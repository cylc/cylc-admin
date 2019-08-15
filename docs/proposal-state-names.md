# Proposal

## Rationalise Task State, Output, and Trigger Names

REF: https://github.com/cylc/cylc-ui/pulls#issuecomment-521121132

Cylc has gradually accrued new task states and associated triggers and outputs
over the years, and the resulting names are variously ambiguous, confusing, or
grammatically inconsistent.

### Task (T) and Job (J) States

| old             | new :question:    | meaning | comment |
|----             |----               | ---     | ---     | 
| T   waiting         | -                 | waiting for prerequisites to be met |
| T   queued          | limited           | held back by a size-limited internal queue | **queued** gets confused with "submitted" (to a batch queue)! |
| T   ready           | submitting        | queued to the subprocess pool for job submission | | 
| T   expired         | -                 | waiting for too long, will never be submitted | | 
| T/J submitted       | -                 | job submitted to batch system for execution | | 
| T/J submit-failed   | submission-failed | job submission failed, did not execute | | 
| T/J submit-retrying | submission-retry  | job submission failed, will try again later | retry**ing** suggests already re-submitted |
| T/J running         | executing         | job execution commenced | | 
| T/J succeeded       | execution-succeeded | job execution completed successfully | | 
| T/J failed          | execution-failed  | job execution completed, but failed | | 
| T   retrying        | execution-retry   | job execution failed, will try again later | retry**ing** suggests already re-executing | 

Note:
- ("held" removed - demoted from state to attribute in Cylc 8)
- ("runahead" - i.e. beyond "max active cycle points" - removed; it is a task pool
  implmentation detail that should not be exposed to users, and it will not
  exist in the spawn-on-demand era)

### Outputs

Should be consistent with state names.

| old             | new :question:      | meaning       | comment |
|----             |----                 | ---           | ---     | 
| expired         | -                   | (see states)  |         |
| submitted       | -                   | (see states)  |         |
| submit-failed   | submission-failed   | (see states)  |         |
| started         | executing           | (see states)  | or execution-started? |
| succeeded       | execution-succeeded | (see states)  |          |
| failed          | execution-failed    | (see states)  |          |

### Graph Trigger Qualifiers

Should be consistent with state names.

However, abbreviating long names is desirable in graph configuration. (The
current aliasing - optional "ed" suffix - is not necessary though).

| old                | new :question:       | abbrev :question:  | meaning      | comment  |
|----                |----                  |----                | ---          | ---      | 
| :expire[d]         | :expired             | :expired           | (see states) |          |
| :submit[ted]       | :submitted           | :submitted         | (see states) |          | 
| :submit-fail[ed]   | :submission-failed   | :sub-failed        | (see states) |          | 
| :start[ed]         | :executing           | :executing         | (see states) | or execution-started?  | 
| :succeed[ed]       | :execution-succeeded | :exe-succeeded     | (see states) | DEFAULT (no qualifier) | 
| :fail[ed]          | :execution-failed    | :exe-failed        | (see states) |          | 
| :(label)           | -                    | -                  | custom message output   |
