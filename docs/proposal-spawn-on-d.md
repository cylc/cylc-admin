# Proposal: Replace Spawn-on-Submit with Spawn-on-Demand

We have considered spawn-on-demand as a major enhancement for some time but
have tended to tie it to Cylc 9 plans such as an abstracted scheduler kernel.

However spawn-on-demand is technically orthogonal to these other plans and we
can do it quite easily within the current framework. This will NOT make other
Cylc 9 changes any harder (it will make them easier if anything). It will
provide a huge efficiency boost by dramatically reducing the size of the task
pool; it will enable workflow branching without suicide triggers; and
there will be no need for dependency matching and task pool housekeeping.

## Currently: spawn-on-submit

We instantiate the first cycle-point instance of *every* task in the workflow
then have each one spawn its own next-cycle successor at job submit time (which
prevents uncontrolled spawning into future cycles but still allows successive
instances of the same task to run concurrently if the opportunity arises). This
has worked remarkably well but it does have the following problems:
- The task pool has to include at least one cycle-point instance of every task
  in the workflow, and more if there are multiple active cycle points. So
  parameterized iteration, for example, can badly bloat the task pool.
- Instances of a task cannot run out of order. So unspawned tasks downstream of
  a failed task can cause a stall (the failed task itself will spawn).
- Suicide triggers are needed to get rid of already-spawned waiting tasks that
  will never run on alternate paths through the graph.
- (dynamic dependency matching and task pool housekeeping can be expensive -
  however these are not strictly required for spawn-on-submit either - see
  NOTES below)

## Proposal: spawn-on-demand

At parse-time:
- decompose the graph to determine prerequisites and outputs of every task (as
  now) **AND** the downstream target task ID(s) of each output.

At start-up:
- instantiate the first (cycle point) instance of every task with no task
  prerequisites, as far ahead as the runahead pool. This is typically a small
  number even in huge suites.

At run-time
- as output messages comes in, instantiate the target tasks (if not already
  instantiated) and update their prerequisite(s)
- when a task finishes it can be removed from the task pool after updating
  downstream tasks with its final output(s)
- change the task pool from the current list to a task-ID keyed dictionary, so
  that no iteration is required to find target tasks

Notes:
- retain the runahead pool for tasks that are spawned too far ahead, including
those with no prerequisites. When the the runhead point moves forward, spawn
more no-prereq tasks out to the new point.
  - future no-prereq tasks don't need to be spawned into the runahead pool
    immediately (e.g. final-point tasks, to pick the extreme case) although
    they could be (TBD).

## Background info

Cylc manages an evolving pool of "task proxy" objects that move along a
potentially infinite workflow graph. To avoid holding up the workflow task
proxies must exist at (or before) the time they are needed. 

Currently cycling sequences are stored by each task to enable spawning of
next-cycle successors - need to change this?

Dynamic dependency matching is technically unnecessary in the current system -
because the graph knows exactly who satisfies whose prerequisites, but (at
Cylc 3?) we bolted the graph on the front, using it to determine prerequisites
and outputs to pass to the original "self-organising scheduler" which works on
a loose collection of task proxies with prerequisites and outputs.

We have discussed allowing self-assembling workflows again (more like the
originally Cylc) but even those won't require dynamic dependency matching - we
can have the graph self-assemble at start-up, after which the dependencies are
static. (Dynamic matching is really only required to allow insertion of new
tasks into a running workflow).
