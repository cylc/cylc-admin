# Cylc Project VC 5 May 2020

*(Meeting notes in italics)*

## Admin

Cylc-7
- 7.8.6 and 7.9.1 released (bugs: cycle point format conversion; spurious
  warnings on restart)

Cylc-8.0a2
- Need to make the release before platforms changes go in
- Agreed last meeting to wait for data-store push on delta updates, but those
  are still not in 
  - Relitigate that decision, or wait just a bit longer?
- *Release 8.0a2 this month*
- *alpha-3 release should be SoD and n=1 window*

Cylc-flow now using GH Actions for CI


## Cylc 8 MVP
- [Development timeline from the mid-Feb
  workshop](https://cylc.github.io/cylc-admin/feb2020-workshop-report#tentative-development-timeline)
- We really need to:
  - get delta PRs in
  - get Spawn-on-demand in
  - get performant tree view done
  - get platforms in
  - THEN focus entirely on the ESSENTIALS for 8.0


*We are probably at least a month behind schedule, partly due to the pandemic,
partly unexpected difficulties.*

## Spawn on Demand

HO
- Simplification on realizing I can avoid keeping finished tasks to prevent
  conditional reflow, without any additional DB access:
  - No need for "parents-finished" housekeeping of finished tasks
  - No need for spawn-on-completion, which also requires "parents-finished"
    housekeeping of additional waiting tasks generated
  - (There is still potential for parents-finished housekeeping of waiting
    tasks downstream of AND triggers, but let's consider nicer ways of doing it
    later - for now, at worst it remains a niche case for suicide triggers -
    much less than in SoS)

- Still updating the document damn it, almost done. Some details are difficult
  to nut out because there isn't an obvious right answer - we just have to
  decide which way to go and document the reasons and consequences ... but it
  is all better than SoS!  Example: should we treat handled and not-handled
  failed tasks differently.

Current status:
- All supported functionality works (all test passing on CI)
- All advertised advantages have proved to be true
- New functionality seems to work nicely: trigger with and without reflow, flow
  merge, flow stop
  - maybe not completely done, but we can refine *new functionality* later
 
- *DM: Keep agree fail handling for now*
- *Need a follow-up meeting with DM and OS to go through the details*

## Back-end: delta updates

DS 

- a few changes since last month, but mainly just waiting on HO review (sorry)
- (OS and BK have had a good poke around - what's the verdict?)
 
 *Ready to be merged*

## UI

BK

- new tree built from deltas, what've we learned from that?

- *Need a follow-up meeting on UI data model issues*
- *OS: integration of control functionality may take ~1 week if we're lucky*
- *Could punt fine-grained auth and UI tweaks till later if we have to*

## Platforms etc.

(UK team)
- *TP: platforms changes very close - 1 week? test batter has been trouble; may
  not be fully functional at first merge, e.g. cylc-7 compat*
- *RD: isodatetime efficiency changes (OS: lru_cache; then make data classes
  hashable and therefore cacheable*
- *MH stripping out `cylc submit` and `cylc jobscript` and tests that use
  them*
- *key generation changes are good to go (private keys don't travel now)*
- *next: suite installation changes*


## AOB?
- *BOM: will ask JR if she will join us soon*

## Next Meeting?

- TP to arrange for early July?
- *July 7*
