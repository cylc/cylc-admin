# Cylc 8 n-distance windows for the UI

Cylc 8 spawn-on-demand makes it possible to display (to users) an intuitive
graph-based window centered on current active tasks.

## Basic Concept

*The UI, in all views, should display current active tasks plus those out to
`n` graph edges from the active ones. So `n = 0` refers to active tasks;
`n = 1` to the parents and children of the active tasks; and so on.*

This is really the only well-founded, easily comprehended, and scalable way to
display large Cylc-style cycling workflows. Only in smaller cases is it
reasonable to show all the tasks in active cycles.

## The Datastores

Windowing will be managed by similar datastores in the scheduler and UI Server
- the scheduler only needs `n = 0` for scheduling
  - but its datastore may go out to `n = 1` for convenience
- the UIS will compute higher-n windows and populate non-active tasks from DB
  - to do that it will need the graph pattern from the scheduler

## Future Tasks

PROPOSE: stop at `n = 1` in the future direction
- `n = 1` shows what comes next in the flow; beyond that nothing has happened
  yet and showing it will unnecessarily reduce screen space for active tasks
- (plus, Cylc 9 changes may make the future graph less predictable)

## Disappearing waiting tasks?

### non-task prerequisites

Tasks may be held back from becoming active even though their graph/task
prerequisites are all satisfied, by runahead limit, queues, xtriggers,
and task hold. These will not be visible in `n = 0` because they're not
active, or in `n = 1` because their parents are no longer active. Users will
see `n = 1` tasks (which are supposed to show what's coming next) disappear
from the UI, and suddenly reappear later when their queue (say) releases them.

*This problem is an artifact of the fact that the graph only represents task
dependencies*. Xtriggers (etc.) should really be represented as entities in the
graph that persist at `n = 0` (c.f. long-running tasks that periodically emit
outputs to satisfy children), then the problem tasks can be seen as just normal
`n = 1` tasks that are still waiting on their active xtrigger (etc.) parents.
They would stay in `n = 1` until *all* prererquisites (not just task
prerequisites) are satisfied, then go straight to `n = 0` as normal. In light
of this:

PROPOSE: keep these `waiting` tasks visible in `n = 1` until active
- (add artificially to `n = 1` as edge-following from `n = 0` won't find them).

POST CYLC 8:
- figure out how to represent all dependence in the graph
- unify task and non-task prerequisites?
- perhaps take an event-driven approach to queues etc. too?

```
foo => bar
@queue => bar
```

### parent gaps

A task with multiple parents will look like it is "coming next" (`n = 1`) with
respect to each active parent, even though that won't really be true until 
the very last of its parents becomes active. That's fine if all the parents run
concurrently, but otherwise the child can repeatedly disappear and reappear
until it becomes active itself. This will likely be disconcerting to users.
```
A & B => C
```
Here, if there is a significant gap between `A` succeeding and `B` becoming
active, `C` will be visible in `n = 1` when `A` is active, then it will
disappear from the UI in the gap between `A` and `B`, and finally appear again
when `B` is active.
 
### Possible solutions

1. live with it? It probably won't happen very often, and it's easy to explain
     to users (strict `n = 1` edge from active tasks)
2. artificially exclude the child from `n = 1` until the last parent is active?
     - check for other unsatisfied prerequisites before adding the child?
3. always show all parents of a child at `n = 1`? 
     - the child will still disappear in a parent gap, but at least that will
       be foreshadowed by visible inactive parents before it does disappear

PROPOSE: implement (1) initially, and later consider (2) and (3) as
enhancements. (2) looks best but it may be tricky avoid wrongly excluding a
child from `n = 1` in alternate path graphs?

## Graph Isolates

Windowing will often result in multiple disconnected graphs in the graph view
(even without considering reflows). We will just have to live with that
(because, see Justification above).

POST CYLC 8: consider ways to display multiple disconnected graphs better, such
as by bridging with "scissor nodes" as in the Cylc 7 GUI?
