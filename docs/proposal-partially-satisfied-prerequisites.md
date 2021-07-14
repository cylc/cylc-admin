# Handling of partially satisfied prerequisites

See https://github.com/cylc/cylc-flow/issues/4256

## New trigger syntax

A new trigger syntax is proposed which (it is hoped) will resolve many issues:

* `a:x ?=> b` is an "optional trigger": `b` is optionally triggered by `a`.
  Even if `a:x` does get generated it does not necessarily imply that `b` is expected to run.
  Note: this is how Cylc 8 currently behaves with the `=>` trigger.

* `a:x => b` is a "required trigger": `b` is required if `a` is run.
  Regardless of whether or not `a:x` was generated, `b` is expected to run eventually (once all its prerequisites are met).
  When `a` finishes, `b` will be spawned whether or not `a:x` was generated.

Note that this discussion will assume that tasks which are are optionally triggered do not get spawned until all their prerequsisites are met.
It may still make sense for them to be spawned but, if they are, we will assume it is into a hidden pool (which would require housekeeping).

A required task will remain in the task pool as `waiting` until all its prerequisities are met.
If it is not possible for the task prerequisities to be met then the task becomes "blocked" (but still `waiting`?).
i.e. it is not possible for the workflow to complete.
Blocked tasks should be highlighted to the user.
We should also add a new "blocked" event handler.
Note that unhandled failures will also be treated as blocked.

**Question**: are `waiting` tasks n=0, or n=1?

## Stalling / Shutdown

If there is nothing left to run yet there are still `waiting` tasks in the task pool then the workflow should stall (otherwise it is complete and should shutdown).
Note that this means that a task can have had some of its prerequisites met via optional triggers without this causing a stall.

## Runahead limit

See https://github.com/cylc/cylc-flow/issues/4258.
Some users understand runahead limiting as *preventing the fastest tasks from getting too far ahead of the slowest*.

Any `waiting` tasks are expected to run and could easily trigger further tasks at the same cycle.
Therefore these tasks need to count towards the runahead limit.
The same is true for "unhandled" failed tasks.

Note that [future triggers](https://cylc.github.io/doc/build/7.8.7/html/suite-config.html#future-triggers) have the potential to cause complications with the runahead limit by spawning tasks at old cycles.
This kind of issue would have been obvious at Cylc 7 because the task waiting for the future trigger would already have been in the task pool.

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
This workflow will now stall due to `qux` being in the `waiting` state with nothing left to run.

Another example:

```ini
[scheduling]
    cycling mode = integer
    initial cycle point = 1
    final cycle point = 10
    [[graph]]
        P1 = """
             model[-P1] => model => archive
             archive[-P1] => archive
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

If you run this with `cylc play --starttask=bar.2` then `baz.3` will remain stuck `waiting` and the workflow will eventually stall due to hitting the runahead limit.

The new trigger syntax also deals with this example:

```
        a | r1 => b ?=> r2
        a:fail ?=> r1 => r2
```

In this case, recovering from the failure of `a` involves running `r1` before running `b` and then running `r2` once `b1` has succeeded.
If `a` succeeds then `r2` will have partially satisified dependencies but this is not a problem since it is optionally triggered.
Note that this is an example of where a task is optionally triggered by one task but required by another.
This is not a problem.
`r2` only becomes required if `r1` is run.

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

This workflow will stall with `b` "blocked because `a:x` is not generated.
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
* We need a mechanism to ensure that any workflows which have stopped unexpectedly don't get missed and that users can easily find their most recent workflows
  (e.g. cylc review orders by last activity).

## Unhandled task failures

Does the new syntax mean we can simplify our handling of unhandled task failures?

In the simple case of `foo => bar`, if `foo` fails we'll be able to identify the unsatisfied `bar` as a problem even if we don't keep the failed `foo` in the pool.
However, keeping it in the pool (n=0) is likely to be more convenient and consistent.
In any case, what happens if `bar` fails - we certainly need to keep that in the pool.

So, we need to continue to keep any failed tasks in the pool (and consider them "blocked") unless they have a failure trigger.

**Question**:
How do we cater for cases where task failure is accepted and does not require any action?
We will need to contine to support suicide triggers for Cylc 7 compatibility so we can simply use:

```
        foo:fail => !foo
```

Alternatively we could introduce a special syntax for this such as:

```
        foo:fail => NULL
```

Note that a special syntax makes it easier to deal with this case:

```
        a | b => c
        a:fail | b:fail => NULL
```

In this case we need `a` or ` b` to succeed but we don't mind if one of them fails.
If both fail we don't get the convenience of them remaining in the task pool but at least `c` will be blocked so `a` and `b` will be visible in n=1.

## How do we tell if it is not possible for the task pre-requisities to be met?

Once all of the tasks referenced in the pre-requisities have completed then clearly we know whether it is possible for the pre-requisities to be met.
However, what happens if some of those tasks never run.
Consider this example again:

```
        foo ?=> bar => qux
        foo:fail ?=> baz => qux
```

If, for example, `foo` succeeds then `qux` becomes stuck waiting for `baz` to run.
In theory we can tell that `baz` cannot run but this would require graph traversal which could be complicated.
For the moment we will not attempt this.

One problem with this is that, even with required tasks, the workflow may never stall (depending on how the runahead limit is set).
However the `waiting` tasks should at least be visible to the user and there should also be another visible cause (blocked or failed tasks) unless there is an error in the graph.

## Cylc 7 compatibility

Cylc 7 workflows will not be taking advantage of the new syntax which means there will only be required triggers.
However there should always be corresponding suicide triggers where needed.

Suicide triggers were required at Cylc 7 to deal with 2 main cases:

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
* If the suicided task is triggered by any task present in the task pool then convert that trigger into a null trigger (can't simply remove it in case it is a failure trigger?).
  This attempts to deal with cases where a task is suicided before all its prerequisites have completed.
* If none of the above apply then no action is required.

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

## Suicide triggers at Cylc 8

So far, we have not thought of any cases where suicide triggers will be needed with the new syntax.
Therefore, suicide triggers will only needed for backwards compatibility.

**Proposal**: Suicide triggers should be flagged as deprecated!

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

**Question**: Why don't we have an "expired" event?
This seems a rather strange omission.

**Question**: How are clock expire triggers going to work at Cylc 8?
Presumably they suffer from the issue with have with late events (see [cylc-flow#4045](https://github.com/cylc/cylc-flow/issues/4045)) - they won't expire until they are spawned.

**Question**: Does [cylc-flow#3293](https://github.com/cylc/cylc-flow/issues/3293) still make sense?
Also, can we close [cylc-flow#3282](https://github.com/cylc/cylc-flow/issues/3282)?
