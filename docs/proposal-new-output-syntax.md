# Proposal: Required and Optional Outputs

Cylc 8 spawn-on-demand solves many problems but in its pure form it can't tell
if the workflow has followed the "intended path" - to completion of the whole
graph, say, vs going down a dead-end side branch. This is because all task
outputs are effectively *optional*, and the scheduler just follows the flow
wherever it leads at runtime.

Cylc 7 could tell (albeit imperfectly!) if the intended path was followed,
because it pre-spawned all possible paths (to the next cycle point instance of
each active task, roughly speaking) and expected users to remove unused paths
with suicide triggers. Any remaining unsatisfied waiting tasks would stall 
the workflow.

This proposal introduces the concepts of **task completion**, and **required**
and **optional** outputs, as a way to convey the intended path of execution to
the Cylc 8 scheduler. *Tasks that finish without completing all required
outputs will be retained in the `n=0` task pool as incomplete tasks*, which
can be flagged as error and (like partially satisfied prerequisites) will stall
the scheduler if it has nothing else to do. Optional outputs do not contribute
to task completion. Incomplete tasks will often be failed tasks that were
required to succeed, otherwise they indicate that the task did not do what the
graph expected despite reporting success.

*Note: required and optional triggers were considered and rejected as another
way of potentially handling this problem - see cylc-admin#128*

## Output Syntax

- `foo:x` means `x` is a *required output* of task `foo`
  - if `foo` finishes without completing `x`, retain it in n=0 as *incomplete*

- `foo:y?` means `y` is an *optional output* of task `foo`
  - it is OK for `foo` to finish without completing `y`

A task is incomplete if:
- it finished executing without completing all required outputs
- or if job submission failed and the `:submit` output was not optional

Abbreviated syntax for the default success case:
- `foo?` means `foo:succeed?` (success optional)
- `foo` means `foo:succeed` (success required)

Examples in trigger expressions:
```
# trigger bar if foo completes x; otherwise flag foo as incomplete
foo:x => bar

# trigger bar if foo completes x; but either way don't flag foo as incomplete
foo:x? => bar
```

### Notes and Caveats

The same output can't be both required and optional.

`foo:succeed` and `foo:fail` (like `foo:submit` and `foo:submit-fail`) are
mutually exclusive opposites.
- if one is required the other can't appear in the graph at all
- if one is optional they must both be optional
- if neither appear in the graph (i.e. only other outputs such as `foo:x`
  appear) we assume success is required

Optional `:submit?` does not imply that task execution outputs must be
optional. You might want to handle job submission failure but require tasks to
(e.g.) succeed if they do submit successfully. 

Failure can be required, but we should document that expecting a task to fail
every time and treating it as incomplete if it succeeds essentially reverses
normal succeed/fail symantics. A task that always fails whilst still doing
what the workflow needs should probably be wrapped to return success on exit.
And note that we don't support retry-on-succeed.

Optional `:start?` is not allowed because it is meaningless. Any task that
finishes (which is when we can check its outputs) must have started, by
definition.

The pseudo output `:finish` (which means "succeed or fail") cannot be optional.
The only way for a task not to finish is if its job gets stuck in an infinite
loop (use execution timeout) or if it never started running in the first place.

The new optional output syntax cannot be used with family collective psuedo
output triggers because it is difficult to interpret that in terms of the
success status of each individual member, which is what we need.
Instead we set appropriate member defaults for each family trigger.
See [Family Triggers](#Family-Triggers) below. 

## Path Branching

### Concurrent Paths 

Concurrent paths can trigger off of required outputs:
```
a:x => b1
a:y => b2
b1 & b2 => c  # join
```

### Alternate Paths

Alternate paths must branch off of optional outputs, and must be joined with
OR triggers because only one side or the other will run:

```
a:x? => b1
a:y? => b2
b1 | b2 => c  # join
```
Note joining can trigger off of required outputs, because the output is
only "required" if the task actually runs.

**Failure recovery** is a common example of this:
```
a? => b1  # success path
a:fail? => b2  # failure path
b1 | b2 => c  # join
```

Note that an artificial dependency can be used to *ensure that at least one
of the alternate paths gets taken*:
```
a:x? => b1
a:y? => b2
b1 | b2 => c  # join
a => c # ARTIFICIAL DEPENDENCY
```
This forces `c` to be spawned as an unsatisfied prerequisite waiting on `b1` or
`b2` even if neither branch runs. If the alternate paths don't actually need to
join, `c` can be a dummy task.

*See Appendix Output Completion Expressions for a possible follow-up that
would allow us to flag `a` as incomplete here and avoid the artificial
dependency.*

## Failed Tasks

No special handling of failed tasks is required.
- if success is required, a failed task automatically gets retained in the task
  pool as incomplete.
- if success is optional (which would usually imply that optional failure is
  explicitly handled), a failed task can be removed as complete

No special syntax (e.g. `c:fail => NULL`) or suicide triggers (e.g. `c:fail =>
!c`) are needed for leaf tasks either, if we don't care if they succeed or fail:
```
a => b => c?  # success of c is optional
```

## Family Triggers

The new optional output syntax cannot be used with family collective psuedo
output triggers because it is difficult to interpret that in terms of the
success status of each individual member, which is what we need.

Instead we will set different optional/required defaults for each collective
pseudo output:

```
A:succeed-all => b  # member success required
A:succeed-any => b  # member success (and hence failure) optional

A:fail-all => b  # member failure required 
A:fail-any => b  # member failure (and hence success) optional

A:finish-all => b  # member success (and hence fail) ooptional
A:finish-any => b  # member success (and hence fail) optional

A:start-all => b  # N/A
A:start-any => b  # N/A

A:submit-all => b  # member submit success required
A:submit-any => b  # member submit success (and hence submit-fail) optional

A:submit-fail-all => b  # member submit-fail required
A:submit-fail-any => b  # member submit-fail (and hence submit success) optional

A:x-all => b  # member x required
A:x-any => b  # member x optional
```

If different family triggers imply both required and optional member output
defaults for the same family, we default to optional. You can use the
notation without a trigger to force this.  E.g.:
```
# Trigger b only if all members of A succeed, but member success is optional:
A:succeed-all => b  # default to member success required ...
A:succeed-any  # ... but now member success is optional
```
To change individual members, just use the normal task output notation:
```
A:succeed-all => b  # members of A default to success required ...
a1?  # ... but success of member a1 is optional
```

**Note** alternatively we could use the task notation to set member defaults.
This would also allow member success required with -any triggers.
```
A:succeed-any => b  # default to member success optional...
A:succeed  # ... then make it required
```

## Backward Compatibility (Cylc 7)

In Cylc 7 graphs, all outputs are effectively required in that all paths are
pre-spawned to the next cycle point, and users are expected to define suicide
triggers to remove unused branches that are really optional.

In Cylc 8, suicide triggers mostly aren't needed because unused branches don't
get spawned at all, but they still work if needed.

Assume we have a Cylc 7 workflow if the config filename is `suite.rc`. If users
change the filename without upgrading syntax they will get validation errors.
- if both `foo:succeed` and `foo:fail` appear (an error for Cylc 8) infer that
  optional success
- similarly for `foo:submit` and `foo:submit-fail`
- we can't know if custom outputs are optional or not (because we don't know if
  they are mutually exclusive or concurrent) so assume they are required and
let the Cylc 7 suicide triggers clean up incomplete tasks (we could try to use
suicide triggers to make the correct inference but that might be difficult).

## Visibility of Incomplete Tasks

Incomplete tasks should be flagged immediately as a critical error.

Most of the time they will be failed tasks that were required to succeed.
Otherwise they represent an error against the workflow configuration that was
not recognized by the task job itself. That could indicate a bug in the task
(it reported success when it shouldn't have), or it could mean the workflow
writer misunderstood what the task is supposed to do.

## Visibility of Partially Satisfied Prerequisites

Partially satisfied prerequisites should be made visible as an indication of a
*potential problem*.

- They can be normal transient states as a task's multiple prerequisites get
  satisfied at different times.
- Or they can result from a subtle workflow design error, where a task
  depends on several *alternate* branches at once:
  (e.g. when `a & b => c` should be `a | b = c`).
- Or they can result from an upstream path being blocked when a required
  output doesn't get completed - which will now also result in an incomplete
  task.

Note **output completion expressions** above will eliminate one source of
partially satisfied prerequisites.

### Can we tell if it is not possible for task prerequisities to be met?

If all tasks referenced in an unsatisfied prerequisite have finished then we
know it can't be satisfied without user intervention. But otherwise we can't
tell without potentially complicated graph traversal, which we have decided not
to attempt for now.

One problem is, depending on how the runahead limit is set a workflow may never
stall even if there are incomplete tasks in the pool. However the partially
satisfied prerequisites (associated with particular waiting tasks, of course)
will at least be visible to the user, and there should also be another visible
cause (incomplete and/or failed tasks) unless there is an error in the graph.

## Scheduler Stall and Shutdown

If the scheduler runs out of tasks to submit it should:
- report the workflow is finished and shut down *only if there are NO
  incomplete tasks or partially satisfied prerequisites present*
- or otherwise, report a stall and stay alive to allow user intervention

Users need the scheduler to be running in order to fix a stalled workflow,
but it doesn't make sense to stay alive forever with nothing more do, so
we should default to `abort on timeout = True` and `timeout = PT1H` in global
config. This will also apply on restarting a stalled workflow that timed out.

We should also improve the visibility of stalled workflows and incomplete tasks
  - `cylc scan` and the UI gscan panel should clearly indicate if a running
  workflow is stalled or has incomplete tasks
  - similarly for stopped workflows (also: "not started", "ran to completion",
    "died", etc.).
  - we need a mechanism to ensure that any workflows that stopped unexpectedly
    don't get missed and that users can easily find their most recent workflows
    (e.g. `cylc review` orders by last activity).

## New Event Handler

We should add a new **incomplete task** event that is triggered whenever a task
finishes without completing all of its required outputs.

## Runahead limit

See https://github.com/cylc/cylc-flow/issues/4258. Some users understand
runahead limiting as preventing the fastest tasks from getting too far ahead of
the slowest.

Incomplete tasks and partially satisfied prerequisites both represent tasks
that will, or at least might after user intervention, still run at their cycle
point, so they should count towards the runahead limit.

(They should not count toward an *active cycle point* based limit, if we decide
to implement that).

Note that future triggers have the potential to cause complications with the
runahead limit by spawning tasks at old cycles. This kind of issue would have
been obvious at Cylc 7 because the task waiting for the future trigger would
already have been in the task pool.

## Cylc 7 compatibility

Cylc 7 workflows will not be using the new syntax which means they will only
have required outputs (but that's OK - Cylc 7 outputs **are** "required" as
explained above).

They also use suicide triggers to remove:
- pre-spawned waiting tasks that would never run (i.e. from unused alternate
  branches)
- failed tasks that don't need to be retained in the pool

For example, in this Cylc 8 graph the `c1 => c2` branch does not get spawned
at all unless `a` fails:
```
a? => b1 => b2
a:fail? => c1 => c2
b2 | c2 => d
```

The Cylc 7 version of this already works out of the box on Cylc 8 master, for
backward compatibility:
```
a => b1 => b2
a:fail => c1 => c2
b2 | c2 => d
a => !c1 & !c2
a:fail => !a & !b1 & !b2
d => !a # prevent "a" disappearing from the GUI too quickly
```

This works because Cylc 8 suicide triggers spawn (or update) children just like
normal triggers. Once all of a task's suicide prerequisites are satisfied it
is removed immediately. A task that has a single suicide trigger and would not
otherwise be spawned in Cylc 8 will have a transient presence in the task pool
because the suicide trigger that spawns it also satisfies its only suicide
prerequisite.

It doesn't matter if a task still has unsatisfied normal prerequisites when it
suicides. The other triggers can't spawn it again in the same flow thanks to
our conditional reflow prevention mechanism.

For the Cylc 7 graph above to work with the new optional output syntax we just
need to infer that `a:succeed` and `a:fail` are optional, to avoid `a` ending
up marked as incomplete both ways.

This example also works on master already:
```
# Cylc 7
[scheduling]
    [[dependencies]]
        # If check-d fails, skip d even if c has not finished yet.
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
The Cylc 8 version still needs a suicide trigger to remove an unsatisfied
prerequisite:
```
# Cylc 8
[scheduling]
    [[graph]]
        # If check-d fails, skip d even if c has not finished yet.
        R1 = """
            a => b => c => d
            a => check-d? => d
            check-d:fail? => !d
        """
[runtime]
   ...
```
This is actually a simple case of the "same task with different dependencies on
difference branches" problem (see the Appendix below). Cylc 9 will be able to
handle this properly:
```
# Cylc 9
[scheduling]
    [[graph]]
        # If check-d fails, skip d even if c has not finished yet.
        R1 = """
            a => b => c
            a => check-d
            check-d:fail? => { }  # or just "check-d:fail?"
            check-d? & c => { d }
        """
[runtime]
   ...
```

Another example:
```
# Cylc 7
a:x => x1
a:y => y1
a:z => z1
x1 | y1 | z1 => b
a:x | a:y => ! z1
a:x | a:z => ! y1
a:z | a:y => ! x1
```
The intention is that `a` completes one (and only one) of the outputs x, y and
z. Note that the pre-spawning of `b` by Cylc 7 ensures that the workflow will
stall if none of the outputs are completed, but the suicide triggers will just
break the workflow if more than one completes.

Using the new syntax this would become:
```
# Cylc 8
a:x? => x1
a:y? => y1
a:z? => z1
x1 | y1 | z1 => b

a(x|y|z)  # a is required to succeed with at least one of x, y, z
  # OR
a => b  # spawn b as a prerequisite that needs at least one branch to be satisfied
```

For backward compatibility all we need to do is, once again, infer that x, y,
and z are optional outputs (the suicide triggers imply this) and add the
completion condition. It is probably sufficient to assume the form `a(x|y|z)`
for all optional output groups.

## Suicide Triggers Still Required at Cylc 8

Suicide triggers will probably be made entirely obsolete by dynamic subgraphs
at Cylc 9. In the meantime we need to keep them for the following reasons:

- primarily, backward compatibility for Cylc 7 graphs

- secondarily, to allow housekeeping of partially satisfied prerequisites
  that can occur because of partial dependence on optional outputs:

```
a & b? => c
b:fail? => !c
```
  or submit failures:
```
a & b => c
b:submit-fail? => !c
```

- finally, to handle the unwanted extra dependencies picked up by tasks
  that appear on multiple graph branches at once (see [Appendix: Same Task on
Multiple Branches](#appendix-same-task-on-multiple-branches)
the Appendix below, and the `check-d` example above).

## Clock Expire Triggers

**(This is off topic, should be moved to a cylc-flow Issue)**

Care is needed with clock expire triggers. Any tasks downstream of a clock
expired task also need to be clock expired if they could get spawned by
another trigger. For example:

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
In this case both late and finish need to be clock expired to avoid an
unsatisfied prerequisite.

Note that the current documentation says "Triggering off an expired task
typically requires suicide triggers to remove the workflow that runs if the
task has not expired". We should change this to recommend using the same clock
expire trigger for these tasks. (But suicide triggers will still work).

**Question** Why don't we have an "expired" event? This seems a rather strange
omission.

**Question** How are clock expire triggers going to work at Cylc 8? Presumably
they suffer from the issue with have with late events (see cylc-flow#4045) -
they won't expire until they are spawned.

**Question** Does cylc-flow#3293 still make sense? Also, can we close
cylc-flow#3282?

## Other Examples

**Graphing error** `qux` triggers off of both alternate branches (should be
   either-or):
```
[scheduling]
    [[graph]]
        R1 = """
             foo? => bar => qux
             foo:fail? => baz => qux
             """
[runtime]
    [[foo,bar,baz,qux]]
        script = true
```
Result: `qux` will be spawned by the path that runs, then it will be stuck as
partially satisfied because it also depends on the other path.

**Graphing error** `archive` needs to trigger off of previous `archive` OR
   previous `recover`:
```
[scheduling]
    cycling mode = integer
    initial cycle point = 1
    final cycle point = 10
    [[graph]]
        P1 = """
             model[-P1] => model => archive
             archive[-P1]? => archive
             archive:fail? => recover
             """
[runtime]
    [[model,archive,recover]]
        script = sleep 2
    [[archive]]
        pre-script = [[ $CYLC_TASK_CYCLE_POINT != 2 ]] || exit 1
```
Result: the next `archive` will still be spawned (by the next `model`) and it
will be stuck as partially satisfied because it also depends on its previous
instance.

**Re-flow, with `cylc play --starttask=bar.2`**

```
[scheduling]
    cycling mode = integer
    initial cycle point = 1
    [[graph]]
        P1 = """
             bar[-P1] => foo => bar & baz
             baz[-P1] => baz
             """
```
Result: `baz.3` stuck as partially satisfied waiting on `baz.2`

# Appendix: Same Task on Multiple Branches

*This is a more in-depth explanation of how our graph syntax works,
which should go into the user guide. I went through the exercise while
investigating the pipeline example for optional outputs, on the PR page,
but it's largely off-topic now.*

Cylc assumes that exactly one instance of each task exists at a given cycle
point (as reflected by the unique task ID being only [name].[cycle-point]). So
in graph parsing, we can add all separately-defined dependencies for a given
task to the one task definition.

In other words, we assume this graph:

```
a => b => z
a => c => z
```
does not describe two separate paths with different instances of a and z on
each. Rather, it is equivalent to this:

```
a => b & c => z
```
i.e. the graph branches at a and joins at z.

If we actually want separate instances of z (say) on each branch, we can use
two logical tasks that inherit the same job content:

```
a => b => z1
a => c => z2
...
[runtime]
   [[Z]]
   [[z1, z2]]
       inherit = Z
```
The example above describes concurrent branches (i.e. both branches are
expected to run). Alternate (i.e. either/or) branches are also OK if each
branch contains different task names and they join again with an OR condition.
Here, the graph branches at a - to one path or the other - and joins again at
z:

```
a? => b  
a:fail? => c 
b | c => z
```

However, a problem arises if we try to put different instances of (say) x in
each alternate branch.
```
# ERROR: this does not result in separate `x`'s on each branch
a? => b => x => m => z
a:fail? => c => x => n => z
m | n => z
```

Cylc assumes again that both x's refer to the same task, effectively joining
the two branches at x which doesn't make sense for alternate branches (only one
of b or c will run, so x will be stalled waiting on the other):

```
b & c => x => m & n
```

Once again, we can fix the problem by using different logical tasks to
represent the same x job content, in each branch:

```
a? => b => x1 => m => z
a:fail? => c => x2 => n => z
m | n => z
...
[runtime]
   [[X]]
   [[x1, x2]]
        inherit = X
```

Rarely, it may not be not possible to use different logical tasks to represent
the same task in different branches. Consider two pipelines:

```
a1 => b1 => d1
a2 => b2 => d2
```
so far so good, but what if we want to use a task c to collate data from b1 and
b2 for some reason, before finishing the pipelines:

```
a1 => b1 => c => d1
a2 => b2 => c => d2
```

The two paths now join and separate again at c (which we can also write like
this: `b1 & b2 => c => d1 & d2`). But if we want to abort a pipeline whose
initial task fails, we can't because unfortunately `c` still depends on `b1`
and `b2` from both branches (so it will be stuck as unsatisfied if one branch
doesn't run), and if it did run it would still trigger both `d1` and `d2`. So
we need to deal with these extra prerequisites and outputs acquired from the
"other" branch.

### Cylc 7 Workaround

```
a1 => b1
a1:fail | b1 => c => d1
a1:fail => !a1 & !b1 & !d1
a2 => b2
a2:fail | b2 => c => d2
a2:fail => !a2 & !b2 & !d2
```
- `a1:fail? | b1 => c` and `a2:fail? | b2 => c` are artificial dependencies
designed to ensure that `c` triggers even if either of its parent tasks fail -
because `c` is needed to trigger both `d`s
- `!d1` etc. are suicide triggers to remove downstream tasks that won't be
needed if their pipelines are cancelled

### Cylc 8 Workaround

```
a1? => b1
a2? => b2
a1:fail? | b1 => c => d1
a2:fail? | b2 => c => d2
a1:fail? => !d1
a2:fail? => !d2
```
Note:
- artificial OR dependencies as for the Cylc 7 case
- suicide triggers are only needed for affected `d`s in Cylc 8 (spawn on demand)

(Note: #128 would require additional artificial dependencies `b1 => d1` and
`b2 => d2` to make the optional trigger work by creating an unsatisfied
dependency for the unused pipelined)

## Another Example

Intent:
- if `a` succeeds, run `b`
- if `a` fails, run `r1 => b => r2`

This is a pathological case for Cylc 7 and 8 because `b` appears with different
dependencies in alternate branches, but for the reasons given above both
branches contribute their `b` dependencies to the same task.

Cylc 9 will handle this properly and intuitively, but we can't do it yet:
```
a? => {b}
a:fail? => {r1 => b => r2}
```
...the lower sub-graph does not add its dependencies to `b` unless `a` fails.

Cylc 7 and 8 workaround: use different logical tasks (with different
dependencies) for the same job in each branch:
```
a? => b1
a:fail? => r1 => b2 => r2

[runtime]
   [[B]]
   [[b1, b2]]
       inherit = B
```

# Appendix: Output Completion Expressions?

A possible follow-up to this proposal.

In an example above we used an artificial dependency (possibly triggering a
dummy task) to deliberately generate an unsatisfied prerequisite that (might
eventually) stall the workflow if neither of two alternate branches run:
```
a:x? => b1
a:y? => b2
b1 | b2 => c  # join
a => c # ARTIFICIAL DEPENDENCY
```

Note we're assuming here that `a` is *supposed to generate x or y*, and if it
doesn't generate either of them something has gone wrong - either `a`
encountered an error but did not report failure, or the workflow writer has
not correctly understood how `a` behaves. Then, the artificial dependency
causes an unsatisfied `c` to be spawned even if neither branch runs, to
(eventually) stall the workflow.

However, this isn't an ideal way to alert users to the problem
- it might not occur to users to use an artificial dependency like this
- unsatisfied prerequisites can't be definitively identified as errors
  until/unless the workflow stalls
- the stall might not happen for some time, or at all, if other parts of the
  workflow can still run
- unsatisfied prerequisites are an indirect indicator: they do not single out
  the task (`a`) that actually caused the problem

So maybe we can (correctly) identify `a` as incomplete instead:

```
a:succeed(x|y) => c

# or in abbreviated success form:
a(x|y) => c

# or if direct triggering of c is not needed:
a(x|y)
```
This (or some variation on the suggested syntax) says that `a` is required to
succeed with at least one of `x` and `y` completed; otherwise it is incomplete
even if it reports success.

Note that the completion condition would not be appropriate if not getting
either output is actually a possible normal outcome of `a`. Or similarly
if the optional outputs belong to different tasks:
```
a => b1? & b2?
b1? | b2? => c  # join
a => c # artificial dependency?
```
If `c` really does depends only on `b1` or `b2` then presumably there is
nothing for it to do when they both fail. There are better ways of alerting the
user, if necessary, to that than spawning an unsatisfied prerequisite
downstream:
```
a => b1? & b2?
b1? | b2? => c
b1:fail? & b2:fail? => alert
```
