# Proposal - Universal ID For Cylc "Objects"

## The Problem

At the moment we have multiple weird and wonderful ways of defining Cylc
"objects" to operate on:

* `[CYCLE-POINT-GLOB/]TASK-NAME-GLOB[:TASK-STATE]`
* `[CYCLE-POINT-GLOB/]FAMILY-NAME-GLOB[:TASK-STATE]`
* `TASK-NAME-GLOB[.CYCLE-POINT-GLOB][:TASK-STATE]`
* `FAMILY-NAME-GLOB[.CYCLE-POINT-GLOB][:TASK-STATE]`

With the GraphQL schema came the requirement for a global identifier
which added yet another format:

* `[user[|]]workflow[|cycle[|(namespace)[|job]]]`

## Motivating Factors For This Proposal

1) The global identifier is nicer and more flexible than the command line
   options.
2) It is not possible to specify jobs with any of the CLI formats.
   https://github.com/cylc/cylc-flow/pull/3880
3) The [rose suite-run migration proposal](https://github.com/cylc/cylc-admin/blob/master/docs/proposal-rose-suite-run.md#cli-changes)
   suggests converting the `REG` (read `FLOW`) arguments into options i.e:

   ```console
   $ cylc stop --flow my_workflow
   ```

   In order to make the edge where users wish to operate on a workflow and
   are already, conveniently, in the run directory of that workflow slightly
   nicer:

   ```console
   $ cd ~/cylc-run/my_workflow
   $ cylc stop
   ```
4) The UI currently doesn't make use of any form of identifier, however,
   it may be beneficial to expose users to the global identifier e.g.
   for mutations
5) The CLI is now implemented via GraphQL anyway meaning that the CLI arguments
   are provided as GraphQL arguments and transformed as required.
6) We should expose users to the concept of the identifier in the UI.

## Proposed Universal Identifier

resolves https://github.com/cylc/cylc-flow/issues/3592

* Use `/` as the separator (as originally intended) but use `//` to separate
  the flow from the cycle.
* Permit globs in each part of the id.
* Support `:<arg>` as a special state selector for convenience in each section
  other than "user".
  (i.e. a direct replacement for the current `cylc trigger flow *:failed`
  syntax).

```bash
# regular form (for identifying objects)
[~user[/]]flow//cycle/task/job

# expanded form with selectors (for quick querying of objects)
[~user[/]]flow[:state]//cycle[:state]/task[:state]/job[:state]
```

Examples (equivalent GraphQL arguments under current scheme)

```bash
# flow under account user (wid="user|flow")
~user/flow

# the task foo at all cycles in flow (wid="flow|*|foo")
flow//*/foo

# all tasks in cycles between (exclusive) 202009 202011 in flow
# (wid="flow|202010*|*)
flow//202010*/*

# all tasks begining foo in flow (wid="flow|*|foo*)
flow//*/foo*

# all running workflows (wid="*", state="running")
*:running

# all cycles in foo with at least one running task
# (wid="flow|*", state="running")
flow//*:running

# all running tasks in foo (wid="flow|*|*", state="running")
flow//*/*:running
```

> **Note:** All syntax implemented here represents functionality currently
  supported in the GraphQL schema.

> **Note:** Currently in GraphQL `state` is a separate argument, there is no
  need to change that in GraphQL though it could be done for consistency if
  desired.

Reference regex:

https://regex101.com/r/XOOpUW/2

```regex
^
(?:
  ~(?P<user>)[^\/]+
  \/?
)?
(?:
  (?P<flow>.*?(?=(?:\/\/|$|:)))
  (?P<flow_sel>.*?(?=(?:\/\/|$)))
  (?:\/\/
    (?:  
      (?P<cycle>[^\/:]+)
      (?::(?P<cycle_sel>[^\/]+))?
      (?:
        \/
        (?P<namespace>[^\/:]+)
        (?::(?P<namespace_sel>[^\/]+))?
        (?:
          \/
          (?P<job>[^\/:]+)
          (?::(?P<job_sel>[^\/]+))?
        )?
      )?
    )?
  )?
)?
$
```

## Proposed Changes

### The Basic Outline

We adapt the CLI to use this identifier in place of the following arguments:

* `REG`
* `TASK_NAME`
* `TASK-ID`
* `TASK_GLOB`
* `TASK-GLOB`
* `TASK-JOB`

Examples:

```bash
cylc stop OPTIONS ID
cylc stop my_workflow
# currently: cylc stop my_workflow
cylc stop my_workflow//123
# currently: cylc stop my_workflow 123
cylc stop my_workflow//123/task
# currently: cylc stop my_workflow 123/task (or task.123)

cylc trigger OPTIONS ID
cylc trigger my_workflow//123/task
# currently: cylc stop my_workflow 123/task (or task.123)
cylc trigger my_workflow//123/*:failed
# currently: cylc stop my_workflow 123/*:failed
```

Benefits:

* Unified universal ID which supports all current CLI syntax.
* Makes use of the great functionality that we have already implemented!
* Very little change from Cylc7, just add `//` between the `REG` and
  `TASK_GLOB` arguments.
* Exposes the full power of GraphQL to the user.
* Enables multi-workflow control from the CLI via the UIS without complicating
  Cylc Flow.
* Enables multi-user control from the CLI via the UIS without Cylc Flow having
  to know anything about authorisation.
* Provides the golden `cylc stop '*'`.

### Bash Completion

Propose a new enhanced (and ridiculously simple) bash completion as a solution
to the *"I would like to run Cylc commands for my workflow without having to
type the `REG`"* use cases.

Reference Implementation:

> **Note:** This is psudocode to illustrate where the bash autocompletion
  would get its suggestions from.

```
# 𝄄 = the users cursor

# autocomplete command name
$ cylc 𝄄
echo $CMD_LIST  # can be built during pip install (would pick up plugins)

# autocomplete flow (REG)
$ cylc trigger 𝄄
if [[ $1 == cylc && trigger == $FLOW_CMD_LIST ]]; then
    # the CLI is expecting a REG
    if [[ -d '.service' ]]; then
        # we are in the run dir
        echo $(basename "$PWD")
    elif [[ -d 'flow.cylc' ]]; then
        # we are in the src dir
        echo $(basename "$PWD")
    else
        # provide a list of workflows
        cylc scan --state=all --format=name
    fi
fi

# autocomplete cycle
$ cylc trigger foo// 𝄄
echo ~/cylc-run/foo/log/job

# autocomplete task
$ cylc trigger foo//123 𝄄
echo ~/cylc-run/foo/log/job/123 | reverse  # newest cycle points first

# works with globs too
$ cylc trigger foo//1* 𝄄
echo ~/cylc-run/foo/log/job/1* | reverse  # newest cycle points first

# autocomplete job
$ cylc trigger foo//123/foo 𝄄
echo ~/cylc-run/foo/log/job/123/foo

# autocomplete flow (REG) for another user
$ cylc trigger ~user/ 𝄄
cylc scan --state=all --format=name --dir=~user/cylc-run
# note the --dir arg would be a two line change to add
```

Benefits:

* Easy to issue commands form the run dir, press tab to autocomplete the `REG`.
* Also easy to issue commands from the src dir, again just press tab.
* Provides autocompletion for cycles/tasks/jobs in the n=0 window and
  from historical execution.
* Cylc commands look the same wherever they are run from making it clear
  what the command is doing.
  All operations are explicit with the name of the workflow
  appearing in the command before issue.

Motivations:

* We need to rewrite the completions anyway.
* Typing cycle points is hard.

## Proposed Implementation Timeline

* 8.0b0 - `cylc install`
* 8.0b1 - CLI via the UIS (if running)
  - Route commands via the UIS where available as per previous proposals.
  - If the workflow ID is a glob attempt to route via the UIS else fail.
  - If a user is specified (and isn't `$USER`) then attempt to route
    via the UIS else fail.
* 8.0bX - universal id
  - Move to universal id in later beta release.
  - Conversion simple in the code thanks to the conversion of the CLI to
    GraphQL.
  - Upgrading the tests and docs more time consuming.
* 8.0.0 - bash autocompletion
  - Remove all remaining `task.cycle`, `cycle/task` references.
* 8.0.0 - bash autocompletion
  - Not required for functionality
  - Aim to make the 8.0.0 release to aid transfer from Rose
* 8.1.0 - Support multi-workflow functionality NOT via the UIS
  - Support workflow globs without the need to go via the UIS.
  - Pretty simple to implement, can be done earlier.
