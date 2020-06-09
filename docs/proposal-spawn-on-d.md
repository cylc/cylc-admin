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
but that requires major refactoring for Cylc 9.

However, the SoD concept is actually orthogonal to many of our other plans and
can be implemented aleady by simply having task proxies spawn graph-based
children on demand instead of next-cycle successors on submit. This will
simplify Cylc internals and usage and it will only make other Cylc 9 plans
easier to implement in the future.

### Advantages

- **A dramatically smaller task pool** unaffected by spread over cycle points
- **An easily-understood graph-based "window on the workflow"**
- **Task instances can run out of cycle point order**
   - no stalling due to unspawned tasks downstream of a failure
- **Achieve ongoing "reflow" by simply re-triggering a task**
- **No need for dynamic dependency matching** - update prerequisites directly on spawning
- **No need for suicide triggers in alternate path branching**
- **No need for manual insertion of first newly-defined tasks** (unless they have no parents)

### Terminology

- **parent** and **child** tasks: upstream and downstream graph relationships.
  (Not runtime inheritance!)
- **task**: an abstract node in the workflow graph (task name at a particular cycle point)
- **task proxy**: an object in the scheduler program representing a particular
  task that the scheduler is currently aware of
- **task pool**: pool of current "active" task proxies
- **runahead pool**: task proxies that have been spawned (created) already but
  are beyond the current runahead limit, or beyond the (non-final) stop point.
  All tasks are initially spawned into the runahead pool then released to the
  main pool as the workflow (and the runahead limit) moves forward.
- **n=0 window**: tasks that anchor the current view (by default, the content of
  the active task pool)
- **n=M window**: tasks within M graph edges of the those in the n=0 window
- **finished**: means *succeeded*, OR *failed* with the *:fail* trigger
  handled. An unhandled failed task is retained in the task pool as "not
  finished"
- **reflow**: when the workflow flows on according to the graph, from a
  manually triggered task

## Implementation Overview

Spawn-on-demand could conceivably be implemented using dumb tasks that know
nothing but their own job config and outputs, and a scheduler that continually
evaluates task outputs and prerequisites in light of the graph, to determine
what comes next. But that would involve massive refactoring, maybe for Cylc 9.

This implementation leverages existing Cylc internals: a pool of task proxies
that know their own prerequisites (logical expressions of parent task outputs)
and can evaluate them to self-determine readiness to run. The new part here is
just that *task definitions (constructed during graph parsing) record who
depends on each of their outputs* so that tasks can spawn their own children
"on demand". So we still have a self-evolving pool of task proxies (albeit much
smaller and conceptually simpler than the SoS one) with no graph computation at
run time:

Initially (at start-up):
- Spawn and submit tasks that have no prerequisites
Then, as the workflow runs:
- [Spawn children of completed outputs](#spawning-on-task-outputs)
   and/or update their prerequisites
   - (and auto-spawn [parentless tasks](#spawning-tasks-with-no-parents)
- If a task's newly updated prerequisites are satisfied, submit its job
- Remove finished tasks immedidately (succeeded, handled-failed, and expired)
- [Shut down](#stall-and-shutdown) if stalled with no unfinished tasks present

So the SoD task pool contains:
- Active tasks (preparing, submitted, running) 
- Waiting tasks with AND prerequisites in the process of becoming satisfied
- [Unfinished (not handled) failed tasks](#failed-tasks)

## Details

### Spawning on outputs

When to spawn a task is obvious if it only has one parent. Below, B should be
spawned ready to run when A succeeds:
```
A => B
```
It's less obvious if the task has an AND trigger because the component outputs
may be be completed at different times (the parent tasks might not even
overlap, in real time):
```
A & B => C
```
If A succeeds first (say), should C be spawned:
- As soon as A succeeds? (*spawn on outputs*)
  - C will be a **waiting task** until B is done
- Or once both A and B have succeeded? (*spawn on satisfied prerequisites*)
  - C can run immediately once spawned

As it happens **spawning on outputs** is much more straightforward with current
Cylc internals, because task proxies already know how to incrementally update
their own prerequisites as the component outputs are completed. Also,
conceptually, tasks depend on upstream *outputs* so it is reasonable to consider
them "on demand" at completion of the first output.

Spawning on outputs generates some waiting tasks: when AND prerequisites are
"in process". This might appear to create a housekeeping problem - the
potential for [stuck waiting tasks](#stuck-waiting-tasks) - unique to this
implementation. However these waiting tasks just represent partially-satisfied
prerequisites that any implementation has to manage, and not storing them in
task proxies would not make that job any easier(*1)

Waiting tasks are not generated in normal [alternate path
branching](#suicide-triggers-mostly-not-needed) scenarios.

Notes:

- (*1) the fact that we could choose to hide waiting task proxies until they
  are satisfied illustrates the fact that they are merely a (convenient) detail
  of implementation. Hiding them (or even taking the prerequisite information
  right out of them) does not solve the underlying problem, which is: when, if
  ever, can we forget partially satisfied prerequisites if they don't get used.

- Use of waiting tasks, or not, makes no difference to users: they will see
  either waiting task proxies with partially satisfied prerequisites, or
  "waiting" abstract tasks that need to have the same prerequisite information
  attached.

- We could avoid waiting tasks (or other in-memory prerequisite management) by
  using the run DB to satisfy prerequisites. But that would require much more
  frequent DB access than we are comfortable with.

- Upstream tasks are of no use for spawning on satisfied prerequisites: they
  do not know about other parents (if any), let alone the exact prerequisites,
  of their children.

- "Test-spawn" on every output, but delete immediately if prerequisites are not
  satisfied yet? This won't help, because parents that finished earlier have
  been removed the pool (can't check their outputs).

- Prerequisites are by definition n=1 graph edges, so could we keep n=1 in the
  scheduler datastore (instead of just n=0) and use that to avoid spawning
  waiting tasks? No, because we're talking about the n=1 edges of a task that
  does not exist yet (until we spawn it!); and earlier parents may have dropped
  out of n=1 too.

- (Letting stuck waiting tasks stall the workflow is a choice).

### Stuck waiting tasks

Can waiting tasks get stuck and if so, is automatic housekeeping of them
desirable and feasible?

Waiting tasks are only [generated](#spawning-on-outputs) when AND prerequisites
are in the process of being satisfied: 
```
A & B => C
```
Above, C will not be spawned until A (say) succeeds. Then there are only two
ways for it to end up stuck as waiting: either B did not run yet, or B ran
and failed (with B:fail handled, or not).

*If B did not run, removing C is not desirable:* 
```
A & B => C  # (A succeeded, C waiting, B has not run yet)
```
If the workflow isn't stalled then we cannot assume B won't show up later to
allow C to run. If the workflow is stalled then we know B won't (automatically)
show up later, but then the stuck C is just a secondary symptom of some upstream
problem that stymied the path to B - and fixing that could restart the workflow
and allow C to run (either a failed task blocked the flow, or a workflow design
error has put A and B on different alternate paths).

*If B failed and B:fail was not handled, removing C is not desirable:* 
```
A & B => C  # (A succeeded, C waiting, B failed)
```
B is considered not finished (because it wasn't expected to fail) and we
leave it in the pool to be fixed and retriggered, allowing C to run.

*If B failed and B:fail was handled, removing C **may be** OK* on grounds that
C will never trigger by itself once its parents have all finished.
```
A & B => C
B:fail => Z
```
However, what's going on here? Is it a workflow design error? (Was B:fail meant
to fix the workflow, but didn't? Even if there are valid workflows that look
like this they are sufficiently niche that it doesn't warrant complicated
in-memory tracking of parents-finished status (which requires additional
spawning on task completion -- see earlier versions of this document) to allow
automatic removal of waiting tasks as soon as their parents have all finished. 
Further, stuck waiting tasks do not actually a problem unless they accumulate
in large numbers (ain't gonna happen?) or cause the worklfow to stall sometime
later. Better to leave this as one remaining niche case for suicide triggers,
and consider infrequent waiting task checks (via main-loop plugin, or if the
workflow stalls) that get parents-finished status from the DB.

Finally, [reflow](#reflow) can also cause stuck waiting tasks (waiting on
off-flow outputs) but these should not be removed.

### Failed tasks

Technically we don't need to retain failed tasks in the pool: they have already
served their logical purpose in the workflow, and retriggering doesn't depend
on presence in the pool.

Indeed, we consider failed tasks to be "finished" if handled by a `:fail`
trigger (which implies the failure was expected by the workflow) and remove
these from the task pool immediately.

Failed tasks that are not handled (which implies the failure was not expected
by the workflow) are not removed from the task pool. This will keep them
visible in the default n=0 view, and it supports the common "manually
retrigger a failed task" use case without the complications of reflow.

```
A & B => C
```
Above, if `A` succeeds but `B` fails unhandled, `C` (spawned by `A:succeed`)
will be stuck waiting for `B:succeed`. If the user retriggers `B` after fixing
it, and it succeeds, the waiting `C` can then run.

### Spawning tasks with no parents

Tasks with no prerequisites (or which only depend on clock or external
triggers) need to be auto-spawned: at start-up auto-spawn the first instances
of these tasks into the runahead pool, then whenever one is released from the
runahead pool, spawn its next instance (into the runahead pool).

### Absolute dependence

Below, start should run in the first cycle point, after which all `foo.n` can 
run (out to the runahead limit):
```
R1 = "start"
P1 = "start[^] => foo"
```
(Note without the first line above the SoS scheduler will stall with `foo`
unsatisfied, but the SoD scheduler will shut down immediately with nothing to
do because `start` is not defined on any sequence and `foo` never gets
spawned.)

A slightly more difficult case: `foo.1,2,3,...` should trigger off of
`start.2`.
```
...
      R1/2 = "start"
      P1 = "start[2] => foo"
```

So: when an absolute parent finishes *spawn its children at the first cycle
point of the sequence.* Above, `start.2` should spawn `foo.1`, not `foo.2`.
Then whenever an absolute child is released from the runahead pool, spawn
its next instance (to the runahead pool) and mark the absolute dependence as
satisfied. This is valid because even the first child does not get spawned
until the associated dependence is already satisfied. 

Retriggering a finished absolute parent (with `--reflow`) will cause it to
respawn its first child, then subsequent children as normal. This is the right
thing to do under SoD (it's what the graph says) but we may want to provide an
option to change the first child spawned to a current cycle rather than going
back to the start (use case: retrigger a start-up task that rebuilds a model or
whatever, but don't re-run old cycles). Alternatively, simply retrigger the
absolute parent without reflow, then retrigger with reflow the first child that
you want to continue spawning forward.

### Stall and shutdown

A stalled workflow cannot carry on without intervention. This could mean:
- the workflow ran to completion
- or an unhandled task failure stymied the workflow

In SoS the scheduler can distinguish the two cases (most of the time!) because
all tasks are spawned ahead as waiting without regard for what is "demanded" by
the graph. If the workflow has completed there will be no waiting tasks left in
the pool, otherwise the stall is premature.

In SoD there may be no tasks waiting ahead (on the "intended path") to show
there is more to do, but we can detect workflow completion as follows:

- If the workflow is NOT stalled,
   - then it has not completed (obviously)
- If the workflow is stalled,
   - no unhandled failed tasks present and all waiting tasks beyond the stop
     point (and in the runahead pool): workflow completed
   - otherwise (unhandled failed tasks and or stuck waiting tasks present):
     premature stall

Note that final cycle point isn't much use in determining workflow completion because:
- it's possible to have no graph recurrence sections that extend to the final point
- we can't know if every task valid in the final cycle point is actually
  expected to run there, because alternate paths are possible

Note that bad failure handling (defined as failure handling that does put the
workflow back on the intended path) can cause premature shutdown, but we can't
second-guess that if it is what the graph logic says to do.
```
A:fail => email_me
A => B => C = archive
```
Above, if `A` fails the workflow will (correctly!) complete on `email_me`
because there is no path that leads to `archive` in that case.

### Spawning and stop points

**Stopping at a stop point prior to the final cycle point:** 
If the scheduler shuts down before the final cycle point (e.g. stop now, or
stop at cycle point P, etc.) the task pool may include waiting tasks in the
runahead pool beyond the stop point. This allows a restart to carry on as
normal, beyond the stop point.

**Stopping at the final cycle point:**
The workflow final cycle point represents the end of the graph. All graph
recurrence expressions end here so in general the "next" instance of task
is undefined beyond that point. When the scheduler shuts down at the final
cycle point the task pool will therefore be empty, and is not possible to
continue from there with a restart. To go beyond the final cycle point
you need to change the final cycle point in the workflow definition and 
restart from an earlier checkpoint, or warm start at any point prior to the new
final point.

### Suicide triggers mostly not needed

Suicide triggers are no longer needed for workflow branching because waiting
tasks do not get spawned on the "other" branch. Below, if A succeeds only C
gets spawned, and if A fails only B gets spawned:
```
A:fail => B
A => C
```

Tim W's case: `B` triggers and determines that `xtrigger1` can never be
satisfied, so we need to remove the waiting `A` (the two xtriggers are watching
two mutually exclusive data directories for a new file):
```
@xtrigger1 => A
@xtrigger2 => B
```
As described this remains a case for suicide triggers (although they won't be
needed here either if we can spawn xtriggered-tasks "on demand" too, in response
to xtrigger outputs).

The other remaining case for suicide triggers is [stuck waiting
tasks](#stuck-waiting-tasks) downstream of an AND trigger:
```
A & B => C 
```
Above if A succeeds and B fails, C will be stuck waiting. 


if we don't clean up
(periodically or on stalling)  multiple parents
that are all finished multiple parents all with downstream of a handled failure
In any case there is no harm in keeping suicide
triggers for backward
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

### Preventing conditional reflow

TODO: modify for new DB USE

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


*ASIDE: the proposal original kept finished task proxies in the pool as well
until their parents are all finished, to prevent "conditional
reflow" (in `A | B => C` if `B` runs after `C` has already triggered off of `A`
and finished, we need to avoid triggering `B` again off of `B`). However,
an alternative to storing the fact that `C` already ran in memory and
housekeeping that information, is to ask the run database if `C` already ran,
before spawning it. As it turns out, we have to ask the database for submit
number at spawn time, and that gives us the same information (did `C` already
run?).
