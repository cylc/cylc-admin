# Cylc Project VC 1 September 2020

*(Meeting notes in italics)*

## Cylc-7

- [7.8.x Milestone](https://github.com/cylc/cylc-flow/milestone/82)
- (last month's GUI sorting bug fixed)
- (Release not a high priority; let's back-port Cylc 8 fixes as we go,
  then make a release later on)

*(no objections)*

## Cylc-8

- [Development timeline from 
  workshop](https://cylc.github.io/cylc-admin/feb2020-workshop-report#tentative-development-timeline)

*(we are quite behind schedule in many respects; reasons include the pandemic
and SoD implementation - but at least the latter puts us ahead in other ways)*

Progress:
- SoD merged (follow-ups from proposal now Issues; priorities TBD)
- Platforms merged
  - *a few small "nice to have" follow-ups to come, but TW will concentrate on
    finishing of the `rose suite-run` migration with MH first*
  - *OS: after Platforms has settled, we should be able to configure our way
    out of FQDN-based portability issues; see cylc-flow#3768*
- Infinite tree not needed! (delta updates made plain tree perform well)
- UI filtering almost ready
- CLI GraphQL almost ready
- Async scan (including stopped flows) almost ready
- Rsync files to install targets (PR up today!)
  - *MH gave us an update on this; it pretty much sews up the file installation
    work apart from symlink follow-up*

Do we need another alpha release? (meh)
- *(no strong opinions expressed)*

Beta release MVP October/November?
- UI
  - performant tree view with filtering
  - integrated mutations
    - *BK and OS to discuss soon - next highest UI priority*
  - scan stopped flows, and allowing starting them
    - *Ability to see and start uninstalled "roses dir" sources important too
      (related to "template suites" and run1, run2, run3, run-dirs.*
- UIS
  - **event-driven n-distance datastore (n=0,1 in scheduler)**
  - load older tasks from disk
  - start stopped flows
- Other
  - `rose suite-run` migration (file installation etc.)
     - *still to do: "template suites" and run1, run2, run3, run-dirs.*
  - CLI via GraphQL
  - task status naming changes (incl. xtrigger PR)

Nice to have (for beta release):
- New graph view - (hierarchical layout less important now - in low-n windows?)
  - *HO: dagre is slow and edge shapes are ugly c.f. graphviz - is that part
    of the layout proper, or configurable?*
  - *OS: still strongly in favor of hierarchical layout*
- Cylc 8 migration guide with all breaking changes listed
- Cross-user functionality
  - *DM noted this is very important, more so than graph view*
- Delta-driven TUI (needs homemade GraphQL subscriptions at the scheduler)

Questions:
- What should gscan show in the post task-pool world? ("number of succeeded
  tasks"??)
  - *n=1 out from: "waiting tasks" is "what comes next"*
  - *also need number of queued and "preparing" tasks (as well as active)*
  - *various options as to what if anything to say about "succeeded" tasks -
    TBD*
  - *gscan currently subscribes to all task proxies and computes its own status
    summaries - need to get that direct from the back end; consider also task vs
    job states.*
 
## cylc/cylc-flow

- [Current PRs](https://github.com/cylc/cylc-flow/pulls)
- [Open Questions](https://github.com/cylc/cylc-flow/issues?q=is%3Aopen+is%3Aissue+label%3Aquestion)


## AOB?

- *HO (and to a lesser extent BK) largely taken out by RSE conf through end of
  next week*
- *TP has off-project stuff to do this week, then on holiday*
- *JR still tied up with other work*
- *OS:*
  - *will move on to the UI after tying up a few cylc-flow loose ends*
  - *his last cylc-flow "curve ball" is test battery use of a simple
    Docker-based remote host.*
  - *tests passing on OSX/BSD now - let's make it official... once we've fixed
    test issues currently affecting Linux (and the the FQDN problems)*

## Next Meeting?

- TP to arrange for early October
  - *same time-ish is fine*
