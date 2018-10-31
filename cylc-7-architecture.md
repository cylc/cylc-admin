# Cylc-7 Architecture

## No Central Server

Cylc is a distributed workflow scheduler in the sense that:
- Each suite is managed by its own ad-hoc server program (suite daemon) that
  runs and submits jobs “as the user” (the suite owner); with inter-suite
  triggering for dependence between workflows.

In contrast to monolithic central-server systems, Cylc:
- Scales easily sideways - just add more VMs.
- Has minimal administration costs - zero in some contexts: individuals can
  download Cylc and run it out of the box with no need to configure and
  administer a central server.
- Has no need for elevated privileges on the system, and therefore a small
  security footprint (with some obvious caveats due to the requirement to
  manage jobs on remote hosts, and authentication of network connections to
  suite server programs).
- Makes it very easy to upgrade large complex systems to new Cylc versions, one
  suite at a time.

At a large site we may therefore have:
- Many suite server programs (aka “suite daemons”) listening on different
  network ports (from the range 43001-43101 by default) on multiple hosts.
- The pool of suite server programs changes over time as some suites finish and
  shut down, and others start anew or grab a different port when restarted.

## Suite Discovery

To manage multiple suites at once, we must be able discover where the suite
server programs are running (i.e. which suites are listening on which ports on
which hosts). The multi-suite monitoring clients cylc scan (CLI) and cylc gscan
(GUI) do this as follows:
- They search standard locations on the local filesystem for suite contact
  files, which store the host and port number (among other things) of running
  suite server programs.
- They scan network ports on configured hosts, to find listening suite server
  programs.

Then they use the REST API of discovered suites to query their identity and status.

## Authentication
Cylc has no central server to manage suites for all users (each suite is
managed by its own ad hoc server program, running as the user) so there is no
absolute requirement for multi-user authorization in suite servers. The current
authentication model is therefore designed to reject, by default, everyone but
the suite owner, with no effort from the owner (it is completely automatic):
- At start up, a suite server program generates a random passphrase and write
  it to disk in a standard location, with owner-only file permissions.
- Client programs (both CLI and GUI) automatically load the passphrase of the
  target suite from the standard location on disk, and use it to authenticate
  with the server program.
- Only clients running as the suite owner can read the passphrase (thanks to
  the restrictive file permissions) so only the owner can control the suite.
  - The owner can deliberately give the passphrase to others if necessary.
  - (C.f. ssh keys stored on disk in user accounts.)
- Proper digest authentication is used – suite passphrases are never
  transmitted in plain text.
- Cylc client programs executed by task jobs (e.g. for status messaging) are
  treated the same as user-executed clients (they automatically load the
  passphrase off disk etc.).
- Note, for convenience we use server-generated self-signed SSL certificates.

## Authorization
- Suite owners can authorize different levels of read-only “public” access
  (this uses the same authentication system, with a built-in “anon” username
  and passphrase).
- But fine-grained authorization of different control functions cannot be
  supported by the single passphrase per suite model.


# cylc-8 Web GUI - Architectural Challenges

Some aspects of the architecture described above are not well-suited to web
technologies:
- Web apps running in browsers cannot interact with the host system:
  - To read suite contact files, to discover suite locations (host:port).
  - To scan ports, to discover suite locations (host:port).
  - To read suite passphrases off disk, for the current automatic authentication system.
- Besides, the browser may not even be running on the suite host system.
- The web services (suite server programs) vary in number and location as
  suites shut down and (re)start as described above.
- Server SSL certificates allow clients to trust the identity of web servers.
  In our context self-signed certificates do not necessarily constitute a risk,
  but they do make life difficult for browsers as an exception must be granted
  manually whenever a new one is encountered.
- Client commands executed by task jobs will probably have to be treated
  differently from those executed by users (CLI or GUI) once a more
  sophisticated and site-integrated authentication model is adopted.
- It is recognized that production suites running under role/service accounts
  would be easier to manage with fine-grained multi-user authentication and
  authorization, and integration with site identity management.

In short, the ways that Cylc clients currently discover suite servers and
authenticate with them, will not work with in-browser clients.
