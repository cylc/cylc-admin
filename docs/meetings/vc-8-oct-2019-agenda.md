# Cylc VC 8 October 2019 Agenda

__ALL__:
- read below and add a placeholder for anything you think we need to talk about
- read the [9 September meeting notes](vc-9-sep-2019-notes.html) as a reminder

## Previous meeting
9 September
- [agenda](vc-9-sep-2019-agenda.html)
- [notes](vc-9-sep-2019-notes.html)

## General Admin etc. - 10 min - HO

- Welcome to Mel!
- NIWA/UM-Partnership contract for BK - STILL in prep
- Hacktoberfest - got a few bites!
  - [So far](https://github.com/search?q=org%3Acylc+label%3Ahacktoberfest&unscoped_q=label%3Ahacktoberfest) 30 issues created, with 6 closed
- OS secondment to NIWA approved (late Nov - mid Feb)
- Need to decide week of second Cylc workshop, and start planning
  - location NIWA, Wellington, NZ
  - late Jan or early Feb?

  **Early Feb, but avoid Waitangi Day**

Cylc-8.0a1 feedback/questions:
  - how to install on secure systems with no internet access - see this comment
    from
    [cylc-admin#27](https://github.com/cylc/cylc-admin/issues/27#issuecomment-534375389)
    for more
  - bug from a user that tested using base_url in JupyterHub (good use case,
    where the hub is behind a rev proxy) -
    https://github.com/cylc/cylc-ui/issues/258

*DM: NRL's problem with non-HTTPS on the back-end is that their remote
platforms are **really** remote. Proper solution is job CLI via the UIS?*

Any new Cylc-7 bug reports or problems?

Other?

## Packaging - 10 min - SB and BK

- cylc-8.0a1 full-system available via conda from kinow channel
- cylc-flow-8.0a1 now available from conda-forge
- what remains to be done on conda-forge? (and ETA?)
  - pending dependencies in the top comment of this issue: https://github.com/cylc/cylc-conda/issues/3

## UI Graph View - 20 min - MR

Update on:
- Cytoscape vs D3 etc
- prospects for custom node icons etc.

*MR: Cytoscape probably best in most respects, e.g. dynamic graphs and practical
updates; "El Grapho" faster?*

*DM and OS: don't forget "cylc graph"; data provision to UI of static graph.*

## WebSockets and Subscriptions - 10 min - BK and DS

Update on the recent work (UI, UIServer, and WFS)
- what has been done / is being done / is still to do; how it works
  - what has been done:
    - https://github.com/cylc/cylc-flow/pull/3390 (pending review): added subscriptions to the schema
    - https://github.com/cylc/cylc-uiserver/pull/82 (pending review): added code from graphene-tornado pending pull request; uses the schema updates from the PR above; adds a tornado WebSocket handler to expose the schema/graphql endpoint
    - https://github.com/cylc/cylc-ui/pull/280: updated project dependencies; fixed build & tests; adjusted previous code to keep working; updated views to use WebSockets/subscriptions
  - what is still to do:
    - Cylc Flow and Cylc UI Server pull requests need review (probably 1 reviewer DW, but needs one more)
    - After Cylc Flow and Cylc UI Server PR's are merged, review Cylc UI PR
    - create follow up issues after review/merge, with what's missing (e.g. data-driven on the websocket side instead of interval-based)

## CLI-WFS Authentication - 20 min - SB

WIP PR status update
- what has been done / is being done / is still to do; how it works

*SB away, but this should be done before she leaves the project; DS agrees on
the approach taken.*

## Config schema unification - 10 min - TP

Status update
#### What has been done
  - [Cylc-flow PR #3348](https://github.com/cylc/cylc-flow/pull/3348) Outlines the new
    config schema without actually making it _do_ anything. It's waiting on a
    review from DM.
  - [Cylc-admin PR #58](https://github.com/cylc/cylc-admin/pull/58) Attempts to
    update the documentation in the light of the comments on
    [Cylc-flow PR #3348](https://github.com/cylc/cylc-flow/pull/3348)
#### What is being done
  - [Cylc-flow PR #3357](https://github.com/cylc/cylc-flow/pull/3357) attempts
    to plug the new schema in for local usage, and starts to look at how
    that changes will continue to support Cylc 7.x `suite.rc` files.
#### What is still to do
  - Approving any of the above PR's.
  - Getting the new config working and well tested with new style `global.rc`
    files.
#### how it works
  - [The `suite.rc` proposal document in cylc-admin] contains details of how
    this change should work.

## Changelogs

- Do we need to add changelog to other projects (like cylc-uiserver, cylc-ui, etc?)

## AOB?
- next meeting? (first Mon/Tues each month)

*OS: Why Cylc? questions at UK RSE conf (plus "cloud integration" etc.); need
good answers to these; Cylc 8 best time to start pushing it*

