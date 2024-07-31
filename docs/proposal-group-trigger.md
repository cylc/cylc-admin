# PROPOSAL: Task Group Triggering

## Background: re-running a sub-graph in Cylc 7 vs Cylc 8

**If** all sub-graph tasks **and** their parents remain in the Cylc 7 task pool
it may be easier to rerun a past sub-graph in Cylc 7 than in (current) Cylc 8.

That's because it arguably requires more graph knowledge to identify initial
task(s) and off-flow prerequisites (Cylc 8) than to identify all member tasks
as a group (by family name or glob) and reset them to waiting (Cylc 7).

## Proposal

Currently, `cylc trigger TASKS` makes all of the target tasks trigger immediately,
which is likely not the desired behaviour if there is any dependence between them
(so, currently, we just avoid doing that).

I propose that we change default manual triggering behaviour to:
 1. respect any internal dependencies between the target tasks, and
 2. automatically satisfy any off-group (i.e., off-flow) prerequisites

If there are no dependencies between the target tasks, this replicates current
behaviour because (2) automatically satisfies all the prerequisites.

If there are dependencies between the targeted tasks, this makes rerunning any
sub-graph in Cylc 8 very easy.

## Implementation

For the selected group of tasks, examine all prerequisites to find any that
point outside of the group. Satsifying these external prerequisites will:
- immediately trigger the "initial tasks" of the group, and
- satisfy any off-flow prerequisites that would cause a stall

### Details

   1. match n=0 and future tasks to command inputs and record all the task IDs
   1. (if not triggering with `--flow=new` then erase the previous flow from
       each task in the group, or perhaps require separate use of
       `cylc remove` for that)
   1. examine the prerequisites of each task in the group
       - for n=0 tasks, just query their prerequisites
       - for future tasks, use taskdef methods to predict their prerequisites
   1. unsatisfy any in-flow prerequisites in existing (n=0) tasks
       - ensures that dependencies get waited on when the tasks run
       - (not needed for future tasks)
   1. satisfy any off-flow prerequisites
       - spawns the owner tasks into n=0, and avoid future stall
   1. `cylc set --pre=all` any parentless in-group tasks
      (i.e. promote them to the task pool)

### CLI

Even though the implementation just sets prerequisites within the target
task group, the trigger command is the appropriate home for this because
it targets multiple tasks at once, and the intention is always to trigger
the (initial tasks of) the group right away - whereas
`cylc set` just sets individual prerequisites or outputs on a single task.

### Outputs?

In principle we could also remove or set out-of-group outputs to prevent
downstream flow-on.

However, we would normally want the flow to continue if the rerun is successful,
and if `cylc remove` is used first (for a past sub-graph) then it won't re-run
downstream tasks beyond the group anyway.

But we could make it option to not flow beyond the bounds of the task group.

### Example

![graph](img/rerun.png)

Task `b` fails and as a result we need to rerun all the yellow tasks.
Off-flow prerequisites and outputs are marked with target icons.

-----

## Appendix: Comparison of Cylc 7 and 8 (current) sub-graph rerun

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

C7_b is the least intuitive of all - it requires understanding the task pool,
task insertion, and the graph (e.g. to identify off-flow parent tasks).

C8_a is conceptually clean, and general, but it does require an understanding of the
graph structure to identify initial tasks and off-flow prerequisites, and it might
result in unwanted downstream activity.

C8_b is like C8_a, but trades off unwanted downstream activity for `cylc remove`.
