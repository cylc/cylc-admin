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
- 

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

- Hub:
  - access to other users' workflows

- UI
  - review current status of tree view
    - what still needs improving?
  - other views: dot, graph, etc.
    - when to start on these?
  - common data store and subscriptions etc.

- spawn-on-demand enhancement
  - POC implemented already, solves many problems
  - consequences (e.g. need n=1,2,3 workflow window)
  - for Cylc 8?

- platform changes

- rose suite-run migration and related changes including new "cylc run"
  semantics (name/run1,2,3... etc.)

- authorization
  - can we settle on an initial set of privilege levels?
  - implementation ideas

- practical debugging session
  - how to debug the UI (with browser and Vue dev tools)
  - how to debug WFS and the UIS

- coding: can we usefully work together on something, for several afternoons
  (say)? E.g.
  - an initial UI dot view?
  - Hub cross-user access
  - authorization

Finally: update the Gantt Chart through mid-2021
