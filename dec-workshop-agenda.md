![UMC Logos](gfx/umc.png)
![ESiWACE - Cylc - Altair Logos](gfx/esiwace-cylc-altair-logos.png)

# Cylc Development Workshop 
__Bureau of Meteorology, Melbourne, Australia__ 

3-7 December 2018

## Table of Contents
- [Context](#context)
- [Format](#format)
- [Prerequisites](#prerequisites)
- [AGENDA](#agenda)
- [Acknowledgements](#acknowledgements)

## Context
Over the next 12 months we need to migrate Cylc to Python 3, and replace the
aging PyGTK GUIs and simple client/server architecture with a more complex
architecture and a web GUI. This is a challenging task!!!

## Format
Mainly discussion sessions, with some demoing and coding (to the extent
possible at this the early stage of development, and with our varying levels of
experience)- not formal presentations (except perhaps Wednesday afternoon).

## Prerequisites
Read and understand (as far as possible at this stage):
- [Cylc-7 Architecture](cylc-7-architecture.md)
- [GUI Replacement Options](gui-replacement-options.md)
- [Cylc-8 Architecture and Web GUI Roadmap](cylc-8-roadmap.md)
  - those of us with more experience should research particular issues in
    depth, if possible, and be prepared to talk about it
- A few Cylc Issues that are relevant:
  - [One GUI using technology for web applications](https://github.com/cylc/cylc/issues/1873)
  - [New GUI: scalable navigable view using quilts](https://github.com/cylc/cylc/issues/2753)
  - [suite http(s) server improvements](https://github.com/cylc/cylc/issues/2563) 
  - [Future Cylc Authorisation/GUI Architecture](https://github.com/cylc/cylc/issues/2729 )
- Consider what you might like to work on, and much effort you will be able to
  contribute in the next year or so.

And for the new people:
- Understand how to define, run, and interact with basic Cylc workflows,
  non-cycling and cycling
- Work through the basic tutorial in the [Cylc User
  Guide](https://cylc.github.io/cylc/documentation.html) and the better
  Cylc tutorial in the newly-rewritten [Rose User
  Guide](https://metomi.github.io/rose/doc/html/index.html) 
 
## Goals
By the end of the week we need to understand:
- How we will work together to develop the new system
  - in principle we have more effort than ever before and should be able to
    make great progress
- Where technological or architectural choices have to be made,
  - which way will we go?
  - or how will we decide very quickly which way to go?
- How can we test the separate parts of the new system, given their interdependence? 
- What intermediate milestones can we achieve (and when) along the way?
- Who will do what?
- (Is the team big enough now that we can start on some important "future Cylc"
  work concurrently with GUI development? - see [Possible Cylc
  Futures](https://github.com/cylc/cylc/wiki/Possible-Cylc-Futures) and
  [Vision for Cylc beyond 2018/2019
  Priorities](https://github.com/cylc/cylc/wiki/Vision-for-Cylc-beyond-2018-2019-Priorities))

# AGENDA
Items listed below are to guide the discussion, but we can be flexible if
needed. Several of us are relatively expert on selected topics already - they
can start by talking the rest of us through it.

### Monday
- __Morning__
  - (Combined session with the BoM/Altair Control Panel Stream)
  - Introductions, with interpretive dance
  - Overview and showcase of work and plans on the BoM/Altair/Cylc project
    (Control Panel, Apache Kafka, reporting DB, authentication etc.)
  - (then: start afternoon topics early if possible)

- __Afternoon__:
  - (Cylc Development and working practices)
  - (as we have a bunch of new team members!)
  - git, GitHub, GitHub Flow, testing, Travis CI, Codacy, Riot.im or Slack?, etc.
  - Then: begin the Architecture discussion (see Tuesday)

### Tuesday
- __All day__:
  - (Architecture)
  - System components:
    - GUI front end, "hub", reverse proxy, sub-services for suite discovery and
      start-up, etc.
    - other? GUI server, "suite state server", ...?
  - What runs where, privileged or as the user?
  - Server-side framework(s)
    - Flask (+gevent?) or Tornado?
  - How do the components communicate?
    - WebSocket and GraphQL seem advantageous (compared with HTTPS and a REST
      API) but do the need to go all the way from the GUI to the suite daemons?
    - what about other alternatives at the back end? ZeroMQ, Protocol Buffers (& gRPC?)?
  - How closely can we follow and/or steal from Jupyter Hub?
  - Suite server data structures, API, and network protocol 
    - REST or GraphQL?
    - HTTPS, or WebSocket, or ZeroMQ, or Protocol Buffers (& gRPC?)?
    - (note this overlaps with the architecture discussion, Tuesday)
  - Do we need a simplified architecture for individual use?
    - or just run all components as the user?

### Wednesday
- __Morning__
  - (Combined session with the BoM/Altair Control Panel Stream)
  - software testing, packaging and distribution
  - "scheduler prediction tool"
  - (Begin afernoon topics, if possible)

- __Afternoon__
  - (Cylc user feedback and discussion)
  - perceived deficiencies (of GUI or otherwise), feature requests?
  - A presentation or two, if needed (e.g. revist our Brussels Worklfow Workshop talk?)

### Thursday
- __All day__
  - (Authentication and authorization)
  - user authentication: site integration, session management
  - session management for CLI commands?
  - automatic job authentication: one-time tokens?
  - if authentication is done by the Hub, how do suite daemons trust the Hub?
    - SSL client certificate?
    - what if not using HTTPS?
  - authorization - how to do it
  - if any time left: coding and demos?

### Friday
- __Morning__
  - (Web GUI)
  - Which Javascript framework: Vue.js (or React.js, or...)?
  - unify gscan and gcylc?
  - UI design ideas
  - How to display very large suites effectively and efficiently
  - Interaction with other components:
    - To suites (via proxy) or to a UI server?
    - Incremental update of suite state data?
    - WebSocket? - no polling by GUI!
    - GraphQL?

- __Afternoon__
  - (Tie it up and nail it down, with reference to the Workshop [Goals](#goals))
  - Retro
  - Review of risks
  - Delivery timeline
  - Who can work on what?

# Acknowledgements

Thanks to BoM, Altair, the UM Partnership, and ESiWACE for sponsoring this
workshop, by providing a venue, and/or travel funding, and/or development
resourcing! 

![ESiWACE - Cylc - Altair Logos](gfx/esiwace-cylc-altair-logos.png)
![UMC Logos](gfx/umc.png)


