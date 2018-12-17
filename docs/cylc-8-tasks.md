
# cylc-8 Implementation Task List
Compiled at the end of the December 2018 Development Workshop.

### New Terminology
_suite daemon_  (old) -> _suite server program_ (recent) -> __workflow service__ (NEW)

## Python 3 Migration

- ~~ISODatetime (Bruno and Matt)~~ - DONE
- Cylc Scheduler (Bruno and Matt)
- Rose (MO)

## Organisation:

- GitHub projects set up and use?
- Discourse
  - ~~apply for a free instance~~ - DONE
  - ~~configure the new Cylc Forum, with GitHub and Google Account Login~~ - DONE 
- ~~Riot.im (open it up to others, etc.)~~ - DONE
- Video Conferencing – report back: who can use what?
  - ~~Riot.im video conferencing and screen sharing?~~ - DONE (should be able to use Riot.im)

## Documentation (Sadie)

- Content:
    1. Old: CUG: LaTeX -> Sphinx (not critical)
    2. New: write in Sphinx
- Infrastructure (doc generation) – [~~copy from Rose etc~~ was not necessary,
  as is fairly trivial to set-up].

## Cylc

- (remove PyGTK GUI)
- Python 3 (above)
   1. Ordered dict gone (what about "ordereddict_with_defaults"?)
- Packaging (above)
- "rose suite-run"
- Cherrypy -> ZeroMQ (with protobuf?)
- REST API to:
   1. ZeroMQ (Direct)
   2. WebSocket (GraphQL)
   3. Representation of API – self-documenting/"API on the fly"
- Callback to UI of async server command results (via ZeroMQ, then pass-thru to GUI?)

## Rose

- Remove Rose Bush
- Python 3  (above)
- Packaging (above)
- Cherrypy -> tornado (rosie web server)

## Packaging – pip and conda (Bruno and Matt)

## Authentication (~~Sujata~~ Hilary, + Damian and Martin ?)
- (Sujata now unavailable till late March?)
- Jupyter Hub user-authentication plug-ins 
- Proxy -> UI Server Trust (Jupyter out of the box?) (via WebSocket)
- UI Server -> Suites Trust (via ZeroMQ)
- CLI -> Suites Trust
   1. Direct - just suite owner (via ZeroMQ)
   2. Via UI Server (via WebSocket)
   3. Session management?
- Jobs -> Suites (one-time token?) (via ZeroMQ)

## Authorization

- Site-level (can user X connect at all?)
- User-level (can user X do this to all of my suites? To my UI server?)
- Suite-level (can user X can do this to suite Y?)
- (simple config files, at least initially)

## Cylc Hub =? Jupyter Hub Investigation (Martin, Hilary + Matt) – (urgent).

- (user auth – Sujata - see above)
- Can we do the following without forking Jupyter Hub:
    1. See other users' UI Servers
    2. users with no local unix account

## UI Server (replaces Jupyter notebook server)

- Separate project and packaging (if possible?)
- GraphQL schema and endpoint
- Tornado web server (client-facing)
- ZeroMQ (server-facing)
- (note: UI server has to server multiple Cylc versions)

## UI Presentation Layer (JS framework)

- Front-end design (Vue.js)  - (Bruno, Martin)
- Testing (e.g. webpack and karma? And selenium … visual testing?) (Martin?)
- Data driver (GraphQL client and JS infrastructure behind it) (David and …?)
- Suite views (in priority order):
   1. Tree view
   2. Dot view
   3. Graph view … (Oliver)
   4. Extra views: "Quilts" etc.

## Services (priority order)
- Suite Discovery (listing running and stopped suites)
- Suite start-up
- "cylc review" (replace cherrypy, consider "integration" with UI server?)
- "rose config-edit"
- "rosie go" (stored suite discovery – MOSRS client)
- "cylc graph" (non-PyGTK!)

## INTEGRATION

- (connecting components: TBD)

## Integration Tests

- more difficult with cylc-8 (more components; privileged components)
- use containers?

## Orthogonal Stuff (ONLY if we have enough resource)

- Job batching
- Cluster support
- (Python API etc. … after cylc-8)

