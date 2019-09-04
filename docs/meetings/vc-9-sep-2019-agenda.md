# Cylc VC 9 September 2019 Agenda

## Previous meeting
9 August
- [agenda](vc-9-aug-agenda.md)
- [notes](vc-9-aug-summary.md)

## Previous Actions (5 min)
- (see 9 Aug notes)
- [outstanding
  actions](https://cylc.github.io/cylc-admin/meetings/left-over-actions.html)

## General Admin (5 min)

- Start using ZenHub, for unified milestones etc.
  [Cylc and Rose Project Board](https://app.zenhub.com/workspaces/cylc-and-rose-5d122023f9628b5d0da532a5/board?repos=1836229)

## Site Updates and News (5 min)

- NIWA
   - NIWA/UM-Partnership contract for Bruno - still in prep
   - discussion started on Cylc development and opportunities - 12 Sept
- BOM?
- UK?

## 7.8 maintenance (2 min)

- [7.8.4 release](https://github.com/cylc/cylc-flow/milestone/80)
- any issues reported?

## Packaging and ALPHA-1 RELEASE

- can we put out a "full system" conda recipe?
- what needs doing?
  - cylc-flow and cylc-uiserver on pypi
  - cylc-ui?
  - version naming?
    - need an uber-project name and version number?
- what do we want to get into the release
  - a funtioning tree view with job data available - yes?
  - incremental UIS udpate - no?
  - websocket and subscriptions - no?

## Cylc 8 Topics

Discuss any progress or problems on the following topics:

- N-distance queries and "hybrid mode" (HO, DS)
  - DS has implemented already
  - shall we use this in the UI?
    - any major concerns or gotchas?
    - if not, let's do it!  (at least as a UI option)
    - implications:
      - a more sensible and more concise set of tasks for the UI
        (c.f. all the waiting and succeeded tasks in the task pool)
      - requires auto-insertion for operations on ghost nodes (easy)
      - instead of scissor nodes, several (not many?) disjoint graphs
      - other?

- Task State Naming and Icons (HO)
  - we tentatively decided to:
    - absorb "queued" and "ready" into a single "submitting" state in the UI
      (icon: a circle, with a smaller central dot that "submitted")
    - and rename "ready" to "preparing" in the back end
  - BUT we probably need to go back to exposing internal queues to the user
    - so what names? (queued and preparing in UI and server?)
    - what icons?
 
- UI Tree View issues (BK)
  - how to display task data (Job ID and batch system etc.)?
    - need tabular form (esp. desktop) and tree collapse (esp. mobile)
    - can the one component do it all "responsively", or do we need two
      different components (pure tree, and tabular)?
    - do we need a temporary solution (e.g. pure tree) for alpha-1 release?

- UI Graph View (MR)
  - progress, problems, or thoughts on:
    - use of our common icons
    - hook-up to live workflow data
    - layout and display of job icons?
  - anything else?
  - any changes needed to conform to UI project directory structure (location
    of components etc?) - BK 

- "rose suite-run" migration (TP, MS)

- CLI-WFS Authentication (SB)
 
- UIS incremental update (DS)
  - thoughts on how to implement

- Websocket and subscriptions (DS, BK)
  - the Tornado PR etc.

- ANYTHING ELSE?


## AOB?
- next meeting - first week of October
