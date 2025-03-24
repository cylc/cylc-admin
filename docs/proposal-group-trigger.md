# PROPOSAL: Task Group Triggering

## Background: re-running a sub-graph in Cylc 7 vs Cylc 8

If all affected tasks and their parents are still in the Cylc 7 task pool it
may be easier to rerun a past sub-graph in Cylc 7 than in current Cylc 8,
because Cylc 8 requires some understanding of the graph to identify initial
tasks to trigger and off-group prerequisites to set to prevent a stall, versus
selecting and resetting pool tasks in the Cylc 7 GUI.

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
> For this proposal we assume that inactive task globbing works
> [TBD cylc-flow 5827](https://github.com/cylc/cylc-flow/issues/5827).
> Replace family IDs with member IDs, to make examples here work without globbing.

- Identify the group of tasks by family name, glob, or list of explicit IDs.
- Examine the prerequisites of all group members (including inactive tasks)
- Satisfy any off-group prerequisites.
  - "initial tasks" with only off-group prerequisites will trigger immediately
  - others (partially satisfied) ensure that the triggered flow won't stall
- Remaining (unsatisfied) prerequisites ensure that the flow respects the graph

### Details

   1. match n=0 and future tasks to command args, and record all the task IDs
   1. if `--flow` is not used, erase (remove) the flow history for *past*
      (not `n=0`) tasks
      - triggering a group of past tasks without starting a new flow or erasing
       flow history is pointless - only the initial tasks will run
      - occasionally we need to not erase flow history in this way (e.g. for
        triggering the same task with `--wait` to complete different outputs):
        use explicit `--flow=all`
      - auto-removing triggered `n=0` tasks may be dangerous, require
        explicit `cylc remove` for that
   1. examine the prerequisites of each task in the group
       - for n=0 tasks, just query their prerequisites
       - for future tasks, use taskdef methods to compute their prerequisites
   1. satisfy any off-group prerequisites
       - this spawns the owner tasks into n=0, avoiding a future stall
   1. spawn any parentless tasks in the group (i.e., `cylc set --pre=all`)

### UI

This should become the default behaviour of the trigger command.

### QUESTIONS

#### Un-satisfy existing in-group preprerequisites in existing n=0 tasks?

No: users might want to pre-set some prerequisites before triggering the group.

#### Prevent flow-on downstream of the group?

No: a triggered group should behave just a single task in this respect.

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
cylc reset FAMILY.1 -s waiting <workflow>
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
cylc trigger --flow=2 <workflow> //1/a
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
cylc trigger //1/FAMILY
```

-----

## Appendix: Comparison of Cylc 7 and 8 (current) sub-graph rerun

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

#### C8_b general,k re-flow

1. `cylc remove` all group members to erase the previous flow
2. trigger the initial task(s) of the sub-graph
3. set any off-group prerequisites

### Comments

C7_a is the simplest. It relies on the group and its parents still being in the task
pool, but that is often the case when dealing with same-cycle problems. If so,
users can get away without understanding the task pool.

C7_b is the least intuitive of all - it requires understanding the task pool,
task insertion, and the graph (e.g. to identify off-group parent tasks).

C8_a is conceptually clean, and general, but it does require an understanding of
the graph structure to identify initial tasks and off-group prerequisites.

C8_b is like C8_a, but trades off unwanted downstream activity for `cylc remove`.
