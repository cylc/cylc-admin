# Proposal: Cylc 8 Spawn-on-Demand

**Hilary Oliver**, *December 2019 - February 2020*

## Table of Contents

- [Terminology](#background)
- [Background](#background)
- [Advantages of SoD](#advantages)
- [Implementation Overview](#implementation-overview)
- [Details, Choices, and Subtleties](#details-choices-and-subtleties)
- [Implications](#implications)
- [Future Enhancments](#future-enhancements)


### Terminology

- "upstream" and "downstream" refer to graph relationships
- "task" is short for "task proxy object"
- "n=0 window": all tasks the scheduler is currently aware of 
- "n=M window": all tasks within M edges of the n=0 pool


### Background

We first considered replacing task-cycle-based
[Spawn-On-Submit](spawn-on-submit-history) (SoS)
with graph-based Spawn-On-Demand (SoD) back in
[cylc/cylc-flow#993](https://github.com/cylc/cylc-flow/issues/993).

The ultimate solution probably looks like
[cylc/cylc-flow#3304](https://github.com/cylc/cylc-flow/issues/3304)
but that involves a major refactoring of the codebase for Cylc 9.

In fact SoD is technically orthogonal to other Cylc 9 plans and can be
implemented easily within the current framework, by simply getting task proxies
to spawn downstream tasks on demand instead of next-cycle successors on submit.
This will simplify Cylc internals and will only make future Cylc 9 changes
easier to implement in the future.

## Advantages of SoD

- Dramatic reduction in task pool size (e.g. 10-100x in large workflows)
  - No need to hold many waiting and succeded tasks in addition to the "active" ones
  - No need for dynamic dependency matching (update prerequisites directly)
  - Lighter load on the UI; the graph view becomes usable in large workflows
- A graph-based "window on the workflow" that is easy for users to understand.
  - (try explaining SoS to users!)
- Suicide triggers not needed for alternate path branching
    - the unused path does not get spawned in the first place
- Task instances can run out of (cycle point) order
  - No stalled workflows due to unspawned waiting tasks downstream of a failed task
- "Re-flow" sub-graphs of the workflow by simply triggering the top task
  - (automatic partial triggering by the original flow is difficult though)
- Solves the first-instance problem of task definitions added mid-run
  - (in SoS users have to add the first instance manually - after which it
    will spawn its own next-cycle successor - because we can't guess which
    instance should be the first one; in SoD the right one will just appear on
    demand)

## Implementation Overview

Details and subtleties are discussed below, but the SoD algorithm is
essentially as simple as this:

1. Record the downstream tasks that depend on each upstream task output
   - Available at graph parsing but not used in SoS
1. At start-up auto-spawn tasks with nothing upstream to spawn them
   - And continue to do this as the workflow evolves
1. As tasks complete outputs, spawn (if not already spawned) the downstream
    tasks that depend on them
   - And update their prerequisites directly
1. Remove finished tasks immedidately, so long as doing so won't empty the task pool

## Details, Choices, and Subtleties

### Spawning on upstream outputs

Tasks can have multiple prerequisites that get completed at different times, so
we have to either:
- spawn on upstream outputs and manage waiting tasks with partially satisfied
  prerequisites.
- OR manage prerequisites separately and only spawn a task when all of its
  prerequisites are completely satisfied.

Spawning on outputs is easier in the current framework, because:
- tasks already know how to evaluate satisfaction of their own prerequisites
- this makes it easier to expose partially satisfied prerequisites to users

(And the memory cost of keeping a few whole task proxies - as opposed to just
the prerequisites and associated methods - around for a bit longer than
strictly necessary is very low).

### Partially satisfied waiting tasks

We still have a few waiting tasks: those with partially satisfied
prerequisites (see just above). Unlike SoS there are no wholly unsatisfied waiting
tasks. We could choose to hide these in the runahead pool until fully satisfied
to make it look as if they are only spawned at the last moment. However:
- users need to know if a task's rerequisites are partially satisfied 
- and any partially satisfied tasks (or sets of prerequisites) that don't get
  fully satisfied (because their upstream outputs are never generated) have to
  be exposed to users or else automatically removed.

So the easiest thing to do at first is simply expose all waiting tasks in the
main pool. We can figure out later which ones can be removed automatically.

Possibly: waiting tasks can be removed if:
- all of their upstream tasks have *finished*
  - (upstream tasks should update their finished status in downstream tasks?)
- all active tasks have moved on to future cycle points?

### Avoiding Conditional-Triggered Reflow

```
A | B => C
```
If `C` triggers off of of `A` we need to avoid spawning `C` again if
`B` runs later (after `C` has already finished). 

So: keep finished conditionally-triggered tasks in the pool until
- all of their upstream tasks have *finished*
- all active tasks have moved on to future cycle points?

(Same as for housekeeping partially satisfied waiting tasks, above)

(restrict to conditional prerequisites, or make it universal for convenience?)

(If cycle point based housekeeping is not valid in general we could cache the
infor for some arbitrary period, and query the DB if it is not in the cache?)

### Finished task removal and scheduler shutdown

In SoS normal shutdown occurs when there is nothing left to run, i.e. there are
no waiting tasks in the pool.

```
foo => bar => baz ...
foo:fail => diagnose => cleanup
```
In this example, the intended path is `foo => bar => baz ...`. If `foo` fails
SoS will stall when `cleanup` finishes, but we know the workflow is not
finished because waiting tasks have already been spawned on the intended path.

SoD seems potentially problematic in this respect because tasks can be removed
immediately once finished, and there may not be any waiting tasks at all
(unless there are partially satisfied preserequisites). In this example,
if `foo` fails `bar` will not be spawned at all as the workflow follows the
failure path, and if `cleanup` is removed once finished there will be nothing
at all left in the task pool. How do we distinguish this from "workflow
finished" when there is no way to automaticaly determine the intended path.

We considered keeping non-terminating finished tasks in the pool if they have
"un-handled" outputs.  For instance, `foo` above failed but it has a task that
depends on `foo:succeed` that was not used. However this doesn't work in
general. Failure may or may not be handled; it may even be expected in some
cases; and Cylc can't know if custom outputs are sequential or mutually
exclusive (one per branch, say).

(And if we stayed alive with an empty task pool, where would the n=0 window be
anchored for users?)

SOLUTION: **remove finished tasks immediately unless doing so
would leave the task pool empty.** Then,
- the n=0 window contains current active tasks OR (if the workflow stalled)
  the most recent active tasks - which is exactly what the window should be
  centered on
- when the workflow stalls the n=0 window may no longer contain the branch
  point where things went off piste (in the example above, it would be just the
  `cleanup` task) but that's fine - the essence of SoD is to follow the graph
  where it leads. If stalled, widen the n=M window to see what led to the stall. 

It may be difficult to automatically distinguish "workflow finished" from other
stalls, in general. ("All tasks finished in the final cycle point"? - nope,
there might be unused branches in the workflow, or it could be a massive
non-cycling workflow...). We could allow users to specify what normal shutdown
looks like, e.g. "`archive.$` succeeded" (where `$` is final cycle point) so
that the scheduler can log "finished" instead of "stalled".

Perhaps we should treat any stall (including workflow finished) like this:
- report (UI and log) "workflow stalled with (list n=0 tasks)"
- wait for user action or shutdown on a 15 minute (say) timeout
- a restarted stalled workflow will similarly wait for 15 minutes before
  shutting down again if not un-stalled

### Tasks with nothing upstream to spawn them

These tasks (those with no prerequisites at all, and those that depend only on
xtriggers - including clock triggers) need to be auto-spawned out to the
runahead limit.

In future we can spawn xtrigger-tasks on demand too, in response to
outputs/events emitted by xtrigger functions,
but for the moment auto-spawning them as waiting tasks is easy to do and makes
it easy to expose what's going on to users.

### Suicide triggers *mostly* not needed

Suicide triggers are no longer needed for normal workflow branching: the path
that's not needed will not get spawned at all. We may still need them for
obscure edge cases though. E.g. (Tim W): if task `foo` is waiting on an
xtrigger `x`, and `bar` determines that `x` will never be satisfied, `bar`
would need to suicide the waiting `foo`. (Not however this is an artifact of
auto-spawning xtriggered tasks - if these tasks were also spawned on demand in
response to xtrigger events, suicide triggers wouldn't be needed here either).

### What's in the n=0 Window?

- current active tasks: preparing, submitted, running
    - or the most recent active tasks, if the workflow has stalled
- waiting tasks with partially satisfied prerequisites
- waiting tasks that are waiting on xtriggers

### REFLOW

Re-flow means triggering a finite partial re-run of some sub-graph of the
workflow, or a full re-run from some earlier task or tasks in the workflow. 

## Reflow in SoS

- A restricted form of reflow will occur automatically in SoS if you
  (re-)insert and trigger a task with only previous-instance dependence
  (`foo[-P1] => foo`). 
- Otherwise it won't work without complicated manual placement of downstream 
  task proxies and/or state reset of still-existing ones. (And it is near
  impossible to get this right without using the graph view).

## Reflow in SoD

In SoD the workflow naturally reflows from any (re-)triggered task.

It is easy and safe to retrigger an isolated sub-graph, or the whole workflow,
from points that flow to all downstream tasks.

But what if some tasks have some prequisites that cannot be satisfied within
the reflow?

Reflowing tasks will not automatically be satisfied by the previous flow
because those task proxies are gone.

INITIALLY we should:
- support connection to the previous flow by manual intervention
  - click on upstream tasks (note in Sod these won't have proxies in the pool)
    to spawn downstream based on specific (or all) outputs (this will allow
    pre-reflow setup all at once)
  - and/or click on waiting tasks to trigger now and/or force-satisfy
    specific prerequisites
- we need to solve the [submit number problem](#submit-number), but that goes
  for SoS too
- The UI will need to make it clear why a task is stuck waiting, as the upstream
  off-reflow will appear as finished (the problem is, they finished *before*
  the reflow was triggered)
  - mark recently-active tasks? e.g. a badge that fades from most recent to old?

FUTURE ENHANCEMENT
- We may want to allow automatic satisfication of reflowed tasks by previous
  flow tasks that are outside of the reflow graph
- (This is not a blocker for initial implementation - we can't do it in SoS!)
- Requires determining which prerequisites will come (eventually, if not
  immediately) from the reflow and which can only be taken from the previous
  flow - a graph traversal problem, potentially difficult?
- (Also: how to know if the reflow has infected the whole workflow so that we
   can stop checking for this?)
- This probably requires implementing a "flow number" that is passed to
  downstream tasks on spawning.

## Implications

### "cylc insert" not needed

In SoD, a forcibly-triggered task gets inserted automatically with
prerequisites satisfied. (If needed for test purposes we could allow the
trigger command to manipulate individual prerequisites?). Inserting tasks for
other reasons does not really make sense in SoD.

A major SoS use case was manual insertion of the first instance of a new task
added to the suite definition mid-run. In SoD the first instance will
automatically appear on demand.

### "cylc spawn" not needed

This SoS niche command was used to make a task spawn its next-cycle successor,
usually before removing the original. No analogous command is needed in SoD.

### "cylc remove" not needed

In SoS removing tasks from the pool will effectively remove them from the
running workflow because future instances won't be spawned. In SoD there's not
much in the task pool, and:
- Removing an active task doesn't make sense (the job is submitted or running
  already)
- A removed partially satisfied waiting task would just get spawned again when
  the next prerequisite gets satisfied (however it would be stuck waiting on
  the original pre-removal outputs). And it would not prevent future instances
  getting spawned (tasks don't spawn their own successors on SoD).

### "cylc reset" not needed

Task state reset is used in SoS to forcibly change the state of existing task
proxies so that dependency negotiation will get a different result (in order
to, usually, cause a downstream task to run).

SoD does not have "existing task proxies" (much) or dependency negotiation. So
`cylc reset` functionality should be removed.

However, we should delineate the use cases for reset and make sure SoD has them
covered:

- SoS: force a bunch of already-finished tasks back to waiting, prior to a reflow
  - SoD: not needed, reflow is automatic without waiting tasks out front
- SoS: force a failed task to succeeded to allow a stalled workflow to carry on
  - SoD: better not to lie about the failure, instead force the downstream
    tasks to carry on in spite of it
- Anything else?


### Commands that target multiple tasks

E.g. to retrigger all tasks `foo*` in cycle points `202303*`.

In SoS this functionality is far from perfect: it only affects the task pool!
And reflow might occur or not, depending on what waiting tasks happen to exist
in the pool.

In SoD the only task-targeting control commands left are:
- `poll` and `kill`
  - easy, these only need to look at active tasks (in the SoD pool)
- `hold`, `release` (do we still need to support these for individual tasks?)
  - `hold` applies to future tasks, so keep globs in memory and check
    them when tasks get spawned? (so held tasks will be waiting in the pool)
  - `release` - applies to held tasks, so need to look at the pool, and tidy
    the recorded hold globs list
- `trigger`
  - historical tasks: just check they exist and are on-sequence, then insert
    and trigger (no need to check the DB unless for submit number - see above)
  - future tasks: do we need to allow this beyond the n=1 window? (not possible
    the future when the upcoming grapy may be determined dynamically)

### Submit number

Retry number increments with automatic retries; submit number increments with
any retry (automatic or forced). Job log path includes submit number, to avoid
clobbering older logs.

In SoD retry number is safe as the task proxy doesn't disappear, it goes from
failed to waiting on a clock trigger (see "tasks with nothing to spawn them"
above). Submit number is a problem because finished tasks disappear from the
pool immediately (so the task proxy can't store it).

Note **this does not work correctly in SoS!**. Inserted tasks get the right
submit numbers via DB look-up, but their spawned next-cycle instances (which
are needed if you want to achieve reflow) start at submit number one again and
clobber the old jobs logs.

To get this right (in SoS or SoD) we would need to either:
- look up submit number every time a task is spawned, OR
- **get it from the job log dir before writing the job script?**

A completely in-memory solution is not feasible without remembering the entire
run history (and that's what the DB is for). Using the log directory may be
best - it's simple, and we have to go to disk at that point anyway.

### Restart

In SoS we store gross task state in the DB and rely on dependency negotiation to
(re-)satisfy prerequisites of waiting tasks at start-up. In SoD prerequisites are
satisfied by upstream tasks at the moment of output completion (and upstream
tasks might be gone by shutdown) so we'll need to store task prerequisites
in the DB.

## Future Enhancements

For Cylc 9: [cylc/cylc-flow#3304](https://github.com/cylc/cylc-flow/issues/3304)

But but several easy wins may be possible before then:
- Refactor the task pool to avoid iterating to find the right task. A simple
  task ID-keyed dict might do it.
- Consider unifying prerequisites and outputs, which might lead to separate
  tracking of prerequisites
- Extend SoD to clock- and external-triggers: tasks should be spawned in
  response to trigger events (depends on xtriggers as main-loop coroutines)
