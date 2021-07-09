# Handling of partially satisfied prerequisites

See https://github.com/cylc/cylc-flow/issues/4256

## New trigger syntax

A new trigger syntax is proposed which (it is hoped) will resolve many issues:

* `a:x ?=> b` is an "optional dependency" (pure spawn-on-demand): if `a:x` does not get generated, `b` does not get spawned.
  Even if `a:x` does get generated it does not necessarily imply that `b` is expected to run.

* `a:x => b` is a "required dependency": when `a` finishes, `b` will be spawned whether or not `a:x` was generated.
  Furthermore, it implies that once `b` is spawned it is "required" - it is expected to run eventually (once all its prerequisites are met).
  If none of a tasks prerequisites are required dependencies then the task is "optional".

There are a number of possible conditions for tasks once they have been spawned.

* If all the task prerequisities are met then the task is ready to run (`waiting` -> `preparing`).

* If it is not possible for the task prerequisities to be met then:

  * If the task is "optional" it is simply removed from the task pool.

  * If the task is "required" then it is considered "blocked" (but still `waiting`?).
    i.e. it is not possible for the workflow to complete.
    Blocked tasks should be highlighted to the user.
    Note that unhandled failures will also be treated as blocked.

* If neither of the previous conditions apply the task simply remains in the task pool (`waiting`).

**Question**: are `waiting` tasks n=0, n=1 or hidden?
We probably want "required" tasks to be visible but not "optional" tasks?

**Question**: does it ever make sense for a task to have a mixture of optional and required prerequisites?
Can we think of a sensible use case?
If not, could we check for this at validation?

## Stalling / Shutdown

If there is nothing left to run yet there are still `waiting` tasks in the task pool then:

* If any of the tasks are "required" then the workflow should stall.

* If all of the tasks are "optional" then the workflow is complete and should shutdown normally.

## Runahead limit

See https://github.com/cylc/cylc-flow/issues/4258.
Some users understand runahead limiting as *preventing the fastest tasks from getting too far ahead of the slowest*.

Any `waiting` tasks which are "required" are expected to run and could easily trigger further tasks at the same cycle.
Therefore these tasks need to count towards the runahead limit.
The same is true for "unhandled" failed tasks.

On the other hand, `waiting` tasks which are "optional" should not affect the runahead limit.
There is no expectation that they will ever run and, if they are going to run then it is reasonable to expect that there are other "required" tasks in the pool at the same cycle or earlier
(not true with [future triggers](https://cylc.github.io/doc/build/7.8.7/html/suite-config.html#future-triggers)?).

In the future we should encourage users to add appropriate dependencies in the graph rather than relying on the runahead limit.
We should consider deprecating runahead limit in its current form and/or changing its default value.

## Partially satisfied prerequisites

Once the changes discussed above are implemented then this should address the concerns raised in https://github.com/cylc/cylc-flow/issues/4256.

Consider the following workflow:

```ini
[scheduling]
    [[graph]]
        R1 = """
             foo ?=> bar => qux
             foo:fail ?=> baz => qux
             """
[runtime]
    [[foo,bar,baz,qux]]
        script = true
```

The user intended `qux` to run but has made a mistake in the graph (should have been `bar | baz => qux`).
This workflow will now stall due to `qux` being "required" and in the `waiting` state with nothing left to run.

Another example:

```ini
[scheduling]
    cycling mode = integer
    initial cycle point = 1
    final cycle point = 10
    [[graph]]
        P1 = """
             model[-P1] => model => archive
             archive[-P1] ?=> archive
             archive:fail ?=> recover
             """
[runtime]
    [[model,archive,recover]]
        script = sleep 2
    [[archive]]
        pre-script = [[ $CYLC_TASK_CYCLE_POINT != 2 ]] || exit 1
```

A simple workflow where a recovery task has been added to deal with the rare case where the archiving fails.
However, a mistake in the graph means the prerequisites for `archive` can't be met in the cycle after a failure (should have been `archive[-P1] | recover[-P1] => archive`).
In this case `archive` will becomed "blocked" and the workflow will eventually stall due to hitting the runahead limit.

Similar situations could occur operationally when they have to intervene due to problems.
Some tasks might get skipped in one cycle which could then lead to partially satisfied prerequisites in the next cycle.
Reload is another potential cause for issues.
Graphing changes to a running workflow could easily result in inconsistencies which cause partially satisfied prerequisites in the initial cycles.
However, any "required" tasks will now get "blocked" and/or cause a stall so will not go unnoticed.

Re-flow is also a potential cause for partially satisfied prerequisites.
For example:

```ini
[scheduling]
    cycling mode = integer
    initial cycle point = 1
    [[graph]]
        P1 = """
             bar[-P1] => foo => bar & baz
             baz[-P1] => baz
             """
```

If you run this with `cylc play --starttask=bar.2` then `baz` will remain stuck `waiting` and the workflow will eventually stall due to hitting the runahead limit.

The new trigger syntax also deals with this example:

```
        a | r1 => b ?=> r2
        a:fail ?=> r1 ?=> r2
```

In this case, recovering from the failure of `a` involves running `r1` before running `b` and then running `r2` once `b1` has succeeded.
If `a` succeeds then `r2` will get spawned when `b` succeeds and get stuck `waiting`.
However, since it is "optional" this is not a problem - it will not cause a stall.

## Optional outputs

The new syntax makes it easy to deal with optional outputs.
Consider this workflow.

```ini
[scheduling]
    [[graph]]
        R1 = a:x => b
[runtime]
    [[a]]
        script = true
        [[[outputs]]]
            x = x
```

This workflow will now stall with `b` "blocked.
If that's not what is wanted (`x` is an optional output) you simply change the graph to

```
        R1 = a:x ?=> b
```

## Shutting down stalled workflows

Should a stalled workflow shut down?
Currently this is not the case with Cylc 7 or 8.
However, it can be configured at the site level via the `timeout` event and the `abort on timeout` setting.

Having a default behaviour where the scheduler process will keep running indefinitely even if there is nothing more for it to do doesn't really make sense.
However, the user needs the workflow to be running in order to address the problem so it makes sense to keep running for a period to give them a chance to do this.

**Proposal**:
Change the default for `abort on timeout` to `True` and set the default for the `timeout` event to `PT1H`.

We should also improve the visibility of stalled workflows and blocked tasks.
*  `cylc scan` and the UI gscan panel should clearly indicate if a running workflow is stalled or has blocked tasks.
* Similarly for stopped workflows (also other states: "not started", "run to completion", "died"?, etc).
* Stopped workflows need to be ordered by last activity time (as they are in cylc review) so that any workflows which have stopped unexpectedly don't get missed.

## Unhandled task failures

Does the new syntax mean we can simplify our handling of unhandled task failures?

In the simple case of `foo => bar`, if `foo` fails we'll be able to identify the unsatisfied `bar` as a problem even if we don't keep the failed `foo` in the pool.
However, keeping it in the pool (n=0) is likely to be more convenient and consistent.
In any case, what happens if `bar` fails - we certainly need to keep that in the pool.

Note that we can't use optional dependencies as an indicator that failure is acceptable.
Consider this earlier example:

```
        a | r1 => b ?=> r2
        a:fail ?=> r1 ?=> r2
```

In this case, `b` only has an optional success dependency but that doesn't mean we're happy for it to fail.

So, we need to continue to keep any failed tasks in the pool (and consider them "blocked") unless they have a failure trigger.

**Question**:

How do we cater for cases where task failure is accepted and does not require any action?
We will need to contine to support sucide triggers for Cylc 7 compatibility so we can simply use:

```
        foo:fail => !foo
```

Alternatively we could introduce a special syntax for this such as:

```
        foo:fail => NULL
        foo:fail => OK
        foo:fail => -
        # It's not a real trigger so maybe better to avoid =>?
        foo:fail =-
        foo:fail =<
```

Note that a special syntax makes it easier to deal with this case:

```
        a | b => c
        a:fail | b:fail => OK
```

In this case we need `a` or ` b` to succeed but we don't mind if one of them fails.
If both fail we don't get the convenience of them remaining in the task pool but at least `c` will be blocked so `a` and `b` will be visible in n=1.

## How do we tell if it is not possible for the task pre-requisities to be met?

Once all of the tasks referenced in the pre-requisities have completed then clearly we know whether it is possible for the pre-requisities to be met.
However, what happens if some of those tasks never run.
Consider this example again:

```
        a | r1 => b ?=> r2
        a:fail ?=> r1 ?=> r2
```

If `a` succeeds then `r2` will get spawned when `b` succeeds and get stuck `waiting`.
In theory we can tell that `r1` cannot run.
However, this would require graph traversal which could be complicated.
For the moment we will not attempt this.

One problem with this is that, even with required tasks, the workflow may never stall (depending on how the runahead limit is set).
For required tasks they should at least be visible to the user and there should also be another visible cause (blocked or failed tasks) unless there is an error in the graph.
However, the example above shows how there can be `waiting` "optional" tasks which will never get run.
This means they could build up to a large number in a long running cycling workflow.

**Question**:
Do we need to housekeep these tasks?
(Presumably yes!)
Once there are no other "required" tasks or preparing/running/failed in the pool at any of the cycles referenced by the incomplete prerequisites or earlier can we remove them?
Do [future triggers](https://cylc.github.io/doc/build/7.8.7/html/suite-config.html#future-triggers) complicate this?

Note that when a task is spawned then, if it has optional prerequisites, we don't know whether these prerequisites have already completed (and not generated the required output).
A database query will be required to determine this.
The same is not true for required prerequisites (the task is always spawned at completion).

An alternative approach for optional tasks would be to only spawn them once their prerequisites have been met.
This would avoid any need to housekeep them.
For tasks with multiple prerequisites it would require additional database queries.
However, this may not be a big deal given that not many workflows are likely to have large numbers of optional tasks.

## Cylc 7 compatibility

Cylc 7 workflows will not be taking advantage of the new syntax which means that Cylc 8 will consider all tasks are required.
However there should always be corresponding suicide triggers.

Suicide triggers were required at Cylc 7 to deal with 2 cases:

1. Tasks that were waiting but would never run.
2. Tasks that had failed but the failure had been handled.

For example, consider this Cylc 8 workflow:

```
        a ?=> b1 => b2
        a:fail ?=> c1 => c2
        b2 | c2 => d
```

At Cylc 7 this would have been something like:

```
        a => b1 => b2
        a:fail => c1 => c2
        b2 | c2 => d
        a => !c1 & !c2
        a:fail => !a & !b1 & !b2
        d => !a # prevent "a" disappearing from the GUI too quickly
```

So, for backwards compatibility, we still need suicide triggers to work at Cylc 8 to prevent stalls.
It is not possible to work out appropriate graph changes in advance so we need suicide triggers to apply at runtime as follows:
* If the suicided task is present in the task pool in the `waiting`, `submit-failed` or `run-failed` state then remove it.
* If the suicided task is present in the task pool in the `preparing`, `submitted` or `running` state then remove it from all flows (to prevent further tasks being spawned) and log a warning (since it doesn't make sense for a task in these states to be suicided).
* If the suicided task is not present in the task pool then no action is required.
* If the suicided task is triggered by any task present in the task pool then convert that trigger into a null trigger (can't simply remove it in case it is a failure trigger?).
  This attempts to deal with cases where a task is suicided before all its prerequisites have completed.

Note that any workflows which rely on suicide triggers are likely to have tasks which become "blocked" for a period before they get cleared by a suicide trigger.
How do we handle this?
* Simply warn users than they may get notified of transient blocks for workflows which still rely on suicide triggers?
* Don't treat any task as blocked if it has a suicide trigger anywhere in the graph?

The above logic won't deal with all cases where a task is suicided before all its prerequisites have completed.
For example:

```ini
[scheduling]
    [[dependencies]]
        graph = """
                a => b => c => d
                a => check-d => d
                check-d:fail => !check-d & !d
                """
[runtime]
    [[a,b,c,d]]
        script = sleep 10
    [[check-d]]
        script = false
```

At Cylc 7 this will skip `d` and shutdown.
With the proposed Cylc 8 logic, `d` will still get run because `c` is not in the task pool when `d` is suicided.
The only obvious way to avoid this is to maintain a list of suicided tasks which gets checked before any task is spawned.
However, that involves extra checks and another list to maintain and housekeep to handle a very obsure case.
Perhaps we should accept that obscure uses of suicide triggers are not guaranteed to work as intended and document this in the migration guide?

## Suicide triggers

So far, we have not thought of any cases where suicide triggers will be needed with the new syntax.
Therefore, suicide triggers will only needed for backwards compatibility and can be flagged as deprecated.

Here is an example where care is needed when replacing suicide triggers:

```
        a:x => x1
        a:y => y1
        a:z => z1
        x1 | y1 | z1 => b
        a:x | a:y => ! z1
        a:x | a:z => ! y1
        a:z | a:y => ! x1
```

The intention here is that `a` produces one (and only one) of the outputs `x`, `y` or `z`.
Using the new syntax this would become:

```
        a:x ?=> x1
        a:y ?=> y1
        a:z ?=> z1
        x1 | y1 | z1 => b
        a:start => b
```

The additional `a:start => b` dependency is required to ensure `b` gets spawned, even if none  of `x1`, `y1` or `z1` are triggered.

Note that care may be needed with [clock expire triggers](https://cylc.github.io/doc/build/7.8.7/html/suite-config.html#clock-expire-triggers).
Any tasks downstream of a clock expired task will also need to be clock expired if they could get spawned via another trigger.
For example:

```
[scheduling]
    initial cycle point = 2000-01-01
    [[special tasks]]
        clock-expire = late(P0D), finish(P0D)
    [[dependencies]]
        [[[ R1 ]]]
            graph = "start => early & late => finish"
[runtime]
    [[start,early,late,finish]]
        script = true
```

In this case both `late` and `finish` need to be clock expired.

Note that the current documentation says
"Triggering off an expired task typically requires suicide triggers to remove the workflow that runs if the task has not expired".
Why didn't we just recommend using the same clock expire trigger for these tasks?
One downside is that this would trigger multiple "late" events rather than one.
Is this something we are happy to live with at Cylc 8?
