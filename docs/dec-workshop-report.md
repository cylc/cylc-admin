![UMC Core Partner Logos](img/umc.png)
![ESiWACE - Cylc - Altair Logos](img/esiwace-cylc-altair-logos.png)
![UMC Assoc Partner Logos](img/umc-associate-logos-2018.jpg)

# 3-7 December 2018 Cylc Development Workshop Report
__Hilary Oliver,__ January 2019

## Table of Contents

- [Executive Summary](#executive-summary)
- [Workshop Goals and Agenda](#workshop-goals-and-agenda)
  - [Background](#background)
  - [Goals](#goals)
- [Workshop Outcomes](#workshop-outcomes)
  - [Communications and Working Practices](#communications-and-working-practices)
  - [Cylc-8 Architecture](#cylc-8-architecture)
  - [Cylc-8 Implementation](#cylc-8-implementation)
- [Acknowledgements](#acknowledgements)

## Executive Summary
[TOP](#table-of-contents)


In December 2018 the Australian Bureau of Meteorology (BOM) hosted the
first-ever Cylc development workshop. Delegates attended from NIWA (3), Met
Office (3), ESiWACE (1), BOM (1), and Altair Engineering (1). Travel funding
was generously provided by the Unified Model Partnership, ESiWACE, and Altair.

The primary goal of the workshop was to plan how to port Cylc from Python 2 to
Python 3, with the complication that the current Cylc (and Rose) GUIs need to
be completely replaced because they are based on a now-obsolete framework
(PyGTK) that will never be ported to Python 3. This has to be done with some
urgency due to the impending demise of Python 2, which will officially be
retired at the end of 2019. The week in Melbourne also provided an ideal
opportunity to introduce new team members, and to meet the BOM and Altair staff
building NWP production infrastructure around PBS Pro, Cylc, and Kafka, and to
learn about that project.

The workshop was a great success. We settled (with caveats) on communication
and working practices for the widely distributed team, and worked through a
porting plan and a radically new Cylc architecture to support a modern web UI,
with site identity management integration and a single point of access for
users.

See below for more information on background, the workshop agenda, and outcomes
including a link to Cylc-8 architecture documentation.

## Workshop Goals and Agenda
[TOP](#table-of-contents)

### Background
[TOP](#table-of-contents)

The official Python 2 end-of-life date is 31 December 2019. After that the
language will no longer be developed or maintained, not even for security
patches. Even worse, the current Cylc GUIs,
which embody several developer-years of effort, are built on the obsolete PyGTK
framework which will never be ported to Python 3. Major Linux distributions
have already announced they will soon be shipping without Python 2 and PyGTK.
Cylc is still Python 2-based as we have not previously had the
resources to address these major issues. 

So there is an urgent need to migrate Cylc to Python 3 and entirely replace the
current GUIs with something more modern, which by general agreement should be a
web UI. This is a challenging task that requires new system components and a
complete rework of Cylc's local client-server architecture, which is not
web-compatible (current clients make use of the local system in ways that
browsers do not allow, for instance). And the first major release of the new
system, Cylc-8, must be performant “out of the box” for existing critical
production suites.

### Goals
[TOP](#table-of-contents)

The primary workshop goal was (therefore) to:

 * plan how to port Cylc to Python 3, and re-architect it with web technologies
   to support an in-browser UI with a single point of access and site identity
   management integration

Toward this end, sub-goals were to:

 * discuss how, as a widely distributed team, to communicate effectively
   and work together on this multi-faceted project
 * decide on the overall architecture of the new solution - the system
   components and how they relate to one another
 * decide which Javascript framework and associated technologies to use for
   the front-end UI
   * including aspects of UI design such as accessibility and new ways to
     display complex suite status information
 * decide on communications protocols and related technologies to connect the
   various components of the new architecture
 * demo and discuss relevant technologies that have already been
   investigated by participants
 * consider intermediate milestones to be achieved along the way
 * decide who will work on which components, in what order
 * consider if the team is big enough now to allow us to make a start,
   concurrently, on other roadmap items beyond cylc-8 

The [WORKSHOP
AGENDA](https://cylc.github.io/cylc-admin/dec-workshop-agenda#agenda)
shows the topics covered each day during the week, to support these goals.

## Workshop Outcomes
[TOP](#table-of-contents)

### Communications and Working Practices
[TOP](#table-of-contents)

Technical working practices centered on git and GitHub were discussed, but
already well-established in the project. With new team members and people
located all over the world, however, we agreed on the need for regular video
conferences (monthly); instant communication via a chat platform of some kind;
possibly a replacement for our old Google Groups email forum (mainly used for
release announcements and infrequent questions from remote users); and
something like Trello boards to help with project management as well.
Unfortunately it is not so easy to solve these problems because our respective
institutions all use mutually incompatible video conferencing technologies and
have wildly different restrictions on what can be used on site. Popular
platforms such as [Slack](https://slack.com) also place significant
restrictions on free accounts (such as loss of history beyond some point) or
charge a lot for full functionality.

We decided to:

* use the up-and-coming free Open Source [Riot.im](https://about.riot.im/) chat
  platform for day-to-day communications, and video conferencing with chat
  Room members.
* consider replacing the Cylc Google Groups forum with
  [Discourse](https://www.discourse.org/) (we were subsequently successful in
  our application for a free Discourse instance as an Open Source project)
* consider GitHub Projects or [Trello](https://trello.com) for project
* management and task tracking

(Since the workshop one of our major sites has decided to block Riot.im for
security reasons, forcing some of us to use personal devices to stay in touch
... so effective communication remains challenging).

### Cylc-8 Architecture
[TOP](#table-of-contents)

We succeeded in working through the detailed Cylc-8 architecture, including all
components, technologies, and communications protocols needed to implement the
new system. The most visible change will be a major new system component: a "hub"
that acts as a single point of access for users, handles authentication
via site identity management plugins, spawns suite UI Servers (which in turn
spawn Workflow Services - i.e. suites) into user accounts, and then proxies
requests to them. Beyond this, a variety of new technologies and communications
protocols will be used to build and connect the various system components.
These include WebSocket (instead of HTTPS) between UIs and UI Servers; the
Vue.js Javascript framework, a GraphQL (instead of REST) API to serve the suite
status data model, and the ZeroMQ network library to connect UI Servers with
Workflow Services at the back end. Naturally, many details - such as secure
backend communications - remain to be worked out during implementation.

The new architecture is written up in some detail, and illustrated, in a
separate report: [CYLC-8
ARCHITECTURE](https://cylc.github.io/cylc-admin/cylc-8-architecture).

### Cylc-8 Implementation
[TOP](#table-of-contents)

The implementation plan can be summarized as follows.

First:
1. finish of critical Cylc-7 developments and release cylc-7.8.0,
   then make a 7.8.x branch for ongoing Cylc-7 maintenance

Then begin Cylc-8 development:
1. implement formal test coverage reporting, and improve coverage where
   possible, to support the Python 3 porting effort
1. port our ISO 8601 date-time library to Python 3
1. delete the old GUIs and port the rest of Cylc to Python 3...
1. ...replacing the Cherrypy network library in the suite server program with
   ZeroMQ
   - result: a working Python 3 Cylc with no GUI and local
     access only; this will provide a platform to build the
     rest of the system on.
1. Python package management (`pip install cylc` etc.)

Concurrent with Python 3 porting:
1. determine whether or not JupyterHub be used "out of the box" as the new Cylc Hub,
   and if not, can it be modified to our purposes, including:
     * user authentication and site plugins
     * spawning user processes
     * access to other user's suites
1. begin work on the UI Server, with Vue.js and other technologies
     * server-facing comms layer: ZeroMQ
     * UI-facing comms layer: Tornado (asynchronous Python web framework), 
       WebSocket, GraphQL
     * sub-services for suite start-up and discovery etc.
1. understand secure communication and session management
1. (and more - see the aforementioned architecture document for details)
1. (and if time allows, continue the mostly-orthogonal "rose suite-run" migration
  and cluster-awareness work)

## Acknowledgements
[TOP](#table-of-contents)

Thanks to BOM for hosting the workshop; and the UM Partnership, ESiWACE, and
Altair for providing travel funding.

![UMC Assoc Partner Logos](img/umc-associate-logos-2018.jpg)
![ESiWACE - Cylc - Altair Logos](img/esiwace-cylc-altair-logos.png)
![UMC Logos](img/umc.png)
