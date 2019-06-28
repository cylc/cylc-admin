# cylc-8 Implementation Task List

### Major Updates and Changes
 - Initially compiled after the December 2018 Development Workshop.
 - Updated 25 Feb 2019: added Gantt Chart.

## Project Gantt Chart

### Key
- `.` - in progress
- `o` - estimated completion date (o1 deployable; o2 celebrate)
- `ooo` - ongoing beyond original estimated completion date
- `x` - done (actual completion date)
- `?` - unsure of status

### Ganntt Chart
(Refer below the Chart for details on some items).

```
o-------------------------------o-----------------------------------------------------------o-------------o
 C Y L C - 8 - T A S K S        | 2019                                                      | 2020
o-------------------------------o-----------------------------------------------------------o-------------o
DOCUMENTATION                   *JAN. FEB. MAR. APR. MAY. JUN. JUL. AUG. SEP. OCT. NOV. DEC.*JAN.| /  SEP. 
  Latex -> Sphinx               |...x|    |    |    |    |    |    |    |    |    |    |    |    | /      |
o-------------------------------o-----------------------------------------------------------o-------------o
CYLC TEST COVERAGE              *JAN. FEB. MAR. APR. MAY. JUN. JUL. AUG. SEP. OCT. NOV. DEC.*JAN.| /  SEP.  
  Reporting                     |....|x   |    |    |    |    |    |    |    |    |    |    |    | \      |
  Coverage > 80%                |  ..|..x |    |    |    |    |    |    |    |    |    |    |    | /      |
o-------------------------------o-----------------------------------------------------------o-------------o
ISODATETIME LIBRARY             *JAN. FEB. MAR. APR. MAY. JUN. JUL. AUG. SEP. OCT. NOV. DEC.*JAN.| /  SEP. 
  Python 3                      |...x|    |    |    |    |    |    |    |    |    |    |    |    | \      |
  packaging, PyPi               |  ..|.x  |    |    |    |    |    |    |    |    |    |    |    | /      |
o-------------------------------o-----------------------------------------------------------o-------------o
WORKFLOW SERVER (WFS) and CLI   *JAN. FEB. MAR. APR. MAY. JUN. JUL. AUG. SEP. OCT. NOV. DEC.*JAN.| /  SEP.  
  Python 3                      |   .|....|...x|    |    |    |    |    |    |    |    |    |    | \      | 
  Cherrypy -> ZeroMQ            |   .|....|...x|    |    |    |    |    |    |    |    |    |    | /      | 
  Command-line clients          |    |    |  ..|...o| ?  |    |    |    |    |    |    |    |    | \      | 
    old passphrase authe        |    |    |  .x|    |    |    |    |    |    |    |    |    |    | /      | 
    one-time token authe        |    |    |    |    |    |    |    |    |....|..? |    |    |    | \      |
    GraphQL API (rm REST endpts)|    |    |    |    |    |....|....|.?  |    |    |    |    |    | \      |
    session mgmt (CLI via Hub)  |    |    |    |    |    |    |    |    |    |    |    |...?|    | /      | (not critical)
  WS: ZeroMQ                    |    |    |    |    |   .|..o |    |    |    |    |    |    |    | /      |
  UI SERVER API                 |    |    |    |    |    |    |    |    |    |    |    |    |    | \      |
    design new data model       |    |    |    |  ..|...o|ooox|    |    |    |    |    |    |    | \      |
    serve new data model        |    |    |    |    | ...|...x|    |    |    |    |    |    |    | /      |
  Async event loop              |    |    |    |    |    |    |    |    |    |    |    |    |..? | \      | (not essential)
  PYTHON PACKAGING              |    |    |    |    |    |    |    |    |    |    |    |    |    | \      |
    pip, setup.py               |    |    |...o|oooo|x   |    |    |    |    |    |    |    |    | /      |
    Cylc available on PyPi      |    |    |   .| ooo|x   |    |    |    |    |    |    |    |    | \      |
    (N) namespacing             |    |    |    |   .|...x|    |    |    |    |    |    |    |    | \      |
o-------------------------------o-----------------------------------------------------------o-------------o
CYLC HUB VIA JUPYTERhUB         *JAN. FEB. MAR. APR. MAY. JUN. JUL. AUG. SEP. OCT. NOV. DEC.*JAN.| /  SEP.  
  Spawn UIS (Local host)        |    |...x|    |    |    |    |    |    |    |    |    |    |    | \      |
  Spawn UIS (Remote)            |    |    |....|...x|    |    |    |    |    |    |    |    |    | \      |
  User Authentication (PAM)     |    |  ..|x   |    |    |    |    |    |    |    |    |    |    | /      |
  User Authentication (LDAP?)   |    |  ..|x   |    |    |    |    |    |    |    |    |    |    | \      | (not important?)
  Server Trust (UIS-Hub)        |    |    |    |    |    |    |    |    |    |    |    |    |    | /      | (need to understnd)
  Session Management            |    |    |    |    |    |    |    |    |    |    |    |    |    | \      | (handled by J-Hub)
  Other-User View (by URL)      |    |    |    |  ..|x   |    |    |    |    |    |    |    |    | /      | 
  Other-User View (UI,JH Admin?)|    |    |    |    |    |    |    |    |    |    |  ..|..? |    | /      | 
  UI Server Authorization       |    |    |    |    |    |    |    |    |    |    |....|..? |    | /      |
  JupyterHub: Dep or Fork?      |    |    |  ..|....|...x|    |    |    |    |    |    |    |    | \      | (prob not fork)
o-------------------------------o-----------------------------------------------------------o-------------o
UI SERVER and UI                *JAN. FEB. MAR. APR. MAY. JUN. JUL. AUG. SEP. OCT. NOV. DEC.*JAN.| \J  SEP.  
  Vue.js app POC                |....|...x|    |    |    |    |    |    |    |    |    |    |    | /      |
  Hub Integration               |    |   .| .. |..x |    |    |    |    |    |    |    |    |    | \      |
  Manage multiple suites        |    |    |    | .  | ...|...x|    |    |    |    |    |    |    | /      |
  UI: Tornado GraphQL           |    |    |    |    | ...|x   |    |    |    |    |    |    |    | \      |
  UI: WebSocket                 |    |    |    |    |   .| .. | ..x|    |    |    |    |    |    | \      |
  UI Workflow State Views       |    |    |    |    |    |    |    |    |    |    |    |    |    | \      |
    table view (basic)          |    |    |    |  . | ...|x   |    |    |    |    |    |    |    | /      | (Not needed now)
    tree view (complete)        |    |    |    |    |  . | ...| ...|...o|    |    |    |    |    | /      |
    tree view (deployable)      |    |    |    |    |  . | ...|....|....|....|..o1|    |    | ...|./o2    |
    dot view                    |    |    |    |    |    |    |    |    |    |    |    | ..?|    | \      |
    graph view (basic)          |    |    |    |    |    |  . | .. | ...|...o|    |    |    |    | /      |
    graph view (selective)      |    |    |    |    |    |    |    |    | .. | .. | ...|...o|    | \      |
    other (adj. matrix etc.?)   |    |    |    |    |    |    |    |    |    |    |    | .  | ..?| /  ...o| (not req 8.0)
  SUB-SERVICES                  |    |    |    |    |    |    |    |    |    |    |    |    |    | \      |
    workflow scan               |    |    |    |    |    |    |    |    |    |    |    |    |    | /      | (now in GraphQL)
    stopped workflow scan       |    |    |    |    |    |    |    |    |    |    |...o|    |    | /      | (now in GraphQL)
    workflow start              |    |    |    |    |    |    |    |    |    |    |    |...o|    | \      | (note "rose s-r")
    "cylc review"               |    |    |    |    |    |    |    |  ..|...o|    |    |    |    | \      | (via UIS, owner)
    graph, validate, etc.       |    |    |   .|..oo|oo  |    |    |    |    |    |    |    |    | \      |
  Packaging UIS (python)        |    |    |    | .  | ...|..o |    |    |    |    |    |    |    | /      |
  Packaging UI  (upload npm)    |    |    |    |    |    |    |..o |    |    |    |    |    |    | /      |
  Fine-grained Authorization    |    |    |    |    |    |    |    |    |    |    |....|.?  |    | /      |
  UI Test framework?            |    |    |    |    |    |    |    |  ? |    |    |    |    |    | /      | 
  auto-shutdown                 |    |    |    |    |    |    |    |  ? |    |    |    |    |    | /      | 
o-------------------------------o-----------------------------------------------------------o-------------o
Initial System Integration      *JAN. FEB. MAR. APR. MAY. JUN. JUL. AUG. SEP. OCT. NOV. DEC.*JAN.| \  SEP.  
    Hub + UI Server             |    |   .| .. |...x|    |    |    |    |    |    |    |    |    | /      |
    UI Server + sub-services    |    |    |    | .  | ...|....|....| .  | .  | ?  |    |    |    | \      |
    UI Server + Workflow Service|    |    |    |   .|  ..|..ox|    |    |    |    |    |    |    | /      |
    Cylc-8a1                                              ......o
    Cylc-8a2                                                               ......o
o-------------------------------o-----------------------------------------------------------o-------------o
"rose suite-run"                *JAN. FEB. MAR. APR. MAY. JUN. JUL. AUG. SEP. OCT. NOV. DEC.*JAN.| \  SEP.  
    migration to Cylc           | .  | .  | .  |  . |  . |  . |  . |  . |  . | ...|...o|    |    | /      | (MS,TP)
    cluster support             |    |    |    |    |    |    |    |  ..|....|...o|    |    |    | /      |
o-------------------------------o-----------------------------------------------------------o-------------o
ROSE                            *JAN. FEB. MAR. APR. MAY. JUN. JUL. AUG. SEP. OCT. NOV. DEC.*JAN.| /  SEP.  
  rosie disco Tornado           |   .|....|....|...x|    |    |    |    |    |    |    |    |    | \      |
  Python 3                      |   .|....|....|....|...x|    |    |    |    |    |    |    |    | /      |
  Setup.py, packaging, PyPi     |    |    |    |....| .. | .. |.o  |    |    |    |    |    |    | \      |
  WS UI Rose Config-Edit        |    |    |    |    |    |    |    | ?  |    |    |    |    |    | \      |
  WS UI Rosie Go                |    |    |    |    |    |    |    | ?  |    |    |    |    |    | \      |
o-------------------------------o-----------------------------------------------------------o-------------o
PRODUCTION (Cylc-8 + Rose-2)    *JAN. FEB. MAR. APR. MAY. JUN. JUL. AUG. SEP. OCT. NOV. DEC.*JAN.| /  SEP.  
  Testing and Refinement        |    |    |    |    |    |    |    |    |    |    |    |    |....|.\. ...o| (hopefully June)
o-------------------------------o-----------------------------------------------------------o-------------o
```

## Notes and Details

### Communication and Collaboration:

- Project Board
  - ~~GitHub projects?~~ - too primitive
  - trying Trello

- Mail forum (mainly for users)
  - Google Groups (now using, but deprecated)
  - Discourse
    - ~~apply for a free instance~~ - DONE
    - ~~configure the new Cylc Forum, with GitHub and Google Account Login~~ - DONE 
    - awaiting decision to go ahead with this

- Chat Platform
  - ~~Slack?~~ - free account is lossy
  - using Riot.im
    - ~~currently blocked at MO~~

- Video Conference and screen sharing
  - ~~Skype~~ - yuck
  ~ ~~Google Hangouts~~ - tested, worked well, but:
  - using Riot.im 

Summary page created [here](project-collab.md).

### Documentation (Sadie) - DONE

- Content:
    1. ~~Old: CUG: LaTeX -> Sphinx (not critical)~~ DONE
    2. ~~New: write in Sphinx~~ DONE
- Infrastructure
    1. ~~doc generation~~ DONE

### Python 3 Migration

- ~~ISODatetime (Bruno and Matt)~~ - DONE
- ~~Cylc Scheduler~~ (Oliver) - DONE
- Rose (WIP - Sadie and Tim)

### Cylc Hub

(Martin, Hilary, Bruno)

Primary question: *can we use JupyterHub "out of the box" as UI Server
spawner?*
- ~~Basic POC~~ - DONE
  - LocalProcessSpawner, PAM, dummy UIS, other-user view
- other authentication plugins?
- remote spawners: ssh, PBS?
- other-user view without having to type /user/[name] in URL?
- site-level authorization via a Hub server?
  - (not a show-stopper; could be done at UIS if necessary)
- low priority but of interest:
  - Docker spawner?
  - support users with no local account?

Update (March): primary functionality demonstrated, with no need to
modify JupyterHub code. Still remains to be seen whether UI support for
other-user views and site authorization can be done with vanilla JupyterHub.

### Workflow Service (WS)

(Mostly Oliver)

- ~~(remove PyGTK GUI)~~ DONE
- ~~Python 3 (Oliver, above)~~ DONE
- Packaging (WIP - Bruno, Matt)
- [rose suite-run migration](proposal-rose-suite-run.md)
- ~~Replace Cherrypy and REST API with ZeroMQ~~ DONE
- ZeroMQ communications model to UI Server: REQ-REP-REP-REP (Oliver)
  - See UI Server (below)
    - and feedback of async command results (not just "command queued")
- Representation of API – self-documenting/"API on the fly"?
- Main loop asyncio.

### UI Server (UIS)

Design options described
  [here](meetings/vc-mar-2019-agenda#workflow-status-summary-data-and-communications)

Decision: implement "MODEL 1" (UIS as WS mirror)
  - but avoid assumptions that restrict us from going to a pass-through model
    in future if necessary

Initial design, therefore:
- UIS subscribes to WS
- WS does initial push of full workflow status data to UIS
- WS then pushes each change to UIS as an increment 
- ~~Separate project (cylc-ui-server)~~ DONE
- UIS resolves GraphQL queries from the UI
  - GraphQL schema and endpoint (WIP - David and Oliver)
  - GraphQl schema should exactly reflect the WS data model
  - WS data model might need redesigning somewhat
- Tornado web server (client-facing)
- ZeroMQ client (server-facing - SuiteRuntimeClient from WS)
- (note: UI server has to serve multiple Cylc versions)

Also:
- packaging (WIP - Bruno)

### UI Presentation Layer (JS framework)

- Front-end design (Vue.js)  - (Bruno, Martin)
- Testing (e.g. webpack and karma? And selenium … visual testing?) (Martin?)
- Data driver (GraphQL client and JS infrastructure behind it) (David and …?)
- Workflow Services views (in priority order):
   1. Tree view
   2. Dot view
   3. Graph view … (Oliver)
   4. Extra views: "Quilts" etc. (Sadie)

### General Authentication issues (~~Sujata~~ Hilary, + Damian and Martin ?)
- (Sujata now unavailable till late March?)
- Jupyter Hub user-authentication plug-ins 
- Hub -> UI Server Trust (JupyterHub out of the box?) (via WebSocket)
- UI Server -> Workflow Service Trust (via ZeroMQ)
- CLI -> Workflow Service Trust
   1. Direct - just Workflow Service owner (via ZeroMQ)
   2. Via UI Server (via WebSocket)
   3. Session management?
- Jobs -> Workflow Service (one-time token?) (via ZeroMQ)

### Authorization

- User X can log in at Hub (at Hub)
- User X can see these UI Servers (at Hub, or at UI Server?)
- User X can see these Workflow Services (at UI server)
- User X can do this to Workflow Services Y (at Workflow Service)
- (initially implement by simple config files that relate users or groups to
   privileges)

### Services (priority order)
- Workflow Services Discovery (listing running and stopped Workflow Services)
- Workflow Services start-up
- "cylc review" (replace cherrypy, consider "integration" with UI server?)
- "rose config-edit"
- "rosie go" (stored Workflow Services discovery – MOSRS client)
- "cylc graph" (non-PyGTK!)

### INTEGRATION

- (connecting components: TBD)

### Integration Tests

- more difficult with cylc-8 (more components; privileged components)
- use containers?

### Orthogonal Stuff (ONLY if we have enough resource)

- Job batching
- Cluster support
- (Python API etc. … after cylc-8)

### Rose (MO)

- ~~Remove Rose Bush (Tim)~~ - DONE
- Python 3  (above)
- Packaging (above)
- ~~Cherrypy -> tornado (rosie web server) (Sadie)~~ - DONE
