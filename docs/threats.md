
## Cylc-8 Threat Modelling Assessment outcomes

(email from BOM staff to HO, 25 March 2019)

> Last Thursday we held a security threat modelling session on Cylc-8 using the
current architecture diagram you have provided. The result was the set of
"user stories" we think need to be considered, which are listed below.  Most of
them are "negative" user stories ie. they are stories we would like Cylc-8 to
stop from happening.  User stories generally spark further conversations,
particularly when it comes to someone trying to work out what to implement and
how. I expect this set of stories will probably do the same!
 
> The other key outcome of the meeting was that we confirmed that JupyterHub is
on our Enterprise Architecture team's technology roadmap, which means it is an
accepted technology for use at the BOM.

> Security User Stories

FIRSTLY: we rely on normal file system permissions to protect tokens on disk
(on suite and job hosts) (like ssh)

Entity-in-the-middle attacks

- Hub-UIS & User-proxy-hub: using TLS certificates
- UIS-Sched: Curve ZMQ (guaranteed! see ZMQ docs)
- Sched-job-CLI: job platform authorization + ssh for key exchange + Curve ZMQ

> As an attacker I want to stop user A suite A from running

 - via the Hub and UIS
  - I cannot authenticate, or
  - I am not authorized at all in the UIS, or
  - I am not authorized to stop user A's suites at the UIS
 - (TODO: allow sites to require explicit elevation to authorized privileges, or
    a toggle button)
 - direct CLI to Sched
  - I cannot connect because of FS permissions (everything runs as the user -

> As an attacker I want to be able to fool authentication on the Cylc Hub

  - use your own preferred plugin

> As an attacker I want to harvest usernames and passwords

  - (shouldn't be possible - see J-Hub security docs) 

> As an attacker I want to be able to get the network map relevant to the Cylc solution

  - cylc global config (with job platform config) must be visible to all cylc
    users (via the CLI)
  - you can see job hosts for a particular suite if you are authorized to view
    the suite

> As an attacker I want to hijack current tokens to take over sessions

  - Hub cookie handling conforms to best practices
  - ZMQ curve keys protected by FS permissions

> As an attacker I want to harvest suite state via the HTTP proxy on Cylc Hub

  - no anon access in Cylc 8 (must authenticate)
  - Hub is ignorant of suite state
  - traffice through hub is HTTPS

> As an attacker I want to execute my own suite/arbitrary code on HPC

  - trigger edit and reload (if/once we edit suites in the UI) requires
    authorization
    -(and permissions elevation process)

> As an attacker I want to inject arbitrary code into an already running suite

  - (see above)

> As an attacker I want to 'spawn' a non UI server (arbitrary code, not cylc)

  - keep your hub configuration file secure

> As an attacker I want to steal SSO from the Cylc Hub

  - (see above - cookie)
  - (hub authentication is specific to the hub, and goes via the proxy)

> As an attacker/user I want to change other users running suites

  - requires explicit authorization (at site and user-owner level)

> As a malicious job I would like to send messages back to UI server and or
  HTTP proxy to affect other users suites

  - our authentication and authorization methods will prevent this

> As an attacker I want to store something that will be executed, when admins
  log in, with admin privileges

  - hub admins have no power over the workflows, can only kill UI Servers (which
    is safe)
  - the hub can be configured to only spawn UI Servers and nothing else

> As a defender I'd like to monitor for unusual (potentially nefarious)
  activity (e.g. DOS attack) and be able to shut-down the threat

  - (this is mainly for network monitoring, not Cylc)
  - TODO: we will consider limiting the number of:
     - logon sessions per user
     - connections to UI Servers (if Tornado allows it?)
     - (schedulers are only exposed internally)

> As an attacker I'd like to snoop or modify communication traffic between
  components, e.g. zeromq traffic

  - everything is encrypted (and nonces used by ZMQ Curve auth)

> As an insider with rudimentary Cylc access who wishes to sabotage operations,
  I'd like to trick the role-based permissions model into providing me
  escalated rights

  - Our authorization system will not allow this
