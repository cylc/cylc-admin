# List of meeting Actions not completed yet.

Feb 2019

- HO to get deep into the hub

- UI testing framework?
  - Vue.js comes with Jest & Mocha for unit testing. Functional/e2e tests:
    Selenium, or other options like Karma, Nightwatch, Cypress, ...

- better use of newer Python 3 technologies in cylc-flow code base
  - (low priority, can do gradually, or once the new system is working)

March 2019

- Hub Service based site authorization layer?
  - feasible, or delegate site auth to the UI Servers (which can read a site
    config file, after all)
  - can site authorization Hub service (above) be co-opted to display a
    clickable list of users whose UI Servers I can connect to?

- investigate the Hub form for entering additional info before spawning
  - is this spawner-specific or generic?

- cylc-flow main-loop asyncio (on critical path but low priority)
  - OS has concerns about what it means to put the ZMQ server in the same event
    loop; but asyncio has a *priority queue* so we should be OK.
    (MS and TP to look at this in due course)

- new repos needed:
  - SB "cheat sheets"
  - profile battery
  - cylc-review?

April 2019

- orthogonal work:
  - cluster awareness
  - "rose suite-run" migration

- OS to put up a PR to do the test battery with pytest (minimally at first)
  (OS did push a branch up and alerted us to it, I think?)

- OS to write down high-level description of UIS-WS data model update

- Create some static  workflow "demo data" (perhaps with limited time
  evolution) - good for standalone and automated testing without the need to
  hook up to a running workflow.

May 2019

- document release process, for less reliance on HO
