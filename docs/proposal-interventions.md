#  Intervention Feature Gaps

At present a few Cylc 7 use cases are not currently possible at Cylc 8 or are
technically possible, but difficult to action, or have little to no feedback
via the user interfaces when actioned.

We have been addressing these limitations one at a time so far. However, since
the remaining use cases/interventions overlap quite a bit it makes more sense
to consider them in one go to ensure we have all bases covered, as changes to
one proposal may mandate changes to another.

Most of the functionality issues which remain are around the `cylc reset`
command which provided a powerful and surprisingly intuitive interface for
performing a much broader range of interventions than it was initially designed
for including:

* Re-running tasks.
* Expiring tasks.
* Testing / development.
* Reverting unwanted changes.
* Skipping sections of a workflow.
* Orphaning jobs.
* Etc.


## Proposals

Here are the new proposals which will be referenced in the use-cases below.

* [1] [Set](proposal-cylc-set.html)
* [2] [Optional Outputs](proposal-optional-output-extension.html)
* [3] [Remove](proposal-remove.html)
* [4] [Skip](proposal-skip-mode.html)


## Use Cases Considered

Here follows a list of use cases which are either not currently possible, or
awkward to action in Cylc 8.

If approved, this list of interventions should padded out with the more
straight-forward cases and turned into a new section of the documentation. The
proposals also open up new use cases which are not covered here but should also
be documented.


### 1. I triggered tasks I didn't mean to, they may have run on. I want to undo this.

**Description:**

* User messed up a glob and ran things they didn't mean to.
* Or triggered a family including things they didn't expect it to.
* Or the graph followed a pathway they didn't want it to.
* Etc.
* They now want to rollback those changes to recover.

**Cylc 7 Intervention:**

* Hold the workflow (suppress downstream consequences until ready).
* Kill any unwanted active tasks.
* Reset unwanted tasks to waiting.
* Resume the workflow.

**Proposed Intervention:**

* [3] Remove any unwanted tasks.
  * `cylc remove <ids>`

This will kill any active tasks, remove any waiting tasks from the pool
and strip the specified flow numbers from any tasks, outputs and corresponding
prerequisites of the selected tasks.

As the new intervention only requires one action it is not necessary to
pause/resume the workflow although that may still be desired.


### 2. I want to Skip a task and allow the workflow to continue.

**Description:**

* A waiting task isn't needed for some reason, continue as if it succeeded.
* Or a completed task has not produced its required outputs.

**Cylc 7 Intervention:**

* Reset task to succeeded.

**Proposed Intervention:**

* [1] Set succeeded task output.
  * `cylc set <id> [--out=all]`


### 3. I want to Skip a cycle of tasks and allow the workflow to continue.

**Description:**

* A more advanced form of the previous example, but where we are skipping
  multiple tasks which are unlikely to all be in the pool so cannot be
  selected with a simple glob.
* Sometimes users want to skip a cycle of tasks and continue as if they had
  run and succeeded.
* Note the user might only want to skip selected families within
  the cycle which suffers from the same globing problem at Cylc 8
  with skipping cycles as family selectors also only match tasks in the pool.

**Cylc 7 Intervention:**

* Reset `*.<cycle>` to succeeded.
* OR Reset `*.<cycle>` to expired and manually trigger downstream tasks as
  required.

**Proposed Intervention:**

* [4] Configure the relevant tasks/families/cycles to skip when they are run.
  The user can configure what outputs are generated when a task skips as
  desired.
  * `cylc broadcast -p <cycle-glob> -n <namespace-glob> -s 'run mode = skip'`


### 4. I need to orphan a "stuck" job submission

**Description:**

* A job submission has got stuck due to a batch system failure or network
  issue.
* I need Cylc to forget about this submission.
* I may want Cylc to re-submit this task to another platform, or I may
  want to allow graph branching to handle the failure.

**Cylc 7 Intervention:**

* Reset task to waiting/succeeded/failed/submit-failed.

**Proposed Intervention:**

* Either, [1] set a completed output on the task.
  * `cylc set <id> --out=failed`
* Or, [3] remove the task.
  * `cylc remove <id>`


### 5. I need to terminate a chain of automatic retries.

**Description:**

* My task has automatic retries configured.
* But there is a systematic problem with my task / platform which means the
  retries cannot succeed.
* I want to terminate the retry sequence to allow graph branching to handle
  the failure OR to avoid waiting compute / confusing operators whilst the
  error is fixed.

**Cylc 7 Intervention:**

* Reset task to waiting/failed/submit-failed

**Proposed Intervention:**

* [1] Set the failed/submit-failed task output.
  This cancels the retry chain as per the cylc-set proposal.
  * `cylc set <id> --out=failed`


### 6. Set outputs on switch tasks ahead of the flow.

**Description:**

* I have a task which controls graph branching, say through the use of optional
  outputs.
* I need to override the automated behaviour and take control of this to
  respond to external issues or for testing purposes.

**Cylc 7 Intervention:**

* Re-write the task's script using `cylc broadcast`.

This use case was not especially prevalent at Cylc 7 due to the limitations
of suicide triggers but will likely become more common at Cylc 8 as switches
currently embedded in task logic are elevated to the workflow level for better
visibility/control/provenance/graph-interaction.

**Proposed intervention:**

* [1] Set the desired outputs on the task and a completion output to prevent
  the task from being rerun by the approaching flow.
  * `cylc set <id> --out=<custom-output>,succeeded --wait`


### 7. Spawn a parentless task.

**Description:**

* I need to spawn a parentless task (which may have xtriggers which I don't
  want to skip).
* E.G. as part of triggering a new flow.

```
@clock => a => b
```

**Cylc 7 Intervention:**

* Insert the task.

**Proposed Intervention:**

* [1] Set all prerequisites of the task to spawn it.
  * `cylc set <id> --pre=all`

The `cylc set` command sets prereqs/outputs and implicitly spawns tasks when
they are not yet present. In this case we want the task to spawn but have no
prerequisites to set.


### 8. I need my graph to branch on an expiry event.

**Description:**

* I have mitigations to perform if the graph falls behind schedule.

**Cylc 7 Intervention:**

* Configure expiry for an appropriate task/family/cycle.
* Use suicide triggers on the `expired` output to action graph changes e.g.
  via suicide triggers.

**Proposed Intervention:**

* Configure expiry for a parentless task (to avoid late event detection).
* [2] Use optional outputs logic to perform graph branching.

This would need to come with the documented advice to use a single
head-of-cycle task to detect expiry, then use graph branching to determine
which tasks are run. This is a change for several Cylc 7 workflows which
configure families or entire cycles of tasks to expire, as this approach will
no longer work at Cylc 8, under this, or the previous proposal. However, having
gone through the use cases, it would appear these cases could all be converted
to use the new approach.


### 9. I want to run a cycle of tasks ahead or behind of the flow.

**Description:**

* I want to re-run a historical cycle.
* I want to run a future cycle.

**Cylc 7 Intervention:**

* Insert the cycle of tasks.
* And or reset to waiting if the selection intersects with the pool.

**Proposed Intervention:**

In situations where users can easily list the start tasks of a cycle, I can
trigger a new flow starting from these tasks. However, this is error prone and
many users do not understand the graph structure of the workflow they are
working on well enough. The list of start tasks may differ depending on the
cycle and could be too long to type out.

* Trigger a new flow at the specified cycle with start tasks automatically
  determined using cold-start logic.
  
This functionality is discussed/tracked in
[cylc-flow#5416](https://github.com/cylc/cylc-flow/issues/5416), however,
is sufficiently orthogonal to be omitted from this proposal pending
future discussion :)

Open questions (to be resolved on #5416):
* What should the syntax be?
* Which commands should support this, `play`, `trigger`, `set`?
* [How] should the stop cycle point be configured in combination?
