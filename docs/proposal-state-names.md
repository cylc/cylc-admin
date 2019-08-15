# Proposal

## Rationalise Task State, Output, and Trigger Names

REF: https://github.com/cylc/cylc-ui/pulls#issuecomment-521121132

Cylc has gradually accrued new task states and associated triggers and outputs
over the years, and the resulting names are variously ambiguous, confusing, or
grammatically inconsistent.

### Task and Job States

| Job?  | old name      | new name :question: | meaning | comment |
|--     | --              |----               | ---     | ---     | 
|       | runahead        |  -                | in the runahead pool (too far ahead) | |
|       | waiting         | -                 | waiting for prerequisites to be met |
|       | queued          | limited           | held back by a size-limited internal queue | **queued** gets confused with "submitted" (to a batch queue)! |
|       | ready           | submitting        | queued to the subprocess pool for job submission | | 
|       | expired         | -                 | waiting for too long, will never be submitted | | 
| :heavy_check_mark:  | submitted       | -                 | job submitted to batch system for execution | | 
| :heavy_check_mark:  | submit-failed   | submission-failed | job submission failed, did not execute | | 
|     | submit-retrying | submission-retry  | job submission failed, will try again later | retry**ing** suggests already re-submitted (TODO: REMOVE) |
| :heavy_check_mark:  | running         | executing         | job execution commenced | | 
| :heavy_check_mark:  | succeeded       | execution-succeeded | job execution completed successfully | | 
| :heavy_check_mark:  | failed          | execution-failed  | job execution completed, but failed | | 
|     | retrying        | execution-retry   | job execution failed, will try again later | retry**ing** suggests already re-executing (TODO: REMOVE) | 

Note:
- ("held" no longer a task state - demoted from state to attribute in Cylc 8)
- "runahead" is really implementation that should not be exposed, and will not
  exist in the spawn-on-demand era, but so long as we still have a task pool
  users occasionally need to understand it (e.g. if they `cylc insert` a task
  beyond "max active cycle points").
- The "retry" states are to be removed as for "held"

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
