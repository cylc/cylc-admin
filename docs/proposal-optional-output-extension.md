# PROPOSAL: Optional Output Extension

A proposal which extends the
[optional outputs model](https://cylc.github.io/cylc-admin/proposal-new-output-syntax.html)
to solve existing limitations and support the expired state by providing more
information to Cylc about the user's intentions for graph branching.

Supersedes:
* [cylc-admin/task expire proposal](https://github.com/cylc/cylc-admin/blob/fc564ddb26c1476dd051b059ca9a31829a20bf30/docs/proposal-task-expire.md)
* [cylc-flow#5423](https://github.com/cylc/cylc-flow/issues/5423)
* [cylc-flow#3294](https://github.com/cylc/cylc-flow/issues/3294)


## Background: Task Expiry

There are currently three problems with expiry (as of cylc-flow 8.1.4):

1. **Expire triggers are a [bit broken](https://github.com/cylc/cylc-flow/issues/5364)**

   This is a bug which can be fixed without fundamental change to the expiry
   implementation so can be ignored for the purpose of this proposal.

2. **Expiry triggers don't fit in with [optional outputs](https://github.com/cylc/cylc-flow/issues/5361)**

   Under the existing logic, if expiry is optional, then success must be too which means
   that failure would go uncaught making expiry unsafe.

   There are fundamentally two solutions to this problem.

   1. Move expiry to a separate model.
      * The existing
        [proposal-task-expire](https://cylc.github.io/cylc-admin/proposal-task-expire.html).
        follows this approach.
   2. Modify optional output logic to allow expiry to fit.
      * The approach this proposal follows.

3. **Expiry events can be detected late, or even missed due to the task pool implementation.**

   This is due to the coupling of expiry logic into the task pool.

   This proposal does not address the late event problem and recommends this
   should be a documented limitation.


## Background: Optional Outputs

The SoD model, initially had implicit graph branching. This was a neat feature, however,
meant that non-success cases were not properly caught so was not suitable behaviour for
real-world use.

The problem was that Cylc had not been provided with the required information to tell
the difference between and "expected" or "permitted" outcome and a "problem" outcome.

Consequently, optional outputs were introduced in order to provide Cylc with enough
information to resolve these situations and stall, where appropriate, or continue/stop
otherwise. This turned implicit branching explicit providing the runtime safety
required for production use.

Unfortunately, the optional outputs system doesn't have enough information at
present to adequately handle expiry as an optional output as `:expire?` would
imply `:succeeded?` and `:failed?`. This is a limitation of the optional output
mechanism which this proposal aims to address by providing sufficient
information to the scheduler to allow it to differentiate between "permitted"
and "problem" outcomes.


## Proposal

1. Expiry should remain implemented as a task output.

   If a task is clock-expired OR if `expired` output is set manually, then the
   task status should change to expired as per
   [cylc set proposal](https://cylc.github.io/cylc-admin/proposal-new-output-syntax.html).

2. The *default* condition for task completion condition should be:

   > If optional outputs are defined, at least one must be generated.

   Where outputs are considered "optional" according to their declaration in
   the graph and `completion` expression if defined.

   This is a breaking change for some examples where failure is permitted, but
   not handled in the graph (`a? => b`). These cases are relatively rare, can
   be caught by validation and are easy to fix.

3. The completion condition should be a configurable expression.

   The completion condition can be set to tolerate cases not handled in the graph or express
   more complex completion criterion.

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

4. Succeeded, failed and expired are three orthogonal completion outcomes.

   * `:expired` must always be optional because that's the nature of clock-expiry.
   * If `:succeeded` or `:failed` are referenced in the graph where expiry
     is present, then they must be optional too.

   Examples:

   ```
   # OK
   a:succeeded => x

   # OK
   a:succeeded? => x

   # OK
   a:succeeded? => x
   a:failed? => y

   # OK
   a:succeeded? => x
   a:expired? => y

   # OK
   a:succeeded? => x
   a:failed? => y
   a:expired? = z

   # ERROR
   a:succeeded => x  # :succeeded must be optional
   a:expired? = z

   # ERROR
   a:failed? => x
   a:expired = z  # :expired must be optional

   # ERROR
   a:succeeded? => x
   a:failed? => y
   a:expired => z  # :expired must be optional
   ```

5. Clock-expire should infer `expired?`:

   The `:expired?` output exists if:

   * Explicitly referenced in the graph e.g. `a:expired? => x`.
   * Or, if clock-expiry is configured for the task.

   Wherever expiry is configured or handled, graph branching can happen
   as the `:succeeded` and `:failed` outputs might not happen.

   So for this example:

   ```
   clock-expire = a
   a? => b
   ```

   The task "a" has two optional outputs:

   * succeeded
   * expired

   And the default completion expression is:

   ```
   completion(a) = succeeded or expired
   ```

   Examples:

   ```
   # OK
   clock-expire = a
   a? => x

   # OK
   clock-expire = a
   a? => x
   a:expire? => y

   # ERROR
   clock-expire = a
   a => x  # :succeeded must be optional

   # ERROR
   clock-expire = a
   a:failed => x  # :failed must be optional
   ```

6. Late delivery of expiry events to be a documented limitation.

   This is not a problem for parentless tasks as they are spawned out to the
   runahead limit.

   Cases where expiry is not a parentless task e.g:

   ```
   a[-P1D] => a?
   a:expire? => x  # can capture the expire event late
   a:succeed? => y
   ```

   Can be translated into the parentless pattern as a workaround:

   ```
   b:expire? => x  # capture the expire event early
   a[-P1D] & b? => a => y
   ```


## Examples

### The "xyz" problem

```
a:x? => x
a:y? => y
a:z? => z
x | y | z => b
```

Optional outputs for task "a":

* x
* y
* z

```
completion(a) = succeeded and (x or y or z)
```

### Expire Branch

```
clock-expire = a

a:succeed? => x
a:expired? => y
x | y => z
```

Optional outputs for task "a":

* succeeded
* expired

```
completion(a) = succeeded or expired
```

### Expire Halt

```
clock-expire = a

a:succeeded? => x
```

Optional outputs for task "a":

* succeeded
* expired (inferred by `clock-expire = a`)

```
completion(a) = succeeded or expired
```

Early halt in the event of expiry now works the same as with optional outputs.
Early halt in this example is now explicit in that it follows an optional output, making it clear
that the `x` branch might not happen and that expiry must be handled if early halt is not desirable.

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

Optional outputs for task "a" (defined in completion expression):

* w
* x
* y
* z

### Flaky Pipe

<!-- * **With 2(i)** -->

```
a? => b? => c?

[a, b, c]
# There is only one optional output here because a:failed? has not
# been handled in the graph.
# So we must permit failure in the completion expression to permit failure.
completion = succeeded or failed
```

Optional outputs for task "a" (defined in completion expression):

* succeeded
* failed

<!--
* **With 2(ii)**

  ```
  a? => b? => c?
  ```
  
  Optional outputs for task "a":
  
  * succeeded
  
  ```
  completion(a) = True  # This is essentially "succeeded or failed"
  ```
-->

### Recovery Task

```
a? | recover => b
a:failed? => recover
```

Optional outputs for task "a":

* succeeded
* failed

```
completion(a) = succeeded or failed
```

### Error Outputs 1

A safer version of the recovery task example where a specific error case is caught and turned into
a custom output in order to avoid random errors (e.g. syntax errors) from triggering recovery logic:

```
a? | recover => b
a:error_x? => recover

[a]
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

Optional outputs for task "a":

* succeeded
* error_x

```
completion(a) = succeeded or error_x
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

Optional outputs for task "a":

* succeeded
* failed
* error_x
* error_y
* error_z

### Complex Conditional

Contrived example to demonstrate how brining expiry into the optional output system allows it to
be used in combination with other triggers:

```
b?
c?
(a? & b:expired?) | (a:expired & c:failed?) => x
```

```
completion(a) = succeeded or expired
completion(b) = succeeded or expired
completion(c) = succeeded or failed
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
7. Expiry, success and failure are fundamentally orthogonal outcomes. Implementing expiry via
   a different model causes this relationship to break down. A consequence of this is that
   the graph branching which results from expiry becomes implicit rather than explicit.
   This has safety implications.


## Footnotes

[1] If a task is run and yields the output `x`, but is then manually re-triggered and yields the
    output `y`, then both the `x` and `y` outputs are completed on the task. Therefore Cylc cannot
    enforce mutually exclusive outputs e.g. `x xor y` so we should not attempt to support this
    in the `completion` expression.
