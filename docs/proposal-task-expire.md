# How Task Expire Should Work

(IN DISCUSSION: NOT ACCEPTED YET)

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
(or rerun) - it's completion is no longer required.

### Expired Tasks Should Not Cause a Stall

If a task expires, the workflow definition, or the user, has decreed that under
current conditions it *should* expire, i.e., we don't need to run it anymore.

Therefore expired tasks should not be retained as incomplete tasks (which is a
surprising/error condition) and allowed to stall the workflow.

### Tasks Downstream of an Expired Task Should Not Cause a Stall

If the workflow definition, or the user, has decreed that under current
circumstances a task should expire, i.e. that it should not run all all, then
by implication nothing downstream of it should run either.


```
a => foo => bar  # a succeeds, foo expires so foo never runs
```
So there is no reason to stall on account of `bar` here. Required outputs are
always conditional on the owner task actually running in the first place, and
the downstream graph should not spawn at all if the outputs it hangs off are
not generated.

From the perspective of `bar` this no different than if `foo` not running
because the branch it lives on wasn't taken at all:

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

(N.B. this is moot if we decide that `:expire` is not a task output - see
below).

### Optional :expire Does Not Mean :succeed Must Be Optional

Required outputs are always conditional on the owner task actually running, and
an expired task does not run at all. So `:succeed` (and other outputs) can
still be required even if expiration is optional. Required success really
means, *if the task runs, its success is required*.

The status of an expired task's required outputs is the same as that of the
required outputs of a task on a branch not taken at runtime.

```
foo:x? => bar
foo:y? => baz
```

Here, bar:succeed is required ONLY if bar runs, i.e. if branch x is taken. If
branch y is taken, the scheduler does not car that bar did not succeed.

```
foo | foo:expire? => bar
```

Here, foo:succeed is required ONLY if foo runs, i.e. if it doesn't expire. If
foo expires, the scheduler should not care that foo (and bar) did not succeed.

### Expired Should Really Be a Task Attribute, Not a Task State

`Expired` is currently a task state. It should really just be an attribute
(like `held`) because the underlying pre-expired state is potentially useful
information: we could instantly distinguish between waiting tasks that expired
without running to achieve completion, and finished-but-incomplete tasks were
force-expired without re-running to achieve completion.

Note this would also make sense in the context of NOT having `:expire` as a
task output at all (see below).

I don't think anyone is deeply invested in expired as a state.


## Is `:expire` Really a Task Output?

Treating `:expire` as a task output is not actually necessary, and it invites 
confusion about how it fits into the optional outputs framework: expiration
can't be required so it it must be optional, but does that imply success (and
every other output!) must be optional too?

As discussed above, optional expire does NOT imply optional success - because
expiration prevents a task from running at all, and the required nature of 
"real" outputs is always contingent on the task running in the first place.
How can a task that never even submits generate an output?

So that's unfortunately a nuance of optional outputs that users will have to
understand, **OR we could remove expiration from the outputs system.**

Expiration prevents a task from running in the first place. It makes more sense
to think of expiration as something the scheduler does TO the task, not
something that is done BY the task. (In fact, a task can in principle expire
long before worfklow activity even catches reaches it in the graph).

So we could use different notation to express this in the graph and allow
triggering off of expiration, e.g.:

```
foo => bar
@expire(foo) => baz  # if the scheduler (or user) expires foo, run baz
```

Now users do not even have to wonder if `foo:expire?` implies `foo:succeed?`.

Also, if we change expired from a state to an attribute, there is no reason to
have an associated triggerable output (we don't have `foo:held => bar`).
