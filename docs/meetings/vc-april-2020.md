# Cylc Project VC 2 April 2020

(Extenuating circumstances: everyone attending from home, Covid-19 lockdown
just instituted over much of the world :grimace:)

## 7.8.5

- We should release this soon, just get MH's PRs in first.
- HO: on the mysterious orphaned job log tail processes on job hosts, it seems
  to be sufficient to configure cylc to run `tail` via the `timeout` command
  (tested). DM may test at Met Office.
  - Need to document this!

## 7.8.x and 7.9.x

- instead of continuing to duplicate PRs to the 7.9.x branch, which is mildly
  painful, we should just delete 7.9.x and re-branch from 7.8.x after changes
  (there shouldn't be many more changes on 7.x) and cherry-pick the Jinja2
  upgrade commit.

## 8.0a2

- Conda Forge setup done, pretty much good to go
    - Just need to update the meta data in the conda recipe
    - BK to document the release process clearly, and make sure that others are
      recorded as maintainers on the conda repos.
- Get it done before the Platforms changes go in
- How to document use (and meaning of alpha status) for users who install by
  conda?
  - Cylc web site and Discourse 

## Spawn on Demand

- HO has all unit tests and most of the test battery working on the SoD branch
  - time consuming due to many tests relying on SOS semantics and task pool
    content.
- Last(??) major unsolved problem is absolute dependence.
  - HO to document this and possible solutions on the SoD proposal doc
- Need to make a decision on this pretty soon.
  - OS agreed if reflow is fine and all test battery functionality is covered
    there's no obvious reason not to go ahead with this.

## Misc

- OS good idea to simplify the test battery by providing a utility to fake the
  DB task pool table to any state, and take the scheduler there directly via
  restart.
- Some discussion on a "universal delimiter" for suite/task IDs.  DS's new ID
  form is good (hierarchy) but can't be used on the command line (the delimiter
  is the shell pipe character).  More thought needed.
- Agreed to get rid of the "task owner" concept in Cylc, used to run jobs on
  other user accounts
  - better to use ssh config for this (need to document for users)
- Agreed to move cylc-flow to GH actions soon

## Next meeting?

Early May (ish).

