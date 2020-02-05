# Proposal: Spawn-on-Demand

*Hilary Oliver, December 2019*

We have talked about replacing spawn-next-instance-on-submit with
spawn-dependent-tasks-on-demand for some time (years!) but have tended to tie
it to Cylc 9 plans such as an abstracted scheduler kernel.

However spawn-on-demand is technically orthogonal to these other plans,
surprisingly easy to implement in the current system, and it won't make Cylc 9
changes any harder in the future (it will make them easier if anything). The
advantages are so great that we should seriously consider doing it for Cylc 8:
- A dramatic reduction in task pool size, and hence a big performance boost
  - e.g.: a pool of 1000+ tasks/cycle (active, waiting, succeeded)
    might drop to 10's of task/cycle max.
  - (Dramatically reduces number of tasks for the UI too)
- Naturally gives a sensible/explicable window on the workflow
  - (we were planning to fake this before Cylc 9).
- Easily re-run sections of workflow (e.g. back up to re-run a whole sub-tree)
- Workflow branching without suicide triggers.
- No need for dependency matching and task pool housekeeping.

## Current: spawn-on-submit

At start-up we instantiate the first cycle-point instance of *every* task in
the workflow, then each spawns its own next-cycle successor at job submit time
(which prevents uncontrolled spawning but still allows successive instances to
run in parallel).
- This achieves the primary goal of ensuring that task proxies exist at or
  before the time they are needed,
  - except that tasks can't run out of cycle-point order (one consequence:
    failed tasks have already spawned their own successors, but tasks waiting
    downstream of them won't have, which stalls the worklow)
- Also, because tasks spawn their own successors the task pool has to include
  at least one cycle-point instance of *every task*, and more if there are
  multiple active cycle points.
  - this is overkill - a lot of tasks are created in the waiting state long
    before they are needed (including parameterized tasks for the whole
    parameter range).
  - the overkill is made even worse by the need to keep succeeded tasks around
  for dynamic dependency matching
- Suicide triggers are needed to get rid of already-spawned waiting tasks that
  will never run on alternate paths through the graph.
- Re-running a sub-tree requires careful manual insertion of relevant task
  proxies while the workflow is held, because spawning is not
  dependency-driven.
- Dynamic dependency matching can be expensive (at least it used to be).

## Proposal: spawn-on-demand

#### At parse-time:
- Decompose the graph to determine prerequisites and outputs of every task (as
  now) **AND** the downstream task ID(s) targetted by each output.

#### At start-up:
- Instantiate the first (cycle point) instance of every task with no task
  prerequisites, as far ahead as the runahead pool. This is typically a small
  number even in huge workflows.

#### At run-time
- As output messages comes in, instantiate the downstream tasks (if not already
  instantiated) and update their prerequisite(s).
- When a task finishes (succeed or fail) it can be removed from the task pool
  as soon as it has updated downstream tasks with its final output(s).
- The task pool should be a task-ID keyed dict, not a list, so no iteration is
  required to find target tasks (to update their prerequisites).

#### Notes, edge cases, etc.:

- Cycling sequences are currently stored by each task to enable spawning of
next-cycle successors - where's the best place to hold this for spawn-on-demand?
- Need to retain the runahead pool for tasks that are spawned too far ahead,
including those with no prerequisites.
  - When the the runhead point moves forward, spawn more no-prereq tasks out to
    the new runahead point.
  - (No need to spawn first instances of late-onset no-prereq tasks into the
    runahead pool at start-up (e.g. stop-point tasks) although could do (TBD).
- Suicide triggers:
  - not needed so long as the alternate paths don't share outputs.
  - self-suicide not needed, as finished tasks need not be kept in the pool.
- Natural workflow window: n=1 based on outputs (not on tasks; if we spawn
  downstream tasks on submit or on start instead of on outputs, suicide
  triggers may become necessary again).
- This makes it much easier to
  [https://github.com/cylc/cylc-flow/issues/2143](hide the concept of insertion
  from users). To retrigger a sub-tree (e.g.) we would still need to
  auto-insert the first task proxy but spawn-on-demand takes care of the
  rest.
- In caught-up real-time operation, between cycles the task pool will only
  contain the few tasks that are waiting on clock-triggers (and the UI can
  optionally show their n-distance descendants as well if requested)
- Conditional triggers are the only tricky thing? Consider "A|B => C"
  with C triggering off of A. Do we remove C once it has finished regardless of
  whether or not B is running or will run? If B does run later we need to stop
  C being spawned again.
  - So: for conditional triggers only, we need to check that the downstream
    tasks have not already spawned and executed? (Ask the DB, or keep some
    record in memory?)
- If a disembodied output message arrives, choice to ignore it or
  instantiate the owner task

## Misc. Notes

The reason for spawn-on-submit and run-time dependency matching is historical:
Cylc originally had no workflow definition and no graph, just a loose
collection of separate task definitions with their own prerequisites and
outputs - it was up to Cylc to make the connections.

Each task also knew its own cycling sequence (e.g. the forecast model should
run on a 6 hour cycle...), so it seemed natural that a task should spawn its
own next-cycle successor.

When I bolted the "new" suite.rc and graph on the front (Cylc 3?) I used the
graph only to determine automatic prerequisites and outputs for each task, and
just passed these to the original "self-organising scheduler". But this dynamic
dependency matching is technically unnecessary even in the current system,
because the graph knows who satisfies whose prerequisites. 

We have discussed allowing self-assembling workflows again (more like the
original Cylc) but even those won't require dynamic dependency matching - we
can have the graph self-assemble at start-up, after which the dependencies are
static. (Dynamic matching is really only required to allow non-predetermined
changes in workflow structure at run time - which we might come back to at some
point).

Cycling is still a special form of iteration compared to parameterization. I
think that is reasonable because cycling can go on indefinitely. Perhaps with the
future Python API we may be able to put the two concepts on equal footing and
just follow the graph - whatever it is - generated by arbitrary user-supplied
iterative functions. But it isn't yet clear whether that is feasible or not,
and anyway this proposal has no impact on our ability to go there in the
future. 
