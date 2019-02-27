# cylc-8 Implementation Task List

### Major Updates and Changes
 - Initially compiled after the December 2018 Development Workshop.
 - Updated 25 Feb 2019: added Gantt Chart.

### New Terminology
_suite daemon_  (original) -> _suite server program_ (more recent) -> __workflow service__ (WS) (new)

## Projectt Gantt Chart

### Key
- `.` - in progress
- `o` - estimated completion date
- `x` - done (actual completion date)

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
WORKFLOW SERVICE (WS) and CLI   *JAN. FEB. MAR. APR. MAY. JUN. JUL. AUG. SEP. OCT. NOV. DEC.*JAN.| /  SEP.  
  Python 3                      |   .|....|...o|    |    |    |    |    |    |    |    |    |    | \      | 
  Cherrypy -> ZeroMQ network    |   .|....|...o|    |    |    |    |    |    |    |    |    |    | /      | 
  Command-line clients          |    |    |  ..|...o|    |    |    |    |    |    |    |    |    | \      | 
    old passphrase authe        |    |    |  ..|...o|    |    |    |    |    |    |    |    |    | /      | 
    one-time token authe        |    |    | ...|....|...o|    |    |    |    |    |    |    |    | \      |
    session management          |    |    | ...|....|....|..o |    |    |    |    |    |    |    | /      |
  UI SERVER API                 |    |    |    |    |    |    |    |    |    |    |    |    |    | \      |
    serve old data model        |    |    |    |...o|    |    |    |    |    |    |    |    |    | /      |
    design new data model       |    |    |    |  ..|...o|    |    |    |    |    |    |    |    | \      |
    serve new data model        |    |    |    |    | ...|...o|    |    |    |    |    |    |    | /      |
  Async event loop              |    |    |    |    |    |  ..|....|..o |    |    |    |    |    | \      |
  Fine-grained Authorization    |    |    |  ..|..o |    |    |    |    |    |    |    |    |    | /      |
  PYTHON PACKAGING              |    |    |    |    |    |    |    |    |    |    |    |    |    | \      |
    pip, setup.py               |    |    |...o|    |    |    |    |    |    |    |    |    |    | /      |
    Cylc available on PyPi      |    |    |   .|o   |    |    |    |    |    |    |    |    |    | \      |
o-------------------------------o-----------------------------------------------------------o-------------o
CYLC HUB VIA JUPYTERhUB         *JAN. FEB. MAR. APR. MAY. JUN. JUL. AUG. SEP. OCT. NOV. DEC.*JAN.| /  SEP.  
  Spawn UIS (Local host)        |    |...x|    |    |    |    |    |    |    |    |    |    |    | \      |
  User Authentication (PAM)     |    |  ..|x   |    |    |    |    |    |    |    |    |    |    | /      |
  User Authentication (LDAP?)   |    |  ..|x   |    |    |    |    |    |    |    |    |    |    | \      |
  Server Trust                  |    |   .|..o |    |    |    |    |    |    |    |    |    |    | /      |
  Session Management            |    |   .|...o|    |    |    |    |    |    |    |    |    |    | \      |
  Other-User View (Admin?)      |    |   .|....|.o  |    |    |    |    |    |    |    |    |    | /      |
  Spawn UI Server (Remote)      |    |    |....|...o|    |    |    |    |    |    |    |    |    | \      |
  UI Server Authorization       |    |    | ...|....|..o |    |    |    |    |    |    |    |    | /      |
  JupyterHub: Dep or Fork?      |    |    |  ..|....|...o|    |    |    |    |    |    |    |    | \      |
o-------------------------------o-----------------------------------------------------------o-------------o
UI SERVER and UI                *JAN. FEB. MAR. APR. MAY. JUN. JUL. AUG. SEP. OCT. NOV. DEC.*JAN.| \  SEP.  
  Vue.js app POC                |....|...x|    |    |    |    |    |    |    |    |    |    |    | /      |
  Hub Integration               |    |   .| .. |...o|    |    |    |    |    |    |    |    |    | \      |
  Manage multiple suites        |    |    |    | .  | ...|...o|    |    |    |    |    |    |    | /      |
  UI: Tornado GraphQL WebSocket |    |    |    |    |   .| .. | .. |...o|    |    |    |    |    | \      |
  WS: ZeroMQ                    |    |    |    |    |   .|..o |    |    |    |    |    |    |    | /      |
  WS UI Workslow State Views    |    |    |    |    |    |    |    |    |    |    |    |    |    | \      |
    table view (basic)          |    |    |    |  . | .. |...o|    |    |    |    |    |    |    | /      |
    table view (complete)       |    |    |    |    |  . | ...| ...|...o|    |    |    |    |    | /      |
    dot view                    |    |    |    |    |    | ...|..o |    |    |    |    |    |    | \      |
    graph view (basic)          |    |    |    |    |    |  . | .. | ...|...o|    |    |    |    | /      |
    graph view (selective)      |    |    |    |    |    |    |    |    | .. | .. | ...|...o|    | \      |
    other (adj. matrix etc.?)   |    |    |    |    |    |    |    |    |    |    |    | .  | .. | /  ...o|
  SUB-SERVICES                  |    |    |    |    |    |    |    |    |    |    |    |    |    | \      |
    workflow scan               |    |    |  ..|....|..o |    |    |    |    |    |    |    |    | /      |
    workflow start              |    |    |  ..|..o |    |    |    |    |    |    |    |    |    | \      |
  Packaging? (separate from WS) |    |    |    | .  | ...|..o |    |    |    |    |    |    |    | /      |
  Authorization                 |    |    |    | .. | .. |...o|    |    |    |    |    |    |    | \      |
  UI Test framework?            |    |    | .  | .. |....|...o|    |    |    |    |    |    |    | /      | 
o-------------------------------o-----------------------------------------------------------o-------------o
Initial System Integration      *JAN. FEB. MAR. APR. MAY. JUN. JUL. AUG. SEP. OCT. NOV. DEC.*JAN.| \  SEP.  
    Hub + UI Server             |    |   .| .. |...o|    |    |    |    |    |    |    |    |    | /      |
    UI Server + sub-services    |    |    |    | .  | ..o|    |    |    |    |    |    |    |    | \      |
    UI Server + Workflow Service|    |    |    |   .|  ..|..o |    |    |    |    |    |    |    | /      |
o-------------------------------o-----------------------------------------------------------o-------------o
"rose suite-run"                *JAN. FEB. MAR. APR. MAY. JUN. JUL. AUG. SEP. OCT. NOV. DEC.*JAN.| \  SEP.  
    migration to Cylc           | .  | .  | .  | .. | .. |....|...o|    |    |    |    |    |    | /      |
o-------------------------------o-----------------------------------------------------------o-------------o
ROSE                            *JAN. FEB. MAR. APR. MAY. JUN. JUL. AUG. SEP. OCT. NOV. DEC.*JAN.| /  SEP.  
  Cherrypy -> Tornado           |   .|....|...o|    |    |    |    |    |    |    |    |    |    | \      |
  Python 3                      |   .|....|...o|    |    |    |    |    |    |    |    |    |    | /      |
  Setup.py, packaging, PyPi     |    |    |    |...o|    |    |    |    |    |    |    |    |    | \      |
  WS UI Rose Config-Edit        |    |    |    |    |    |    |    |    |    |    |    |    |    | \      |
    basic                       |    |    |    |   .| .. |....|...o|    |    |    |    |    |    | \      |
    complete                    |    |    |    |    |    |    | .. | .. | .. |....|....|...o|    | \      |
  WS UI Rosie Go                |    |    |    |    |    |    |    |    |    |    |    |    |????| \      |
o-------------------------------o-----------------------------------------------------------o-------------o
PRODUCTION (Cylc-8 + Rose-2)    *JAN. FEB. MAR. APR. MAY. JUN. JUL. AUG. SEP. OCT. NOV. DEC.*JAN.| /  SEP.  
  Testing and Refinement        |    |    |    |    |    |    |    |    |    |    |    |    |....|.\. ...o|
o-------------------------------o-----------------------------------------------------------o-------------o
```

## Notes and Details

### Communication:

- GitHub projects set up and use?
- Discourse
  - ~~apply for a free instance~~ - DONE
  - ~~configure the new Cylc Forum, with GitHub and Google Account Login~~ - DONE 
- ~~Riot.im (open it up to others, etc.)~~ - DONE
- Video Conferencing – report back: who can use what?
  - ~~Riot.im video conferencing and screen sharing?~~ - DONE (can use Riot.im)

UPDATE: Riot.im now blocked at one site

### Documentation (Sadie) - DONE

- Content:
    1. ~~Old: CUG: LaTeX -> Sphinx (not critical)~~ DONE
    2. ~~New: write in Sphinx~~ DONE
- Infrastructure
    1. ~~doc generation~~ DONE

### Python 3 Migration

- ~~ISODatetime (Bruno and Matt)~~ - DONE
- Cylc Scheduler ~~(Bruno and Matt)~~ Oliver
- Rose (MO: Sadie and Tim)

### Workflow Service

- ~~(remove PyGTK GUI)~~ DONE
- Python 3 (Oliver, above)
   1. Ordered dict gone (what about "ordereddict_with_defaults"?)
- Packaging (Bruno, Matt)
- [rose suite-run](proposal-rose-suite-run.md)
- Replace Cherrypy with ZeroMQ ~~(with protobuf?)~~ DONE (minimally)
- Replace REST API to ZeroMQ (Direct)
- 3. Representation of API – self-documenting/"API on the fly"?
- Callback to UI of async server command results (via ZeroMQ, then pass to GUI?)

### Rose (MO)

- Remove Rose Bush
- Python 3  (above)
- Packaging (above)
- Cherrypy -> tornado (rosie web server) (Sadie)

### Cylc Hub =? Jupyter Hub Investigation (Martin, Hilary + Matt) – (urgent).

- (user auth – Sujata - see above)
- Can we do the following without forking Jupyter Hub:
    1. See other users' UI Servers
    2. (users with no local unix account?)

### Authentication (~~Sujata~~ Hilary, + Damian and Martin ?)
- (Sujata now unavailable till late March?)
- Jupyter Hub user-authentication plug-ins 
- Hub -> UI Server Trust (JupyterHub out of the box?) (via WebSocket)
- UI Server -> Workflow Service Trust (via ZeroMQ)
- CLI -> Workflow Service Trust
   1. Direct - just Workflow Service owner (via ZeroMQ)
   2. Via UI Server (via WebSocket)
   3. Session management?
- Jobs -> Workflow Service (one-time token?) (via ZeroMQ)

### UI Server (replaces Jupyter notebook server)

- Separate project and packaging (if possible?)
- GraphQL schema and endpoint
- Tornado web server (client-facing) (Sadie)
- ZeroMQ (server-facing)
- (note: UI server has to server multiple Cylc versions)

### UI Presentation Layer (JS framework)

- Front-end design (Vue.js)  - (Bruno, Martin)
- Testing (e.g. webpack and karma? And selenium … visual testing?) (Martin?)
- Data driver (GraphQL client and JS infrastructure behind it) (David and …?)
- Workflow Services views (in priority order):
   1. Tree view
   2. Dot view
   3. Graph view … (Oliver)
   4. Extra views: "Quilts" etc. (Sadie)

### Authorization

- User X can log in at Hub (at Hub)
- User X can see these UI Servers (at Hub, or at UI Server?)
- User X can see these Workflow Services (at UI server)
- User X can do this to Workflow Services Y (at Workflow Service)
- (initially implment by simple config files that relate users or groups to
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
