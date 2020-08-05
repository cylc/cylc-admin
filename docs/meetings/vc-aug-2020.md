# Cylc Project VC 4 August 2020

*(Meeting notes in italics)*

## Cylc-7

- [7.8.x Milestone](https://github.com/cylc/cylc-flow/milestone/82)
- see also: [#3664](https://github.com/cylc/cylc-flow/issues/3664)
- [GUI sorting and lagginess](https://github.com/cylc/cylc-flow/issues/3698)
  - *Met O: sorting issue not seen; GUI a bit laggy for huge suites but not
    that bad; note that both local scheduler and task jobs contribute to load;
    network issues may be a problem for large suites with frequent global data
    updates*
- Release 7.9.2/7.8.7?
  - *Not a high priority right now; let's back-port Cylc 8 fixes as we do them, then make a release later on*

## Cylc-8.0a2:

- released and announced :tada:
  - *no feedback received; but we're not pushing heavy testing till beta-zero release*

## Top priorities

Is first beta release feasible by late September? (Probably not!)

*New 8.0b0 milestone needed; essentially content:*
- *infinite tree*
- *n-distance data store*
- *integrated mutations*
- *platforms and other config changes*
- *rose suite-run migration* (suite installation etc.)

*We will probably publish more alpha releases before then*


- [Development timeline from 
  workshop](https://cylc.github.io/cylc-admin/feb2020-workshop-report#tentative-development-timeline)

- merge UI delta tree [DONE]
- review and merge platforms PR
- n-distance windowing (mainly UIS)
    - *n=1 fine for the scheduler, and will be useful for clients such as `cylc tui`*

Then:

- infinite tree
- mutations integration
- (CLI GraphQL)
- `rose suite-run`
- task status naming (complete the changes)
- Graph View, UIS subservices, cross-user functionality, etc...

`rose suite-run` migration:
- *OS: do this last before beta-0; might take the longest to implement; still some open questions*
- *MH is looking at suite installation aspects of this*

CLI via GraphQL
- *DS working on this now, done by end of week?*
- *straightforward but a bit laborious (each command needs its request)*

Platforms PR:
- *HO to review by end of week*
 
## cylc/cylc-flow

- [Current PRs](https://github.com/cylc/cylc-flow/pulls)
- [Open Questions](https://github.com/cylc/cylc-flow/issues?q=is%3Aopen+is%3Aissue+label%3Aquestion)
- platforms: ready for review

*Note the almost-year-old retry as xtrigger PR is part of the task state
renaming effort - need to get this done (a couple of questions on the issue to
be decided)*

*Async scan PR: use JSON output, drop the old output formats*

*DM: Cylc 8 migration guide will be needed, with all breaking changes listed*

## Spawn on Demand

Merged :tada: (One non-trivial problem found and fixed in final review)

Now:
- Prioritize and address follow-up issues
- adapt the system to it:
  - n-distance window! (historical tasks loaded by UIS from DB)
  - *plus event-driven datastore update*


*HO to transfer all TODO items from proposal to GitHub Issues*

## UI
- (currently blocked waiting on merge of delta tree PR, but more or less agreed now)
- *PR merged; BK says infinite try likely done by end of August*
- *known missing families bug predates SoD but made worse by it - UIS/datastore at fault*

## AOB?

- *OS: delta-driven `cylc tui` needed for beta release, to work with large suites*
- - *needs GraphQL subscriptions at the scheduler*
- - *no built-in support for this in ZMQ: reimplement ourselves*

## Next Meeting?

- TP to arrange for early September?
