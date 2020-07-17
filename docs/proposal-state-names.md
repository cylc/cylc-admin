# Proposal (Sept-Oct 2019)

## Rationalise Task State, Output, and Trigger Names

Cylc has gradually accrued new task states (and associated triggers and outputs
over the years) and they need tidying up. By thinking this through again from
the top we can actually halve the number of abstract task states from 8 to 4.

### Task/Job States

(i.e. task states that have a corresponding job state)

| current name      | NEW name            | meaning                       |
| --                |----                 | ---                           |
| `submitted`       | `submit-succeeded`  | job submitted to batch system |
| `submit-failed`   | `submit-failed`     | job submitted but did not execute |
| `running`         | `running`           | job started, but did not finish yet |
| `succeeded`       | `run-succeeded`     | job finished successfully    |
| `failed`          | `run-failed`        | job finished unsuccessfully   |

Notes:
- Prefixes `run-` and `submit-` added to disambiguate and make consistent
- More verbose, but:
  - that does not matter for documentation or display in the GUI
  - for CLI commands that target tasks by state, we will allow the
    old/abbreviated names for back-compat anyway

### Abstract Task States

(i.e. task states with no corresponding job state)

#### Current Abstract Task States

These are problematic in several ways, detailed below the table.  

| current name       | meaning                                                   |
|--                  |---                                                        |
| `runahead`         | task held in the non-active runahead pool while too far ahead |
| `waiting`          | waiting for prerequisites to be met                       |
| `held`             | will not submit job even if prerequisites met                   |
| `queued`           | held back in an internal queue                            |
| `ready`            | preparing job submission, including subprocess pool queue |
| `expired`          | waited too long, do not bother to submit                  |
| `submit-retrying`  | job submission failed, will try again later               |
| `retrying`         | job execution failed, will try again later                | 

Problems:
- `runahead` is really an artifact of the task-pool implementation and should not
  be exposed to users; it will not exist in the spawn-on-demand era
- `queued` (internal queue) gets confused with `submitted` (to a batch queue)
- `ready` is not descriptive of what is actually going on (see "meaning" in the
  table)
- `held` is problematic as a state, e.g. what state to release to after forced
  state manipulation during the hold?
- `retrying` is ambiguous (should be `run-retry`) but in fact both retrying
  states are problematic: a run-retry necessarily involves a submit-retry; the
  word `retrying` wrongly suggests already running again. In fact "retrying"
  is the *reason* why a previously-failed or previously submit-failed task has
  been returned to the `waiting` state, with a new clock-trigger

#### New Abstract Task States

(These make perfect sense, and are easy to explain and understand!)

| NEW name      | meaning                                             |
|----           | ---                                                 |
| `waiting`     | (formerly `waiting`, `queued`, and `runahead`) waiting on **all** prerequisites |
| `preparing`   | (formerly `ready`) all prerequisites met; preparing for job submission |
| `expired`     | (unchanged) waited too long, do not bother to submit    |

Notes:
- removed the `runahead` and `queued` status, but absorbing them into into
  `waiting`, which now means waiting on all prerequisites: task dependencies,
  clock and xtriggers, internal queues, runahead release, etc.
- removed the `held` state
  - "held" is now just an attribute of a task, whatever its state
  - it only affects `waiting` tasks, but we should still attach a "held" badge
    to a task in any state to indicate that it will not submit its job even
    if it achieves or is put in the `waiting` state
- removed `submit-retry` and `(run-)retry` (see above)
  - the fact that a task is going to retry will now be obvious from a) the task
    state; and b) the job icon of the previous try
- the `preparing` state includes:
  - Writing and verifying the job file.
  - Selecting a remote job host.
  - Initialising the remote job host.
  - (Waiting for user to upload the job file in a trigger-edit?)

### Outputs

(These are mostly an internal concept; just make consistent with task states)

| current           | new             |
|---                |----             |
| `expired`         | `expired`       |
| `submitted`       | `submit-succeeded`     |
| `submit-failed`   | `submit-failed` |
| `started`         | `run-started`   |
| `succeeded`       | `run-succeeded` |
| `failed`          | `run-failed`    |
| (custom)          | (custom)        |

Notes:
- added `run-` prefixes for consistency and to avoid any ambiguity

### Graph Trigger Qualifiers

| old                  | new              |
|----                  |----              |
| `:expire[d]`         | `:expired`       |
| `:submit[ted]`       | `:submit-succeeded`     |
| `:submit-fail[ed]`   | `:submit-failed` |
| `:start[ed]`         | `:started`       |
| `:succeed[ed]`       | `:run-succeeded` (default) |
| `:fail[ed]`          | `:run-failed`  |
| :(custom)            | :(custom)      |

Notes:
- changes to disambiguate, and for consistency with state names
- however, `run-succeeded` is the default and rarely be used explicitly
- need to support the old/abbreviated names for back-compat anyway

### Implementation Plan

(Roughly; can be refined on appropriate Issue or PR pages)

Code
- rename states throughout the code base, including functional tests, using
  constants defined in one place
- deal with absorption of `queued` into `waiting`
- (TODO: do we still need the "retry(ing?)" event - probably OK as it triggers
  when a retrying task submits, and distinguishes original try from retries).  
- create back-compat for all changed items, and write associated tests 
- write code to support interrogation of a waiting task's queue status

User Guide
- rename states throughout, document their meanings and relation to cylc-7 states

UI
- need a new `preparing` task icon
- new new "held", "queued", and "runahead" badges/icons 
  - graph view: represent as dependencies (like xtriggers in cylc 7 GUI)
    (diagram: https://github.com/cylc/cylc-admin/pull/47#issuecomment-531849902)
  - other views: represent as badges on top of the task state icon
  - (TODO: can runahead be treated as a built-in queue?)
- (allow interrogation of a waiting task's queue status)
