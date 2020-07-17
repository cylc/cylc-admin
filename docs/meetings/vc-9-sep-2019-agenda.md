# Cylc VC 9 September 2019 Agenda

## Previous meeting
9 August
- [agenda](vc-9-aug-2019-agenda.html)
- [notes](vc-9-aug-2019-summary.html)

## Previous Actions - 5 min
- (see 9 Aug notes)
- [outstanding
  actions](https://cylc.github.io/cylc-admin/meetings/left-over-actions.html)

## General Admin - 5 min

- Start using ZenHub, for unified milestones etc.
  [Cylc and Rose Project Board](https://app.zenhub.com/workspaces/cylc-and-rose-5d122023f9628b5d0da532a5/board?repos=1836229)

## Site Updates and News - 5 min

- NIWA
   - NIWA/UM-Partnership contract for Bruno - still in prep
   - discussion started on Cylc development and opportunities - 12 Sept
- BOM?
- UK?

## 7.8 maintenance - 2 min

- [7.8.4 release](https://github.com/cylc/cylc-flow/milestone/80)
- any issues reported?

## Cylc 8 Topics

Any progress or problems on the following topics:

__Packaging and ALPHA-1 RELEASE - 10 min__

- can we put out a "full system" conda recipe?
- what needs doing?
  - cylc-flow and cylc-uiserver on pypi
  - cylc-ui?
  - version naming?
    - need an uber-project name and version number?
- what do we want to get into the release
  - a functioning tree view with job data available - yes?
  - incremental UIS update - no?
  - websocket and subscriptions - no?


__N-distance queries and "hybrid mode" (HO, DS) - 10 min__

DS has implemented already; shall we use this in the UI?
- any major concerns or gotchas?
- if not, let's do it!  (at least as a UI option)
- implications:
  - a more sensible and more concise set of tasks for the UI
    (c.f. all the waiting and succeeded tasks in the task pool)
  - requires auto-insertion for operations on ghost nodes (easy)
  - instead of scissor nodes, several (not many?) disjoint graphs
  - other?

__Task State Naming and Icons (HO)  - 5 min__
we tentatively decided to:
- absorb "queued" and "ready" into a single "submitting" state in the UI
  (icon: a circle, with a smaller central dot that "submitted")
- and rename "ready" to "preparing" in the back end
BUT we probably need to go back to exposing internal queues to the user
- so what names? (queued and preparing in UI and server?)
- what icons?
 
__UI Tree View issues (BK)  - 10 min__
how to display task data (Job ID and batch system etc.)?
- need tabular form (esp. desktop) and tree collapse (esp. mobile)
- can the one component do it all "responsively", or do we need two
  different components (pure tree, and tabular)?
- do we need a temporary solution (e.g. pure tree) for alpha-1 release?

__UI Graph View (MR)  - 10 min__

progress, problems, or thoughts on:
- use of our common icons
- hook-up to live workflow data
- layout and display of job icons?
anything else?
any changes needed to conform to UI project directory structure (location
of components etc?) - BK 

__"rose suite-run" migration (TP, MS) - 5 min__

__CLI-WFS Authentication (SB) - 2 min__
 
__UIS incremental update (DS)  - 5 min__
  - thoughts on how to implement

__Websocket and subscriptions (DS, BK)  - 5 min__
  - the Tornado PR etc.

__ANYTHING ELSE?__

## AOB?
- meeting schedule: first Mon/Tues every month
- UK team announcements from Dave and/or Matt
