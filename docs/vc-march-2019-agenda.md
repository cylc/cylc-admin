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

## New Architecture 

1. **Demo** (Bruno) Hub + UI server + Cylc-7 Workflow Service with Flask +
  GraphQL

1. **JupyterHub** more or less proven?
  - tested with local PAM authentication and local process spawner
  - no mods required so far
  - other authentication
  - other spawners (PBS spawner? - Martin)
  - other-user view?

1. Can we do any more on the **UI** without real workflow data?
  - (fake data? - don't know structure yet, below)

1. state summary data structures in **Workflow Service**?
  - form? 
  - efficient incremental update from task pool?
  - how to push data increments to UI Servers?

1. state summary data data structure in **UI Server**?
  - form? (same as in WS?)
  - one per workflow, or one per UI?

1. UI Server subscription model?

1. How do current APIs compare to what we want to end up with?
  - ZeroMQ (Oliver)
  - GraphQL schema (David)

1. Any **Workflow Service** logic changes needed?
  - state summary data structures and update (above)
  - main loop asyncio?
  - other?

1. **Testing**
  - Unit test + Battery test -> Pytest?
  - UI testing - any decisions here? (Selenium?)

1. **Next steps toward cylc-8.0a1?**
  - Packaging - nearly done? (Bruno)
  - Connecting UI Server to Python 3 Workflow Service 
  - Review Gantt Chart? (Projectt Gantt Chartt!?)
 
1. **Graph View**
  - representation in Workflow Service, and UI Server?
  - internal processing, or external Graph Engine, e.g. Neo4j
  - server has to compute the "concrete graph" and client populates nodes (as now)?
  - begin investigating now or leave until other views are done?

## **Project Management**

1. **Who does what?**

1. **Task tracking**
  - ZenHub? Trello? GitHub Projects?
  - (let's decide and be done with it!)

1. **Riot.im** - the saga continues!
  - (note todays email CC-list screw-up)
  - (switch to Slack?)

1. **Other?**
  - Do the other new `cylc/*` repositories need milestones?
  - Do we need to choose coding styles?
      * Python Docstring Conventions - would be nice to have one that works with most tools (i.e. sphinx, vim syntax hightlight, IDE, napoleon, etc)
      * A `CONTRIBUTING.md` with guidelines for people creating PR for the first time (the 2 approvers rule is nice, but not well known until you send a contribution), also explaining that test coverage is desired when possible, as well as documentation update, etc
      * Maybe worth filling in the full set of community profile e.g. https://github.com/cylc/cylc-admin/community as well.
      * Should we use something like [Black](https://github.com/ambv/black)?
  - Change name of main `cylc/cylc` repo to `cylc/cylc-workflow-service`?
      - (along with `cylc/cylc-web-ui`, `cylc/cylc.github.io`...)
      - or does `pip install cylc-VERSION` dictate that we keep `cylc/cylc`?
      - Sadie's Cheat Sheets idea: new repository `cylc/cylc-cheat-sheets`?
      - `cylc review` to new repo `cylc/cylc-review`? 
  - Pull Request template: e.g. add a message for the release change log?
  - YAML vs suite.rc vs Python API
      - is it worth doing both? or Python API for cylc-8 (probably too big a job!)
  - `suite.rc` -> `cylc-workflow.rc`? (or `cylc-workflow.yaml`?...)
  - *anything else?*

1. **END**
  - any other business?
  - next meeting date - in 2 weeks?
