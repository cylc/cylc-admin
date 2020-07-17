# Cylc Project VC 5 May 2020

*(Meeting notes in italics)*

## Admin

Cylc-7
- 7.8.5 and 7.9.0 released
  - (7.9.0 by copying 7.8.5 and cherry-picking the Jinja2 patch)

Notes:
- *UK team to check how `task not found, skip` warnings affect large suite
  restart - this may warrant another quick cylc-7 release*
 
Cylc-8.0a2
- Conda Forge ready to go
- BK's release HowTo (for devs) merged to cylc-admin
- BK's user instructions merged to cylc-admin
  - To be posted to the web site and Discourse 
- (Make the release before Platforms changes go in - any day now)

Notes:
- *Agreed we may want to release a2 with data-store push on delta updates
  (instead of five second intervals) after performance and memory leak tests*

## Spawn on Demand

HO
- Absolute dependence solved and implemented
- Proposal document finished and comprehensive
- All unit and functional tests passing on Travis
- PR branch ready for review
   - HO: what remaining details need polishing?
   - Reviewers (OS? DS? others?)

Notes:
- *~~HO to update PR with remaining TODO items~~ DONE*
- *~~HO to update proposal UI section: need to show users outputs that have been
  "reset" by the spawn command~~ DONE*

## Back-end: delta updates

Update from DS 

- almost done?
- recent Protobuf issue with deltas and scalar default

Notes:
- *more to do, e.g. delta counters, but can be follow-up PRs*
- *OS to start testing as a priority*
 
## UI

Update from BK

- latest thoughts on use of deltas and implications for reactivity etc.

Notes:
- *some unsolved issues to do with Vuex store and reactivity, with deltas*
- *OS will prioritise helping with this stuff once the deltas are done*

## Platforms etc.

(UK team)
- latest on progress, any road-blocks

Notes:
- *TP has made progress on Platforms but is currently moving house*
- *OS auto-doc PRs are up, import to have these before the big config changes*
    - *sections replaced with meta-variables*
- *OS auto build and deployment of cylc-docs by GH Actions*
    - *agreed to make ~5 previous releases including 7.8.5, plus latest doc
      build, available
    - will just link to cylc-doc GH Pages from the web site*
- *MH working on generation of client keys on remote hosts*
- *... and restoring the old "ssh messaging" functionality*
- *RD syntax wizardry for Cylc in VSCode, PyCharm etc., done and announced*
- *MH next priority: remote file installation and symlinks, for `rose suite-run` migration*
  - *HO TODO: there's an outstanding PR on cylc-admin for config changes*

## Misc

- GH Actions good to go? (What about coverage?)

Notes:
- *still a couple of test failures*
- *coverage was broken by moving re-invocation of sub-commands from bash to Python!*
   - may need to use click or similar
- *Agreed to get this in quickly even if we need to fix a few tests later on*

- *OS portable `getfqdn` needed for Cylc portability - will post issue*

## Next Meeting

Notes:
- *TP to arrange for early June*
- *We should look at the timeline decided at the workshop*
