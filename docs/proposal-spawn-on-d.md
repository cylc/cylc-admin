# Proposal: Replace Spawn-on-Submit with Spawn-on-Demand

## Proposal

We have been considering spawn-on-submit as a major enhancement for some time,
but have treated it as tied to other Cylc 9 plans such as abstracting out the
scheduling kernel.

HOWEVER: we can do spawn-on-demand easily with the current task pool framework,
and it is in fact orthogonal to an abstracted kernel (etc.) - this change will
not make it harder to do that in the future (easier if anything).

This will provide a major efficiency boost by greatly reducing the size of the
task pool and eliminating the need for dynamic dependency matching, and it will
enable workflow branching without the need for suicide triggers.

## Spawn-on-submit (OLD)

Cylc manages an evolving pool of "task proxy" objects that move along a
potentially infinite workflow graph. Task proxies must exist by the time (or
before) they are needed, to avoid holding up the workflow. Currently we
achieve this by:

At parse-time:
- decompose the graph to determine inputs and outputs of every task

At start-up:
- instantiate the first instance (task.cycle-point) of every task

At run-time:
- tasks with no inputs can submit their jobs to run immediately
- whenever a task message comes in, see if it matches unsatisfied inputs in
  the pool (dependency matching)
- once a task has all inputs satisfied, it can submit its job to run AND spawn
  its own next-cycle-point successor
- Succeeded task proxies are kept until no longer needed to satisfy the inputs
  of waiting tasks in their cycle point or earlier (adjusted for inter-cycle
  dependencies)

Historical background: early versions of Cylc had no "suite definition" or
graph, just a collection of separate task definitions with inputs and ouputs to
be matched at run time by the "self-organising scheduling algorithm". Tasks
also knew their own cycling sequences so it seemed natural to have them to
spawn their own next-cycle-point successors. Spawning at submit-time prevents
uncontrolled spawning into future cycles but still allows successive instances
of the same task to run concurrently if the opportunity arises.

Nowadays the suite.rc file provides a central suite definition and the graph
allowed users to work directly with the structure of their workflow.
Technically the graph makes dynamic dependency matching unnecessary (the graph
knows exactly whose outputs satisfies whose inputs) but it is just bolted on
the front: we use it to determine the inputs and outputs of each task and pass
them to the original self-organising scheduling algorithm.

PROBLEMS:

Spawn on submit works remarkably well but it has the following problems:
- successive instances of a task can run concurrently (depedendencies and
  history allowing) BUT they can't run out of order. This can stall a suite due
  to tasks waiting unspawned downstream of a failed task (the failed task
  itself will have spawned).
- the task pool is unnecessarily large because it has to include at least one
  cycle-point instance of every task in the suite, more if the suite extends of
  several cycle points (succeeded tasks are kept in the pool until the fall out
  the back end). Parameterized tasks particularly bloat the task pool.
- suicide triggers are needed to get rid of already-spawned waiting tasks that
  will never run due to graph branching (this shouldn't happen if tasks are
  only spawned when needed).


## Spawn-on-demand

At parse-time
- determine inputs, outputs and output-targets (new) of each task

At start-up
- instantiate all tasks with no inputs, out to runahead limit

At run-time
- when an output message comes in, the owner task should (on demand)
  instantiate or update

