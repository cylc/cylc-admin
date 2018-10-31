# Cylc-7 System Architecture

(Updated 1 Nov 2018)

## Distributed (No Central Server)

Cylc is a distributed system in the sense that:
- Each suite is managed by its own ad-hoc **suite server program** (aka **suite
  daemon**), which
  - starts up on-demand (`cylc run SUITE`)
  - shuts down down automatically when/if the suite runs to completion
  - and runs and submits jobs *as the user*
- Inter-suite triggering is supported
  - by polling other-suite databases
    - formerly, by polling tasks in the workflow
    - from cylc-7.8.0, by external triggers

In contrast to monolithic central-server systems, Cylc:
- Scales easily sideways - just add more VMs.
- Has minimal admin costs (zero in some contexts)
  - individuals can run Cylc "out of the box" in their own account
- Has no elevated privileges on the system
  - and therefore a relatively small security footprint
    - (caveats: we need to manage jobs on remote hosts, and authenticate
      network connections to suite daemons)
- Allows easy upgrade of large systems, one suite at a time

At a large site we may therefore have:
- Many suite daemons listening on different ports, on multiple hosts
  - ports 43001-43101 by default
- The pool of suite daemons changes over time as some suites finish and
  shut down, and others start anew or grab a different port when restarted

## Suite Discovery: `cylc (g)scan`

To manage multiple suites, we must be able discover running suite daemons,
- i.e. which suites are listening on which ports, on which hosts

The multi-suite monitoring clients `cylc scan` (CLI) and `cylc gscan` (GUI):
- Scan your local suite run directory for **suite contact files**
  (which store the host and port number, etc.) of running suite daemons
  - (by default, for your own suites)
- Scan ports on configured hosts, to find listening suite suite daemons
  - (optionally, or to find other users' suites)
- Then they query the REST API of discovered suites for identity and status info

## Authentication

Because Cylc has no central server to manage suites for all users, and
everything (suite daemons and jobs) run as the user, there is no absolute
requirement for multi-user authentication and authorization.

The current authentication model is therefore designed to **automatically**
(with no need to manually enter credentials at any point) reject, by default,
everyone but the suite owner.
- At start up, suite daemons generate a random passphrase
  - and write it to `~/cylc-run/<suite>/.service/passphrase` with owner-only
    permissions
- Client programs (CLI and GUI) automatically load the passphrase of the
  target suite from the standard location on disk, and use it to authenticate
  with the server program.
  - Only clients running as the suite owner can read the passphrase (thanks to
  the restrictive file permissions) so only the owner can control the suite
  - However, the owner can give the passphrase to others if necessary.
  - (C.f. ssh keys stored on disk in user accounts.)
- Proper digest authentication is used – suite passphrases are never sent in
  plain text.
- Cylc client programs executed by task jobs (e.g. for status messaging) are
  treated just the same as user-executed clients (they automatically load the
  passphrase off disk).
- For convenience we currently use server-generated self-signed SSL certificates

## Authorization

- Suite owners can authorize different levels of read-only “public” access
  - from suite identity only, to full read-only 
  - this uses the same authentication system, but with a built-in “anon”
    username and passphrase
- But fine-grained authorization of different control functions for different
  users cannot be supported by the simple shared passphrase model


# cylc-8 Web GUI - Architectural Challenges

Some aspects of the architecture described above are not well-suited to web
technologies:
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

In short, the ways that Cylc clients currently discover suite servers and
authenticate with them, will not work for the future "web Cylc".
