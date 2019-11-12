# Cylc-8 Architecture

_Last Updated:_ November 2019.

_Author:_ Hilary Oliver.

_Contributors:_ Dave Matthews, Matt Shin, Oliver Sanders, Sadie
Bartholomew, Martin Ryan, Bruno Kinoshita, David Sutherland, Sujata Patnaik. 

_This document is a primary output of the 3-7 December 2018 Cylc Development
Workshop at the Bureau of Meteorology, Melbourne, Australia_.

## Table of Contents
[TOP](#cylc-8-architecture)

- [Cylc Terminology](#cylc-terminology)
- [Technology Glossary](#technology-glossary)
- [Background](#background)
- [Cylc-8 Architecture Diagram](#cylc-8-architecture-diagram)
- [Cylc Hub](#cylc-hub)
- [Cylc UI Server](#cylc-ui-server)
- [Cylc UI](#cylc-ui)
- [Cylc Workflow Services](#cylc-workflow-services)
- [Command Line Interface](#command-line-interface)
- [Authorization](#authorization)

Appendices:
- [Deployment](#deployment)
- [Similarity with JupyterHub](#similarity-with-jupyterhub)

## Cylc Terminology
[TOP](#cylc-8-architecture)

- A Cylc __workflow__ is a single (possibly cycling) __suite__ of inter-dependent
tasks.

- A Cylc __workflow service__ is workflow manager program for a single workflow,
formerly known as a __suite server program__ or a __suite daemon__. (Cylc has
no central server - each workflow gets its own ad-hoc service that runs as the
user).

## Background
[TOP](#cylc-8-architecture)

Cylc-7 is written in Python 2, with PyGTK desktop GUIs and relatively simple
[local client/server architecture](cylc-7-architecture.md) in which
everything runs as the user, all clients are treated equally (user GUI and CLI,
and job CLI), clients get some server information via the filesystem and port
scanning, and automatic owner-only authentication via a suite-specific
passphrase file. Unfortunately:
- Python 2 end-of-life is 1 Jan 2020, after which ["there will be no [Python 2]
  updates, not even source-only security
  patches"](https://github.com/python/devguide/pull/344).
- PyGTK is Python 2 based and has not been updated for years.

We have decided __not__ to port the existing Cylc GUIs to PyGObject (the
successor to PyGTK) because _there is strong demand for a new architecture that
supports an in-browser web GUI and integration with site identity management
systems._ The full web architecture is (necessarily) more complicated, but it is
more powerful. It will enable us to:
1. Provide a single point of access to many Cylc workflows on a pool of servers.
1. Run and interact with workflows via a web browser, without requiring:
    - a Cylc installation on the front-end (browser) platform
    - a shared filesystem or SSH access between the front-end and workflow
      platforms
1. Drop the requirement for port scanning by users.
1. Retire the suite-specific passphrase files and self-signed SSL certificates.
1. Integrate with site identity management.
1. Support fine-grained authorized access to individual Workflow Services.

## Cylc-8 Architecture Diagram
[TOP](#cylc-8-architecture)

![Cylc-8 Architecture](img/cylc-8-architecture.png "Cylc-8 Architecture")

__Figure 1__ Cylc-8 Architecture: The "user A" box represents processes owned
by one user, but potentially spread over multiple _workflow hosts_ (on a shared
filesystem) and multiple _job hosts_. The term "HPC Platform" is used rather
loosely - potentially only the jobs reside on actual HPC nodes. Yellow boxes
show the technologies and protocols that will be used to implement each component. 


## Cylc Hub
[TOP](#cylc-8-architecture)

The Hub and Proxy architecture described below is inspired by
[JupyterHub](#jupyterhub). JupyterHub is a proven technology that solves a very
similar problem of managing back-end services spawned into user accounts. And
it is commonly used in scientific modeling and HPC contexts, See below for details:
[Similarity with Jupyter Hub](#similarity-with-jupyterhub)).

We hope to use JupyterHub "out of the box" for the Hub and Proxy components of
the new archtitecture. Our back-end components are very different from Jupyter
Notebook, but some of the technologies involved remain relevant.



Overview:
- At start-up, the Hub launches a web proxy.
- The proxy forwards requests to the Hub by default.
- The Hub handles user login (authentication) and spawns UI Servers on demand.
- The Hub configures the proxy to forward URL prefixes to the UI Servers.

Detail:
- The Hub must be a privileged process - either root or sudo.
  - See
  [running JupyterHub without root
  privileges](https://jupyterhub.readthedocs.io/en/stable/reference/config-sudo.html).
  - (As it must be able to spawn [Cylc UI Servers](#cylc-ui-server) as user processes).
- Implemented in [Python 3](#python-3) with [Tornado](#tornado).
- Spawns a _Proxy_ that it dynamically configures to route requests to
    [Cylc UI Servers](#cylc-ui-servers). The proxy is:
  - A single point of access for users.
  - The only process that listens on a public interface.
  - Implemented with
    [jupyterhub/configuable-http-proxy](https://github.com/jupyterhub/configurable-http-proxy)
    - ([Node.js](https://nodejs.org/); wraps
    [node-http-proxy](https://github.com/nodejitsu/node-http-proxy)).
- Hub Authenticator: calls out to host or site identity management, with plugins for:
  - PAM, LDAP, OAuth (GitHub and Google accounts), etc.
  - (PAM sufficient for sites where local accounts are driven by AD or LDAP?)
  - (Extendable wit: custom authenticators)
- Hub User Database
  - Stores Hub state (which users are running which workflows where - user
    names only, nothing sensitive).
  - Default `sqlite` (light-weight serverless, zero-admin).
    - But [https://jupyterhub.readthedocs.io/en/stable/reference/database.html]
    (supports full RDBMS) if needed.
- Hub Spawner: spawn [Cylc UI Servers](#cylc-ui-server) on user accounts; plugins for:
  - ssh, sudo, PBS, Docker, ...
  - (Extendable with custom spawners).

For more detail on component interaction, including session management, see
[JupyterHub Technical
Overview](https://jupyterhub.readthedocs.io/en/stable/reference/technical-overview.html).

## Cylc UI Server
[TOP](#cylc-8-architecture)

Overview:
- Serves the UI to the user's browser.
  - For uniform presentation of stopped suites and static services as well as
    running suites (a workflow service can only be queried if it is running).
- Spawned by [Cylc Hub](#cylc-hub) on demand, into "suite owner" user accounts.

Detail:
- Must run as the user (that is the suite owner, not the UI user) because it
  must run sub-services as the user.
  - (consider another user authorized to view your suites: she must be able to
    read your suite files without relying on local file permissions - she might
    not even have a local account on the workflow host).
- Implemented in [Python 3](#python-3)
- User-facing server communications:
  - [Tornado](#tornado) web server, with [GraphQL](#graphql) API over
    the [WebSocket](#websocket) protocol.
  - WebSocket allows server _push_ to UI Servers (no need for polling) and the
    server to return a response to clients even a command has to be queued for
    asynchronous execution.
  - GraphQL allows the UI to request exactly what it needs and no more.
- Workflow Service-facing server, API, and network protocol:
  - [JSON](#json) over [ZeroMQ](#zeromq).
  - (ZeroMQ is used between Jupyter Notebook Servers and Kernels).
  - (Later: consider Protocol Buffers and/or gRPC - possibly better efficiency).
- One UI Server per (suite owner) user - i.e. a UI server fronts multiple suites.
  - Efficiency benefits for multiple UIs looking at the same suite?
  - Relieves workflow services of some comms load.
  - BUT consider one UI server per UI (i.e. per browser tab) for simplicity, if
    the aforementioned efficiency benefits can't be realized.
- (Could potentially scrape suite databases rather than query Workflow
  Services, to remove all comms load from the suites ... but this has the
  potential for disk latency problems on NFS?)
- Pluggable Sub-Services:
  - Suite Listing Sub-Service:
    - Location and identity of running workflow services (host:port).
    - Location and identity of inactive suites (stopped or never started).
    - Status of stopped suites (e.g. "stopped with N failed tasks").
    - (use existing `cylc scan`, as the user)
  - Suite Start Sub-Service:
    - Start up new workflow services (from inactive suites).
    - (use existing `cylc run`, as the user):
  - Static Sub-Services, e.g.:
    - `cylc graph` (dependency and inheritance graph visualization).
    - `cylc review` (formerly Rose Bush).
    - View suite definition.
    - Suite analytics.
    - `rose edit`.
    - etc.

## Cylc UI 
[TOP](#cylc-8-architecture)
- Javascript with the [Vue.js](#vue-js) framework
- [Target design](Cylc8-UI-design-June2019.pdf)  
- Multiple workflow views:
  - "dot"
  - text tree
  - dependency graph
- Responsive design (e.g. mobile compatible)
- Separates the concepts of "task" and "job" (unlike Cylc 7)
- Presents a simpler array of 
- Contains a shared data-store that collates data subscriptions from views into
  a single subscription, for incremental update from the UI Server by GraphQL
- Unifies the two Cylc 7 GUIs into one (multi-suite gscan side-bar)

## Cylc Workflow Services
[TOP](#cylc-8-architecture)

Largely unchanged from Cylc-7 "suite server programs", except:
- [Python 3](#python-3).
- [JSON](#json) over [ZeroMQ](#zeromq) for communication with
  [Cylc UI Servers](#cylc-ui-server).

## Command Line Interface
[TOP](#cylc-8-architecture)

- User-executed commands go via the Proxy.
  - Allows remote commands, and other authorized users.
  - Might need some kind of in-memory or on-disk CLI session management, akin
    to use of cookies for session management in the browser (TBD).
- Job-executed commands (e.g. for job status messages) should go direct to the
  parent Workflow Service.
  - Job clients know where their own Workflow Service is.
  - Suites must carry on even if the Hub and Proxy are down.
  - Server authentication (trust) - some kind of single-use token? (TBD).
- CLI clients need to talk both ZeroMQ (direct mode) _and_ WebSocket (indirect). 

## Authorization
[TOP](#cylc-8-architecture)

- Authenticated user name sent with requests.
- Two-level authorization:
  1. Who is allowed to connect to my UI Server?
  1. Who is allowed to see or do what to which of my suites?
- Both levels _could_ be enforced by the UI Server, which runs as the user and
  can therefore see the same config files the Workflow Services can.
- Simple text files that map user names or groups to privileges might do.
  - (Use Unix group names to authorize service/role accounts).

# Appendix
[TOP](#cylc-8-architecture)

## Deployment

- Cylc-8 will be packaged for installation with `pip` and `conda`.
- (This wasn't possible at Cylc-7 and earlier due the extreme difficulty of
  PyGTK installation).
- This will enable us to stop bundling 3rd party libraries like Jinja2.
- It should also eliminate problems with version compatibility of software
  dependencies on the system. 
- Tools like `safety` will be able to scan software dependencies (listed in
  a standard `requirements.txt`) for security vulnerabilities.
- No Cylc installation will be needed on the UI (browser) platform (the UI is
  served up by the [Cylc UI Server](#cylc-ui-server).
- We'll also consider `.rpm` and `.deb` packaging for system package managers,
  and containers.

## Similarity with JupyterHub
[TOP](#cylc-8-architecture)

[Jupyter Notebooks](https://jupyter.org/) are a proven technology commonly used
in scientific and educational institutions worldwide for interactive
programming and sharing web documents that contain embedded code.
Architecturally, the user's browser talks to a Python
[Tornado](https://www.tornadoweb.org)-based _Notebook Server_ that communicates
via the [ZeroMQ](http://zeromq.org) network library with a back-end _Notebook
Kernel_ that executes the code. Both the notebook server and kernel run as the user.

[JupyterHub](https://jupyterhub.readthedocs.io) is used to deploy and manage
single-user notebooks for large numbers of users within an institution. It
consists of a privileged _multi-user Hub_ (a Python
[Tornado](https:www.tornadoweb.org) process) that:
- is a single point of access for all users
- handles user authentication, with plugins to integrate with site identity
  management
- spawns single-user Notebook Servers on user accounts
- spawns a configurable Web Proxy to route requests to the single-user
  Notebook Servers
- provides a REST API for "convenient administration of the Hub, its users, and
  services"

[JupyterHub is extremely well documented](https://jupyterhub.readthedocs.io),
including (for example) a [security
overview](https://jupyterhub.readthedocs.io/en/stable/reference/websecurity.html)
of the architecture.

__This architecture is (almost) exactly what we need for Cylc-8__, with:
- _Notebook Server_ => _Cylc UI Server_, and
- _Notebook Kernel_ => _Cylc Workflow Service_

For an in-browser GUI to spawn and access a distributed set of Cylc workflow
services running under various user accounts, we need a central "hub" that can
spawn user processes and proxy requests to them, and a "UI Server" component to
construct the UI (HTML/Javascript) around workflow status data obtained from
the workflow services (and also static data about stopped workflows).

JupyterHub is Open Source, with the [3-Clause BSD
License](https://opensource.org/licenses/BSD-3-Clause)) 


### Differences from Jupyter Notebook

The Jupyter Notebook back-end is highly specific to the Notebook Document
format. For Cylc, we have Workflow Services instead of Kernels, and we need a
Cylc-specific UI Server (with a bunch of Cylc-specific sub-services for suite
discovery and start-up etc.) instead of the Notebook Server. Further, Jupyter
Notebook Servers fronts a single Kernel, whereas a Cylc UI Server may need to
access (and possibly collate) data from multiple workflow services.

However, we do share the two-level "web server / kernel" back-end model, and we
have therefore decided to use ZeroMQ for back-end communications. It is a
well-used, efficient network library, and we may be able to learn by studying
the Jupyter model - including the automatic trust (server authentication)
mechanism.

### Differences from JupyterHub

There is only one difference that we're aware of:
- JupyterHub [does not
  yet](https://github.com/jupyterhub/jupyterhub/issues/394) support access to
  other users' Notebooks, whereas we need (authorized) access to other users'
  workflows services. 

However, multi-user access is presumably as simple as allowing access to
`/user/{other-name}/` URLs (in the Jupyter case the show-stopper is that the
Notebooks themselves, at the back end, don't support shared use yet).

We therefore hope __we can use JupyterHub "out of the box" for the Hub and
Proxy components of the Cylc-8 architecture__. The only question is,
can the (minor?) multi-user access change be done (e.g. by plugin) without
modifying the core of JupyterHub so that we can treat it as a third-party
software requirement, or can we contribute a change back to JupyterHub to
enable that, or do we need to fork the project and maintain our own "Cylc Hub"
in the future?

## Technology Glossary
[TOP](#cylc-8-architecture)

(Hyperlinks in the text below point here for further information).

- <a name="python-3"></a> [Python 3](https://wwww.python.org)
  - _An interpreted high-level programming language for general-purpose
    programming._
  - The primary language of Cylc implementation.

- <a name="tornado"></a> [Tornado](https://www.tornadoweb.org)
  - _A Python web framework and asynchronous networking library._

- <a name="graphqL"></a> [GraphQL](https://www.graphql.org)
  - _A data query language ... that provides an alternative to REST and ad-hoc
    web service architectures. It allows clients to define the structure of the
    data required, and exactly the same structure of the data is returned from
    the server._
  - Originally out of (and backed by) Facebook.
  - A single flexible endpoint, instead of many fixed inflexible REST endpoints.
  - Should allow the UI to request just what it needs very easily.

- <a name="websocket"></a> [WebSocket](https://en.wikipedia.org/wiki/WebSocket)
  - _A communications protocol providing persistent full-duplex communication
    channels over a single TCP connection._  
  - Alternative to HTTPS (and initiated by HTTPS handshake).
  - Good when server-side data changes quickly and unpredictably.

- <a name="zeromq"></a> [ZeroMQ](http://zeromq.org)
  - _A high-performance asynchronous messaging library aimed at use in
    distributed or concurrent applications._ 
  - For back-end server-to-server communications.

- <a name="javascript"></a>
  [Javascript](https://developer.mozilla.org/en-US/docs/Web/JavaScrip)
  - _A lightweight interpreted or JIT-compiled programming language with
    first-class functions, most well-known as the scripting language for Web
    pages_ (in which context it runs inside web browsers).

- <a name="nodejs"></a> [Node.js](https://developer.mozilla.org/en-US/docs/Glossary/Node.js)
  - _A cross-platform JavaScript runtime environment that allows developers to
    build server-side and network applications with JavaScript_.

- <a name="vuejs"></a> [Vue.js](http://vue.js.org)
  - _A JavaScript framework for building user interfaces._
  - The smallest but fastest-growing of the current top Javascript
    frameworks.
  - Lighter than Angular.js and React.js, and reputedly the easiest to learn.
  - In terms of UI components our needs are quite modest, so we will try the
    simplest modern framework first.

- <a name="json"></a> [JSON](https://www.json.org)
  - _JavaScript Object Notation, an open-standard format for human-readable
    text transmission of data objects as attribute–value pairs and array types,
    commonly used for asynchronous browser–server communication._

- <a name="jupyterhub"></a> [JupyterHub](https://jupyterhub.readthedocs.io)
  - _A multi-user Hub that spawns, manages, and proxies multiple instances of
    the single-user Jupyter Notebook Server._ 
  - Architecturally analogous to Cylc-8, with:
    - "Jupyter Notebook Server" -> "Cylc UI Server"
    - "Jupyter Notebook Kernel" -> "Cylc Workflow Service"


