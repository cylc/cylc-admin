# Cylc VC 4/5 May 2019 Agenda

## Previous Meeting Actions (10 min)

Go through  **ACTIONS** in [March meeting notes](vc-mar-2019-summary.md)

## Site Updates and News (5 min)

- New [Cylc Trello Board](https://trello.com/b/6k6KRUJM/cylc-main-board)
- CiSE paper: preprint published
  - add to Cylc web site doc page.
- NIWA
   - we need [root-dir{log} PR](https://github.com/metomi/rose/pull/2297)
   - formal NIWA/UM-Partnership contract for Bruno in prep
- Any new technical issues at sites?

## 7.8 maintenance (5 min)

- [7.8.2](https://github.com/cylc/cylc/milestone/75) release still pending
- any blockers?
- release date?

## cylc-8 packaging (5 min)

Bruno: status update

## cylc-8 Hub (10 min)

Martin: status update
- BatchSpawner?
    - Bruno: Docker with PBS?
- ssh RemoteSpawner?
- (other: auth etc.?)

## cylc-8 WS, UIS, UI (20 min)

Oliver, Bruno, David: status update
  - UIS: we settled on the WS mirror data model
     - base on "cylc-jupyterhub" prototype
  - ZeroMQ, data model, and incremental update
  - Tornado, WebSocket, GraphQL (to UI)
  - WS asyncio
  - [API-on-the-fly?](https://github.com/cylc/cylc/pull/3005#issuecomment-479512438) 

## Orthogonal work (5 min)

Matt: status update
- Rose Python 3
- "rose suite-run" migration?
- cluster awareness
- other?

## Other (10 min; if time allows)

- Source repositories and naming:
  - `cylc/cylc` --> `cylc/cylc-workflow-service`? (or not, for `pip install cylc`?)
  - (along with `cylc/cylc-web-ui`, `cylc/cylc.github.io`...)
  - or does `pip install cylc-VERSION` dictate that we keep `cylc/cylc`?
  - Sadie's Cheat Sheets idea: new repository `cylc/cylc-cheat-sheets`?
  - `cylc review` to new repo `cylc/cylc-review`? 
  - `cylc profile-battery` to new repo.

- Improved release process (Bruno) 
   - less reliance on Hilary
   - Pull Request template, require a message for the release change log?
   - automatically generate CHANGES.md?

- Testing
  - Unit test + Battery test -> Pytest?
  - UI testing - any decisions here? (Selenium?)

- suite.rc, YAML, Python API?
  - `suite.rc` -> `cylc-workflow.rc`? (or `cylc-workflow.yaml`?...)
  - my feeling is we could force users to change config file format, and we
    need a Python API for advanced users/applications, but we can't force
    users to use Python (simpler suites don't *need* Python)
  - so is it worth considering suite.rc -> YAML? for cylc-8, or later?
  - Does the intention to make a Python API change this?
  - Is a Python API possible for cylc-8? (probably not)

- Oliver's API-only-the-fly POCs:
  - CLI? web UI?
  - (GraphQL in WS would impact this? ... but we're currently aiming for
    GraphQL in UI S, not WS)

## END
- **anything else?**
- next meeting date
