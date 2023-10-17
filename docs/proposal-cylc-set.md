# PROPOSAL: A New Command for Manual Task Forcing

## Background

Users need to be able to intervene and affect the progression of workflows.

Currently we only have only `cylc trigger` and `cylc set-ouputs` which don't
cover all requirements. (And note the latter command does not actually set task
outputs: it just spawns downstream children with the corresponding
prerequisites set, *as if* the outputs had been completed).


## Set & Task Status

When setting outputs we have the option to change the task status or not.
Do we want (e.g.):
- A force-succeeded task that actually failed but is indistinguishable from a
  naturally succeeded one without looking at run history
- Or a failed task which the workflow nevertheless
  continued on from because the user said to carry on *as if* it had succeeded
  but which is indistinguishable from a failed task on which no intervention
  has been performed.

This proposal considers the former option for reasons of visibility,
provenance and consistency with the task-job model.


## Requirements

1. Force set specified **prerequisites** of a target task. 
   - This contributes to the task's readiness to run.
   - It is not equivalent to setting the parent output, unless the task is an only child.
   - This will spawn the task if not already in the pool.

2. Force set specified **outputs** of a target task.
   - This sets the corresponding prerequisites of child tasks.
   - And it contributes to the completion of *incomplete tasks*.
   - It will not spawn a task with the specified outputs completed - that would create an incomplete task.
   - Set *implied outputs* as well (see command help below).
   - If the `succeeded` or `failed` outputs are set, disable automatic retries.
   - DEFAULT: set all required outputs.


## CLI

```
$ cylc set [OPTIONS] [TASK(S)]

Command for intervening in task prerequisite, output, and completion status.

Setting a prerequisite contributes to the task's readiness to run.

Setting an output contributes to the task's completion, and sets the
corresponding prerequisites of child tasks.

Setting an output also results in any implied outputs being set:
 - started implies submitted
 - succeeded and failed imply started
 - custom outputs and expired do not imply any other outputs.

By default this sets all required outputs for the given task(s).

[OPTIONS]

   --flow=INT: Flow(s) to attribute spawned tasks. Default: all active flows.
      If a task already spawned, use current flows.

   --meta=DESCRIPTION: description of triggered flow (with --flow=new).

   --wait: Wait for merge with current active flows before flowing on.

   --pre=PRE: Set a prerequisite to satisfied e.g. `foo:succeeded`
              (multiple use allowed, may be comma separated).

   --pre=all: Set all prerequisites to satisfied. Equivalent to adding the task
              into the workflow in the waiting state.

   --out=OUT: Set an output e.g. `succeeded` (and implied outputs) to completed
              (multiple use allowed, may be comma separated).

   --out=required Or if no outputs or prerequisites are specifed: set all required
             outputs.
```

This would replace `cylc set-outputs`.

### Validation

Setting a prerequisite or output which does not apply to the task should result
in a warning to ensure typos and simple mistakes are communicated back to the
user.

To support this work, Cylc custom outputs will need to be validated
to prohibit the `all` and `required` keywords as well as the comma character.
The `_cylc` prefix should also be reserved for any future internal uses as
already done for some other interfaces (e.g. xtriggers).

Examples:
```ini
[runtime]
  [[<namespace>]]
     [[[outputs]]]
       # OK
       foo = <msg>
       foo_bar = <msg>
       foo-bar = <msg>
       # ERROR
       foo,bar = <msg>  # commas aren't valid in the graph
       foo bar = <msg>  # spaces aren't valid in the graph
       all = <msg>      # all is a keyword
       required = <msg> # required is a keyword
       _cylc = <msg>    # _cylc is a special prefix
```


### Extension

When setting prereqs/outputs it would be useful to provide visual confirmation
of the result.

We could display the `cylc show` output (which can be generated Scheduler side
without the requirement for a second API call).

This can also help users to discover valid prereqs / outputs in the event of
warnings.


## QUESTIONS

### Un-setting prerequisites?

If a task has already run, its prerequisites have already been "used". There's
no point in unsatisfying them.

If a task has not been spawned yet, none of its prerequisites are satisfied
anyway.

Is there a case for unsatisfying the prerequisites of a waiting
partially satisfied task that has not run yet?

Use cases aren't obvious, suggest omitting this functionality until a
compelling use case is discovered for the sake of interface/implementation
simplicity. See also `cylc remove` which reduces the pressure on this.

See: [Remove Proposal](proposal-remove.html)


### Un-completing outputs?

There's no point in unsetting a completed output because any downstream action
will have already been spawned (on demand).

Use cases aren't obvious, suggest omitting this functionality until a
compelling use case is discovered for the sake of interface/implementation
simplicity. See also `cylc remove` which reduces the pressure on this.

See: [Remove Proposal](proposal-remove.html)


## Example Use Cases

> Note: In these CLI examples the workflow ID and cycle point have been omitted
> for brevity, so `cylc set //a` is shorthand for
> `cylc set <workflow-id>//<cycle>/a`.

### 1. Carry on as if a failed task had succeeded

`foo => post<m>`

E.g. `foo` failed and is incomplete, but I fixed some files externally
so the workflow can carry on as if it had succeeded.

```
$ cylc set //foo --out=succeeded
# Or simply:
$ cylc set //foo
```

Set the task's `succeeded` output, to:
- spawn children of that output, with the corresponding prerequisite set
- complete the task, so scheduler can forget it

The database will record:
- foo: state=succeeded, outputs={submitted,started,failed,succeeded}


### 2. Set off-flow prerequisites, to prep for a new flow

If a flow is not entirely self-contained, you can prime the appropriate tasks
before triggering the flow, by either (whichever is more convenient):
- setting their off-flow prerequisites, or
- setting the parent outputs of the off-flow prerequisites

E.G. if we wanted to start a new flow at task `a`:

```
# the tasks we want the flow to run
a => b => c
# the off-flow prerequisites
a_cold => a
b_cold => b
c_cold => c
```

Then we could either set the prerequisites of the downstream tasks:

```
# set off-flow prerequisites and trigger a new flow on a
$ cylc set //a //b //c --flow=new --pre=a_cold:succeeded,b_cold:succeeded,c_cold:succeeded
WARNING //a has no prerequisite b_cold:succeeded
WARNING //a has no prerequisite c_cold:succeeded
WARNING //b has no prerequisite a_cold:succeeded
WARNING //b has no prerequisite c_cold:succeeded
WARNING //c has no prerequisite a_cold:succeeded
WARNING //c has no prerequisite b_cold:succeeded
```

Or the outputs of the upstream tasks:

```
# set off-flow outputs and trigger a new flow on a
$ cylc set //a_cold //b_cold //c_cold --flow=new [--out=succeeded]
```

Note, in both cases, using `cylc set` will spawn task `a` with all
prerequisites satisfied so a subsequent trigger of task a is not required.

(Note this is more than a convenient alternative to `cylc trigger` because the
flow needs to progress according to the graph edges, and we don't want to
trigger the off-low parent tasks just to provide the off-flow prerequisites).


### 3. Skip to next cycle after externally-caused failures

> Not necessarily a `cylc set` use case, but included for completeness.

Many tasks failed due to (say) external disk error. I want to start over at the
next cycle point rather than re-run the failed tasks.

Either skip all tasks in the cycle and complete any incomplete tasks:

See: [Skip Mode Proposal](proposal-skip-mode.html)

```
# configure all tasks in the cycle to skip
$ cylc broacast -n '*' -p <cycle> -s 'run mode = skip'
# handle any incomplete tasks by setting their required outputs
$ cylc set //* --out=succeeded
```

Or, with [cylc-flow#5416](https://github.com/cylc/cylc-flow/issues/5416)
trigger all start tasks in the next flow:

```
# trigger all start tasks in the next cycle
$ cylc trigger --start-cycle=<new-cycle>  # syntax not yet decided
# remove all n=0 tasks in the cycle
$ cylc remove //<old-cycle>/*
```


### 4. Set jobs to failed when a job platform is known to be down

This case describes the scenario where a job has successfully submitted (or
even started) on a remote platform which subsequently becomes un-contactable
leaving us with a job "stuck" in the submitted/running state. We cannot poll
or kill these tasks, in order to allow the task to be retried, we must orphan
the broken submission.

```
$ cylc set <id> --out=failed
```

The database will record:
- foo: state=failed, outputs={submitted,started,failed}


### 5. Set switch tasks at an optional branch point, to direct the future flow

Graph branching can occur automatically via optional graph outputs.
Users may need to manually control this logic by setting outputs manually E.G:

```
foo:a? => a => ...
foo:b? => b => ...
```

```
$ cylc set //foo --out=a,succeeded --wait
```

The database will record:
- foo: state=waiting, outputs={a}

Notes:
* A completion output (`succeeded` in this case) must be specified if you
  don't want the task to be re-run by an approaching flow.
  The task may have a completion condition like
  `completion = succeeded or (failed and a)` so we don't attempt to guess the
  completion status for the user. They may also have legitimate use cases for
  not specifying a completion output.
* The `--wait` is necessary if you don't want the workflow to flow on from this
  task until the flow washes over it as in this case.


### 6. Expire a task

Clock-expires allow tasks to be automatically expired, however, users may wish
to action this logic manually e.g. for testing.

This is no different to setting any other output, task completion is evaluated
as normal.

```
$ cylc set //<task> --out=expired
```

The database will record:
- <task>: state=expired, outputs={expired}


### 7. Spawning parentless tasks

Parentless tasks cannot currently be spawned because there are no upstream
outputs which can spawn them.

See [cylc-flow#5572](https://github.com/cylc/cylc-flow/issues/5572).

If `--pre=all` is used but there are no prerequisites to satisfy, then
`cylc set` should still spawn the task as it would have if there had been
prerequisites to set.

E.G:

```
@clock => foo => bar
```

```
# spawn task foo but don't satisfy its xtriggers
$ cylc set --pre=all //foo
```
