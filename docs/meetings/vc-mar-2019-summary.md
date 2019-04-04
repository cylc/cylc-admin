# Cylc VC March 2019 Summary Notes

[Agenda](vc-mar-2019-agenda.md)

## Site Updates, News

[Met Office]
- Operational with 7.8.1. Still having some problems with tasks that get
stuck in the "ready" state, possibly after failed job submissions are
not identified correctly (particularly after a suite reload?) 
- Strange problem reported with a cylc library on a job host becoming
unlistable. Probably a filesystem issue, but `cylc` commands are now
need on job hosts (job submit, poll, and kill) now, but weren't in the past (although that was some time ago now).

[NIWA]
- Operational with 7.8.0, no issues noted.

## Next Release

Not desperately needed at MO as patched 7.8.1 with subsequent fixes, but we
should aim to get it out soon for other sites.

**ACTION: release cylc-7.8.2 by end of week** (Hilary)

cylc-7.8.3 needed by May for MO PS43, hopefully with fix for the "stuck in
ready state" problem.

**ACTION: make a new milestone for 7.8.3, May 1 end date** (Hilary)

## Python 2-to-3

- Rosie disco + Tornado port - *Sadie almost done*
- Rose suite and task wrappers - *Tim starting on this now*

## JupyterHub

Successfully tested:
 - local PAM authentication
 - spawn proto Cylc UI Server with LocalProcessSpawner
 - view other user's UI Server (it seems that single user J Notebook Server
   was the blocker here, not the Hub) .. but requires typing in the URL
   manually (`/usr/other-name`)?

**ACTION: demonstrate ssh spawner with our proto UI Server** (Martin)
(do we need to take on ssh spawner development - it's not currently maintained?)

**ACTION: demonstrate batch spawner with our proto UI Server** (Martin) 

**ACTION: consider and demo Hub Service based authorization** (Martin, Bruno)
We may be able to use a J-Hub Service to implement site-level authorization
(which users can even look at which other user's UI Server)
- if this does not work out we could delegate site authorization to the UI
    Servers (which can read a site config file just as cylc servers do now)

**ACTION: investigate the J-Hub form for additional info pre-spawn** (Martin) 
- is this spawner-specific or generic (or potentially generic, just not
  currently used by other spawners)?

**ACTION: can the site authorization Hub service (above) be co-opted to display
a clickable list of users whose UI Servers I can connect to?**

## UI Server and Workflow Status Communications

**ACTION: work through and decide which of model 1 or 2 (see agenda) to
implement first.** (Matt and Oliver, initially).

**ACTION: begin implementation** (Oliver, David, Bruno)
- DEPENDS ON model (above)
- need "req, rep, rep, rep..." pub sub model too
- use dummy workflow status data at the back end initially, if necessary

**ACTION: rename `cylc-jupyterhub` repository to cylc-ui-server** (Bruno)

## Workflow Service

**ACTION: main loop asyncio** (Oliver, Matt)

**ACTION: clean up the workflow status data structure (if nec.)** (Hilary)
- item names, dictionary layout, grew organically over a long time

## Next Steps

**ACTION: finish off packaging PRs** (Bruno)

## Project Admin, Code Management

- temporary solution to our chat platform access issues: Oliver to use a dirty
  laptop at work.

**ACTION: carry over any important agenda items that we did not get to in this
meeting** (Hilary)

**ACTION: arrange for a meeting every 2 weeks, swap morning/night** (Hilary)
