![UMC Logos](img/umc.png)
![ESiWACE - Cylc - Altair Logos](img/esiwace-cylc-altair-logos.png)

# Cylc Development Workshop 
__Bureau of Meteorology, Melbourne, Australia__ 

3-7 December 2018

## Table of Contents
- [Context](#context)
- [Format](#format)
- [Preparation](#preparation)
- [What to bring](#what-to-bring)
- [Goals](#goals)
- [AGENDA](#agenda)
- [Participants](#participants)
- [Acknowledgements](#acknowledgements)

## Context
Over the next 12 months we need to migrate [Cylc](https://cylc.github.io/cylc)
to Python 3, and replace the
aging PyGTK GUIs and simple client/server architecture with a more complex
(but more powerful) architecture and a web GUI. And the new system has to be
performant "out of the box" for existing critical production use. This is a
challenging task!!!

## Format
Mainly discussion sessions, with some demoing and coding (to the extent
possible at this early stage of development, and with our varying levels of
experience)- not formal presentations (except perhaps Wednesday afternoon).

## Preparation
Read and understand (as far as possible at this stage):
- [Cylc-7 Architecture](cylc-7-architecture.md)
- [GUI Replacement Options](gui-replacement-options.md)
- [Cylc-8 Architecture and Web GUI Roadmap](cylc-8-roadmap.md)
  - those of us who have researched particular aspects in depth should
    be prepared to talk about it
- A few Cylc Issues that are relevant:
  - [One GUI using technology for web applications](https://github.com/cylc/cylc/issues/1873)
  - [New GUI: scalable navigable view using quilts](https://github.com/cylc/cylc/issues/2753)
  - [suite http(s) server improvements](https://github.com/cylc/cylc/issues/2563) 
  - [Future Cylc Authorisation/GUI Architecture](https://github.com/cylc/cylc/issues/2729 )
- [Jupyter Hub Architecture](https://jupyterhub.readthedocs.io/en/stable/) - we
  think we need something very much like this (with _notebook_ = _suite daemon_)
- Consider what you might like to work on, and how much effort you can
  contribute in the next year or so.

And for the new people:
- Understand how to define, run, and interact with basic Cylc workflows,
  non-cycling and cycling
- Work through the basic tutorial in the [Cylc User
  Guide](https://cylc.github.io/cylc/documentation.html) and the better
  Cylc tutorial in the newly-rewritten [Rose User
  Guide](https://metomi.github.io/rose/doc/html/index.html) 
 
## What to bring
- Your development laptop, with a Cylc repository clone on board and tested
- Working demos or other material to present, if you have any

## Goals
By the end of the week we need to understand:
- How we will work together to develop the new system
  - in principle we have a bigger team than ever before, and we should be able
    to make great progress
- Who will do what?
- Where technological or architectural choices have to be made,
  - which way will we go?
  - or (at the least) how to decide very quickly which way to go
- How to develop and test the different components separately, given their interdependence
- When can intermediate milestones be achieved along the way?
- (Is the team big enough now that we can start on some important "future Cylc"
  work concurrently with GUI development? - see [Possible Cylc
  Futures](https://github.com/cylc/cylc/wiki/Possible-Cylc-Futures) and
  [Vision for Cylc beyond 2018/2019
  Priorities](https://github.com/cylc/cylc/wiki/Vision-for-Cylc-beyond-2018-2019-Priorities))

# AGENDA
Items listed below are to guide the discussion, but we can be flexible if
needed. Several of us are relatively expert on selected topics already and
can start by walking the rest of us through it. Tentative decisions have
already been made in a few areas (e.g. to use Vue.js for the GUI). We can
consider coding and demo opportunities on any day, if time allows.

### Monday
- __Morning__
  - (Combined session with the BoM/Altair Control Panel Stream)
  - Introductions, with interpretive dance
  - Overview and showcase of work and plans on the BoM/Altair/Cylc project
    (Control Panel, Apache Kafka, reporting DB, authentication etc.)
  - Then: begin afternoon topics early, if possible

- __Afternoon__:
  - (development working practices - we have a bunch of new team members!)
  - [Sadie's write-up](practices-prompts)
  - git, GitHub, GitHub Flow, testing, Travis CI, Codacy
  - 2FA about to be enforced on cylc repository
  - Riot.im? (UK team now has access); or Slack?
  - Define Python code style guidelines (add to CONTRIBUTING.md)?
  - Development roadmap: what's been discussed and decided so far?
  - Then: begin Tuesday's Architecture discussion early, if possible

### Tuesday
- __All day__:
  - (Architecture)
  - System components:
    - GUI, "hub", reverse proxy, sub-services for suite discovery and start-up, etc.
    - other? GUI server, "suite state server", ...?
    - [suite discovery not needed?](
      https://github.com/cylc/cylc/issues/1873#issuecomment-416000070)  
  - How closely can we follow (and even borrow code from) Jupyter Hub?
  - What runs where? Privileged, or as the user?
  - Server-side [Python web frameworks](https://steelkiwi.com/blog/top-10-python-web-frameworks-to-learn/)
    - [Flask](http://flask.pocoo.org/) (+gevent?)
    - [Tornado](https://www.tornadoweb.org/en/stable/) (async even at Python 2; "ideal for websocket or long polling")
    - ([popularity](https://python.libhunt.com/compare-tornado-vs-flask?rel=cmp-cmp))
    - Other? Django; [Sanic](https://github.com/huge-success/sanic) (flask-like async); [AIOHTTP](https://aiohttp.readthedocs.io/en/stable/)
  - Inter-component communication - network protocols and API(s)
    - WebSocket and GraphQL seem advantageous (c.f. HTTPS and a REST API) but
      do they need to go all the way from the GUI to the suite daemons? If not,
      what are the implications?
    - alternatives at the back end? ZeroMQ, Protocol Buffers (& gRPC?)?
  - Suite server data structures
    - are lists of nodes and edges sufficient for all views? 
    - exposed via GraphQL? (by the suite server, or a GUI server?)
  - GraphQL demo with cylc-7 (David) 
  - Do we need a simplified architecture for individual use?
    - or just run all components as the user?

### Wednesday
- __Morning__
  - (Combined session with the BoM/Altair Control Panel Stream)
  - Software testing, packaging and distribution
  - Scheduler prediction tool
  - Cylc `setup.py`
  - (Begin afernoon topics, if possible)

- __Afternoon__
  - (BoM Cylc user feedback and discussion - if needed)
  - Deficiencies, feature requests, problem solving
  - A presentation or two, if called for
    - e.g. the Brussels Worklfow Workshop Cylc and Rose keynote presentation?
  - Testing Cylc with Docker

### Thursday
- __Morning__
  - (Authentication and authorization)
  - User authentication: site integration (plugins?), session management
  - Session management for CLI commands?
  - Automatic job authentication: one-time tokens?
  - If authentication is done by the Hub, how do suite daemons trust the Hub?
    - SSL client certificate?
    - what if not using HTTPS?
  - Authorization - how to do it?
- __Afternoon__
  - __Begin the Web GUI discussion__ (see Friday morning)

### Friday
- __Morning__
  - (Web GUI)
  - Which Javascript framework: Vue.js (or React.js, or...)?
    - [A recent comparison](https://belitsoft.com/front-end-development-services/react-vs-angular)
  - Typescript? [Maybe not](https://medium.com/javascript-scene/the-shocking-secret-about-static-types-514d39bf30a3)
  - Oliver's UI design ideas
    : [one](https://github.com/cylc/cylc/issues/1873#issuecomment-405373915)
    : [two](https://github.com/cylc/cylc/issues/1873#issuecomment-417481655)
    : [three](https://github.com/cylc/cylc/issues/1873#issuecomment-419742930)
    : [four](https://github.com/cylc/cylc/issues/1873#issuecomment-420084752)
    : [graph view](https://github.com/cylc/cylc/issues/1873#issuecomment-423555699)
  - [Sadie's mind maps!](https://github.com/cylc/cylc/issues/1873#issuecomment-421856260)
  - Provide a small number of built-in themes? or give users control over the look?
  - Accessibility
  - Do we need (/can we get?) professional UI design advice?
  - How to display very large suites effectively and efficiently
  - Interaction with other components:
    - To suites (via proxy) or to a UI server?
    - Incremental update of suite state data?
    - WebSocket? - no polling by GUI!
    - GraphQL?
  - Who serves the actual HTML pages (as opposed to just data)?
    - suite daemon, or GUI server? (covered in "Architecture" above)
  - Automated UI testing?

- __Afternoon__
  - (Tie it up and nail it down, with reference to the Workshop [Goals](#goals))
  - Retro
  - Review of risks
  - Delivery timeline
  - Who can work on what?

# Participants

- Hilary Oliver - [NIWA](https://www.niwa.co.nz), (Wellington, New Zealand) - <hilary.oliver@niwa.co.nz>
- Bruno Kinoshita - [NIWA](https://www.niwa.co.nz), (Auckland, New Zealand - <bruno.kinoshita@niwa.co.nz>
- David Sutherland - [NIWA](https://www.niwa.co.nz), (Wellington, New Zealand) - <david.sutherland@niwa.co.nz>
- Raghavendra (Raghu) S. Mupparthy - [NCMRWF](http://www.ncmrwf.gov.in), (Noida, India) - <raghav@ncmrwf.gov.in> 
- Dave Matthews - [ESiWACE](https://www.esiwace.eu), [Met Office](https://www.metoffice.gov.uk), (Exeter, UK) - <david.matthews@metoffice.gov.uk> 
- Sadie Bartholomew - [Met Office](https://www.metoffice.gov.uk), (Exeter, UK) - <sadie.bartholomew@metoffice.gov.uk>
- Matt Shin - [Met Office](https://www.metoffice.gov.uk), (Exeter, UK) - <matthew.shin@metoffice.gov.uk>
- Oliver Sanders - [Met Office](https://www.metoffice.gov.uk), (Exeter, UK) - <oliver.sanders@metoffice.gov.uk>
- Martin Ryan - [BoM](https://www.bom.gov.au), (Melbourne, Australia) - <martin.ryan@bom.gov.au>
- Sujata Patnaik - [Altair](https://www.altair.com), (Bangalore, India) - <sujata.patnaik@altair.com>

# Acknowledgements

Thanks to BoM, Altair, the UM Partnership, and ESiWACE for sponsoring this
workshop, by providing a venue, and/or travel funding, and/or development
resourcing! 

![ESiWACE - Cylc - Altair Logos](img/esiwace-cylc-altair-logos.png)
![UMC Logos](img/umc.png)


