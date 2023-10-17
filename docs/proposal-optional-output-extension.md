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

   Under the existing logic, task expiry cannot cause incomplete tasks. This makes expiry unsafe.
   This proposal addresses this.

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

3. A task is incomplete if any of the following are true:

   * It finished executing without satisfying the completion condition

   * If job submission failed and the `:submit` output was not optional

   * If the task expired and the `:expire` output was not optional
     * This is a new condition to handle expiry

4. `:expire` cannot be required - it can only be optional or not allowed.
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

   # ERROR
   completion = not failed
   completion = (x and not(y or z)) or (y and not(x or z)) or (z and not(x or y))
   completion = import os
   ```

   Expression to be validated (i.e. parsed) at validate time.

   If the completion expression is defined, any outputs which are optional in the expression
   should be marked as optional in the graph if used in the graph:

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

   Note: the completion condition only covers outputs generated when a task is executed - it cannot
   include `:expired`, `:submit` or `:submit-fail`. It also cannot include `started` (this has to
   happen if a task executes) or `finished` (this isn't an output).

   The default condition is complete all required outputs.

6. Expiry is only optional if it is explicitly referenced in the graph e.g. `a:expired? => x`.

   If a task is configured to clock-expire it must be made optional via the graph - otherwise this
   will cause a validation failure. This is designed to reduce the risk of unintended early shutdown.

   For this example:

   ```
   clock-expire = a
   a => b
   a:expired?  # This does nothing but is sufficient to pass validation
   b:submit-fail? => c
   ```

   Task "a" is completed in one of 2 ways:

   * It expires
   * It executes and succeeds

   If task "a" succeeds then task "b" must complete which can happen in one of 2 ways:

   * It fails to submit
   * It executes and succeeds

   So, the expression `a => b` in the graph can be interpreted as:
   * If "a" executes it must succeed
   * If "a" succeeds then "b" must complete
   * If "b" executes it must succeed

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

`a:expired?` has to be included in the graph to avoid validation failure.
This makes it clear that the halt is intentional.

### Output Groups

A contrived example demonstrating how more complex completion conditions can be implemented.

In this example, we don't consider the task complete unless one pair of outputs is generated:

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

If either `a` or `b` fail then they will be incomplete and the workflow will stall.

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
