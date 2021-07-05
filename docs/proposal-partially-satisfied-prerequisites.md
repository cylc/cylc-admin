# Handling of partially satisfied prerequisites

See https://github.com/cylc/cylc-flow/issues/4256

## Partially satisfied prerequisites

Should tasks with partially satisfied prerequisites cause a stall if the scheduler has nothing else to run?
Currently this is not the case with Cylc 8.

Consider the following workflow:

```ini
[scheduling]
    [[graph]]
        R1 = """
             foo => bar => qux
             foo:fail => baz => qux
             """
```

Currently this will just run `foo` and `bar` and then shutdown.
The user intended `qux` to run but has made a mistake in the graph.
It's too easy for a user to miss this (especially if this is part of a complex workflow or if it's something that only happens in some failure scenario).
For example:

```ini
[scheduling]
    cycling mode = integer
    initial cycle point = 1
    final cycle point = 5
    [[graph]]
        P1 = """
             model[-P1] => model => archive
             archive[-P1] => archive
             archive:fail => recover
             """
[runtime]
    [[model,archive,recover]]
        script = sleep 2
    [[archive]]
        pre-script = [[ $CYLC_TASK_CYCLE_POINT != 2 ]] || exit 1
```

A simple workflow where a recovery task has been added to deal with the rare case where the archiving fails.
However, a mistake in the graph means the archiving will be skipped in any cycle after a failure.
This kind of error is just too easy to make in complex suites - we need to stall in this situation.

Similar situations could occur operationally when they have to intervene due to problems.
Some tasks might get skipped in one cycle which could then lead to partially satisfied prerequisites in the next cycle - these require intervention.

Reload is another potential cause for issues.
Graphing changes to a running workflow could easily result in inconsistencies which cause partially satisfied prerequisites in the initial cycles.
Imagine making an operational change to the graph which then resulted in part of the graph being skipped - confidence in Cylc would quickly be destroyed.

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

If you run this with `cylc play --starttask=bar.2` then `baz` will never run and the workflow will currently shutdown (not stall) after `bar.7` runs due to the runahead limit
(see also https://github.com/cylc/cylc-flow/issues/4258).
Again, it seems clear this isn't what the user would want and it might be impossible to avoid this situation when doing reflows within a complex workflow.
Stalling is the best way to flag the problem and allow the user to intervene.

**Proposal**:
Once a task has partially satisfied prerequisites then it should be expected to run.
Therefore, if there are any tasks with partially satisfied prerequisites and there are no more tasks to run then this indicates a problem and we should stall.
This is to ensure that graphing errors (either errors in the actual graph or situations resulting from triggering/re-flow) will not go unnoticed by users.
They should also count towards the runahead limit.

**Question**:
Are tasks with partially satisfied prerequisites always shown in n=1?

Note that there are cases where tasks with partially satisfied prerequisites do not indicate a problem so stalling is not desirable.
Ways to address this are covered in later sections.

# Shutting down stalled workflows

Should a stalled workflow shut down?
Currently this is not the case with Cylc 7 or 8.
However, it can be configured at the site level via the `timeout` event and the `abort on timeout` setting.

Having a default behaviour where the scheduler process will keep running indefinitely even if there is nothing more for it to do doesn't really make sense.
However, the user needs the workflow to be running in order to address the problem so it makes sense to keep running for a period to give them a chance to do this.

**Proposal**:
Change the default for `abort on timeout` to `True` and set the default for the `timeout` event to `PT1H`.

We should also improve the visibility of stalled workflows.
*  `cylc scan` and the UI gscan panel should clearly indicate if a running workflow is stalled.
* Similarly for stopped workflows (also other states: "not started", "run to completion", "died"?, etc).
* Stopped workflows need to be ordered by last activity time (as they are in cylc review) so that any workflows which have stopped unexpectedly don't get missed.

# Required versus optional outputs

Stalling based on unhandled failed tasks and tasks with partially satisfied prerequisites doesn't address some potential runtime issues.
For example:

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

At present Cylc 7 would stall when running this workflow but Cylc 8 would not.

**Proposal**:
Introduce a new trigger syntax `?=>` to indicate that an output is optional.
If a finished task has any incomplete "required" outputs then it should remain in the task pool (n=0).
Any tasks with incomplete outputs should cause a stall if there are no more tasks to run.
They should also count towards the runahead limit.

In the above example, if `x` is an optional output we can indicate this in the graph as follows:
```
        a:x ?=> b
```

Another example:

```
        # success branch
        a:succeed ?=> a1 => a2
        # failure branch
        a:fail ?=> b1 => b2
        a2 | b2 => c
```

The `?=>` trigger indicates that the succeed and fail triggers are both optional so there will not be any incomplete outputs whichever path is chosen.

# Suicide triggers

With the changes proposed above, any task with required triggers for both succeed and fail will now cause a stall.
The same was true at Cylc 7 (although for different reasons) - suicide triggers were used to prevent this.
Therefore, we need to ensure that suicide triggers still work at Cylc 8 and prevent this kind of stall.

**Proposal**:
If a task is suicided then:
* remove it from the task pool if present
* remove it from the list of tasks with partially satisfied prerequisites if present
* remove it from the list of tasks with incomplete outputs if present
* go through the list of tasks with incomplete outputs and change any outputs which trigger the suicided task from required to optional

Hopefully this proposal handles all the ways in which suicide triggers were used at Cylc 7 and also enables them to be used to handle edge cases at Cylc 8 (if necessary).

# Conditional triggers

OR operators are a potential problem.
Consider this graph:

```
        a | b => c
```

This makes succeed a required output for both `a` and `b` so if either of them fail the workflow will stall.
However, what if we don't mind if one of them fails?
We can't use `a | b ?=> c` because that would allow both of them to fail.

**Proposal**:
Introduce a new trigger syntax `=>?` to indicate that an output becomes optional once the target prequisites have all been met.

Note that this is needed for trivial recovery tasks, for example:

```
        a | r =>? b
        a:fail ?=> r
```

Without this syntax we would need a suicide trigger:

```
        a | r => b
        a:fail ?=> r
        r => !a
```

(you can't just use `a | r ?=> b` since the workflow would complete if `b` failed)

# Unhandled task failures

The new `?=>` trigger means we need to re-think how we deal with "unhandled task failures".
Consider this graph:

```
        a => b ?=> c
```

If `a` fails then it will have incomplete outputs and the workflow will stall (no special unhandled failure logic required).

If `b` fails then the workflow is complete since the "succeed" output is optional (which implies that failure is acceptable).

If `c` fails we want the workflow to stall.
However, there are no outputs so we still need special logic for this unhandled failure.
One way to deal with this would be to add a dummy required "succeed" output (`=> NULL`) to any task which doesn't have either a "succeed" or a "fail" trigger.

Note that we have a problem if we want the workflow to complete even if `c` fails.
We would have to add a suicide trigger to allow this:

```
        c:fail => !c
```
# Partially satisfied prerequisites revisited
 
With the changes proposed above we are in a much better position to deal with various graphing issues in a consistent way and it should be easier to explain the behaviour to users.
However, we are still left with the problem that there are cases where partially satisfied prerequisites are expected and shouldn't cause a stall.
For example:

```
        a | r1 ?=> b => r2
        a:fail ?=> r1 => r2
        a & b => !r2
```

In this case `r2` is only meant to run if `a` fails but it will have partially satisfied prerequisites if `a` succeeds.
A suicide trigger is needed to prevent a stall (note that the suicide trigger must be actioned last, after the  `b => r2` trigger).

# Runahead limit

See https://github.com/cylc/cylc-flow/issues/4258.
Some users understand runahead limiting as *preventing the fastest tasks from getting too far ahead of the slowest*.
Tasks with partially satisfied prerequisites are expected to be run.
Similarly, tasks with incomplete outputs are expected to be re-run.
In both cases the tasks could trigger further tasks at the same cycle.
Therefore these tasks need to count towards the runahead limit.

In the future we should encourage users to add appropriate dependencies in the graph rather than relying on the runahead limit.
We should consider deprecating runahead limit in its current form and/or changing its default value.
