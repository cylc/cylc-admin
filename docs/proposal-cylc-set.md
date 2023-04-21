# PROPOSAL: A New Command for Manual Task Forcing

(IN DISCUSSION: NOT ACCEPTED YET)

## Background

Users need to be able to intervene and affect the progression of workflows.

Currently we only have only `cylc trigger` and `cylc set-ouputs` which don't
cover all requirements. (And note the latter command does not actually set task
outputs: it just spawns downstream children with the corresponding
prerequisites set, *as if* the outputs had been completed).


### Task Status Reset is Not Needed

I am proposing that we do NOT support Cylc 7 style task status reset, e.g.
resetting a `failed` task to `succeeded`.

Primarily this is for provenance reasons: the run history will be clearer if a
task's status reflects what its last job did, followed by evidence that the user
told the scheduler to carry on *as if* something else had happened.

NOTE we *could* choose to change the task status as well as carry on *as if*
the forced status had been achieved naturally. However, that's not needed to
make the workflow continue in Cylc 8, so we can choose based on clarity of run
history. Do we want (e.g.):
- A force-succeeded task that actually failed but is indistinguishable from a
  naturally succeeded one without looking at run history
- Or a failed task that really failed, which the workflow nevertheless
  continued on from because the user said to carry on *as if* it had succeeded.

This may be a matter of opinion, but I prefer the latter.


## Requirements

0. Force **trigger** a target task.
  - Make it run now, regardless of prerequisites.

1. Force set specified **prerequisites** of a target task. 
  - This contributes to the task's readiness to run.
  - It is not equivalent to setting the parent output, unless the task is an
    only child.
  - It is not equivialent to triggering, unless the task has only one
    unsatisfied prerequisite.

2. Force set specified **outputs** of a target task.
   - This contributes to the task's completion, and sets the corresponding
    prerequisites of child tasks.
  - Set any *implied outputs* as well (see command help below).
  - If the `succeeded` or `failed` outputs are set, disable automatic retries.

3. Force expire tasks.
  - Expire means "we don't need to run this task anymore".
    - Can be automatic (clock-expire) or manual.
    - Allow waiting tasks to expire without running all.
    - Allow the scheduler to forget incomplete tasks without re-running
      to complete them.
  - Make `cylc remove` obsolete (currently, incomplete tasks have to be
    "removed" if not re-run to completion).
  - This amounts to a Cylc 7 style "state reset", but - see below - `expired`
    should really be demoted to a task attribute

Expiration is a bigger topic in its own right, due to its connection to task
outputs and automatica triggering. See
[Task Expire Proposal](proposal-task-expire.md)


## The New Command

```
$ cylc set [OPTIONS] [TASK(S)]

Command for intervening in task prerequisite, output, and completion status.

Setting a prerequisite contributes to the task's readiness to run.

Setting an output contributes to the task's completion, and sets the
corresponding prerequisites of child tasks.

Setting a future waiting task to expire allows the scheduler to forget it
without running it. WARNING: nothing downstream of the task will run
automatically after this.

Setting a finished-but-incomplete task to expire allows the scheduler to forget
it without completing its required outputs. WARNING: nothing downstream of 
of the incomplete outputs with run automatically after this.

Setting an output also results in any implied outputs being set:
 - started implies submitted
 - succeeded, failed, and custom outputs, imply started
 - succeeded implies all required custom outputs
 - failed does not imply custom outputs

[OPTIONS]

   --flow=INT`: flow(s) to attribute spawned tasks. Default: all active flows.
      If a task already spawned, merge flows.

   --pre=PRE: set a prerequisite to satisfied (multiple use allowed).

   --pre=all: set all prerequisites to satisfied. Equivalent to trigger.

   --out=OUT: set an output (and implied outputs) to completed (multiple use allowed).

   --out=all or [no-options]: set all required outputs.

   `--expire`: allow the scheduler to forget a task without running it, or
      without rerunning it if incomplete.

```


## QUESTIONS

### Un-setting prerequisites?

If a task has already run, its prerequisites have already been "used". There's
no point in unsatisfying them.

Is there a case for unsatisfying the prerequisites of a multi-prerequisite
`n=0` waiting task that has not run yet?


### Un-completing outputs?

Normally there's no point in unsetting a completed output. Any downstream
action will have already been triggered (on demand), except in a flow-wait
scenario.


## Example Use Cases


### 1. Forget an incomplete failed task

Expire it.


### 2. Carry on as if a failed task had succeeded

E.g. `foo` failed and is incomplete, but I fixed some files externally
so the workflow can carry on as if it had succeeded.

Set the task's `succeeded` output, to:
- spawn children of that output, with the corresponding prerequisite set
- complete the task, so scheduler can forget it

The database will record:
- `foo` state `failed`, and with the `failed` output
- then `foo:succeeded` set by manual forcing


### 3. Trigger a new flow downstream of target task 

`foo => post<m>`

If we don't want to run `foo` again, setting `foo:succeed` (for a new flow) is
more convenient than triggering all the downstream children individually.

This will spawn all `post<m>` at once, with prerequisite `foo:succeed` satisfied.


### 4. Set off-flow prerequisites, to prep for a new flow

If a flow is not entirely self-contained, you can prime the appropriate tasks
before triggering the flow, by either (whichever is more convenient):
- setting their off-flow prerequisites, or
- setting the parent outputs of the off-flow prerequisites

(Note this is more than a convenient alternative to `cylc trigger` because the
flow needs to progress according to the graph edges, and we don't want to
trigger the off-low parent tasks just to provide the off-flow prerequisites).


### 5. Skip to next cycle after externally-caused failures

Many tasks failed due to (say) external disk error. I want to start over at the
next cycle point rather than re-run the failed tasks.

- Trigger a new flow from next cycle (using `cylc trigger` or `cylc set`)
- Expire all the incomplete failed tasks so the scheduler can forget them


### 6.?? Set jobs to failed when a job platform is known to be down

I don't think this case is valid. (Unless I've misunderstood the requirement?).

If jobs were submitted already they will have already failed naturally.
Otherwise, you can prevent job submission by holding the tasks, or expiring
them if necessary.


### 7.?? Set switch tasks at an optional branch point, to direct the future flow

I'm not sure this is valid either. Why would we need to do this?

```
foo:x? => bar  # take this branch regardless of x or y at run time?
foo:y? => baz

# or:

foo:fail? => bar  # take this branch regardless of succeed or fail at run time?
foo? => baz
```

Should `foo` still run when the flow reaches it, but only trigger the right
branch regardless of what outputs it completes? That amounts to "the graph is
wrong"!

Or do we want to pretend that `foo` has run already and generated the chosen
output, but don't flow on from there until the flow catches up?

In that case: `cylc broadcast` to change the `script` to simply send the chosen
output and exit.
