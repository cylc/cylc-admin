![Cylc Workshop 2020 Logos](img/logos-2020.png)

# CylcCon 2020 Report
## Cylc Development Workshop 10-14 Feb, NIWA, Wellington, New Zealand

**Hilary Oliver, NIWA**

[Participants](feb2020-workshop-agenda.md#participants)
attended from NIWA, Met Office, BOM, and NRL.

**Acknowledgement:** Thanks to NIWA for hosting, and the UM Partnership and
IS-ENES3 for funding travel for some participants.

## Table of Contents

- [Goal and topics](#goal-and-topics)
- [Executive summary](#executive-summary)
- [Progress to date](#progress-to-date)
- [Prioritization of remaining tasks](#prioritization-of-remaining-tasks)
  - [Essential for Cylc 8.0](#essential-for-cylc-80)
  - [Nice to have for Cylc 8.0](#nice-to-have-for-cylc-80)
  - [Cylc-8.1](#cylc-81)
- [Tentative development timeline](#tentative-development-timeline)

## Goal and topics

The primary goal of the workshop was to review Cylc 8 progress and chart the
roadmap to completion by early 2021.

For specific discussion topics covered during the workshop and links to
relevant background material see the [Workshop
Agenda](feb2020-workshop-agenda.md). For additional detail
see [workshop notes by HO](feb2020-workshop-notes.md) and
[JR](feb2020-workshop-notes-jr.md).

## Executive summary

Python 2 end of life, obsoletion of the PyGTK GUI toolkit, and a near-universal
modern preference for web UIs over traditional desktop GUIs, has led to an
urgent need to re-engineer Cylc to future proof our investment in workflow
orchestration.

The new [Cylc 8 architecture](cylc-8-architecture.md) was agreed during the
[December 2018 Cylc workshop](dec-workshop-repord.md) at BOM. Much has been
achieved since then despite some unexpected resourcing issues (staff turnover
etc.). At this point the back-end workflow engine and CLI have been migrated to
Python 3, and the major components Cylc 8 and the connections between them
are all in place and working. It is already possible to run workflows
of moderate size with Cylc 8 and view and control them in the new web UI. For
details, see [Progress to date](#progress-to-date) below.

However, there is still much to do before Cylc 8 is production-ready. Every
component needs detailing, extensive testing, and performance optimisation; and
some important sub-systems are still missing. For example, the primary tree
workflow status view does not yet perform well enough for large workflows;
control functionality is not integrated with the view components; we don't have
efficient incremental data updates flowing through to the UI; some aspects of
security have not been addressed; and we haven't implemented fine-grained
authorization yet. For details of what remains to be done, see [Prioritization
of remaining tasks](#prioritization-of-remaining-tasks).

*We are aiming to have a major preview release of Cylc 8 ready for widespread
testing in September of this year, and to officially release cylc-8.0 in
the first quarter of 2021.*

## Progress to date

(First read the [Executive summary](#executive-summary).)

A recent screenshot of the new web UI showing live workflow data:
![Cylc 8 web UI as of February 2020](img/cylc-8-ui-feb2020.png)

As of February 2020:
- The previously monolithic Cylc project has been split into multiple modular
  projects on GitHub (primarily because the UI is no longer Python based)
- Our new communications platforms Riot (developer chat and video
  conferencing) and Discourse (mail forum) are proving effective
- Web UI design settled after consultation with users (we anticipate
  more feedback during user testing)
- Packaging and installation is done, with `pip` for Python
   components and `conda` (via Conda Forge) for the full system
- Major Cylc 8 components and connections implemented and working (but not
  finished):
  - Cylc Hub: the central point of access for user authentication, a
     JupyterHub instance configured to spawn Cylc UI Servers
  - Cylc Flow (scheduler and CLI) fully migrated to Python 3
  - Cylc UI Server: takes data from the workflow schedulers and 
  - Cylc web UI: the new Javascript (Vue.js) in-browser UI
  - Scheduler to UI Server data pipe: Protocol Buffers over ZeroMQ 
  - UI Server to web UI data pipe: GraphQL over Websocket
- We are beginning to take advantage of new Python 3 features (e.g. entry
  points for extensibility via proper plug-ins)
- Much of the new web UI is implemented:
   - The primary tree workflow status view is well on the way to completion
   - The integrated gscan sidebar is working for live workflows
   - The dashboard is ready to be populated with useful information
   - Flexible tabbing of multiple workflow views is in place
- Control capability has been implemented in the web UI, but not integrated
   into the primary views yet.
- Back-end authentication (between CLI and workflow schedulers) is mostly done

## Prioritization of remaining tasks

(First read the [Executive summary](#executive-summary).)

The following items were thoroughly discussed, and problems worked out, in the
workshop. Now they just need to be implemented or completed.

### Essential for Cylc 8.0

- Do we go ahead with [Spawn on Demand](proposal-spawn-on-d.md)?
  - If yes: SoD solves many problems of efficiency and usage, but needs
    n-distance windowing to show nodes beyond the scheduler task pool
  - If no: we probably need to replicate the Cylc 7 task pool window and
    continue to live with its many deficiencies until Cylc 9
- Event-driven incremental data update throughout the system
- Do we stick with the flat shared UI data store plan or allow views to
  request their own data in the form they need via GraphQL?
- Complete [task state name rationalisation](proposal-state-names.md)
  throughout the system
- Performant virtual-scrolling UI tree view
- Integration of control functionality in the UI (currently available
  via a special "mutations view")
- Display (in the UI) held and queued badges, retries and xtriggers, etc.
- Cross-user functionality: how to get this via the Hub and/or the UI?
- Authorization: configuration and implementation
- CLI - UI Server (for other-user workflows, and remote jobs)
- Platforms support and associated config changes (including config file names)
- `rose suite-run` migration
- Restore "ssh messaging" via ssh tunnel with ZMQ
- Solve UI Server back-compatibility
- New `cylc run` command names and semantics, and "run1/2/3..." run directories
- Workflow view window
  - If spawn-on-submit: the scheduler task pool and ghost nodes, as now
  - If spawn-on-demand: n-distance window, with n=0 as the scheduler pool
  - (plus option to show whole cycles in SoS or SoD)
- `cylc review` (Rose Bush) functionality:
  - contingency: make the old Python 2.7 system compatible with Cylc 8
  - preferably: integrate with the new UI (but must be as convenient to use
      as the old system for historical data)
- Review User Guide and tutorials for Cylc 8 
- New UI User Guide (in the UI)
- Document use of central `cylc` wrapper with Cylc 8
- Document security-related aspects of the new architecture (including
  Hub-proxy comms)

### Nice to have for Cylc 8.0

These can be bumped to 8.1 if necessary, but we'll do as much as we can to get
them done before then:
- Graph view conforming to our common design requirements
  - Probably requires dagre layout + custom display engine
  - Current window, and n edges around selected tasks by request
- Dot view (concise view of spread over cycle points)
- Flat table view (to allow sorting by job parameters such as submit time)
- Selection sharing/linking between views
- Display command progress (not just "command queued")
- Consider load balancing with multiple UI Servers (just use current `cylc run`
  mechanism?)
- Minimal data stores (UIS should just store what is subscribed to, not the
  whole scheduler pool)
- Scheduler main loop refactor: asyncio co-routines
- User Preferences via the Hub
- Consider "pre-graph" tasks for file installation
- Change "batch system handler" to "job runner", and re-implement as plugins
- New Urwid `cylc monitor`: if performant for big workflows
  - (Need to address Tim W's objection to this)
  - add control functionality
- Multi-user gscan (this goes beyond the cross-user functionality above)
- Cylc Hub as an entry point plugin of the UIS package (just J-Hub, but needs
  to pick up our config file automatically)
- Minimal install for job platforms

### Cylc 8.1

- Finish off remaining main Cylc 8 items, including [Nice to have for Cylc
  8.0](nice-to-have-for-cylc-80) (above)
- Other workflow views (Gantt, Quilt, ...)
- Allow starting the UI in read-only mode with deliberate escalation required
  for execute and read-write mode
- ...
- Begin Cylc 9 development

## Tentative development timeline

The following are endpoints to aim for, for the major tasks. (Smaller
tasks above are not listed here as they can be fitted in around the bigger
ones as time allows). Many of these items are already in progress or have at
least been planned in advance. Initials have been given for those leading the
development, but others may also be involved, and we have to be flexible
depending on availability.

- March 2020:
  - flat shared UI data store decision (OS, BK)
- April 2020:
  - spawn-on-demand decision (HO, DM, OS)
  - performant tree view with virtual scrolling (BK)
  - incremental data update (DS)
  - complete spawn-on-demand, if agreed, e.g. adapt the test battery (HO)
- May 2020:
  - UI integrated control functionality (BK, OS)
  - Complete remaining UI design elements - xtriggers etc. (BK, OS)
- June 2020:
  - platforms support and config changes (TP, MH)
  - authorization at the UI Server (HO)
  - thoroughly document security (JR)
- July 2020:
  - UI Server access to historical data within the view window (DS)
  - cross-user functionality (BK, OS)
- August 2020:
  - `rose suite-run` migration (TP, OS) including new run command semantics and
    run directory structure (HO)
  - `cylc review` decision - integration or contingency? (OS)
- September 2020:
  - review and upgrade all documentation and training material (ALL)
  - preview release on Conda Forge, encourage heavy testing :tada:
- October 2020-March 2021:
  - address feedback, bugs, performance, stability (ALL)
  - nice-to-have features, esp. the graph view
- March 2021 (or earlier):
  - Cylc 8.0 major release :tada:
- March-June 2021: continue to address feedback and bugs
- June 2021
  - Cylc 8.1 with the remaining "Nice to have for Cylc 8" features
- July 2021 onward: Cylc 9 development (Python API etc.)

![Cylc Workshop 2020 Logos](img/logos-2020.png)
