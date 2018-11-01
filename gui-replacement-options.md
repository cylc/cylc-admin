# Options for Replacing the Python 2 PyGTK GUIs

(Latest update: 1 Nov 2018)

## Motivation

- Python 2 end of life 2020
- PyGTK GUIs obsolete and not Python 3 compatible
- everyone wants modern web GUIs
  - modern web browsers are the most sophisticated display engines
  - remote suite monitoring and control
  - platform agnostic (portable)
  - easy integration with site authentication

# Architectural Challenges

Some aspects of the cylc-7 architecture are not well-suited to web technologies:
- Web apps running in browsers cannot interact with the host system:
  - To read suite contact files (to discover suite locations host:port)
  - To scan ports (to discover suite locations host:port)
  - To read suite passphrases off disk, for automatic authentication
- Besides, the browser will likely not even be running on the suite host system
- The web services (suite daemons) vary in number and location as suites shut
  down and (re)start as described above
- Server SSL certificates allow clients to trust the identity of web servers.
  In our context self-signed certificates do not necessarily constitute a risk,
  but they do make life difficult for browsers as an exception must be granted
  manually whenever a new one is encountered
- Client commands executed by task jobs will probably have to be treated
  differently from those executed by users (CLI or GUI) once a more
  sophisticated and site-integrated authentication model is adopted
- It is recognized that production suites running under role/service accounts
  would be easier to manage with fine-grained multi-user authentication and
  authorization, and integration with site identity management

In short, the ways that Cylc clients discover and authenticate with suites will
not work for an in-browser web GUI.

## Options

- __PyGObject desktop GUIs__
  - PyGobject is the successor to PyGTK
  - __pros__
    - a relatively easy (?) port of the old desktop GUIs
    - doesn't bring other major advantages of an in-browser GUI
  - __cons__
    - PyGObject is not well-used (everyone wants web apps now)
    - doesn't bring other major advantages of an in-browser GUI

- __Electron native desktop web apps__
  - chromium browser display engine with node.js wrapper
  - __pros__
    - architecturally, a straightfoward replacement for current GUIs
    - the node.js wrapper could still read the filesystem and scan ports
    - is "web technology"
  - __cons__
    - not really a short-cut to an in-browser GUI (major architectural differences)
    - doesn't bring other major advantages of an in-browser GUI

- __In-browser web GUI__
  - __pros__
    - the ultimate in remote monitoring and control
    - platform agnostic (portable)
    - no cylc installation needed at the GUI end (the GUI is "served" up)
    - easy integration with site authentication systems
    - new system components required (see below) but these bring other
      advantages, e.g. single point of access for users
  - __cons__
    - requires massive architectural change (new system components) because
      an in-browser GUI can't interact with the system to discover suites, and
      start them, etc.
      
__DECISION: in-browser GUI__ (it's what we ultimately need)

