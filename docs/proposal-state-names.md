# Proposal

## Rationalise Task State, Output, and Trigger Names

REF: https://github.com/cylc/cylc-ui/pulls#issuecomment-521121132

Cylc has gradually accrued new task states and associated triggers and outputs
over the years, and the resulting names are variously ambiguous, confusing, or
grammatically inconsistent.

### Task (and Job) States

| old             | new :question:    | meaning | comment |
|----             |----               | ---     | ---     | 
| runahead        | -                 | beyond "max active cycle points" | (not needed in spawn-on-demand future) |
| waiting         | -                 | waiting for prerequisites to be met |
| queued          | limited           | held back in a limited internal queue | **queued** gets confused with batch queue! |
| ready           | submitting        | in the process pool for job submission | | 
| expired         | -                 | waiting for too long, will never be submitted | | 
| submitted       | -                 | job submitted to batch system for execution | | 
| submit-failed   | submission-failed | job submission failed, did not execute | | 
| submit-retrying | submission-retry  | job submission failed, will try again later | retry**ing** suggests already re-submitted |
| running         | executing         | job execution commenced | | 
| succeeded       | -                 | job execution completed successfully | | 
| failed          | execution-failed  | job execution completed, but failed | | 
| retrying        | execution-retry   | job execution failed, will try again later | retry**ing** suggests already re-executing | 
| held | - | held back even if ready to submit | (not actually a task state in Cylc 8) |

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

Should be consistent with state names. Current aliasing (optional "ed"
suffix) is really not necessary.

| old                | new :question:       | meaning      | comment  |
|----                |----                  | ---          | ---      | 
| :expire[d]         | :expired             | (see states) |          |
| :submit[ted]       | :submitted           | (see states) |          | 
| :submit-fail[ed]   | :submission-failed   | (see states) |          | 
| :start[ed]         | :executing           | (see states) | or execution-started? | 
| :succeed[ed]       | :execution-succeeded | (see states) |          | 
| :fail[ed]          | :execution-failed    | (see states) |          | 
| :(label)           | -                    | custom message output | |
