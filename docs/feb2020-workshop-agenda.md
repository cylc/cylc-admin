## Cylc Workshop
### 10-14 Feb 2020, NIWA, Wellington, NZ

#### Attendees:

Hilary Oliver, NIWA
Bruno Kinoshita, NIWA
David Sutherland, NIWA
Mel Hall, Met Office
Tim Pillinger, Met Office
Oliver Sanders, Met Office
David Matthews, Met Office

**Primary purpose:** review Cylc 8 progress to date and map out our roadmap to
completion

**Secondary purpose:** learning as much as we can about parts of the system
mostly worked on by others, to gain understanding and enable future work

## Prelimary Agenda Topics

(TBD: collate with others' suggestions and make a timetable)

- architecture
  - review the new architecture
  - make sure we all know how to get the full system up and running
  - what still needs improving?
  - potential problems? (remote spawning, scaling, load balancing?)

- Hub:
  - multi-user aspects

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

- rose suite-run migration and related changes
  - including new "cylc run" semantics (name/run1,2,3... etc.)

- practical debugging session
  - how to debug the UI (with browser and Vue dev tools)
  - how to debug WFS and the UIS

- coding, if time allows
  - can we usefully work together on something?
    (e.g. an initial UI dot view?)

- authorization
  - can we specify an initial set of access levels?
  - implementation ideas

Finally: Gantt Chart
