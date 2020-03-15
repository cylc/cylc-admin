# Proposal: Cylc 8 Spawn-on-Demand

**Hilary Oliver**, *December 2019, and February/March 2020*

## Table of Contents

- [Introduction](#introduction)
  - [Background](#background)
  - [Advantages](#advantages)
  - [Terminology](#terminology)
- [Implementation overview](#implementation-overview)
- [Discussion of details](#discussion-of-details)
  - [Use of task proxies](#use-of-task-proxies)
  - [Spawning tasks with no parents](#spawning-tasks-with-no-parents)
  - [Spawning on task outputs](#spawning-on-task-outputs)
  - [Keeping finished tasks to prevent conditional reflow](#keeping-finished-tasks-to-prevent-conditional-reflow)
  - [Spawning on task completion](#spawning-on-task-completion)
  - [Failed tasks](#failed-tasks)
  - [Stall and shutdown](#stall-and-shutdown)
  - [Suicide triggers not needed](#suicide-triggers-not-needed)
  - [Intervention and reflow](#intervention-and-reflow)
     - [Reflow in SoS](#reflow-in-sos)
     - [Reflow in SoD](#reflow-in-sod)
  - [Submit number](#submit-number)
  - [CLI implications](#cli-implications)
  - [Succeeded tasks](#succeeded-tasks)
- [Notes](#notes)
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
- *reflow*: when the workflow flows on from a manually triggered task

## Implementation overview

1. Get tasks to record (from the graph) the children that depend on each of
      their outputs 
1. Continually [auto-spawn tasks with no
      parents](#auto-spawn-tasks-with-no-parents) to the runahead limit
1. (Tasks with no prerequisites can submit their jobs, as usual)
1. As outputs are completed [spawn waiting
     children](#spawning-waiting-tasks-on-outputs) that depend on them
    (if not already spawned) and update their prerequisites directly
1. (Tasks with prerequisites all satisfied can submit their jobs, as usual)
1. As tasks finish, spawn remaining children (if not already spawned) and
      update their [parent-is-finished
      status](spawning-and-parent-is-finished-status) directly
1. [Remove waiting and finished tasks](#housekeeping) if their parents have
    finished *or* the pool has moved beyond the point it can affect them
1. [Shut down](#scheduler-shutdown) if stalled with no failed tasks present

## Discussion of details

### Use of task proxies

The implementation proposed here uses waiting task proxies, which appear "on
demand" when the first output they depend on is completed, to track
prerequisite satisfaction. It also keeps finished task proxies in the
pool until their parents are all finished, to prevent "conditional reflow".

So the task pool contains:
- Active tasks (preparing, submitted, running) 
- Waiting tasks with at least one prerequisite satisfied (and/or unsatisfied
  xtriggers)
- Finished tasks (succeeded and failed) that still have some unfinished
  conditional parents
- [unhandled failed tasks](#failed-tasks) and [unhandled succeeded
  tasks](#succeeded-tasks)

This constitutes vastly fewer `waiting` and `finished` tasks than in SoS.

It would be possible to implement SoD without any waiting or finished tasks:
the scheduler could keep track of disembodied prerequisite data, only spawning
tasks that are ready to run; and similarly it could separately keep track of
who finished.

However the same information has to be tracked and housekept either way, and
using task proxies is simpler - at least with current Cylc internals.
Prerequisites are associated with particular tasks after all, and task proxies
already know how to track and evaluate their own prerequisites. The cost of
keeping a few whole tasks a little longer than strictly necessary is very low.
And having these tasks in the pool makes it easy to expose what is going on to
users (otherwise prerequisite data has to be sent to the UIS and connected 
to the right abstract tasks there).

### Spawning tasks with no parents

Tasks with no prerequisites at all, and those that depend only on xtriggers,
need to be auto-spawned out to the runahead limit (they have no parents to do
it for them).

However, see [future enhancements](#future-enhancements) (below) for plans to
spawn on xtrigger events.

### Spawning on task outputs

First, see [use of task proxies](#use-of-task-proxies) (above) for
justification of using waiting task proxies to track satisfaction of
prerequisites.

When a task output gets completed the scheduler should spawn (if not already
spawned) children of that output in the waiting state and update their
prerequisites directly. Note this only creates waiting tasks between generation
of the first and last outputs dependended on, for tasks with multiple parents.

```
A & B => C
```
If `A` finishes before `B`, `C` will be spawned on `A:succeeded` and will
remain as waiting until `B` finishes.

### Keeping finished tasks to prevent conditional reflow

First, see [use of task proxies](#use-of-task-proxies) (above) for
justification of using finished task proxies to prevent conditional reflow.

Conditional prerequisites require keeping some finished tasks in the pool,
to prevent unintended reflow:

```
A | B => C
```
If `C` triggers off of `A:succeeded` we need to avoid spawning `C` again if `B`
runs after `C` has finished and been removed from the pool. To avoid this we
can keep finished tasks in the pool until there is no danger of automatic
reflow (a task will not be spawned if it already exists in the pool).

### Spawning on task completion

[Housekeeping](#housekeeping-waiting-and-finished-tasks) (below) relies on
children knowing when their parents are finished (a waiting task can never
be satisfied automatically if its parents are finished). This requires, for
children with multiple parents, spawning on parent completion as well as on
outputs, but only to update their parent-is-finished status (not
prerequisites):

```
A:fail => X
A & B => C
```
Here `A:fail` needs to spawn `C` as well as `X` so that `C` exists and
remembers that `A` finished when `B` finishes later. Then `C` can be
removed (all parents finished).

Note that spawning all children in this way does not create a problem even for
mutually exclusive outputs:

```
A:out1 => B
A:out2 => C
```
Here, if `A` generates output `:out1` (say) both `B` and `C` will be spawned
when `A` finishes, but only `B` will run (prerequisites satisfied). The waiting
`C` can then be removed immediately (parents finished).

### Housekeeping waiting and finished tasks

Tasks with only one parent will run as soon as they are spawned, and can
be removed as soon as they are finished.

But tasks with multiple parents may spend some time waiting on the rest of
their prerequisites, and as finished (to prevent conditional reflow).

**Waiting tasks can be removed if all of their parents have finished.**
(Parents that are finished can no longer automatically satisfy anyone's
prerequisites; if retriggered after that, users need to be aware of reflow).

**Finished tasks can be removed if all of their parents have finished.** 
(Parents that are finished can no longer spawn any children and cause reflow;
if retriggered after that, users need to be aware of reflow).

However, it is not quite quite enough to wait for all parents to finish,
because it is possible for parents to be stuck as waiting or to never be
spawned at all, due to upstream failures or alternate path choices:

```
A:out1 => post1
A:out2 => post2
post1 | post2 => plot
```
Here the intention is probably that either one or the other of the outputs
`out1` and `out2` will be generated, so either `post1` or `post2` will run, and
`plot` will trigger of whichever `post` happens to run. But Cylc cannot
know if `out1` and `out2` are mutually exclusive or not. It is possible that
they could both be generated (and both paths will run), so we have to keep
`plot` around in the finished state to prevent possible conditional reflow.

Alternate branching like this won't stop the workflow from flowing on,
however, so we can remove finished tasks like `plot` once the task pool has
moved on to a cycle point beyond which nothing short of intervention could
trigger their "missing" parents.

Stuck waiting tasks are a little less straightforward as they can be caused by 
upstream failures.

```
x => B
A & B => C
```
If `x` fails, `B` will be spawned (on `x` finished) but will be stuck as
waiting on `B` to succeed, so `C` will be stuck as waiting once `A`
is finished. But retriggering `x` will cause `B` to run, and then `C` - so no
problem here.

```
x => y => B
A & B => C
```
If `x` fails, `B` will not be spawned, so `C` will be stuck as waiting once `A`
is finished.  But retriggering `x` will cause `y` and then `B`, and hence `C`
to run - so no problem here either.

But a badly handled upstream failure can potentially cause orphaned waiting
tasks:
```
[scheduling]
   cycling mode = integer
   initial cycle point = 1
   final cycle point = 5
   [[graph]]
       P1 = """x:fail => alert
               x => B
               A & B => C"""
[runtime]
   [[root]]
       script = sleep $(( 5 + RANDOM % 5 ))
   [[x]]
       script = """
   if (( CYLC_TASK_CYCLE_POINT == 1 && CYLC_TASK_SUBMIT_NUMBER == 1 )); then
      false
   fi"""
```
When `x.1` fails `B.1` is spawned (on `x.1` finished), but `x.1` gets removed
as finished (the failure was handled). `B.1` gets removed (parents finished)
but `C.1` will be stuck as waiting because not all of its parents finished.
This is essentially a workflow design error: the handled failure does not
result in the workflow continuing. However, we can still remove such orphaned
waiting tasks once the pool moves beyond the point where anyone could possibly
trigger them (in the example, once `C.1` is the only task remaining in cycle
point 1 there is no point in retaining it).

### Failed tasks

Technically we don't need to retain failed tasks in the pool: they have already
served their logical purpose in the workflow, and visibility and retriggering
don't depend on presence in the pool (just watch notifications and/or widen the
view beyond the pool).

Failed tasks can be removed immediately if handled by a `:fail` trigger (which
implies the failure was *expected by the workflow*). 

Failed tasks that are not handled (which implies the failure was *not expected
by the workflow*), however, will be kept in the task pool and considered
*not finished* for [housekeeping](#housekeeping) purposes.

This makes unhandled failed tasks stay visible in the default n=0 view.

And it supports the typical "retrigger a failed task" use case without the
complications of reflow:

```
A & B => C
```
If `A` fails it will [spawn](#spawn-on-task-completion) `C` as waiting. Both
`A` and `C` will remain in the pool after `B` completes (`A` because its
failure was not handled, and `C` because its parents did not all finish). Then
if the user retriggers `A` and it succeeds, `C` can run becuase it remembers
that `B` finished.  If we did not keep `A` and `C` in the pool, the user could
still retrigger `A`, but its success would respawn `C` which would not remember
that `B` had finished in "the previous flow". So the proposed implementation
supports the failed task retriggering use case without requiring further
intervention downstream.


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
   - Then it has not completed (obviously)
- If the worklfow has stalled,
   - With unhandled failed tasks present: premature stall
   - With no unhandled failed tasks present: workflow completed

At stall we can choose to keep the most recent active tasks in the pool, or to
empty the pool (apart from unhandled failed tasks). Keeping recent active tasks
may be preferred as this keeps the default view (n=0) focused on what most
recently happened.

We can (and already do) shut down on a configurable timeout on premature stall.
There's no point in keeping a stalled scheduler alive if it isn't doing
anything, so long as we can easily display and restart stopped workflows.

Note that final cycle point isn't much use in determining completion
before it happens:
- It is possible to have no graph recurrence sections that reach the final
  cycle point
- Cylc can't know if every task valid in the final cycle point is actually
  expected to run there, because alternate paths are possible

Note that bad failure handling, which does not cause the workflow to continue,
will cause a premature shutdown:
```
A:fail => email_me
A => B => C = archive
```
Here, if `A` fails the workflow will shut down as completed with `email_me`
succeeded. But, that is what the graph logic says to do! If `A` fails, there is
no path to normal completion.

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

### Succeeded tasks

If a task fails unhandled (i.e. without a `fail` trigger) we consider it
unfinished, and keep it in the task pool to support easy retriggering (above).

By analogy if a task succeeds without a `:succeeded` trigger, should we also
consider it unfinished and keep it in the pool (presuming that it was
"expected to fail")?

Failure by definition implies the task failed to do what it was designed to do.
But it is possible for us to expect a task to fail, and to build the workflow
around that expectation (that's what `:fail` triggers are for).

Similarly, success implies the task succeeded in doing what it was designed to
do. But it is possible for us to expect a task not to succeed, and for the
workflow to be built around that expectation.

If a task generates multiple outputs, failure can occur at any point during
execution, so any of the outputs (apart from `:succeeded`) could be generated
before failure.

Similarly success implies only that the `:failed` output was not generated,
because multiple internal outputs can be mutually exclusive (e.g. to drive
alternate paths).

So it seems there is no fundamental difference between `:fail` and `:succeed`
in terms of workflow logic, and we should really treat unhandled success
and failure in the same way.

But there may be a difference in practical terms: *success is probably assumed
in cases where internal outputs are used, and failure handling probably always
assumes that no internal outputs were generated before the failure??*
```
A:out1 => B1
A:out2 => B2
 # ...
```
We can't expect this to be written:
```
A:out1 & A => B1
A:out2 & A => B2
 # ...
```
because then `B1` etc. have to wait until A succeeds before running.

**TODO: should success be considered handled if any non-fail output is
handled?**

## Notes

The task pool might not contain tasks from the highest cycle point reached
(e.g.  if a user-triggered [reflow](#reflow) is still running when the original
flow finishes)
  - This is fine; the scheduler's job is to manage active tasks, not to
    maintain an awareness of the past. To see beyond the task pool users just
    need to widen the view window (and watch for important events.

## Future Enhancements

For Cylc 9: [cylc/cylc-flow#3304](https://github.com/cylc/cylc-flow/issues/3304)

Several easy wins may be worthwhile before Cylc 9:
- Refactor the task pool to avoid (most) iteration over tasks.
- Refactor prerequisite and output handling, which remains largely as designed
  for SoS dependency negotation
- Consider not [using waiting and finished task proxies](#use-of-task-proxies)
- Extend SoD to clock- and external-triggers: tasks should be spawned in
  response to trigger events (depends on xtriggers as main-loop coroutines)
