# Cylc VC 9 September 2019 Summary Notes

(all present except for SB)

## Site Updates and News

__Met Office__
- deployed 7.8.4 as `cylc-next` for testers.

__NIWA__
- NIWA/UM-Partnership contract for Bruno - still in process with lawyers
- discussion on Cylc development funding and opportunities: 12 Sept
- cylc-7 on Spectrum Scale (GPFS) Maui HPC: "unable to open db file"
  frequent-ish cause of suite shutdown; may be filesystem error; looking into
  it; try strace

__Packaging and ALPHA-1 RELEASE__

- "full system" alpha-1 release by end of month, needed for Met Office team
  key deliverable
- Need:
  - a funtioning tree view with job data available 
- Omit from alpha-1?:
  - n-distance window
  - incremental UIS update from WFS
- Maybe:
  - websocket and subscriptions - maybe?

- conda forge: upload may require approval, get everything working locally
  first
- however conda can have "latest build"
- versioning:
  - conda package "cylc-8.0a1" includes:
  - cylc-flow-8.0a1
  - cylc-ui-0.1        (move to 1.0 with cylc-8 release)
  - cylc-uiserver-0.1  (ditto)
- cylc-flow and cylc-uiserver on pypi
- OS: full package needs some kind of command to list cylc component versions
  installed
  - for now just document "conda list | grep cylc"; maybe improve later

__UI Table view__
- table still needed by some users
- agreed can have a separate filterable, sortable detailed "table view" post
  alpha
  - (or *maybe* same as part of tree view but off by default)
  - easy with off-the-shelf components, unlike the tree view
  - call it the "job status" view?
  - on back-burner milestone for now

__UI Gantt Chart View__
- DS: easily spot down-times in the schedule, and bottle-necks
- backburner for post Cylc 8

__cylc-flow.rc__
- TP: coming along well but not hooked up/functional yet

__Task status naming__
- HO: do not group "queued" and "ready" into "submitting" as "queueud" is
  important to users (they configure the queues)
- also: "queued" never fails, but "ready" can in many ways
- change name "ready" to "preparing"
- MS: "queued" is more like "waiting"
- OS: represent queue "dependence" like xtriggers, in the graph?
- DS: eventually cross-suite triggers can bridge the gap in the UI

__N-distance queries and "hybrid mode"__

- try to do it for cylc-8
- may be some edge cases (what to display)

__UI Graph View__

- MR still has issues with polling, subscriptions, and dynamic styling of
  nodes, which we need to get to the bottom of as soon as we can (HO: polling
  was always just a temporary bridge to get us to websockets while waiting on
  Tornado PR)
- HO: need common styling and data use across views
- (Did not discuss: component directly structure needs to conform to that of
  the tree view and other components)

__ANYTHING ELSE?__

- meeting schedule: first Mon/Tues every month
- UK team announcements from Dave and/or Matt:
  - Matt moving on with the Met Office end of this month :-(
