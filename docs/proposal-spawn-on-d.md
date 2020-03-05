# Proposal: Cylc 8 Spawn-on-Demand

**Hilary Oliver**, *December 2019, and February/March 2020*

## Table of Contents

- [Introduction](#introduction)
  - [Terminology](#terminology)
  - [Background](#background)
  - [Advantages](#advantages)
- [Implementation overview](#implementation-overview)
- [Implementation details](#implementation-details)
  - [What is in the task pool](#what-is-in-the-task-pool)
  - [Scheduler shutdown](#scheduler-shutdown)
  - [Active waiting tasks](#active-waiting-tasks)
  - [Active finished tasks](#active-finished-tasks)
  - [Auto-spawning tasks with no parents](#auto-spawning-tasks-with-no-parents)
  - [Suicide triggers not needed](#suicide-triggers-not-needed)
  - [Intervention and reflow](#intervention-and-reflow)
  - [Submit number](#submit-number)
  - [CLI implications](#cli-implications)
- [Future enhancments](#future-enhancements)

## Introduction

*Companion POC Pull Request: https://github.com/cylc/cylc-flow/pull/3474*

(To be superceded and re-described by a new PR shortly).

### Terminology

- *parent* and *child* refer to upstream and downstream graph relationships
- *task* is short for *task proxy object*
- *task pool*: just the "active tasks" the scheduler is currently aware of
- *n=0 window*: tasks that anchor the current view (by default, the task pool)
- *n=M window*: view of tasks within M graph edges of the n=0 window
- *finished* means *succeeded* or *failed*

### Background

We first considered replacing task-cycle-based
[Spawn-On-Submit](spawn-on-submit-history) (SoS)
with graph-based Spawn-On-Demand (SoD) back in
[cylc/cylc-flow#993](https://github.com/cylc/cylc-flow/issues/993).
The ultimate event-driven solution might look something like 
[cylc/cylc-flow#3304](https://github.com/cylc/cylc-flow/issues/3304)
but that requires significant refactoring and is slated for Cylc 9.

However, SoD itself can actually be implemented quite easily now, within the
current framework, by simply getting task proxies to spawn children on demand
instead of next-cycle successors on submit. This will simplify Cylc internals
and usage, and will make Cylc 9 changes easier to implement in the future.

### Advantages

- **Dramatically smaller task pool**
- **No need for dynamic dependency matching** (update prerequisites directly)
- **An easily understood graph-based "window on the workflow"**
- **Suicide triggers not needed for alternate path branching**
- **Task instances can run out of (cycle point) order**
- **No stalling with unspawned tasks downstream of a failed task**
- **Achieve "re-flow" by simply re-triggering the top task**
- **No need to manually insert the first instance of new task defs**

## Implementation Overview

Get task proxies to record who depends on each of their outputs (according to
the graph). Then:
1. At start-up auto-spawn tasks with no one upstream to do it for them,
   and continue to do this as the workflow evolves
1. As outputs are completed, spawn (if not already spawned) children and update
   their prerequisites directly
   - see [active waiting tasks](#active-waiting-tasks), below
1. Remove finished tasks (succeeded or failed) when all of their parents have
   finished, unless that would empty the task pool (which implies a stall)
   - see [active finished tasks](#active-finished-tasks), below
1. If the workflow stalls, shut down after a configurable timeout
   - see [What is in the task pool](#what-is-in-the-task-pool) and [Scheduler
     shutdown](#scheduler-shutdown), below

Details are discussed below because some aspects of this implementation are
choices that are not fundamental to the SoD concept. E.g. is a task "on demand"
as soon as its first prerequisite is satisfied, or only when all of them are
satisfied? - SoD can be made to work either way, but the former happens to be
more straightforward, at least initially.

The proposed implementation requires some housekeeping of information
associated with partially completed and conditional prerequisites (below) but
it turns out that can be done entirely in memory (unless/until we need
automatic use of previous-flow outputs - below).

## Implementation Details

### What is in the task pool

The task pool (n=0) contains **only** current active tasks, or (if stalled) the
most recently active tasks, i.e. it represents *what is currently happening* or
(if stalled) *what happened most recently*.

The definition of "active" has been extended slightly to include:
- [active waiting tasks](#active-waiting-tasks) (those with partially
  satisified prerequisites)
- [active finished tasks](#active-finished-tasks) (those with some unfinished
  conditional parents)

But these constitute vastly fewer waiting and finished tasks than in SoS.

(The task pool does not **need** to retain failed tasks. However we might
choose to retain them until we have implemented better failure notification
and/or the UI has access to n>0 windows)

The task pool might not contain "tasks of interest" such as those that caused
a stall. E.g. `foo:fail => diagnose => notify` will stall with only `notify`
succeeded (not `foo` failed) in the pool. It might not contain tasks from
the highest cycle point reached either. E.g. because a later cycle finished
before an earlier one, or because the user triggered a reflow from an earlier
cycle.

This is fine! The scheduler's job is to manage active tasks, not to maintain an
awareness of what happened in the (chronological, or graph) past. To see beyond
the task pool users just need to widen the window (and watch for important
events) - the UI Server's problem.

### Scheduler shutdown

A stalled workflow cannot carry on without intervention. This could mean it has
run to completion (nothing left to run) or it could be a premature stall (e.g.
after a task failure).

In SoS the scheduler can distinguish the two cases (most of the time!) because
all tasks are spawned ahead as waiting without regard for what is "demanded" by
the graph. If the workflow has completed there will be nothing left to run (no
waiting tasks left in the pool) otherwise it is a premature stall.

In SoD tasks can be removed immediately once finished and there are no wholly
unsatisfied waiting tasks, so there will often be nothing waiting ahead to
show whether or not there is more to do. In fact *there is no foolproof way to
automatically distinguish premature stall from workflow completion.* In both
stalled and active situations success or failure can be handled or not, or
expected or not, or possibly handled by custom outputs which might be
sequential or mutually exclusive; and we can't even know if every
task in every cycle point is supposed to execute (there might be dynamically
determined alternate paths).

Fortunately this doesn't really matter! I propose:

- If the workflow stalls - finished or not - shut down on a timeout
  - the timeout allows intervention if the worklfow is actively monitored
  - otherwise the user can intervene later, after restarting the workflow
  - there is no good reason to keep a stalled workflow alive indefinitely,
    whether it has stalled or completed (so long as the UI can display the
    right information about stopped workflows).

- Allow users to give an optional condition that distinguishes completion from
  premature stall, mainly for notification and logging purposes. E.g.:
  `completion = archive.$ succeeded` where `$` is the final cycle point.

### Active waiting tasks

```
A & B => C
```
If `A` and `B` finish at different times, `C`'s prerequisites have to be
managed over that time. So we have to either:
- Spawn children (`C`) on the first output (`A:succeeded`) and manage the
  resulting "active waiting tasks" until all prerequisites are satisfied
- OR manage prerequisites separately and spawn the associated tasks only when
  all of their prerequisites are satisfied (no waiting tasks)

Spawning earlier is more straightforward, at least initially, because:
- Tasks already know how to manage and evaluate their own prerequisites
- The same information needs houskeeping either way, but this way exposes
  what's happening very clearly (and otherwise we'd have to hook live
  prerequisite status up to upcoming n>0 tasks)
- (And the additional cost of keeping a few waiting tasks a bit longer than
  strictly necessary is very low).

See also [Tasks with no parents to spawn
them](tasks-with-no-parents-to-spawn-them)

#### Housekeeping active waiting tasks

Waiting tasks can be removed from the pool if there is no way, short of
intervention, their prerequisites can be satisfied (by their parents) anymore:
- if all of their parents have finished
- OR if the active pool has moved on to the point that their unfinished parents
  could not be triggered

Waiting tasks that are critical to the ongoing workflow will remain in the pool
when the workflow stalls. If the workflow can go on without them, they will be
removed according to the above criteria.

### Active finished tasks

```
A | B => C
```
If `C` triggers off of `A` we need to avoid spawning `C` again if `B` runs
later, after `C` has already finished and disappeared. To avoid this
"conditional reflow" we have to either:
- keep "active finished tasks" in the pool until reflow is no longer possible
- OR separately remember who finished until reflow is no longer possible

This first option is more straightforward, at least initially, because:
- The mere presence of the finished task actively prevents respawning
- The same information needs houskeeping either way, but this way
  exposes what's happening very clearly
- (And the additional cost of keeping a few finished tasks a bit longer than
  strictly necessary is very low).

#### Housekeeping active finished tasks

Similar to active waiting tasks, finished tasks can be removed if there is no
way, short of intervention, that the status of their parents can change:
- if all of their parents have finished
- OR if the active pool has moved on to the point that their unfinished parents
  could not be triggered

### Auto-spawning tasks with no parents

Tasks with no prerequisites at all, and those that depend only on xtriggers -
including clock triggers - need to be auto-spawned out to the runahead limit.

In future we shouuld spawn xtriggered-tasks on demand too, in response to
outputs/events emitted by xtrigger functions, but for the moment auto-spawning
them as active waiting tasks is easy to do and makes it easy to expose what's
going on to users.

### Suicide triggers not needed

Suicide triggers are no longer needed for workflow branching. 
```
A:fail => B
A => C
```
The path that's not needed here will not get spawned at all.

```
A:fail => B
A => C
X => C
```
Even here, C will be spawned by X as active waiting if A fails, but will be
removed by housekeeping (above) once both A and X are finished.

```
@xtrigger1 => A
@xtrigger2 => B
```

Tim W's edge case: `B` triggers and determines that `xtrigger1` can never be
satisfied, so we need to remove the waiting `A`. This is arguably the only
remaining case for suicide triggers, and even this will go away once we are
spawning xtriggered-tasks "on demand" too in response to xtrigger outputs.
For the moment we can either keep suicide triggers to support this use case, or
suggest the workaround: `B` calls `cylc remove A`.

## Intervention and reflow

Re-flow means having the workflow continue to "flow on" according to the graph
from a manually re-triggered task.

### Reflow in SoS

- A restricted form of reflow occurs automatically in SoS if you (re-)insert
  and trigger a task with only previous-instance dependence (`foo[-P1] => foo`). 
- Otherwise it requires laborious manual insertion of downstream task proxies
  and/or state reset of existing ones to set up the reflow. (And it is near
  impossible to get this right without using the graph view).

### Reflow in SoD

Reflow naturally happens in SoD, but we need to enumerate any limitations and
dangers of this powerful feature.

Consider the cycling workflow graph:

![reflow-example](img/sod.png)

*Retriggering a finite sub-graph from a bottleneck task* is straightforward and
safe:
- Trigger `woo.2`, then `bar.2`, `baz.2`, and `qux.2` will follow, and the
  reflow will stop.

*Retriggering an ongoing cycling workflow from a bottleneck task* is equally
straightforward:
- Trigger `foo.2`, then the workflow will carry on from that point just like
  the original flow
- But we may want optional restrictions based on a "flow number" passed down on
  spawning events?:
  - don't reflow at all (run the retriggered task alone)
  - stop the reflow at cycle point X
  - stop the reflow at a list of task IDs
  - don't reflow if the original flow is still running? (is there a valid use
    case for multiple active flow-fronts at the same time?)

*Retrigging a non-bottleneck task will cause a stalled reflow* without manual
intervention, because previous-flow outputs are not automatically available
(those tasks are gone from the pool):
- Triggering `baz.2` will result in an active waiting `qux.2` with an
  unsatisfied dependence on `bar.2`
- But the intervention to deal with this is straightforward and consistent:
  - Tell Cylc to spawn the child of `bar.2:succeeded` (before triggering
    `baz.2` to set up an automatic reflow, or after to un-stall)
  - OR trigger `qux.2` after `baz.2` finishes, to make it run despite
    its unsatisfied prerequisite

**Automatic use of previous-flow outputs** is a possible future enhancement.
- But it may be difficult
  - Graph traversal is required to determine whether prerequisites can be
    satisfied within the new flow or not
  - If not, we have to query the database for previous-flow outputs
  - (and how to know when/if we can stop doing this?)
- The lack of this feature should be be a blocker for SoD 
  - As-is this is still a massive improvement on SoS reflow! (less intervention
    is needed, and it is straightfoward and consistent)

*TODO: the UI will need to make it clear why a task is stuck waiting, as off-reflow
parents will appear as finished in the n>0 window (they finished *before*
the reflow was triggered). Can we mark recently-active tasks? Or flow number?*

## Submit number

Retry number should increment with automatic retries, and may be needed in job logic. 
Submit number should increment with any retriggering (automatic or forced) and
is needed to avoid clobbering old job logs (it appears in the log file path).

In SoS retry number increments safely as the task proxy doesn't disappear
between retries. If the task proxy disappears, however, submit number has to be
looked up in the DB, and **this is broken in SoS!** Inserted tasks do get the
right submit number via DB look-up, but their next-cycle successors
(automatically spawned) do not - they start at 1 again, and the original job
logs get clobbered.

In SoD, retry number is safe for now because a retrying task will revert to
"active waiting" (it will not disappear from the pool). Submit number, however
(and retry number in future, if we re-spawn retrying tasks in response to
clock trigger events) is a problem because finished tasks disappear from the
pool immediately (so the task proxy can't reliably store submit number).

To get submit number right (for SoS too, actually! - see above) we need to either:
- Look it up in the DB every time a task is spawned
   - Even the first time, otherwise how do we know it is the first time?
- OR get it from the job log dir before writing the job script?
   - Could be good as we have to go to disk here anyway
   - (But what about log dir housekeeping in long runs?)

A completely in-memory solution for submit number is not feasible because any
task in the entire history of the workflow could be retriggered at any point.
Remembering the whole run is what the DB and log directory is for.

## CLI implications

Some SoS task-targeting commands can be removed or repurposed.

### "cylc insert" not needed

In SoD, a re-triggered task will get inserted automatically with prerequisites
satisfied. Inserting tasks for other reasons does not really make sense in SoD.

A major SoS use case was manual insertion of the first instance of a new task
added to the suite definition mid-run. In SoD the correct first instance will
automatically appear on demand.

### "cylc spawn" repurposed

In SoS this made a task instances (already in the task pool) spawn their
next-cycle successors.

In SoD this should spawn the children of specified outputs of abstract tasks
in order to set up reflow from a non-bottleneck task (see above).

### "cylc remove" still needed

In SoS removing tasks from the pool can remove them entirely from the ongoing
workflow, if the next instance isn't spawned before the removal.

In SoD there's not much in the task pool to "remove" and
- removing active tasks doesn't make sense (the job is already active)
- removing active waiting tasks will not stop future instances of the task
  appearing on demand
- but we can still keep the command to allow occasional removal of active
  waiting and active finished tasks if necessary (to preempt housekeeping?)

### "cylc reset" not needed

In SoS state reset is used to force-change the state of existing task instances
(in the pool). Usually to change the result of dependency negotiation and get a
task to run, or to "lie" that a failed task succeeded in order to un-stall
the workflow.

SoD does not have non-active tasks or dependency negotiation. Abstract tasks
out front (beyond n=0) are already "waiting" and don't need to be in the task
pool. Finished tasks are removed immediately. Failed tasks are just historical
information, and pretending they succeeded is unnecessary - better force
downstream tasks to carry on despite the upstream failure.

### CLI task globbing:

In SoS we glob on:
- name and cycle point to match instances in the task pool:
   - `spawn`, `poll`, `kill`, `remove`, `reset`, `trigger`
- name only, for a given cycle point, to match abstract tasks:
   - `insert` 

In SoD we need to glob on:
- name and cycle point to match active tasks in the task pool:
  - `poll`, `kill`
- name only, for a given cycle point, to match abstract tasks:
  - `spawn`, `trigger`

### Restart

In SoS we only store gross task state in the DB infer prerequisite status from
that on restart (then rely on dependency negotiation to (re-)satisfy
prerequisites of waiting tasks at start-up). For SoD we'll need to store actual
prerequisite status in the DB, not just task state. But that's easy to do.

## Unsolved Problems?

- How to find and display graph isolates once they've left the n=0 window?
    - e.g. cycling workflows with no inter-cycle dependencies

## Future Enhancements

For Cylc 9: [cylc/cylc-flow#3304](https://github.com/cylc/cylc-flow/issues/3304)

Several easy wins may be worthwhile before Cylc 9:
- Refactor the task pool to avoid (most) iteration over tasks.
- Refactor prerequisite and output handling, which remains largely as designed
  for SoS dependency negotation
- Extend SoD to clock- and external-triggers: tasks should be spawned in
  response to trigger events (depends on xtriggers as main-loop coroutines)
