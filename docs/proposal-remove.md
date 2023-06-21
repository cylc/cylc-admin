# PROPOSAL: Cylc Remove Extension

The `cylc remove` command currently only removes tasks which are in the pool
but not active. Otherwise it logs a warning and does nothing.

This proposal extends remove-like functionality to cover other cases and
better address the remaining use cases.

> By remove-like, I mean things currently covered by several related terms /
> ideas which all pertain to erasing the past, present or future of a specific
> task instance. The resulting functionality does not *need* to be provided in
> a single command or interface or use the same terminology, however, it is
> helpful to consider all cases together even if they subsequently go their
> separate ways in implementation.


## Background

There are some cases where users want to wipe historical data, e.g:

> I triggered some tasks I did not mean to, I need to undo this.

The Cylc 7 intervention was:

* Hold the workflow.
* Kill unwanted active tasks.
* Reset unwanted tasks to waiting.
* Resume the workflow.

The users intention is to erase tasks from the workflows history in order
to allow it to continue as if they hadn't run. The Cylc 7 reset capability
combined with the task pool model appears to have been a remarkably
intuitive model for users in these situations.

At present (8.1.4) the Cylc 8 intervention is:

* Pause the workflow.
* Kill unwanted active tasks.
* Wait for wanted active tasks to complete.
* Start a new flow by setting outputs on all of the tasks you want the workflow
  to run on from.
* Stop the original flow.
* Play the workflow.

Or another combination of flow, stop, trigger and set-outputs to the same effect.

This is far too challenging and may require a level of knowledge of graph
structure that the operator is not partial to.

There are also potentially some use cases which overlap with interventions
where `--flow=new` might be used, but would be cumbersome due to the
trouble of tracking down outputs/prerequisites and starting/stopping
flows.

This proposal considers alternatives to `--flow=new` in order to reduce the
intervention to:
* "Forget" unwanted tasks.

The need for pausing/resuming, originating from the need to suppress spawning,
alleviated.


## Removing Tasks In Different N-Window Cases

There are four cases to consider for remove-like functionality.

### 1. Historical tasks (n<0).

**Proposed removal behaviour:**

Remove the requested flow(s) from:
* The task (DB);
* its outputs (DB);
* and any corresponding prerequisites this task satisfied on waiting tasks in
  the pool (and DB), removing them if these outputs were the only ones to be
  satisfied for the task.

**Notes:**

* This is the closest we can get to "erasing" the task from the workflow's
  history.
* Anything downstream which has already run will also have to be "forgotten" if
  that is the intention.

**Visibility:**

* In the GUI the user will still see the task, but in a different flow
  (when flows are supported in the GUI).

### 2. Active tasks {preparing,submitted,running} (n=0).

**Proposed removal behaviour:**

Remove the requested flow(s) from:
* The task (DB);
* its outputs (DB);
* and any corresponding prerequisites this task satisfied on waiting tasks in
  the pool (and DB), removing them if these outputs were the only ones to be
  satisfied for the task.

If removing the specified flow(s) would leave the task in no flow
(i.e. `flow=None`), then we also send the kill command to the running job but
don't track its outcome. Then reset the job to failed/submit-failed as
appropriate to orphan the submission and remove the task (i.e. complete it,
bypassing required output checking). The failed/submit-failed output should
not be yielded into the pool.

This avoids the issue of having un-tracked active jobs in the DB (which could
appear in the GUI and confuse users) as well as preventing resource contention
issues if the task is re-run and bypassing any possible flow-merge conditions.

**Visibility:**

In the GUI the user will still see the task in the [submit-]failed state with
a [submit-]failed job, but in a different flow (when flows are supported in the
GUI).

### 3. Waiting tasks which are ready to run (n=0).

**Proposed removal behaviour:**

Remove the task from the pool.

Insert a waiting no-flow task into the task history in the DB.

**Notes:**

- If we just remove a task from the pool, then the GUI will still show it
  because the n>0 window is generated from the workflow definition not the
  task pool! We will also have no record of the removal.
- The only way to impact the task's representation in the GUI is to put
  something there for it to display. The data store will subsequently load this
  in along with its flow numbers.

**Visibility:**

In the GUI the user will still see the task, but in a different flow
(when flows are supported in the GUI).

### 4. Future tasks (n>0).

It is proposed that remove functionality shall not be extended to future tasks.

Instead another proposal will handle this case with the addition of a new task
run mode called "skip" which can be configured via `cylc broadcast`.


## Proposal

1. The `cylc remove` command is to be extended to cover cases 1 and 2 with
   its behaviour modified for case 3.
2. When outputs are removed, any corresponding prerequisites on downstream
   tasks in the pool shall be unset providing they were "naturally satisfied".

   This means prerequisites which have been satisfied by `cylc trigger` or
   `cylc set` will not be unset as the result of `cylc remove` as these
   have been manually satisfied by separate interventions.
3. If removing a task causes all of the prerequisites on a downstream task to be
   unset (i.e. if the downstream task was spawned as a result of outputs from
   this task alone) then the downstream task shall be removed from the pool.
4. This command shall not be extended to cover the selective un-setting of
   specific prerequisites or outputs.
5. When all flows are removed from active tasks, Cylc should first attempt to
   kill the job as it serves no further purpose and may cause resource
   contention issues if left running, however, failure of the kill command
   should be tolerated, though logged.
   The state should be reset to "failed" (for running tasks)
   or "submit-failed" (for submitted tasks) in order to orphan the submission
   if the kill failed, but also prevent the problem of active tasks appearing
   in the workflow history despite no longer being tracked/managed by Cylc and
   to allow the task to be re-run as part of a subsequent flow-front as needed.

   * The task should then be removed from the pool as completed.
   * The failed/submit-failed output should not spawn any downstream tasks.
   * The failed/submit-failed event handlers should not be run.
6. The `cylc remove` command shall accept the `--flow` option:
   - if specified, only the listed flow numbers should be removed.
   - If not specified, and the task being removed is in the pool, then the
     flow numbers from the task proxy should be used.
   - If not specified and the task being removed is not in the pool,
     the standard `all` default should apply.
7. To clarify removal operations as well as flow logic in general, visual
   filtering of flows shall be supported in `cylc gui` and `cylc tui`
   as already planned - https://github.com/cylc/cylc-ui/issues/470.

## CLI

```
cylc remove [OPTIONS] <id>

Erase a task from the workflow's history.

This removes the specified flow numbers from a task and any outputs it
generated. The task will still exist, however, not in the specified flows so
will not influence workflow evolution in those flows.

Removing a task will also unset any prerequsites it satisfied on downstream
waiting tasks.

If all flows are removed from a task, it and its outputs will be left in the
`None` flow. This preserves a record that the task ran, but it will not
influence active flows in any way.

If the task is active (i.e. preparing, submitted or running), then, Cylc
will attempt to kill the task and will change its status to either failed, if
it is running, or submit failed, if it is not.

Waiting tasks which have one or more of their prerequsities satisfied can be
removed.

Waiting tasks which are ahead of the flow (i.e. which do not have any of their
prerequsites satisfied) cannot be removed because they do not exist yet. You
can, however, configure them to "skip" by chaing the "run-mode" e.g:
  $ cylc broadcast <workflow> -p <cycle> -n <task> -s 'run mode = skip'

Or you could remove them from the workflow definition and reinstall + reload.

Examples:
  # remove a task which has already run
  # (any tasks downstream of this task which have already run or are currently
  # running will be left alone The task and its outputs will be left in the
  # None flow)
  $ cylc remove <id>

  # remove a task which is running
  # (Cylc will attempt to kill the task and will set the status to failed or
  # submit-failed. The task and its outputs will be left in the None flow)
  $ cylc remove <id>

  # remove a task from a specified flow
  # (the task may remain in other flows)
  $ cylc remove <id> --flow=1


[OPTIONS]

  --flow=all   The flow(s) to remove this task from.
```
