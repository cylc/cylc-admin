## Cylc Workshop
### 10-14 Feb 2020, NIWA, Wellington, NZ

#### Attendees:

- Hilary Oliver, NIWA
- Bruno Kinoshita, NIWA
- David Sutherland, NIWA
- Mel Hall, Met Office
- Tim Pillinger, Met Office
- Oliver Sanders, Met Office
- David Matthews, Met Office
TBC?:
- (Tim Whitcomb, NRL) 
- (Developer, BOM)

### Workshop Goals
- Primarily: review Cylc 8 progress to date and map out our roadmap to
  completion
- Secondarily: learn as much as we can about parts of the system mostly
  worked on by others, to gain understanding and enable future work

## Prelimary Agenda Topics

(TBD: collate with others' suggestions and make a timetable)

- architecture
  - review the new architecture
  - make sure we all know how to get the full system up and running
  - what still needs improving?
  - potential problems? (remote spawning, scaling, load balancing?)
  - support for CLI via the UI Server (e.g for task communication)

- Hub:
  - access to other users' workflows

- UI
  - review current status of tree view
    - what still needs improving?
  - other views: dot, graph, etc.
    - when to start on these?
  - common data store and subscriptions etc.
  - whiteboard/inkscape design session
      - Gantt view
      - Multi-workflows/dashboards
      - selecting sharing (drag n drop?)
      - other?

- Security
  - review BOM cylc-7 pen testing concerns
  - review BOM threat modeling notes
  - which J-Hub options should we be using
  - single users: e.g. should users be able to run the UIS standalone as
  - you can with notebooks?

- spawn-on-demand enhancement
  - POC implemented already, solves many problems
  - consequences (e.g. need n=1,2,3 workflow window)
  - for Cylc 8?

- platform changes

- config rationalization
  - potential user & global config changes (going back to the suggestions made
    during the aborted unified config work). We'll try to prepare some notes /
    a proposal in advance.

- rose suite-run migration and related changes including new "cylc run"
  semantics (name/run1,2,3... etc.)

- brief discussion on contingency planning for a delayed Rose2 release - New
  Python2 Rose release with rose suite-run etc stripped out for use with Cylc8.

- authorization
  - can we settle on an initial set of privilege levels?
  - implementation ideas

- practical debugging session
  - UI (using browser and Vue dev tools)
  - WFS (including subprocesses)
  - UIS (including async routines)
  - inspect network traffic between components

- coding: can we usefully work together on something, for several afternoons
  (say)? E.g.
  - an initial UI dot view?
  - Hub cross-user access
  - authorization

- deployment
  - versioning strategy (now we have many repos)
  - future of cylc wrapper / multi-version support?
  - minimal client install?
  - use of conda pack?
  - reducing the size of conda environments?
  - optional dependencies?
  - installing without conda?
  - reducing size of UI dist/ package
  - containers: how many docker files; use of docker compose; non-docker?

- component version compatibility
  - how should new versions deal with existing (running) WFS at older versions?

Finally: update the Gantt Chart through mid-2021
