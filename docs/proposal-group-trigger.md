# PROPOSAL: Task Group Triggering

## Background: re-running a sub-graph in Cylc 7 vs Cylc 8

It may be easier to rerun a past sub-graph in Cylc 7 than in current Cylc 8, if
all affected Cylc 7 tasks and their parents are still present in the task pool,
because Cylc 8 requires some understanding of the graph to identify initial
tasks (to trigger) and off-group prerequisites (to set to prevent a stall) versus
selecting and resetting pool tasks  to waiting in Cylc 7.

("Off-group prerequisite" means a dependency on tasks outside of the group).

## Proposal

Currently, `cylc trigger TASKS` makes all of the target tasks trigger immediately,
which is likely not the desired behaviour if there is any dependence between them
(so, currently, we just avoid doing that).

We should change the default manual triggering behaviour for a group of tasks to:

 1. force satisfy any off-group prerequisites, to prevent a stall
 1. respect any in-group dependencies (i.e., leave them to be satisfied by the
 triggered flow)

If there are no dependencies between the target tasks, this replicates current
behaviour because all prerequisites get satisfied as off-group.

If there are any internal dependencies, this makes rerunning any sub-graph in
Cylc 8 (where the tasks can be identified as a group) very easy.

This also solves the problem of how to "trigger a whole cycle" easily in Cylc 8,
because it will automatically identify the initial tasks of the cycle, and satisfy
any inter-cycle (off-group) prerequisites.

## Implementation

> [!NOTE]
> Commands for triggering (etc.) multiple tasks look simpler when targeting a
> family name or glob, vs a long list of individual tasks. For maximum clarity,
> this proposal therefore assumes in a couple of places that inactive task
> globbing works already in Cylc 8.
> Until [that work](https://github.com/cylc/cylc-flow/issues/5827) is done just
> replace family names with member names below to get functional commands.

### Overview

- Identify the group of tasks by family name, glob, or list of explicit IDs.
- Examine the prerequisites of all group members (including inactive tasks)
- Satisfy any off-group prerequisites.
  - group start tasks (with only off-group prerequisites) will trigger immediately
  - others (partially satisfied) ensure that the triggered flow won't stall
- Remaining (unsatisfied) prerequisites ensure that the flow respects the graph

> [!NOTE]
> Manually triggering an individual task causes it to run, even if it already
> ran in the same flow. Similarly, manually triggering a group will cause the
> whole group to run, even if all or part of it already ran in the same flow.
>
> If you want to trigger a sub-graph without re-running members that already
> ran in the same flow, use flow-based methods rather than group trigger
> (i.e., just trigger the initial tasks, and manually set off-flow
> prerequisites to avoid a stall).

### Details

>[!NOTE]
>
>- A "group start task" is parentless OR has only off-group prerequisites
> (these need to trigger first to kick off the flow).
>- Use the same flow numbers, as determined by the trigger command in the
> usual way, throughout the operation (referred to as "the flow" below)
> to run the group in a new flow or to rerun it in the same flow.
>- For active (n=0) tasks, recompute prerequisites from the taskdef in
> case the task proxies were spawned prior to a reload.

1. Provide `cylc trigger` with task IDs, by list or (in future) glob.
2. Match the command line to find all group members, both active (n=0) and
   inactive.
3. Examine all active group members:
   - Identify group start tasks, by absence of in-group prerequisites.
   - Satisfy the prerequisites of `waiting` group start tasks, in the flow.
   - Merge the flow to `preparing`, `submitted`, and `running` group start
   tasks, but otherwise leave them be (they already triggered).
   - (*) Record all the other active tasks, for removal (next).
4. Remove all inactive and type-(*) active group members, in the flow,
  to allow rerun.
   - This will kill other killable active tasks (if any).
   - Do not shut down if removal empties the task pool (actually
     not a problem if the entire operation is done in a single method).
5. Examine all removed group members (both inactive and formerly active):
   - Spawn parentless members in the flow (as per `cylc set --pre=all`).
   - Satisfy all off-group prerequisites (as per `cylc set --pre=...`).
   - (Group start tasks will trigger, others will spawn with off-group
     prerequisites satisfied.)
6. Satisfy any xtriggers in the group (consistent with current trigger
     behaviour).
7. Submit any queued tasks in the group (bypassing the queue, consistent
   with current trigger behaviour).

### UI

This should become the default behaviour of the trigger command.

-----

## Example

![graph](img/rerun.png)

Task `b` fails and as a result we need to rerun all the yellow tasks.
Off-group prerequisites and outputs are marked with target icons.

```cylc
# suite.rc  (Cylc 7 / 8 compatible example)

[scheduling]
  cycling mode = integer
  initial cycle point = 1
  [[dependencies]]
    [[[R1]]]
      graph = """
        start => a
        x => f_m1
        a => f_m1 => g_m1 => b
        a => f_m2 => g_m2 => b
        a => f_m3 => g_m3 => b
        b => end
        g_m3 => y
      """

[runtime]
  [[b]]
    script = [[ "${CYLC_TASK_SUBMIT_NUMBER}" != '1' ]]
  [[FAMILY]]
  [[a, f_m1, f_m2, f_m3, g_m1, g_m2, g_m3, b]]
    inherit = FAMILY
  [[start, x, end, y]]
```

Task `b` fails and as a result we need to rerun all the yellow tasks.
Off-group prerequisites and outputs are marked with target icons.

### Cylc 7 intervention

```bash
cylc reset -s waiting <workflow> FAMILY.1
```

### Cylc 8.4.0 intervention

(Aside: see note on family globbing above, and be aware of
https://github.com/cylc/cylc-flow/issues/6681.)

#### New flow

(a) set all off-flow prerequisites:

```bash
cylc set --flow=new --pre=1/x --pre=1/start <workflow> //1/f_m1 //1/a
```

Or (b) trigger initial tasks and set remaining off-flow prerequisites:

```bash
cylc trigger --flow=2 <workflow>//1/a
cylc set --flow=2 --pre=1/x <workflow>//1/f_m1
```

#### Original flow

Same as for a new flow above, but omit the `--flow` options, after first erasing
the original flow with `cylc remove`, and enclose the lot between pause/resume
to avoid shutdown if `remove` could empty the task pool. E.g. as for (a) above:

```bash
cylc pause <workflow>
cylc remove <workflow>//1/FAMILY  # (assumes inactive task globbing)
cylc set <workflow> //1/a //1/f_m1 --pre=1/start --pre=1/x
cylc play <workflow>
```

### Proposed intervention

```bash
cylc trigger <workflow>//1/FAMILY
```

-----

## Appendix: Comparison of Cylc 7 and 8.4 sub-graph rerun

### Cylc 7 sub-graph rerun

#### C7_a group members and their upstream parents remain in the task pool

1. reset all group members to waiting (e.g. by family name)

#### C7_b (if any group members or parents have left the pool)

1. insert all group members as waiting
2. insert all off-group parents as waiting
3. reset all the parents to succeeded

Dependency matching will then cause the sub-graph to run correctly.

### Cylc 8 sub-graph rerun

#### C8_a general, new flow

1. trigger the initial task(s) of the sub-graph (with `--flow=new`)
2. satisfy any off-group prerequisites (with `--flow=n`, the new flow number)

or,

1. satisfy all initial prerequisites (with --flow=new)

#### C8_b general, same flow

1. pause the workflow, in case `remove` empties the task pool
2. `cylc remove` all group members to erase the previous flow
3. trigger the initial task(s) of the sub-graph
4. set any off-group prerequisites
5. resume the workflow

### Comments

C7_a is the simplest. It relies on the group and its parents still being in the task
pool, but that is often the case when dealing with same-cycle problems. If so,
users can get away without understanding the task pool.

C7_b is the least intuitive of all - it requires understanding the task pool,
task insertion, and the graph (e.g. to identify off-group parent tasks).

C8_a is conceptually clean, and general, but it does require an understanding of
the graph structure to identify initial tasks and off-group prerequisites.

C8_b is like C8_a, but trades off unwanted downstream activity for `cylc remove`.
