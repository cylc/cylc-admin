# Cylc VC 8/9 August 2019 Summary Notes

(Notes written up 3 weeks late, so I may have missed a few things - anything
not written down at the time) (Agenda topics omitted here if nothing much arose
from the discussion).

Present: HO, BK, DS, SB, OS, MS (others away, jet lagged, or ill).

## Outstanding Actions

UI testing framework discussed
- may be able to start on this next month (BK)
- could use Travis CI for selenium, but getting screenshots back out is a problem
- writing and running the tests is easy, but integration and cross-browser
  testing is difficult

## General Admin

ZenHub:
- cross-project view should be useful
- will probably come into its own for unified milestones etc.
- We should play with it 

## CLI-WFS Authentication (20 min)

Decision: one-time token per command:
- client generates token
- then server uses it
- then client deletes it when done.

## UIS - WFS
- need encryption
- initial connection is authenticated just like clients (passphrase at the
  moment
- one-time token will be longer-lived, but that doesn't matter if comms are
  encrypted.

## UI
- lots of tree views in the UI, can they all use the same Vue component?
- if off-the-shelf not available, write or adapt our own one (note OS
  requirements checklist)
- we had decided to put task job data at lowest tree level (i.e. a pure tree
  view with no table aspect) but user feedback shows we still **need a table
  view**, e.g. to sort jobs by submit time
  - could we have a separate table view with no expand/collapse but fast
    filtering
  - or can we do both table and tree with the same component?
    - task job info (job ID, submit time etc.) at lowest level of tree OR in a
      table layout, by preference and/or screen width
    - should be feasible

## Packaging

- release numbers for UI, UIS need to be decoupled
- even if we started with same version numbers, they would inevitably get out
  of sync due to different release schedules and fixes
- we can still use unified milestones, just not based on version number

alpha-1 release:
- aim for end of August
- will a) show users; and b) help us to work on packaging:
- conda "super package"
  - conda recipe needs its own version?
- needs a command to list installed versions of each component
  - or can we just use conda commands for that?
- do we need incremental UIS update first?  Not really
- do we need subscriptions and web socket first?  Not really
- TODO: migrate UIS to websocket first, then subscriptions

## AOB
- OS noted JEDI (data assimilation) will support ECflow, Cylc, and Airflow, but
  they have unusual ideas about how to use the workflow engine within the DA
  system

## meeting schedule
- regular: first week of every month
- other: ad hoc, as needed
