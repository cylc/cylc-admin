# How Spawn-On-Submit Works

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

## Implementation:

At start-up we instantiate the first cycle-point instance of *every* task in
the workflow, then each spawns its own next-cycle successor at job submit time
(which prevents uncontrolled spawning but still allows successive instances to
run in parallel). This works remarkably well (at ensuring task proxies exist
before they are needed) BUT:
- Tasks can't run out of cycle-point order
- The successors of a failed tasks can run (tasks spawn before failing) but
  those of tasks waiting downstream of a failed task can't, which stalls the
  workflow
- The task pool has to include at least one cycle-point instance of *every
  task*, and more if there are multiple active cycle points.
 - this is overkill - a lot of tasks are created in the waiting state long
   before they are needed (including parameterized tasks for the whole
   parameter range).
 - this overkill is made even worse by the need to keep succeeded tasks around
   for dynamic dependency matching
- Suicide triggers are needed to get rid of waiting tasks that will never run
  (e.g. on alternate paths through the graph).
- Re-running a sub-tree requires careful manual insertion of relevant task
  proxies while the workflow is held, because spawning is not
  dependency-driven.
- Dynamic dependency matching is expensive in large suites.

