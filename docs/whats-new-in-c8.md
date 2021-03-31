# What's New in Cylc 8

#### Cylc-8.0b0 beta preview release
**1 April 2021**

Cylc 8 is different from Cylc 7 in many ways, but it can run Cylc 7
workflows out of the box (please take note of deprecation warnings though).

## Terminology

- *Suite* is now **workflow**
  - (a much more widely used and understood term)
- *suite daemon* (or *suite server program*) is now **scheduler**
  - (ditto)
- *batch system* is now **job runner**
  - (our job runners are not necessarily "batch systems")

## Configuration Filename

- `suite.rc` has been deprecated to `flow.cylc` (see "Terminology" above)

## Architecture

Cylc has been re-architected to support a remote web UI:
- a new Hub component, where you authenticate
  - (can be run by a regular user or privileged user)
- a new UI Server component that runs as the user
- a new UI that runs in your web browser
- new network layers to efficiently feed data from schedulers and
  filesystem, through the UI Server, to the UI

## Web UI

The new web UI features:
- a useful front dashboard page
- integrated gscan side-panel
- responsive design (from desktop to table to mobile)
- a tabbed interface to display multiple workflow views
- full command integration, for interaction with task, jobs, and schedulers

## Scheduling Algorithm

Managing infinite workflows composed of repeating tasks is difficult.

The Cylc 7 used a *"spawn-on-submit"* model had some downsides (see below).

Cylc 8 has a new *"spawn-on-demand"* scheduler that:
- only has to be aware of current active tasks, no matter how big the workflow
- does not need suicide triggers for alternate path branching
- can run tasks out of cycle point order
- enables a sensible active task based view of the workflow

It also supports a new feature called **reflow**: you can trigger multiple
"wavefronts" of activity in the same workflow graph at the same time.

## Task/Job Separation

A "task" represents a node in your abstract workflow graph. A job is a real
process that runs on a computer. Tasks can submit multiple jobs to run, through
automatic retries or manual re-triggering.

The Cylc 7 GUI showed tasks, with job data from just the latest job.

The Cylc 8 UI shows both task and jobs. Tasks are circles, jobs are squares.

## Fewer Task States

Cylc 7 had 13 task/job states, and increasing. In Cylc 8, Task/Job separation
(above) means we can use just 8 states. For example, a *retrying* task in Cylc
8 just has the plain *waiting* state (it is waiting on a clock trigger) but
you can see it will be a retry because it already has finished jobs.

The simpler complete list of essential task/job states is now:
- *waiting* - (task only) - not ready to run yet, waiting on upstream task
  prerequisites, or clock triggers, or xtriggers, or queue limiting, or runhead
  limiting
- *preparing* - (task only) - ready to run, preparing for job submission
- *submitted* - (task/job) - job submitted but not yet running
- *submit-failed* - (task/job) - job submission failed
- *running* - (task/job) - job executing
- *succeeded* - (task/job) - job finished successfully
- *failed* - (task/job) job - finished unsucessfully 
- *expired* - (task only) - job will not be submitted

## Platform Awareness

Cylc 7 was aware of individual job hosts.

Cylc 8 is aware of groups of hosts, called *platforms*, specified in global
configuration, that can be treated equally in job management terms. E.g.
if a job submission host goes down Cylc 8 can use other hosts on the
platform to interact with jobs.

## Workflow Installation, Separation of Workflow Source and Run Directories

The functionality of `rose suite-run` has been migrated into Cylc 8.

- `cylc install` copies all workflow source files into a dedicated
  run-directory (see also...)
- (~~`cylc register`~~ is no longer neeeded)
- by default each new install creates a new numbered run-directory:
  `<workflow>/run1, <workflow>/run2, ...`

## New Safe Run Semantics

In Cylc 7 the default workflow run semantics were dangerous. If you
accidentally typed `cylc run` instead of `cylc restart`, a new cold-start run
would blithely overwrite your existing run directory and destroy your workflow
state so that the desired restart was no longer possible.

In Cylc 8, we use `cylc play` to start, restart, or unpause a workflow. If you
want to do a new run from scratch (a "cold start") do a fresh install and run
it safely in the new incremented run directory.

## Better Security

TBD...

# Appendices

## Long-standing Scheduling Deficiencies Fixed by Cylc 8

- every task had an implicit depedence on the job submission event of its
  own previous-instance (i.e. same task, previous cycle point)
- the scheduler had to be aware of at least one active and one waiting instance
  of every task in the workflow, plus all succeeded tasks in the current
  active task window
- to fully understand what tasks the GUI showed (e.g. why particular *waiting*
  or *succeeded* tasks appeared in some cycles but not in others) you had to
  understand the scheduling algorithm
- *suicide triggers* were needed to clear unused graph paths to avoid stalling
  the scheduler
- tasks could not run out of cycle point order, and workflows could stall
  due to next-cycle-point tasks not being spawned downstream of failed tasks
