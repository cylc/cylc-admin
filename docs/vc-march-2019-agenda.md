# Cylc VC March 2019 Agenda

[Previous meeting notes](vc-feb-2019-summary.md)

## Site Updates, News

- CiSE paper accepted.
- Met Office PS42->OS42 at Met Office using Cylc 7.8.1 (with mod).
- NIWA operations - 7.8.1? (David)
- BOM? (Martin) - Cylc + SMS, a long-term marriage made in heaven?

## Upcoming 7.8.2 maintenance release

- quick review of issues
- any blockers?
- release date?

## Python 2-to-3

- Cylc Workflow Service + 0mq port  - **DONE**
- Rose standalone toolkit - **DONE**.
- Rosie disco + Tornado port - WiP
- Rose suite wrappers - TBD

## Cylc-8 Architecture 

### Demo
- (Bruno) Hub + UI server + Cylc-7 Workflow Service with Flask + GraphQL

### JupyterHub
- more or less proven, unmodified, as Cylc Hub?
- what remains to be done?
  - other authentication, spawners (PBS?), other-user view?
  - anything else?

### UI (cylc-web)
- Can we do any more without real workflow data?
  - (fake data? - don't know structure yet, below)

### Workflow Status Summary Data and Communication

Some disagreement on the fundamental model here.
**MODEL 1: UI Server Mirror**:
- UI Server holds a data structure that mirrors the Workflow state
- One of these per Workflow Service (not one per UI)
- Workflow service updates this incrementally by dumb push
- UI "indexes into" this via GraphQL
- GraphQL schema exactly mirrors the status data structure
- Workflow Service does not see multiple UIs, just the UI Server (for status
  requests, not commands)
**MODEL 2: UI Server Pass-Through**
- UI Server just passes UI requests through to Workflow Services
- Needs GraphQL at the backend too? 
- What does incremental update mean in this case?
- Is this simpler or not? 
-
**TODO**
- decide on model, with implications for API and subscription model, etc.
- (what does incremental update mean or imply at the UI end?)
- what (dict) form should the summary data structure take, exactly?
- document method and data "schema" clearly before implementation

### UI Server
- As a project: `cylc-ui-server` (Python Tornado), depends on cylc-web?
- Can we start implementing this?  (questions above need resolving)
- Initially serve fake workflow data

### Workflow Service
- any logic changes *necessary* for cylc-8?
  - how summary status data is updated from main loop?
  - main loop asyncio?
  - other?

### Graph Representation and Processing
- stick with a flat list of nodes and edges?
- internal processing, or external Graph Engine, e.g. Neo4j
- server has to compute the "concrete graph" over range of cycle points,
  and client populates nodes (as now)?  (otherwise client needs graph pattern
  and full cycling capability)
- begin investigating now or leave until other views are done?

### Next steps toward cylc-8.0a1
  - Packaging - nearly done? (Bruno)
  - UI Server?
  - Review Gantt Chart? (Projectt Gantt Chartt!?)
  - Who does what?

## Project Admin and Code Management
- ZenHub? Trello? GitHub Projects? (let's decide and be done with it!)
-. **Riot.im**, the saga continues!
  - (note todays email CC-list screw-up)
  - status at MO? switch to Slack?
- Do the other new `cylc/*` repositories need milestones?
- Do we need to choose coding styles?
   - (only PEP8 so far)
   - Python Docstring Conventions - would be nice to have one that works
     with most tools (i.e. sphinx, vim syntax hightlight, IDE, napoleon,
     etc)
- A `CONTRIBUTING.md` with guidelines for people creating PR for the
  first time (the 2 approvers rule is nice, but not well known until you
  send a contribution), also explaining that test coverage is desired
  when possible, as well as documentation update, etc
- Maybe worth filling in the full set of community profile e.g.
  https://github.com/cylc/cylc-admin/community as well.
- Should we use [Black](https://github.com/ambv/black)?
- Source repository names:
  - `cylc/cylc` --> `cylc/cylc-workflow-service`? (or not, for `pip install cylc`?)
  - (along with `cylc/cylc-web-ui`, `cylc/cylc.github.io`...)
  - or does `pip install cylc-VERSION` dictate that we keep `cylc/cylc`?
- Sadie's Cheat Sheets idea: new repository `cylc/cylc-cheat-sheets`?
   - `cylc review` to new repo `cylc/cylc-review`? 
   - `cylc profile-battery` command to new repo.
   - `suite.rc` -> `cylc-workflow.rc`? (or `cylc-workflow.yaml`?...)
- Improved release process (Bruno) 
   - less reliance on Hilary
   - Pull Request template, require a message for the release change log?
   - automatically generate CHANGES.md?
- Testing
  - Unit test + Battery test -> Pytest?
  - UI testing - any decisions here? (Selenium?)

## Speculative, or off the cylc-8 critical path
- YAML vs suite.rc vs Python API
   - is it worth considering suite.rc -> YAML for cylc-8
   - or not, given intention to make a Python API in time (is the API possible
     for cylc-8? ... probably not)

## END
- anything else?
- next meeting date: in 2 or 4 weeks?
