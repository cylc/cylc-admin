# Cylc VC 8/9 August 2019 Agenda

Previous: week-long meeting(s) in Exeter in June.

Present: HO, BK, DS, SB, OS, MS (others away, jet lagged, or ill).

## Actions (5 min)
- [May meeting actions](vc-22-may-2019-summary.md) - largely achieved
- [outstanding
  actions](https://cylc.github.io/cylc-admin/meetings/left-over-actions.html)

## General Admin (5 min)

- Decision made to use ZenHub, which provides a unified view of multiple repos:
  [Cylc and Rose Project Board](https://app.zenhub.com/workspaces/cylc-and-rose-5d122023f9628b5d0da532a5/board?repos=1836229)

## Site Updates and News (5 min)

- NIWA
   - NIWA/UM-Partnership contract for Bruno - still in prep
   - discussion started on Cylc development and "opportunities"
- Any new issues at other sites?

## 7.8 maintenance (2 min)

- nothing urgent; bumped
  [next-release](https://github.com/cylc/cylc-flow/milestone/80) till 31 Aug

## Cylc 8 Topics

We now have a workable end-to-end system.

### CLI-WFS Authentication (20 min)

[proposal PR](https://github.com/cylc/cylc-admin/pull/41)

SB:
- plans/thoughts so far
- what still needs to be decided
- implementation plan?
 
### UI Server and WFS (15 min)

DS:
- status update
- incremental data WFS data store update
- next steps?
  - incremental update to UIS
  - hook up CLI to GraphQL endpoint

### Orthogonal work (15 min)

- "rose suite-run" migration
- "cluster awareness"
- `flow.rc` config unification

TP (if present?) or MS: 
- status updates
- next steps

### Other Topics (5 min)

Nothing further done on:
- Packaging: 
  - once we have the first semi-usable system, start thinking about the total
  packaging solution for first users? 
- Hub:
  - have proved it works
  - need to document use with Cylc
  - and consider other-user display
 
### UI (10 min)

BK (if he has audio at home):
- status update
- next steps?

### AOB?
- next meeting - 2 weeks?
