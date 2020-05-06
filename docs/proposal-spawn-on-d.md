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
  - [Absolute dependence](#absolute-dependence)
  - [Spawning on task outputs](#spawning-on-task-outputs)
  - [Keeping finished tasks to prevent conditional reflow](#keeping-finished-tasks-to-prevent-conditional-reflow)
  - [Spawning on task completion](#spawning-on-task-completion)
  - [Failed tasks](#failed-tasks)
  - [Stall and shutdown](#stall-and-shutdown)
  - [Spawning and stop points](#spawning-and-stop-points)
  - [Suicide triggers mostly not needed](#suicide-triggers-mostly-not-needed)
  - [Reflow](#reflow)
     - [Reflow in SoS](#reflow-in-sos)
     - [Reflow in SoD](#reflow-in-sod)
     - [Reflow restrictions](#reflow-restrictions)
     - [Reflow TBD](#reflow-tbd)
  - [Submit number](#submit-number)
  - [CLI implications](#cli-implications)
  - [Succeeded vs failed tasks](#succeeded-vs-failed-task-handling)
- [UI TODO](#ui-todo)
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
instead of next-cycle successors on submit. In addition to the major advantages
below this will simplify internals and usage, and it will only make Cylc 9
changes easier to implement in the future.

### Advantages

- **Efficiency: a dramatically smaller task pool** that does not increase in
     size if spread over multiple cycle points
- **No need for dynamic dependency matching** - parents update child
     prerequisites directly at spawning time
- **An easily-understood graph-based "window on the workflow"**
- **Suicide triggers not needed for alternate path branching**
- **Task instances can run out of cycle point order**
   - no stalling because of unspawned tasks downstream of a failed task)
- **Achieve "reflow" by simply re-triggering a task**
- **No need to manually insert the first instance of newly-defined tasks** on
  restart or reload (unless perhaps they have no parents)

### Terminology

- **parent** and **child**: up- and down-stream graph relationships (not runtime inheritance!)
- **task**: an abstract node in the workflow graph (task name at a particular cycle point)
- **task proxy**: an object in the scheduler program that represents a particular
  task instance (task name at a particular cycle point)
- **task pool**: task proxies representing "active tasks" that
  the scheduler is currently aware of
- **runahead pool**: task proxies that have been spawned (created) already but
  are beyond the current runahead limit, or beyond the (non-final) stop point.
  All tasks are initially spawned into the runahead pool then released to the
  main pool as the workflow (and the runahead limit) moves forward.
- **n=0 window**: tasks that anchor the current view (by default, the content of
  the active task pool)
- **n=M window**: tasks within M graph edges of the those in the n=0 window
- **finished**: means *succeeded*, OR *failed* with ":fail" handled. An
  unhandled failed task is consider to be "not finished"
- **reflow**: when the workflow flows on according to the graph, from a
  manually triggered task

## Implementation overview

1. During graph parsing, record (in parent task definitions) the children that
     depend on each output
1. At start-up, spawn the first instance of any [task with no
        parents](#spawning-tasks-with-no-parents) to the runahead pool 
    - Then whenever such a task is released from the runahead pool, spawn its
      next instance (into the runahead pool)
1. Tasks with all prerequisites satisfied (in the main task pool) can submit
      their jobs
1. As task job outputs are completed [spawn
     children](#spawning-on-task-outputs) that depend on them
    (if not already spawned) and update their prerequisites directly
1. As tasks finish, [spawn remaining children](#spawning-on-task-completion)
      and update their *parent-finished* status (needed for housekeeping)
1. [Housekeep waiting and finished tasks](#housekeeping-waiting-and-finished-tasks):
   - Remove waiting tasks if all of their parents have finished
   - Remove finished tasks if all of their parents have finished OR the pool
     has moved beyond the point it can affect them
1. [Shut down](#stall-and-shutdown) if stalled with no (unhandled) failed tasks
      present

## Discussion of details

### Use of task proxies

The implementation proposed here uses waiting task proxies, which appear "on
demand" when the first output they depend on is completed, to track
prerequisite satisfaction. It also keeps finished task proxies in the
pool until their parents are all finished, to prevent "conditional reflow".

So the task pool contains:
- Active tasks (preparing, submitted, running) 
- Waiting tasks with at least one prerequisite satisfied (and/or unsatisfied
  xtriggers) - these can be considered "active" in the sense that *their
  prerequisites are in the process of becoming satisfied*.
- Finished tasks (succeeded and failed) that still have some unfinished
  conditional parents
- [unhandled failed tasks](#failed-tasks)

These constitute vastly fewer `waiting` and `finished` tasks than in SoS.

It would be possible to implement SoD without these waiting or finished tasks:
the scheduler could keep track of disembodied prerequisite data and only spawn
tasks that are ready to run; and it could separately track finished tasks for
conditional reflow prevention. But the same information has to be tracked and
housekept either way, and using task proxies is much simpler because:
- Partially satisfied prerequisites have to be associated with the same tasks
  anyway (in the scheduler and the UI!)
- Our task proxies already know how to track and evaluate their own
  prerequisites

Further, users will see the same thing either way in n>=1 windows: waiting
tasks with partially satisfied prerequisites (in the approach of this proposal
those are backed by task proxies in the pool, in the other approach they would
be abstract tasks that we have to attach the prerequisite data to).

### Spawning tasks with no parents

Tasks with no task prerequisites, and those that depend (only?) on xtriggers,
need to be auto-spawned out to the runahead limit (they have no parents to do
it for them).  I'll call these "orphans".

**Implementation:** at start-up auto-spawn the first orphan instances into the
runahead pool. Subsequently, whenever releasing orphans from the runahead
pool auto-spawn their next instance (into the runahead pool).

See [future enhancements](#future-enhancements) (below) for plans to spawn on
xtrigger events.

### Absolute dependence

Simplest example:
```
R1 = "start"
P1 = "start[^] => foo"
```
(Note without the first line above the scheduler will stall on master with
foo unsatisfied, and will shut down immediately (nothing to do) under SoD
because `start` is not defined on any sequence and `foo` never gets spawned.) 

Here, start should run in the first cycle point, after which all `foo.n` can 
run (out to the runahead limit).


A slightly more difficult case:
```
...
      R1/2 = "start"
      P1 = "start[2] => foo"
```
Here, `foo.1,2,3,...` should trigger off of `start.2`.

**Implementation:** during graph parsing identify tasks with absolute-offset
parents. When an absolute parent finishes *spawn the child at the first cycle
point of the sequence.* (Above, `start.2` should spawn `foo.1`, not `foo.2`).
Then whenever an absolute child is released from the runahead pool auto-spawn
its next instance (to the runahead pool) and mark the absolute dependence as
satisfied. This works because in SoD even the first child does not appear in
the pool until the associated dependence is already satisfied. 

We could choose to keep absolute parent in the n=0 pool or not, once finished.

Retriggering a finished absolute parent causes it to respawn its first child,
then subsequent children as normal (i.e. reflow). This is the right thing to do
under SoD (it's what the graph says!) but we may want to provide an option to
change the first child spawned to a current cycle rather than going back to the
start (use case: retrigger a start-up task that rebuilds a model or whatever,
but don't re-run old cycles).

### Spawning on task outputs

First, see [use of task proxies](#use-of-task-proxies) (above) for
justification of using waiting task proxies to track satisfaction of
prerequisites.

When a task output gets completed the scheduler should spawn (if not already
spawned) children of that output in the waiting state and update their
prerequisites directly. Note this only creates waiting tasks between generation
of the first and last outputs depended on, for tasks with multiple parents.

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

Note that spawning all children in this way does not create a problem even
at the branching of mutually exclusive alternate paths:

```
A:out1 => B
A:out2 => C
```
Here, if `A` generates output `:out1` (say) and spawns `B` as a result, `C`
will be spawned as well when `A` finishes. But `C` can then be removed
immediately (parents finished).

### Housekeeping waiting and finished tasks

Tasks with only one parent will run as soon as they are spawned, and can
be removed as soon as they are finished.

But tasks with multiple parents may spend some time waiting on the rest of
their prerequisites, and as finished (to prevent conditional reflow).

**Waiting tasks can be removed if all of their parents have finished.**
(Parents that are already finished can no longer satisfy anyone else's
prerequisites).

**Finished tasks can be removed if all of their parents have finished.** 
(Parents that are already finished can no longer spawn any children and cause
reflow).

For finished tasks at the end of mutually exclusive alternate paths it is not
quite enough to wait for all parents to finish however, because in that case
some parents should not be spawned at all - but there is no way for Cylc to
know if the paths are mutually exclusive or not:

```
A:out1 => post1
A:out2 => post2
post1 | post2 => plot
```
Here if `out1` and `out2` are mutually exclusive, only one of `post1` or
`post2` will be spawned, but `plot` has to be kept around in the finished 
state in case the other path runs. (Actually the other `post` will also be
spawned when `A` finishes but it will immediately be cleaned up as a waiting
task whose parents have all finished).

Alternate branching like this won't stop the workflow however, so we can remove
finished tasks like `plot` once the task pool has moved on to a cycle point
beyond which nothing short of intervention could trigger their "missing"
parents.

Options:
- Use graph traversal to find the branching task (`A` above). If that
  task is finished we can remove `plot` because the other path cannot be
  spawned. 
- SIMPLE but blunt: cycle point housekeeping - if no active tasks exist anymore
  in cycles that can affect this cycle, the finished task can be removed.

#### Stuck waiting tasks

Stuck waiting tasks (**I think**) are only a symptom of bigger problems 
upstream that *should* stall the suite, so they do not need housekeeping:

```
x => B
A & B => C
```
If `x` fails, `B` will be spawned (on `x` finished) but once `A` is finished
`C` will be stuck as waiting on `B` to succeed. But retriggering `x` will cause
`B` to run, and then `C` - so no problem here (no stuck waiting task).

```
x => y => B
A & B => C
```
If `x` fails, `B` will not be spawned, so `C` will be stuck as waiting on `B`
once `A` is finished.  But retriggering `x` will cause `y` and then `B`, and
hence `C`, to run - so no problem here either (no stuck waiting task).

But a *handled* upstream failure that does not fix the workflow can potentially
cause orphaned waiting tasks:
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
result in the workflow continuing.

We *could* still remove such orphaned waiting tasks *if* the pool moves beyond
the point where anyone could possibly trigger them (in the example, once `C.1`
is the only task remaining in cycle point 1 there is no point in retaining it)
but (a) as argued above there's not much point; and (b) we don't want reflow
tasks that are waiting on an off-flow output to be cleaned up by cycle point
housekeeping (however, we could just turn off this housekeeping for reflows).

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

This makes unhandled failed tasks stay visible in the default n=0 view, and it
supports the typical "retrigger a failed task" use case without the
complications of reflow:

```
A & B => C
```
If `A` fails it will spawn `C` as waiting. Both
`A` and `C` will remain in the pool after `B` completes (`A` because its
failure was not handled, and `C` because its parents did not all finish). Then
if the user retriggers `A` and it succeeds, `C` can run because it remembers
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

In SoD there may be no tasks waiting ahead (on the "intended path") to show
there is more to do, but we can detect workflow completion as follows:

- If the workflow has NOT stalled,
   - Then it has not completed (obviously)
- If the workflow has stalled,
   - No unhandled failed tasks present and all waiting tasks beyond the stop
     point (and in the runahead pool): workflow completed
   - Otherwise (unhandled failed tasks and or stuck waiting tasks present):
     premature stall

At stall we can choose to keep the most recent active tasks in the pool, or to
empty the pool (apart from unhandled failed tasks). Keeping recent active tasks
may be preferred as this keeps the default view (n=0) focused on what most
recently happened.

We could (should?) choose to shut down stalled workflows on a configurable
timeout. There's no point in keeping a stalled scheduler alive if it isn't
doing anything, so long as we can easily display and restart stopped workflows.

Note that final cycle point isn't much use in determining completion
before it happens:
- It is possible to have no graph recurrence sections that reach the final
  cycle point
- Cylc can't know if every task valid in the final cycle point is actually
  expected to run there, because alternate paths are possible

Note that bad failure handling, which does not cause the workflow to continue,
will cause an (arguably!) premature shutdown:
```
A:fail => email_me
A => B => C = archive
```
Here, if `A` fails the workflow will shut down as completed with `email_me`
succeeded. **But, that is what the graph logic says to do!** If `A` fails,
there is no path to `archive` succeeded.

### Spawning and stop points

**Stopping prior to the final cycle point:** 
At any shut down prior to the final cycle point (e.g. stop now, or stop at
cycle point P, etc.) the scheduler shuts down with the task pool as-is, which
may include waiting tasks in the runahead pool beyond the stop point.
This allows a restart to carry on as normal, beyond the stop point.

**Stopping at the final cycle point:**
The workflow final cycle point represents the end of the graph. All graph
recurrence expressions end here so in general it is not possible to say what
the "next" instance of task should be beyond the final point. When the
scheduler shuts down at the final cycle point the task pool will therefore
represent only the last thing that happened before shutdown - i.e. the last
task(s) to complete. It is not possible to carry on beyond this point
with a restart. To go beyond the original final cycle point users would need to
change the final cycle point in the workflow definition and warm start beyond
the original final point.

### Suicide triggers mostly not needed

Suicide triggers are no longer needed for workflow branching. In this example,
the path that's not needed will not get spawned at all:
```
A:fail => B
A => C
```

Even here, C will be spawned by X as waiting if A fails, but will be
removed by housekeeping (above) once both A and X are finished:
```
A:fail => B
A => C
X => C
```

Tim W's case: `B` triggers and determines that `xtrigger1` can never be
satisfied, so we need to remove the waiting `A` (the two xtriggers are watching
two mutually exclusive data directories for a new file):
```
@xtrigger1 => A
@xtrigger2 => B
```
This is arguably a remaining case for suicide triggers (although this will go
away once we are spawning xtriggered-tasks "on demand" too in response to
xtrigger outputs).

In any case there is no harm in keeping suicide triggers for backward
compatibility and in case they are needed for a few edge cases like this. See
also [stuck waiting tasks](#stuck-waiting-tasks): "a *handled* upstream failure
that does not fix the workflow can potentially cause orphaned waiting tasks".

## Reflow

Reflow means having the workflow continue to "flow on" according to the graph
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

Note that retriggering an unhandled failed task does not cause reflow
([above](#failed-tasks)).

### Reflow restrictions

Out of the box any re-triggered task will automatically lead to reflow, and
this could result in multiple flows at once in different parts of the graph.
This seems dangerous, so we will (initially at least) restrict reflow:

- Don't reflow by default: just run the retriggered task and stop
- Allow only one reflow at a time: tasks will be part of the one reflow, or not
    - So we need a **scheduler reflow flag**
    - Task proxies should pass a **task reflow flag** to spawned children.
- Stop the reflow:
    - at cycle point P
    - at a list of task IDs (allows less than a whole cycle)
    - (note some reflows will peter out naturally - at which point we can
      detect there are no tasks present with the reflow flag)
- Cancel a reflow

If a reflow catches up to the main flow new reflow tasks will not get spawned
if they are already in the task pool (but the existing task's prerequisites
will be updated, unless we prevent that). If the existing task is:
  - waiting: run it as normal when its prerequisites are met
  - submitted, running, or finished: leave it alone
(Note we can always tell if a task is part of the reflow or not because of the
task reflow flag, above)

**Automatic use of off-reflow outputs?** This is possible in principle but we
have decided against it as difficult and dangerous. Also, it is really not
necessary. SoD reflow is a big improvement even without this (less
intervention, straightfoward and consistent).

(Note we actually have a similar-but-worse problem in SoS: previous-flow task
outputs will not be used automatically unless those tasks happen to still be
present in the task pool - and users do not generally understand what
determines whether or not those tasks will still be present).

(For the record, in case we ever reconsider automatic use of off-reflow
outputs, graph traversal would be required to determine whether or not an
unsatisfied prerequisite can be satisfied later within the reflow or not, and
if not, to go the the database. As Oliver noted: this could probably be worked
out once at the start of the reflow; it could also help us show users
graphically the consequences of their intended reflow).

### Reflow TBD

TODO: Do we always need to identify a reflow as a reflow (i.e. set the reflow
flag)? E.g. if you trigger a reflow without specifying any stop point for it?
(Submit number)?

TODO: if a reflow can become the main flow (e.g. trigger a reflow behind the
main flow but let it go on forever), how do we stop identifying the ongoing
flow as a reflow and start identifying it as the main flow? If it goes beyond
the original flow it becomes the main flow? (Note this may affect how we
determine [submit number](#submit-number)).

TODO: to document what to expect, for users

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
waiting (it will not disappear from the pool... unless we start re-spawning
retrying tasks in response to clock events, that is).

Unfortunately we cannot assume that submit number always starts at 1 in the
original flow (and so only look at the DB during a reflow) because it would be
possible (if we allow it??) to trigger a reflow beyond the front of the main
flow, before resuming the main flow. If so, options are (TODO):
- Always look up submit number in the DB when a new task is spawned?
  - (Foolproof, and I doubt this would be a significant performance problem)
- Assume 1 if not a reflow, but double check the log directory when the job
  file is written?
  - (log file housekeeping could break this check)

## CLI implications

Some SoS task-targeting commands can be removed or repurposed.

### "cylc insert" not needed

In SoD, a re-triggered task will get inserted automatically with prerequisites
satisfied. Inserting tasks for other reasons does not really make sense in SoD.

A major SoS use case was manual insertion of the first instance of a new task
added to the suite definition mid-run. This isn't needed in SoD [at least
for tasks with parents](#reload-and-restart-after-graph-changes).

### "cylc spawn" repurposed

In SoS this command forced task instances already in the pool to spawn their
next-cycle successors. In SoD it should spawn the children of specified outputs
of abstract tasks, e.g. to set up reflow from a non-bottleneck task.

TODO - consider changing the command name, e.g. `cylc spawn-on-outputs`?

### "cylc remove" needed less

In SoS removing tasks from the pool can remove them entirely from the ongoing
workflow, if the next instance isn't spawned before the removal. In SoD there's
less in the task pool to "remove"; removing active tasks doesn't make sense
(the job is already active); and removing waiting tasks will not stop future
instances of the task appearing on demand.  However, we can keep the command to
allow occasional removal of stuck waiting and finished tasks if necessary.

### "cylc reset" not needed

In SoS this forces the state of task instances in the pool to change, to change
the result of subsequent dependency negotiation (to get a task to run) e.g. to
lie that a failed task succeeded. SoD does not have non-active tasks or
dependency negotiation. Abstract tasks out front are already "waiting" and
don't need to be in the task pool. Finished tasks may be removed immediately.
Failed tasks are just historical information and pretending they succeeded is
unnecessary - better force downstream tasks to carry on despite the upstream
failure. (I'm proposing `cylc spawn` - possibly renamed - as a way to force
downstream tasks to carry on as if certain outputs had been generated.)

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
- actually trigger needs both kinds of globbing: we might still want to
  re-trigger multiple unhandled failed tasks at once (i.e. match in the task
  pool) for instance.

### Restart

In SoS we only store gross task state in the DB, infer prerequisite status
from that on restart and then rely on dependency negotiation to (re-)satisfy
prerequisites of waiting tasks at start-up.

In SoD we no longer have dependency negotation so we need to store the full
prerequisite-satisfaction status of tasks in the DB, not just the gross task
state. More precisely, task pool snapshots need to list:
- the prerequisite components (upstream outputs) completed so far
- the parent_finished status populated so far

### Reload and Restart after graph changes

This gets much better under SoD: if new task definitions the first instance will
automatically be spawned on demand so long as an upstream parent already
exists. The first instance of a newly-defined task with no parents *may* still
need manual insertion (or triggering) though. TODO: we should consider
automatically adding such tasks at some sensible cycle point - e.g. the nearest
valid point for the task relative to the current oldest or newest point, or to
the current runhead limit.

### Succeeded vs failed task handling

If a task fails unhandled (i.e. without a `fail` trigger) we consider it
unfinished, and keep it in the task pool to support easy retriggering (above).

Failure by definition implies the task failed to do what it was designed to do.
But it is possible for us to expect a task to fail, and to build the workflow
around that expectation (that's what `:fail` triggers are for).

By analogy if a task succeeds without a `:succeeded` trigger, should we also
consider it unfinished and keep it in the pool (presuming that it was
"expected to fail")? The answer is NO! Consider `A => B` (or just `B`).  If `B`
is at the end of the workflow we should still presume that it is expected to
succeed even though no other task depends on its success.

So it seems we should treat success and failure differently in the following
sense: an unhandled failed task is kept in the pool as "unfinished", but an
unhandled succeeded task is considered to have finished and can be removed.

## UI TODO

Primarily:
- n>0 windowing:
  - UIS needs to populate n>0 abstract tasks with historical data
  - Probably only allow n=1 (next task) in the future direction

Approaching a stop point:
- If the workflow is told to shut down before the final cycle point (i.e.
  before the end of the graph) there may be some waiting tasks held back in the
  runahead pool because they are beyond the stop point even though they have all
  prerequisites satisfied.  TODO: do we need the UI to make this clear?
  Probably not (other than making the approaching scheduler stop point clear) -
  it isn't really any different from waiting satisfied tasks held back because
  they are beyond the runahead limit. 

Graph isolates: expanding the n-distance window will not reveal tasks that are
not connected (via the graph) to n=0 tasks (nasty example: cycling workflows
with no inter-cycle dependencies).How should the UI make these easily discoverable?
- Provide and "n-max" window that includes whole cycle points?
- Or is searching or filtering by name sufficient?

Reflow:
- Need to distinguish between reflow and main-flow in the UI so that it is
  clear to users why a task with off-reflow dependence is stuck waiting
  (off-reflow parents will appear as finished if the window is expanded).
  Simply distinguishing reflow from main flow should be sufficient because it
  will be relatively easy to explain the problem to users (compared to SoS)
- Allow selecting part of the graph, or particular tasks, to define the
  bounds of a reflow?

Task Outputs:
- The SoD analogue of "task state reset" in SoS, is to tell a task to spawn
  graph-children of particular outputs. E.g. If a task failed but we want to
  the workflow to carry on as if it succeeded, in SoS we would reset the failed
  to succeeded and dependency matching would result in triggering of downstream
  tasks that depend on its success.  Changing a failed task to succeeded isn't
  ideal because it hides the fact that it actually succeeded, but at least the
  change to succeeded is obvious to users. For SoD we would leave the task as
  failed (changing its state to succeeded wouldn't have any effect) but tell it
  to spawn based on the `:succeeded` output even though it failed. This is more
  elegant, but we need to make what happened obvious to users by getting the UI
  to display the state of individual outputs (in the workshop we considered UI
  design for custom outputs - can use the same for standard outputs?)

## Future Enhancements

For Cylc 9: [cylc/cylc-flow#3304](https://github.com/cylc/cylc-flow/issues/3304)

Follow-on changes may (or may not) be worthwhile before Cylc 9:
- Refactor the task pool (which remains largely as for SoS) to avoid (most)
  iteration over tasks?
- Refactor prerequisite and output handling (which remains largely as for SoS)
- Consider not [using waiting and finished task proxies](#use-of-task-proxies)
  to track partially-satisfied prerequisites and prevent conditional reflow.
  (However, I don't think this matters much).
- Extend SoD to clock- and external-triggers: tasks should be spawned in
  response to trigger events (depends on xtriggers as main-loop coroutines)
- Consider using graph traversal for better housekeeping of finished tasks at
  the base of alternate paths.
