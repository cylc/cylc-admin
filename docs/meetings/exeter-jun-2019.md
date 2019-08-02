# Exeter Meeting Notes

June 24-28
Present HO, DM, MS, OS, SB, BK, TP

-----

- ZenHub - use instead of Trello, as it provides a multi-project view
  of GitHub.

- Reviewed and merged a bunch of outstanding PRs
  - esp. DS pair for Protobuf-over-ZMQ WFS-to-UIS comms 

- UI Design Session led by OS
  - refined our ideas based on OS excellent Inkscape mock-ups.
  - OS has now updated the design document and posted to the cylc-ui issue.

- MR provided instructions for his Cytoscape-based graph view experiments,
  which look very promising, and a PR to the UI project. Uses fossilized test
  data at this stage, but fine for demos. 

- UI: prioritize Tree View based on OS mock-ups.
  - Dot-view is low priority.

- UIS-WFS authentication/trust - similar to CLI, e.g. one-time or timed token.

- need encryption between Hub and UIS (not the default in J-Hub).
  - high priority once we have the end-to-end system

- UIS to do all authorization if possible. It has to do some, e.g. for suite
  start-up, so might as well do it all.
  - so audit trail for all client interaction needed at the UIS

- User feedback request on Discourse
  - agreed we should do this
  - not wide spectrum of use cases - can't promise to please everyone
  - SB to write up for review before posting to Discourse.

- We will rely on J-Hub to be present, even for single-user installations

- UIS needs to abort (or prob just be useless) if not started/owned by the Hub.

- we may eventually need a native e-app (e.g. electron) for those without the
  right network routing.

- JupyterLab is a different/better notebook interface. Has a nice user-movable
  tiling tab feature that we should try to use. 

- Most tree view info should be retrieved on request only (e.g. Job ID etc. in
  the current tree view, not needed unless the final level is expanded in the
  new UI.

- "rose suite-run" - need plugin-based file-installation, e.g. to support fcm
  extract. MS says: I guess you are talking about supporting 3rd party file
  install schemes? The current functionality in Rose is already modular and
  plugin friendly. With the packaging change, it should become very easy to
  install plugins.

- rose-suite.conf should be an importable Python file.
  - Jinja2 variables in rose-suite.conf are currently copy-and-pasted into the
    install suite.rc file. Instead we should define those variables in a Python
    module, and import it in the suite.rc (which is now possible:
    https://cylc.github.io/doc/built-sphinx/suite-config.html#importing-additional-python-modules)
  - MS: I suppose we can also enhance the --set-file=... option of the cylc run
    command to do this?

- UIS has to see source dir
  - need to see diffs of src vs run-dir
  - MS: could use git (whole suite, minus work tree etc.); tie commits to runs,
    restarts, reloads.

- Agreed(?) to proposed new default installation dir structure:
   ```
    src ---> src/run1
                /run2
                /run3
                ...
    ```
    -  every cold-start creates a new run-dir by default
    - with symlink to src, so that cylc knows it is a template suite.
    - reload should show a diff of old vs new
    - in the UI gscan sidebar: collapse on src suite; select src for a new
      run; select run1 to stop suite etc.
    - housekeeping: select to delete runX
    - show latest runs and/or running suites by default
    - show out-of-date config (with respect to src)
    - only support editing the source suite via UI, at least initially
      - although separate run-dirs make run dir editing safer now
      - edit run-dir, save and reload
    - need to be able to validate and graph the src dir (unlike now)
    - use `run%d` by default, but allow users to specify individual run names if
      they want to.
      - MS: Allow users to parameterize run labels, with the above as default?

  - MS: I have written the following in the Trello board. Probably lost in the
    ether now. Considerations:
     - How to easily start multiple runs with multiple versions (or tree-ish objects)?
     - How to easily start multiple runs with multiple configuration sets?
     - How to easily start multiple runs with multiple ad-hoc changes?
     - Where are the settings stored? How to present these to users?
     - Version control of configuration on run and reload.
     - How to apply changes to source to all current runs?
     - Configurable run names.
     - Plugin to improve UX for version controlled source.
     - Multiple sources. Source direct from a VCS or online resource?
     - Managing user expectation on any major changes.
     - If a suite is considered a normal software project, running it should be
       considered an installation/deployment issue like any other software
       project.

- UI form for Jinja2 params
- Note need static graph vis too, in Cylc 8 (tell MR)

- TODO: can we "query the API" over GraphQL (UIS-WFS) so to allow multiple
  cylc-flow versions on the back end?

- OS agrees with DS PRs in large part; GraphQL schema looks good.
  - not convinced on protobuf yet (but see below)

- "stop suite" should report to user "waiting for active tasks to finish"
  (e.g.) and:
  - report the list of active tasks it is waiting for?
  - provide hint to users about other stop options?
  - (See also #1032. Reason for shutdown should be recorded.) 

- GraphQL has typing too (like Protobuf); not sure  if validation/checking is done
  - BK wants types in the UI for safety/robustness

- FUTURE: display multiple suites together with inter-suite triggers!
  - great for small modular workflows

- DS PRs: push back functional and unit tests, and "granularise data
  structure", to follow-up devel.

- TODO: consider Travis CI cron jobs for cross-project integration testing.
  - use cylc/cylc-admin for this (T-CI config can pull from other projects)

- tests should run against master all the time, not release versions
  - need setup.py to install from branch URL not PyPi

- TODO (HO) - check JOSS paper repo link.

- SMS-like ghost nodes in all views? is a broken concept because it elevates
    "cycle points" to be more fundamental than they will be in Cylc 9 - some
    workflows might iterate over parameters first (or just wherever the graph leads).
   - we just need to educate users on the strengths of the cylc "adaptive
     window on an infinite graph" world view.
         
- What to display in the new UI:
   - first, as now, just display the task pool
   - then consider a new HYBRID MODE that looks a bit like spawn-on-demand will:
      - display N-edges out from "active tasks", in all views.
      - at start-up request N edges out, populate from suite and DB (only need to
        connect to DB at start-up)
      - "data display" technology if needed later (??Que??)
      - "level 0" is "active tasks" (submitted, running, satisfied, ... if
        prereqs are met OR "cannot be met")
      - allow configurable n-level zoom
      - max-n could be determined by query time on page
      - TODO ask DS: are n=1,2,3 edges already paged?
      - max tasks permitted per page?
   - UIS can query the DB to populate ghost nodes (and in future for "exploring
     the graph" beyond active tasks)
   - use distinct edge style to indicate that manual intervention forced
     override of prerequisite status.
   - failed tasks should need to be "dismissed" (not "removed"!)
   - show a separate list of failed tasks that have not been dismissed yet
   - failed tasks should not be treated differently to succeeded in terms of
     workflow
       - instead of keeping them around in the task pool should we have
          a different failure alerting system?
        - need to distinguish tasks that are expected to fail
           - FUTURE: generalise this to allow users to specify "expected
           completion state" of all tasks (default to expect success)
   - NOTE two kinds of ghost nodes!
      1. graph node with no corresponding task proxy currently in the pool
         (this kind will cease to exist in the spawn-on-demand future).

        - MS: Nodes N-generations away from active tasks. Can be N-parents or
        N-children. Should these be ghost tasks? Or just call them inactive or
        non-current tasks?
        - HO: I think just "active" and "inactive" will do in the future. Ghost
        implies something a bit mysterious, which "not present in the task
        pool" is. But "active or not" should be much less mysterious to users,
        I hope.

      2. "wrong graphing": intercycle dependence on an undefined task
        - MS and HO: just call these "undefined tasks"

------

UI Session

- need themes (e.g. color-blind) at 8.0.0
  - store in Hub?
  - store on disk?
  - store in cookies?  (not browser-portable)
  - preferred option: hub preferences service (Hub Sqlite)

- gscan sidebar: 
  - filtering
  - collapse hierarchy: just combine all suite badges/icons to get the group status summary 
  - collapse/expand state should be a preference
  - don't persist current preference; revert to stored is better
  - select two suites: see two suites in same tree view!
    - future graph view: with inter-suite triggers

- keep preview view alive (data feed), e.g. for 20 secs, in case user returns to it.

- UI data feed needs to be flat list of nodes (proxies and ghosts); edges;
  family structure, so data model can be shared by all views.
  - graph view = tree view + edges

- note the tree view should support a "flat list" (no family info) mode, like current GUI.

- Graph view (MR): needs to use same task and job icons etc. as other views.

- DS: can we request the "active view" and abstract this out (in hybrid mode
  this will be active tasks + level 1 etc.); later swap in the spawn-on-demand
  selected graph window.

- for now, each node could have a flag to say "I'm not a ghost node"? (there is
  a task proxy behind me).

- GLOSSARY
  - needed!
  - e.g. WFS = "workflow server"  (DM prefers "server" to "service")

- OS will do a UI design doc; SB will link to it in the Discourse UI feedback post

- UIS may need to query the user's group (from the OS).

- Firefox F12 (or Ctrl-Shift-M) device sim mode (mobile etc.) ... Cylc UI on
  smart watch, he heh.

- DM: found non-root J-Hub pretty straightforward to set up.
- Note we need to be able to start UIS as other users (I want to look at other
  user who does not have a UIS at the moment).

- UIS could have a query to list other users that I am authorized to see?
  - (If Hub-level auth for this is not feasible)

- TODO immediately after first e2e system:
   default authorization should be owner-only.

--------

Job to UIS/WFS authentication/trust:

- one-time or timed token
- trust the OS/FS
- same for encryption (use the same token)
- don't transport the token, stays on disk (except for remote jobs - need to
  install on remote host, one-time token best for this)
- event-driven expiry is probably the way to go (e.g. expire when job finished)

- Interactive CLI needs a timed token (on hosts with shared FS)
- Remote jobs- timed or one-off  

-----------

Cylc Workshop 2019/2020
- funding agreed by UMPB
- prob Dec 2019 or Jan 2020
- location?  Maybe Singapore?
- Need decent IT and WiFi!

-------------

GraphQL:
- note graphene (Python client/server) is much less powerful than Apollo (JS)
- request for "hold all of my suites": "workflows/*, status=held"
- (what about: hold all tasks submitting to host X?)

----------

Protobuf discussion (as implemented by DS already):
- decision: Protobuf is fine for the current model where UIS is *mirroring* all
  data in the "active view" of WFS.  GraphQL backend potentially better *if* in
  future we can make the UIS merely a transparent conduit for exactly what
  subset of the data that the UI requests.
- still need protobuf feed somewhat granularized to level of nodes and edges,
  so UIS doesn't store edges if no one is looking at the graph view.
- Ask DS not to granularize further than this, if not strictly needed.
- (See DS quotes below)

---------

Future, Cylc 9:
  - Python API gives modularity
  - parameterizing sub-graphs 
  - spawn-on-demand: avoids need for multi-dimensional cycling (just follow the
   graph, whether it is generated by cycles or parameters)
  - Housekeep - port `rose_prune` to Cylc, and improve
  - Run a sub-graph as a single job
  - WFS populate in memory DB (redis etc.) ... then UIS can just query the DB?

--------

Other

- UI auto-hiding of side-bar - flex box
  - (HO: note this is already working with BK's UI - tested!)

- TODO - how to get command results back?

- Note UK summer hols: 24 July - 5 Sept, ish.

- Default access to UIS "cylc review" read only

----

Cylc-8 release schedule?
- Cylc-8.0.0 first release candidate for researchers
- OR fully production-ready?
- need a series of beta releases first

- Need to investigage Discourse email digest settings (are users getting
  notifications if not checking the web site?)

------
DS Notes:
> I've already discussed some of the questions/features, but just thought I
> would responed to other points relevant to me:
> 
>>  - GraphQL has typing too (like Protobuf); not sure if validation/checking is done
>>  - BK wants types in the UI for safety/robustness
> 
> As discussed with Bruno, GraphQL does request/query validation (both field
> and Arg/Var type checking) and type validation on data returned by the
> resolvers..
> This covers all bases, and means you don't have to follow any bread crumbs in
> finding errors.. The fact that both protobuf and graphql definitions are in
> cylc-flow makes it quite robust (both WRT development and production).
> 
>>  GraphQL:
>> 
>>    - note graphene (Python client/server) is much less powerful than >Apollo (JS)
> 
> We're not using python client, but would agree Graphene is probably not as
> "powerful" as Apollo GraphQL Server..
> There's technology out there to write language agnostic GraphQL schema, so
> cylc-flow could possibly still share it's schema with a non-python
> implementation, and with use of a Redis like data store perhaps we could
> migrate the UIS/Tornado to something like Apollo GraphQL Server... However:
> 
>   - It would need to play nice with Jupyterhub (unless the UIS is split in two).
>   - I think it could interact with ZeroMQ? (https://github.com/zeromq/zeromq.js/ ?)
>   - We would loose other functionality imported from cylc-flow
>     (scanning/spawning suites, ZeroMQ server/client code ...etc)
> 
> So not sure this is worth entertaining unless we are suffering in performance
> after optimisation (or there is other interplay from having Apollo client &
> server)..
> 
>>  -  request for "hold all of my suites": "workflows/*, status=held"
> 
> Already capable... Use the GraphiQL interface docs to find out (or just ask!)
> 
>>  - (what about: hold all tasks submitting to host X?)
> 
> Would need to add an arg for host, and possibly a field to taskProxy (and
> that could be cross/multi-workflow also)
> 
>>  Protobuf discussion (as implemented by DS already):
>> 
>>      decision: Protobuf is fine for the current model where UIS is >mirroring all
>>      data in the "active view" of WFS. GraphQL backend potentially >better if in
>>      future we can make the UIS merely a transparent conduit for exactly >what
>>      subset of the data that the UI requests.
> 
> I think I may have spoken to this point. We don't want to place strain on the
> WS by the GUI (something BOM also stressed) and it becomes highly
> inefficient/impractical for multi-workflow queries... If you still want
> GraphQL at the UIS (it not just be a proxy to the WS), in order to unify the
> CLI we import the schema from cylc-flow so you can't have both..
> 
>>   - still need protobuf feed somewhat granularized to level of nodes >and edges,
>>   - so UIS doesn't store edges if no one is looking at the graph view.
>>   - Ask DS not to granularize further than this, if not strictly needed.
>
> Not sure there's a massive benefit from doing this... I think we could
> definitely only sync when a user is connected.. In the spawn-on-demand era we
> may have to generate edges and nodes on the fly from the workflow pattern
> (else looking into the future might be hampered), but we still have to store
> "what happened" somewhere..
>
>>    Future:
>>
>>   - WFS populate in memory DB (redis etc.) ... then UIS can just query >the DB?
> 
> Could be beneficial if directly exposed (like redis), then no need for data store sync with WS => UIS (as they would both use this).
