# Cylc Project VC 7 July 2020

*(Meeting notes in italics)*

## Admin

Cylc-7

- Need to release 7.9.2/7.8.7? https://github.com/cylc/cylc-flow/issues/3664 

Cylc-8.0a2
- UI and conda meta-package still to do? 

*TBD soon as possible:*
- *Need UI and UIS releases*
- *Builds on conda can be incremented and tested pre-release*
- *Add OS as "admin" to UIS*

- *Remove MS as contact on security.md (cylc-flow)*

Current cylc-flow PRs
- quick triage?

*Job script subshell PR: we should get this in soon, but DM needs to review and
is busy; OS to help TT rebase if necessary*

BOM
- HO had a catch-up meeting with JR; she should be able to join us soon :tada:

*JR to spin up slowly over next 2 weeks*

## Cylc 8 MVP
- [Development timeline from the mid-Feb
  workshop](https://cylc.github.io/cylc-admin/feb2020-workshop-report#tentative-development-timeline)
- We really need to:
  - delta tree in? [AlMOST]
  - Spawn-on-demand in [ALMOST]
  - platforms in [ALMOST]
  - infinite tree done [BK hopes pretty easy?]
  - THEN focus entirely on the ESSENTIALS for 8.0

## Spawn on Demand

HO
- Had a meeting on this DM, OS, HO
- Latest proposal and implementation agreed
- Functional tests for reflow added
- NOW: hundreds of conflicts to resolve post integration tests merge :grimace:

- *SOD needs merge not rebase, so we can review the conflict resolution*
- *need to think about "flow hold" -what exactly does that mean?*

## Back-end: delta updates

DS on leave (may attend?), but:
- delta PRs now merged
- next: post-SoD n-distance windowing

*(discussed and agreed)*
 
## UI

BK
- data model "essentials" clarified and agreed
- delta tree status?
- next: infinite tree

*(discussed and agreed)*

## Platforms and suite installation etc.

UK team
- Initial platforms PR ready to go?
- `rose suite-run` work underway or about to start? 
- other?

*(discussed and agreed)*

*RD on isodatetime improvements: classes mutable -> hashing -(next)-> caching*

## AOB?

- Any side-meetings needed?

## Next Meeting?

- TP to arrange for early August?
