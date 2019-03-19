# Cylc VC March 2019 Summary Notes

[Agenda](vc-march-2019-agenda.md)

## Site Updates, News

[Met Office]

Operational with 7.8.1. Still having some problems with tasks that get
stuck in the "ready" state, possibly after failed job submissions are
not identified correctly (particularly after a suite reload?) 

Strange problem reported with a cylc library on a job host becoming
unlistable. Probably a filesystem issue, but `cylc` commands are now
need on job hosts (job submit, poll, and kill) now, but weren't in the past (although that was some time ago now).

[NIWA]

Operational with 7.8.0, no issues noted.

## Next Release

Not desperately needed at MO as they've patched 7.8.1 with subsequent
fixes, but we should aim to get it out soon for other sites.

**ACTION: release cylc-7.8.2 by end of week** (Hilary)

cylc-7.8.3 needed by May for MO PS43, hopefully with fix for the "stuck in
ready state" problem.

**ACTION: make a new milestone for 7.8.3, May 1 end date** (Hilary)

## Python 2-to-3

Rosie disco + Tornado port - *Sadie almost done*

Rose suite and task wrappers - *Tim starting on this now*

## JupyterHub

- Successfully tested:
   - local PAM authentication
   - spawn proto Cylc UI Server with LocalProcessSpawner
   - view other user's UI Server (it seems that single user J Notebook Server
     was the blocker here, not the Hub)

- we need ssh spawner (not maintained, we may need to take it on)

  **ACTION: demonstrate ssh spawner with our proto UI Server** (Martin)

- we also want to get PBS batch spawner working (Martin)

  **ACTION: demonstrate batch spawner with our proto UI Server** (Martin) 

- may be able to use a J-Hub Service alongside the Hub to implement site-level
  authorization (i.e. which users can even look at which other user's UI
  Server); probably set in a config file. 
  - if this does not work out we could delegate site authorization to the UI
    Servers (which can read a global/site config file just as cylc servers do
    now)
  **ACTION: consider and demo Hub Service based authorization** (Martin, Bruno)

- J-Hub can serve a web form to get additional config information before
  spawning. 

  **ACTION: is this generic (any spawner)? how can we use it?**

- cross-user view works, but by typing in the URL (`/usr/other-name`)

  **ACTION can the authorization Hub service (above) be used to display
  a clickable list of users who's UI Servers I can connect to?**

