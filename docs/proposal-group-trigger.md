# PROPOSAL: Task Group Triggering

## Background: re-running a sub-graph in Cylc 7 vs Cylc 8

For typical use cases where all affected tasks and their parents remain in the
Cylc 7 task pool, it may be easier to rerun a past sub-graph in Cylc 7 than in
Cylc 8.

This is because it may require more graph knowledge to identify the
initial task(s) and off-flow prerequisites of the sub-graph (Cylc 8) than to
identify all of the sub-graph member tasks as a group (by family name or glob) 
and reset them to waiting (Cylc 7).

## Proposal

Currently, `cylc trigger TASKS` makes all the target tasks trigger immediately.

That's almost certainly not the desired behaviour if there is any dependence
between the targeted tasks.

Note that *for any given group of tasks we can examine all prerequisites
to find those that point outside of the group*. I propose changing the default
triggering behaviour to:
 1. respect any internal dependencies between the target tasks
 2. automatically satisfy any off-group (i.e., off-flow) prerequisites 

If there are no dependencies between the target tasks, this replicates
current behaviour because (2) automatically satisfies all the prerequisites.

If there are dependencies between the targeted tasks, this makes rerunning
any sub-graph in Cylc 8 very easy.

-----

## Comparison of Cylc 7 and 8 (current) for sub-graph rerun

### Cylc 7 sub-graph rerun

#### C7_a group members and their upstream parents remain in the task pool

1. reset all group members to waiting (e.g. by family name)

#### C7_b (if any group members or parents have left the pool)

1. insert all group members as waiting
2. insert all off-flow parents as waiting
3. reset all the parents to succeeded

Dependency matching will then cause the sub-graph to run correctly. 

### Cylc 8 sub-graph rerun

#### C8_a general, new flow

1. trigger the initial task(s) of the sub-graph (with `--flow=new`)
2. satisfy any off-flow prerequisites (with `--flow=n`)

In this case, you might need to deal with flow-on downstream of the sub-graph if it
doesn't dead-end or merge with an existing (typically failed incomplete) task.

#### C8_b general, re-flow

1. `cylc remove` all group members to erase the previous flow
2. trigger the initial task(s) of the sub-graph
3. set any off-flow prerequisites

In this case, flow-on downstream of the sub-graph is not a problem. Any
downstream tasks that already ran won't rerun in the same flow, and otherwise
the flow should continue as normal if the sub-graph re-run was successful.

### Comments

C7_a is the simplest. It relies on the group and its parents still being in the task
pool, but that is often the case when dealing with same-cycle problems. If so,
users can get away without understanding the task pool. 

C7_b is the least intuitive of all - it requires some understanding of the task pool,
task insertion, and the graph (e.g. to identify off-flow parent tasks).

C8_a is conceptually clean, and general, but it does require an understanding of the
graph structure to identify initial tasks and off-flow prerequisites, and it might
result in unwanted downstream activity.

C8_b is like C8_a, but trades off unwanted downstream activity for `cylc remove`.

## Cylc 8 group trigger proposal

This makes re-running any Cylc 8 sub-graph as easy as the C7_a case.

#### C8_c: Cylc 8 (general, proposal):
1. trigger the group (e.g. by family name, or as a list of task IDs)

### Implementation

For each group member, the trigger method should:
   - remove the previous flow (if not `--flow=new`)
   - set all off-flow (i.e., out-of-group) prerequisites
   - unset all in-flow (i.e., in-group) prerequisites (existing n=0 only)
   - "set all" any parentless in-group tasks (i.e., promote them to the task pool)

To achieve this, on matching inconming arguments to n=0 and future tasks:
   - record all task IDs in the group
   - examine the prerequisites of each task
     - for n=0 task proxies, just query their prerequisite objects
     - for future tasks, use tasdef methods to see what their prequisites would be
   - unset any group-internal prerequisites in n=0 task proxies
   - set any off-flow prerequisites
     - this will spawn the associated tasks into n=0

### CLI

Even though the implementation just "sets prerequisites" the trigger
command is the more appropriate home for this functionality.
- `cylc set` just sets prerequisites or outputs on individual tasks
- the intention here is always to trigger (the group) right away,
  which is not the case with `cylc set`

### Outputs?

In principle we could also remove or set out-of-group outputs to prevent
downstream flow-on.

However, we would normally want the flow to continue as normal if the rerun is
successful, and if `cylc remove` is used (for re-flow as opposed to new-flow) 
then it won't re-run downstream tasks that already succeeded in the same flow. 

So managing off-flow outputs is not necessary.
