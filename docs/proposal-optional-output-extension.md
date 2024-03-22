# PROPOSAL: Optional Output Extension

A proposal which extends the
[optional outputs model](https://cylc.github.io/cylc-admin/proposal-new-output-syntax.html)
to solve existing limitations and support the expired state.

Supersedes:
* [cylc-admin/previous optional output extension proposal](https://github.com/cylc/cylc-admin/blob/4c5bf7f50125472a1c022ad9108ac7fd2a7ec8bf/docs/proposal-optional-output-extension.md)
* [cylc-flow#5423](https://github.com/cylc/cylc-flow/issues/5423)


## Background: Task Expiry

There are currently three problems with expiry (as of cylc-flow 8.1.4):

1. **Expire triggers are a [bit broken](https://github.com/cylc/cylc-flow/issues/5364)**

   This is a bug which can be fixed without fundamental change to the expiry
   implementation so can be ignored for the purpose of this proposal.

2. **Expiry triggers don't fit in with [optional outputs](https://github.com/cylc/cylc-flow/issues/5361)**

   Under the existing logic, task expiry cannot cause a stall (due to a task finishing with incomplete outputs).
   This makes expiry unsafe. This proposal addresses this.

3. **Expiry events can be detected late, or even missed due to the task pool implementation.**

   This is due to the coupling of expiry logic into the task pool.

   This proposal does not address the late event problem and recommends this
   should be a documented limitation.

## Proposal

1. Expiry should remain implemented as a task output.

2. Expiry is considered separately from task execution in the same way that submission is already
   considered separately. Hence `:expire?` does *not* imply `:succeed?` and `:fail?`.

   In this proposal users have to understand that `a => x` only means that `a` must succeed if it is
   executed and that might not happen if expiry or submission are optional (or if the pre-requisites
   for `a` are not met).

3. A task's outputs are "complete" if the completion condition is satisfied.

   The completion condition can be defined by the user, otherwise the
   expression is automatically generated according to the rules:

   1. All required outputs (referenced in the graph) must be satisfied.
   2. If success is optional, then the outputs are complete if the task fails.
   3. If submission is optional, then the outputs are complete if the task submit-fails.
   4. If expiry is optional, then the outputs are complete if it the task expires.

   I.E:

   ```python
   is_complete = (
     all(output for output in outputs if not is_optional(output))
     or (failed if is_optional(succeeded))
     or (submit_failed if is_optional(submitted))
     or (expired if is_optional(expired))
   )
   ```

   Similarly, a task's outputs are "incomplete" or "not complete" if the completion
   condition is not satisfied.

3.5 A task is "finished" if it has final task status

   Final task statuses are: expired, submit-failed, failed, and succeeded.

   A finished task will not run again, complete other outputs, or change state, without manual
   intervention (in the same flow, that is).

   A task can finish with complete or incomplete outputs.


3.6 Task pool removal depends on final status *and* output completion.

   A finished task with complete outputs can be removed as "done".

   A finished task with incomplete outputs is retained as "not done",
   pending intervention to complete its outputs or to remove it manually.

  N.B. giving a short name for "finished with complete outputs" is not strictly
  necessary, but it is convenient. I propose "done": a that finishes with completed
  outputs has done its job and can be forgotten by the scheduler. "Not done" is the
  logical opposite of that.


4. `:expire` cannot be required.

   Similarly `:submit-fail` cannot be required.

   * We currently allow `:submit-fail` to be required but this doesn't really make sense.

5. The completion condition should be a configurable expression. The completion condition can be
   set to tolerate cases not handled in the graph or express more complex completion criterion.

   The expression should be pure Python, evaluated in a restricted context which permits only
   `and` & `or` operators, but not:

   * The `not` operator, because we cannot logically support negation or
     exclusive `or`'s in Cylc [1].
   * Imports, function calls, etc.

   ```
   # OK
   completion = succeeded or failed
   completion = succeeded and (x or y or z)
   completion = (succeeded and x) or (failed and y)
   completion = (succeeded and (x or y or z)) or failed or expired

   # ERROR
   completion = not failed
   completion = (x and not(y or z)) or (y and not(x or z)) or (z and not(x or y))
   completion = import os
   ```

   Expression to be validated (i.e. parsed) at validate time.

   If the completion expression is user-defined, then it must be logically
   consistent with the graph.

   ```
   completion(a) = succeeded and (x or y or z)

   # OK
   a:x? => x
   a:y? => y
   a:z? => z
   x | y | z => b

   # ERROR
   a? => w  # ":succeeded" must be required
   a:x? => x
   a:y? => y
   a:z? => z
   x | y | z => b

   # ERROR
   a:x => x  # ":x" must be optional
   a:y => y  # ":y" must be optional
   a:z => z  # ":z" must be optional
   x | y | z => b
   ```

   Note that the validator can use the following logic to determine which outputs are optional
   in a given expression:

   ```python
   { # output: is_optional
     output: restricted_eval(condition, {o: o != output for o in outputs})
     for output in outputs
   }
   ```

   The completion expression cannot include `finished` (this isn't an output).

6. Expiry is only optional if it is explicitly referenced in the graph (e.g.
   `a:expired? => x`) or permitted in the completion expression
   (e.g. `succeeded or expired`).

   E.G. in these examples:

   ```
   a => z

   b => z
   b:expired?

   c
   # completion = succeeded or expired
   ```

   If the `expired` output is set on both tasks `a` and `b`:

   * `a` will finish with incomplete outputs - i.e. it is "not done"
   * `b` will finish with complete outputs - i.e. it is "done" (permitted in graph).
   * `c` will finish with complete outputs - i.e. it is "done" (permitted in completion expression).

   If a task is configured to clock-expire, but expiry is not handled in the
   graph or permitted in the completion expression, a warning should be raised
   informing the user that the workflow may stall. The warning should
   recommend that they either handle the expired output in the graph if no
   completion expression is defined, or permit it in the completion expression
   if one is defined.

7. If neither `:succeed` nor `:fail` are specified for a task in the graph then `:succeed` should
   be required.

   So for this example:

   ```
   a:x? => b
   ```

   Task "a" must succeed if it executes.

   Apparently this is already the case but is undocumented?

8. Expiry should be considered for tasks with partially satisfied prerequisites.

   This is to prevent workflows from stalling on tasks with partially satisfied
   prerequisites where expiry is configured.

9. Late delivery of expiry events to be a documented limitation.

   Cases where expiry is not a parentless task e.g:

   ```
   a[-P1D]? | a[-P1D]:expire? => a?

   a:succeed? => x
   a:expire? => y  # this cannot happen until a[-P1D] has succeeded/expired
   ```

   Could receive expire events late due to the previous cycle running behind
   schedule.

   To handle this, these cases will need to be adapted so that a parentless task
   is a dependency of the task to be expired e.g:

   ```diff
   + dummy => a
     a[-P1D]? | a[-P1D]:expire? => a?

     a:succeed? => x
     a:expire? => y  # this cannot happen until a[-P1D] has succeeded/expired
   ```

   In this example, the task `dummy` will run and succeed as soon as it enters
   the runahead window which will cause `a` to be spawned with partially
   satisfied prerequisites at which point it will be considered for expiry
   events according to (8).

10. Manually triggering a task should not cause an expiry event.

    If a task has expiry configured and a user force-triggers it, it should not
    expire in response to this trigger.

    This is special to the trigger command (which infers run), manually
    setting prerequisites on a task should not disable expiry.


## Examples

### The "xyz" problem

```
a:x? => x
a:y? => y
a:z? => z
x | y | z => b
```

By default we have:

```
completion(a) = succeeded
```

This gives a reasonable level of protection. If task "a" is meant to output one of x, y or z
then it shouldn't succeed unless it does so (and there is nothing Cylc can do if tasks succeed
when they should fail). If you want Cylc to check this happens then you can add:

```
[a]
  completion = succeeded and (x or y or z)
```

### Expire Branch

```
clock-expire = a

a => x
a:expired? => y
x | y => z
```

Succeeded is required for task "a" but only if it executes.

### Expire Halt

```
clock-expire = a

a => x
a:expired?
```

`a:expired?` has to be included in the graph to avoid stall.
This makes it clear that the halt is intentional.

### Output Groups

A contrived example demonstrating how more complex completion conditions can be implemented.

In this example, we don't consider the task's outputs complete unless one pair of outputs is generated:

```
a:w? => w => w1 => ...
a:x? => x => x1 => ...
a:y? => y => y1 => ...
a:z? => z => z1 => ...
...
# with some conditional join later on along the lines of:
(w9 & x9) | (y9 & z9) => end

[a]
  completion = succeeded and ((w and x) or (y and z))
```

### Flaky Pipe

```
a? => b? => c?
```

This continues to work as at present with no need to define a completion condition.

### Flaky Submission Pipe

```
a:submit? => b:submit?
```

This workflow will shutdown if:

* `a` fails to submit
* `a` succeeds and `b` fails to submit
* `a` and `b` succeed

If either `a` or `b` fail then they finish with incomplete outputs and the workflow will stall.

### Recovery Task

```
a? | recover => b
a:fail? => recover
```

Again, no change in this case.

### Error Outputs 1

A safer version of the recovery task example where a specific error case is caught and turned into
a custom output in order to avoid random errors (e.g. syntax errors) from triggering recovery logic:

```
a? | recover => b
a:error_x? => recover

[a]
  completion = succeeded or error_x
  script = """
    this
    that
    if [[ $? == 42 ]]; then
      cylc message -- x
    fi
    tother
  """
  [outputs]
    error_x = x
```

### Error Outputs 2

An example where multiple failure cases have been assigned different outputs, not all of which
are used in the graph.

```
# break the chain if a fails with an expected error but stall otherwise
a? => b

[a]
  script = """
    ...
  """
  completion = succeeded or (failed and (error_x or error_y or error_z))
  [outputs]
    error_x = ...
    error_y = ...
    error_z = ...
```

## Preference To The Expire Proposal

This proposal replaces the
[task expire proposal](https://github.com/cylc/cylc-admin/blob/fc564ddb26c1476dd051b059ca9a31829a20bf30/docs/proposal-task-expire.md)
which recommended converting expiry from an output to a task attribute in order
to work around the issue with `:expired?` inferring `:succeeded?`.

This proposal has technical merit as expiry might be better thought of as a
sort of inverse prerequisite, however, there are drawbacks with this approach.

Arguments in favor of keeping expiry as a task state:

1. The "expired" task state records the expiry event/intervention in a way which is intuitive to
   users and visible with existing tools.
   Review, GUI and Tui already work with task states (incl expired) and provide tools for
   filtering tasks by state.
2. Expired tasks are not instantly removed so remain in the n-window and can be manually triggered
   if necessary.
3. The "expired" state can be used in inter-workflow triggers which may be necessary where downstream
   workflows depend on a real-time workflow with catch-up logic.
4. `:expire` triggers are used in the graph making them work the same as task outputs.
   Implementing them differently to task outputs but representing them the same is illogical.
   Xtriggers might be considered a workaround, however, they
   cannot be used with optional outputs or conditional triggers so would not be able to replace
   `:expire` triggers for all cases meaning that task expiry will continue to be represented as
   a task state in the graph.
5. Other than implementation "correctness", there doesn't seem to be any practical advantage to
   users in changing the implementation of expiry from task state to attribute from a
   behaviour perspective.
6. At the moment expiry fits into the task state model in a way which is fairly intuitive. Changing
   this means that expiry will be bespoke edge-case which user's will have to learn. The ideal
   solution would be to unify expiry with the existing model rather than creating a new one.


## Footnotes

[1] If a task is run and yields the output `x`, but is then manually re-triggered and yields the
    output `y`, then both the `x` and `y` outputs are completed on the task. Therefore Cylc cannot
    enforce mutually exclusive outputs e.g. `x xor y` so we should not attempt to support this
    in the `completion` expression.
