# PROPOSAL: Skip Run-Mode For Tasks

A proposal for a configuration to "skip" tasks in order to resolve the issue
of skipping families or cycles of tasks which are not yet present in the pool
or which have non-uniform required outputs by utilising broadcast task
matching.

This is effectively an automated solution to calling `cylc set --output=skip`
which allows tasks to be "selected" according to broadcast rules opening
up a wider range of use cases.

## Background

Sometimes users may wish to "skip" one or more tasks in order to short
circuit a graph.

With Cylc 7 they would do something like:

```bash
# skip a family
cylc reset <wid> '<family-id>.*' -s succeeded
# skip a cycle
cylc reset <wid> '*.<cycle>' -s succeeded
```

However, at Cylc 8 tasks cannot be "selected" by globs in this way as globs
apply only to the pool and Cylc 7 style reset is not permitted.

> **Note:** Resetting to "waiting" use cases are catered for by
  `cylc trigger --flow=new`.


## Proposal

1. A new task run mode called "skip" to sit alongside the existing "live"
   and "simulation" modes. The implementation would follow the same
   code-pathway as simulation mode. I.E. it would fake submission and
   execution, but yield real outputs into the workflow.

2. The "skip" mode would be configured by
   `[runtime][<namespace>][skip]` to separate it from
   `[runtime][<namespace>][simulation]`.
   The valid configurations would be:
   * `outputs` - Define the outputs to be generated when this task runs
     in skip mode. By default, all required outputs will be generated.
   * `disable task event handlers` - Disable the event handlers which would
     normally be called on task lifecycle events. By default event handers
     will be turned off.

   The skip mode will not support simulation mode configurations such
   as `speedup factor` which are not relevant to this functionality.

3. The run mode should be controlled by a new task configuration
   `[runtime][<namespace>]run mode` with the default being `live`.
   As a runtime configuration, this can be defined in the workflow for
   development / testing purposes or set by `cylc broadcast`.

   If the `run mode` is set to `simulation` or `skip` in the workflow
   configuration, then `cylc validate` and `cylc lint` should produce a
   warning (similar to development features in other languages / systems).

   Examples:
   ```bash
   # run plotting tasks in skip mode (i.e. turn it off)
   cylc broadcast -n plotting -p '*' -s 'run mode=skip'

   # run debugging tasks in live mode (i.e. turn them on)
   cylc broadcast -n DEBUG -p '*' -s 'run mode=live'
   ```

4. The `cylc set --out` option should accept the `skip` value which should
   set the outputs defined in  `[runtime][<namespace>][skip]outputs`.
   The `skip` keyword should not be allowed in custom outputs.

5. Tasks with `run mode = skip` will continue to abide by the `is_held`
   flag as normal.

6. Force-triggering a task will not override the `run mode`.

7. [Extension] The configured `run mode` should be made available as a task
   attribute so that the UI's can display skip/simulation mode tasks with
   a task modifier (the held and runahead "badges" are task modifiers).

   The order of precedence for this modifier should be bellow the held and
   runahead states, but above queued.

   Examples:

   ```
   Task(is_held=True, run_mode='skip') => is_held modifier
   Task(is_runahead=True, run_mode='simulation') => runahead modifier
   Task(is_queued=True, run_mode='skip') => skip modifier
   ````

8. [Extension] When tasks are run in skip mode, the prerequisites which
   correspond to the outputs they generate should be marked as
   `satisfied by skip mode` rather than `satisfied naturally` for
   provenance reasons.

   For the purpose of `cylc remove` logic, `satisfied by skip mode` should
   be treated the same as `satisfied naturally`.
