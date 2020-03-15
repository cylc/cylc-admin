# Proposal: Cylc 8 Spawn-on-Demand

**Hilary Oliver**, *December 2019, and February/March 2020*

## Table of Contents

- [Introduction](#introduction)
  - [Background](#background)
  - [Advantages](#advantages)
  - [Terminology](#terminology)
- [Overview](#overview)
- [Discussion](#discussion)
  - [Use of task proxies](#use-of-task-proxies)
  - [Spawning waiting tasks on outputs](#spawning-waiting-tasks-on-outputs)
  - [Auto-spawning tasks with no parents](#auto-spawning-tasks-with-no-parents)
  - [Spawning and parent-is-finished status](#spawning-and-parent-is-finished-status)
  - [Finished tasks](#finished-tasks)
  - [Failed tasks](#failed-tasks)
  - [Stall and shutdown](#stall-and-shutdown)
  - [Suicide triggers not needed](#suicide-triggers-not-needed)
  - [Intervention and reflow](#intervention-and-reflow)
  - [Submit number](#submit-number)
  - [CLI implications](#cli-implications)
- [Future enhancments](#future-enhancements)

## Introduction

Companion POC PR [cylc-flow#3515](https://github.com/cylc/cylc-flow/pull/3515).

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

- **Efficiency: a dramatically smaller task pool** (few waiting and finished tasks)
- **No need for dynamic dependency matching** (parents update child prerequisites on spawning)
- **An easily-understood graph-based "window on the workflow"**
- **Suicide triggers are not needed for alternate path branching**
- **Instances of the same task can run out of cycle point order**
- **No unnecessary stalling** (in SoS, caused by unspawned tasks downstream of a failed task)
- **Achieve "re-flow" by simply re-triggering a task**
- **No need to manually insert the first instance of newly defined tasks** on restart or reload

### Terminology

- *parent* and *child*: up- and down-stream graph relationships
- *task*: short for *task proxy object*
- *task pool*: just the "active tasks" the scheduler is currently aware of
- *n=0 window*: tasks that anchor the current view (by default, the task pool)
- *n=M window*: view of tasks within M graph edges of the n=0 window
- *finished*: means *succeeded*, OR *failed* *with ":fail" handled*

## Overview

1. Get tasks to record (from the graph) the children that depend on each of
      their outputs 
1. Continually [auto-spawn tasks with no
      parents](#auto-spawn-tasks-with-no-parents) to the runahead limit
1. (Tasks with no prerequisites can submit their jobs, as usual)
1. As outputs are completed [spawn as waiting
    tasks](#spawning-waiting-tasks-on-outputs) any children that depend on them
    (if not already spawned) and update their prerequisites directly
1. (Tasks with prerequisites all satisfied can submit their jobs, as usual)
1. As tasks finish, spawn any multi-parent children (if not already spawned) and
      update their
      [parent-is-finished status](spawning-and-parent-is-finished-status)
      directly
1. Housekeep the task pool:
   1. Remove waiting tasks if their parents have finished
      - see [Waiting tasks](#waiting-tasks)
   1. Remove finished tasks if their parents have finished, *or* if the
   pool has moved beyond any missing parents
      - see [Finished tasks](#finished-tasks) and [Failed tasks](#failed-tasks)
1. Shut down (completed) if stalled with no failed tasks present
   - see [What is in the task pool](#what-is-in-the-task-pool) and [Scheduler
     shutdown](#scheduler-shutdown), below

## Discussion

### Use of task proxies

The proposed implementation uses waiting task proxies, which appear "on demand"
when the first output they depend on is completed, to track prerequisite
satisfaction (below).

It also keeps finished task proxies in the pool until their parents are all
finished, to prevent "conditional reflow" (below).

It would be possible to implement SoD without any waiting or finished tasks.
The scheduler could keep track of disembodied prerequisite data, and only spawn
tasks that are ready to run; and similarly it could separately keep track of
who finished.

However the same information has to be tracked and housekept either way, but
using task proxies is simpler (with current Cylc internals at least). Tasks
already know how to track and evaluate their own prerequisites, and the cost of
keeping a few whole tasks a little longer than strictly necessary is very low.
Having these tasks in the task pool also makes it easier to see what is
happening.


### Spawning waiting tasks on outputs

When an output gets completed, the scheduler should spawn as [waiting
tasks](#waiting-tasks) any children that depend on that output, if they are
not already spawned, and update their prerequisites directly.

```
A & B => C
```
If `A` and `B` finish at different times `C` will be spawned on the first
relevant output (`A:succeeded`, say) and remain as waiting until all
of its prerequisites are satisfied (`B:succeeded`).

See also [Tasks with no parents to spawn
them](tasks-with-no-parents-to-spawn-them)

#### Housekeeping waiting tasks

Waiting tasks can be removed if their parents have all finished, because at
that point they can no longer automatically satisfy the remaining
prerequisites. Note that "finished" means succeeded OR [failed but
handled](#failed-tasks).

### Auto-spawning tasks with no parents

Tasks with no prerequisites at all, and those that depend only on xtriggers -
including clock triggers - need to be auto-spawned out to the runahead limit.

In future we should spawn xtriggered-tasks on demand too, in response to
outputs/events emitted by xtrigger functions, but for the moment auto-spawning
them as waiting tasks is easy to do and makes it easy to expose what's
going on to users.

### Finished tasks

```
A | B => C
```
If `C` triggers off of `A` we need to avoid spawning `C` again if `B` runs
later, after `C` has already finished and disappeared. To avoid this
"conditional reflow" we have to keep finished tasks until reflow is no longer
possible.

#### Housekeeping finished tasks

Finished tasks can be removed if their parents have finished (at which point
there is no danger of reflow).

This doesn't clean up all finished tasks, however. At alternate branch joins,
parents may never show up:
```
A:out1 => post1
A:out2 => post2
post1 | post2 => finish
```
Cylc cannot know if the `out1` and `out2` outputs of `A` are mutually exclusive
or not. If they are, `finish` will trigger off of one of them, but the other
will never show up. Failure recovery branching is of this type too.

These finished tasks can be removed if the task pool has moved on to a cycle 
point beyond which nothing short of intervention could trigger their parents.

 Note that "finished" means succeeded OR [failed but handled](#failed-tasks).

### Spawning on finished

Housekeeping of [waiting](#waiting-tasks) and [finished](#finished-tasks) tasks
as described above requires children to know when their parents are finished.
This requires spawning on finished as well as on outputs, because parents can
finish without spawning depending on which outputs were completed.

```
A:fail => X
A & B => C
```
Here, if `A` fails without spawning `C`, then when `B` spawns `C` later on, `C`
will only know that `B` finished. But if `A` spawns both `X` and `C` when it
finishes, and updates their parent-is-finished status, `C` can be removed
(parents finished) once `B` succeeds.

Note this does not result in unsatisfiable (stuck) waiting tasks regardless of
what outputs `A` is supposed to generate. Housekeeping depends only on parents
being finished or not, not on specific parent outputs:

### Failed tasks

Technically, we don't need to retain failed tasks in the pool: they have
already served their purpose in the workflow, and visibility and retriggering
don't depend on presence in the pool (just watch notifications and/or widen the
view beyond the pool). However...

Failed tasks handled by a `:fail` trigger, which implies the failure was not
unexpected, can be removed as *finished* just like succeeded ones.

Failed tasks NOT handled by a `:fail` trigger, which implies something went
unexpectedly wrong, will:
- be kept in the task pool, and
- considered to be *not finished*

This has the following advantages:
- Visibility in the default n=0 view
  - Users don't have to go looking for failures if they missed a notification
- It supports simple retriggering of failed tasks without the complications of
  general reflow (namely off-flow dependence of downstream tasks)
  - (Children waiting on success of the failed task, spawned on *finish* as
    described above, are kept in the pool until their parents are finished)

### Task pool content

So the task pool contains:
- active tasks (preparing, submitted, running) 
- [waiting tasks](#waiting-tasks) with one or more prerequisites (but not all
  of them) satisfied, and/or unsatisfied external triggers
- [finished tasks](#finished-tasks) (succeeded and failed) that still have some
  unfinished conditional parents
- [unhandled failed tasks](#failed-tasks)

Notes:
- This constitutes vastly fewer `waiting` and `finished` tasks than in SoS
- The pool might not contain tasks from the highest cycle point reached (e.g.
  if a user-triggered [reflow](#reflow) is still running when the original flow finishes)
  - This is fine; the scheduler's job is to manage active tasks, not to
    maintain an awareness of the past. To see beyond the task pool users just
    need to widen the view window (and watch for important events.

### Stall and shutdown

A stalled workflow cannot carry on without intervention. This could mean:
- The workflow ran to completion
- Premature stall due to an unexpected task failure

In SoS the scheduler can distinguish the two cases (most of the time!) because
all tasks are spawned ahead as waiting without regard for what is "demanded" by
the graph. If the workflow has completed there will be no waiting tasks left in
the pool, otherwise the stall is premature.

In SoD there may be no tasks waiting ahead to show there is more to do, but we
can detect workflow completion as follows:

- If the workflow has NOT stalled,
   - Not completed (obviously)
- If the worklfow has stalled,
   - Failed tasks (and waiting children) present: premature stall
   - No failed tasks present: workflow completed

At stall we can choose to keep the most recent active tasks in the pool, or to
empty the pool (apart from unhandled failed tasks). Keeping recent active tasks
may be preferred as this keeps the default view (n=0) focused on what most
recently happened.

We can (and already do) shut down on a configurable timeout on premature stall.
There's no point in keeping a stalled scheduler alive if it isn't doing
anything, so long as we can easily display and restart stopped workflows.

Note that a workflow final cycle point isn't much use in determining completion
before it happens:
- It is possible to have no graph recurrence sections that reach the final
  cycle point
- Cylc can't know if every task valid in the final cycle point is actually
  expected to run there, because alternate paths are possible

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
Even here, C will be spawned by X as waiting if A fails, but will be
removed by housekeeping (above) once both A and X are finished.

```
@xtrigger1 => A
@xtrigger2 => B
```

Tim W's edge case: `B` triggers and determines that `xtrigger1` can never be
satisfied, so we need to remove the waiting `A` (the two xtriggers are watching
two mutually exclusive data directories for a new file). This is (?) the only
remaining case for suicide triggers, and even this will go away once we are
spawning xtriggered-tasks "on demand" too in response to xtrigger outputs.
For the moment we can either keep suicide triggers to support this use case, or
perhaps suggest a workaround: `B` calls `cylc remove A`.

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

Reflow naturally happens in SoD. This is very powerful, but we need to be careful!

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

*Retrigging a non-bottleneck task will cause a stalled reflow* without manual
intervention, because previous-flow outputs are not automatically available
(those tasks are gone from the pool):
- Triggering `baz.2` will result in an waiting `qux.2` with an
  unsatisfied dependence on `bar.2`
- But the intervention to deal with this is straightforward and consistent:
  - Tell Cylc to spawn the child of `bar.2:succeeded` (before triggering
    `baz.2` to set up an automatic reflow, or after to un-stall)
  - OR trigger `qux.2` after `baz.2` finishes, to make it run despite
    its unsatisfied prerequisite

We will need optional reflow restrictions, based on a "flow number" passed down
on spawning events?:
- initially at least, allow only one reflow at a time (tasks will either be
  part of the reflow or not)
- by default, don't reflow at all - just run the retriggered task alone
- stop the reflow at cycle point X
- stop the reflow at a list of task IDs (to do less than a whole cycle)

Notes:
- Need to be able to cancel a reflow
- Need to clearly document what to expect
- At the moment, if the reflow catches up to the main flow a task will not get
spawned if it was already spawned by the main flow (and the existing one will
get its prerequisites updated)
  - this is probably the right thing to do if the existing task is active already
    (if you don't want it to run, kill it); but what if the existing one is
    finished (still in the pool because its parents aren't finished)? Should
    it to run again in this case?

**Automatic use of previous-flow outputs** is a possible future enhancement, but:
- It may be difficult
  - Graph traversal is required to determine whether prerequisites can be
    satisfied within the new flow or not
  - If they can't, query the database for previous-flow outputs (and: how to
    know when/if we can stop doing this?)
- It could be dangerous, and anyway isn't really needed!
  - SoD reflow is a massive improvement on SoS reflow as-is: less
    intervention (usually very little is needed) and it is more straightfoward
    and consistent
- We should provide an option to ignore dependence on pre-reflow cycle points

Semantically, reflow as described here is really just a generalisation of
"retriggering tasks within the same flow". This is not really fundamentally
different than what we have currently in SoS, it just works better and is
easier to do.

Note that retriggering of unhandled failed tasks (still in the pool) does not
cause reflow.

*TODO: the UI will need to make it clear why a task with off-flow dependence is
stuck waiting, as off-reflow parents will appear as finished in the n>0 window.
Do we need to mark scheduler task pool members to distinuish them?*

(Note we actually have a similar-but-worse problem in SoS: previous-flow task
outputs will not be used automatically unless those tasks happen to still be
present in the task pool - and users do not generally understand what
determines that presence).

## Submit number

Retry number should increment with automatic retries, and may be needed in job logic. 
Submit number should increment with any retriggering (automatic or forced) and
is needed to avoid clobbering previous job logs (it appears in the log file path).

In SoS retry number increments safely as the task proxy doesn't disappear
between retries. Submit number is safe on retriggering so long as you retrigger 
while the finished task is still in the pool, otherwise submit number has to be
looked up in the DB. However **this is broken in SoS:** re-inserted tasks get
the right submit number from the DB, but their spawned next-cycle successors
start at submit number 1 again.

In SoD, retry number is still safe because a retrying task will revert to
waiting (it will not disappear from the pool... until we re-spawn retrying
tasks on clock events).

Submit number always starts at one in the original flow, and increments safely
for automatic retries. 

For any manual triggering *and subsequent reflow* we have to look up submit
number in the DB in case the task already ran earlier.

TODO - if a reflow takes over the whole workflow, can we stop lookup at the DB
for submit number? (or will this naturally happen when the reflow catches up to 
the original flow - if that doesn't happen, it is still "the reflow").

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

TODO - consider changing the name of the command for users,
to `cylc set-outputs-completed` or similar?

### "cylc remove" still needed

In SoS removing tasks from the pool can remove them entirely from the ongoing
workflow, if the next instance isn't spawned before the removal.

In SoD there's not much in the task pool to "remove" and
- removing active tasks doesn't make sense (the job is already active)
- removing waiting tasks will not stop future instances of the task
  appearing on demand
- but we can still keep the command to allow occasional removal of waiting and
  finished tasks if necessary (to preempt housekeeping?)

### "cylc reset" not needed

In SoS state reset is used to force-change the state of existing task instances
(in the pool). Usually to change the result of dependency negotiation and get a
task to run, or to "lie" that a failed task succeeded in order to un-stall
the workflow on the next dependency matching round.

SoD does not have non-active tasks or dependency negotiation. Abstract tasks
out front (beyond n=0) are already "waiting" and don't need to be in the task
pool. Finished tasks may be removed immediately. Failed tasks are just
historical information, and pretending they succeeded is unnecessary - better
force downstream tasks to carry on despite the upstream failure: use
`cylc spawn` (to become `cylc set-outputs-completed`?).

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

In SoS we only store gross task state in the DB, and infer prerequisite status
from that on restart (then rely on dependency negotiation to (re-)satisfy
prerequisites of waiting tasks at start-up). For SoD we'll need to store actual
prerequisite status in the DB, not just task state. But that's easy to do.

Also need to auto-spawn tasks with no prerequisites again, like at cold start.

Similarly for reload, if parentless task defs were added.

## Future Enhancements

For Cylc 9: [cylc/cylc-flow#3304](https://github.com/cylc/cylc-flow/issues/3304)

Several easy wins may be worthwhile before Cylc 9:
- Refactor the task pool to avoid (most) iteration over tasks.
- Refactor prerequisite and output handling, which remains largely as designed
  for SoS dependency negotation
- Extend SoD to clock- and external-triggers: tasks should be spawned in
  response to trigger events (depends on xtriggers as main-loop coroutines)
