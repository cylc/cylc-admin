# How Task Expire Should Work


## What Expire Means

A task can be configured to *expire* instead of run:
- if too far behind the wall clock, or
- if forced to expire by the user

Either way, the implication is that there is no point in running the task now.
Perhaps (e.g.) its output files have been obtained by other means now, or are
not needed anymore.

NOTE that forced expiration is exactly the same, except that it is caused by
the user rather than the clock. In both cases we expire an incomplete task
(whether waiting or finished-but-incomplete) because it does not need to run
(or rerun) in order to be considered complete.

### Expired Tasks Are "Complete" and Should Not Cause a Stall

If a task expires, the workflow definition, or the user, has decreed that under
current conditions it *should* expire, i.e., we don't need to run it anymore.

That is an kind of success, and we should therefore mark expired tasks as
complete so the scheduler can forget them - not retain them as incomplete to
cause a stall.

### Tasks Downstream of an Expired Task Should Not Cause a Stall

```
a => foo => bar  # a succeeds, foo expires so foo never runs
```
There is no reason to stall on account of `bar` here. Required outputs are always
conditional on the owner task actually running in the first place, and the
downstream graph should not spawn at all if the outputs it hangs off are not
generated.

From the perspective of `bar` this no different than if `foo` not running because
the branch it lives on wasn't taken at all:

```
a:x? => foo => bar  # a does not generate :x, so foo never runs
```

### Expire Triggers Must Be Used To Avoid Early Halt, if That's What's Needed

An expired task obviously terminates the branch that it belongs to. If a task
does not run, whatever the reason, then downstream tasks that depend on it
running should not spawn.

Depending on the graph structure that has the potential to result in an early
scheduler shutdown, because the graph says there's nothing else to run.

However that *could be exactly what's wanted*, even in a cycling workflow that
has all future cycles cut off by an expired task. We cannot presume that the
graph is wrong if it says this should happen.

If downstream tasks, or a different graph branch, need to run after a task
expires, then :expire triggers must be used to achieve that.

### The :expire Pseudo-Outuput Should Be Marked Optional

We should enforce that :expire triggers be marked optional, because "required
expiration" doesn't really make sense.


### Optional :expire Does Not Mean :succeed Must Be Optional

Required outputs are always conditional on the owner task actually running, and
an expired task does not run at all. So `:succeed` (and other outputs) can
still be required even if expiration is optional. Required success really
means, *if the task runs, its success is required*.

The status of an expired task's required outputs is the same as that of the
required outputs of a task on a branch not taken at runtime.

foo:x? => bar
foo:y? => baz

Here, bar:succeed is required ONLY if bar runs, i.e. if branch x is taken. If
branch y is taken, the scheduler does not car that bar did not succeed.

foo | foo:expire? => bar

Here, foo:succeed is required ONLY if foo runs, i.e. if it doesn't expire. If
foo expires, the scheduler should not care that foo (and bar) did not succeed.

### Expired Should Really Be a Task Attribute, Not a Task State

`Expired` is currently a task state. It should really just be an attribute
(like `held`) because the underlying pre-expired state is potentially useful
information: we could instantly distinguish between waiting tasks that expired
without running to achieve completion, and finished-but-incomplete tasks were
force-expired without re-running to achieve completion.

I don't think anyone is deeply invested in expired as a state.
