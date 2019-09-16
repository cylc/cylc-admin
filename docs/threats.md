
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
>
- As an attacker I want to stop user A suite A from running
- As an attacker I want to be able to fool authentication on the Cylc HUB
- As an attacker I want to harvest usernames and passwords
- As an attacker I want to be able to get the network map relevant to the Cylc solution
- As an attacker I want to hijack current tokens to take over sessions
- As an attacker I want to harvest suite state via the HTTP proxy on Cylc Hub
- As an attacker I want to execute my own suite/arbitrary code on HPC
- As an attacker I want to inject arbitrary code into al already running suite
- As an attacker I want to 'spawn' a non UI server (arbitrary code, not cylc)
- As an attacker I want to steal SSO from the Cylc Hub
- As an attacker/user I want to change other users running suites
- As a malicious job I would like to send messages back to UI server and or HTTP proxy to affect other users suites
- As an attacker I want to store something that will be executed, when admins log in, with admin privileges
- As a defender I'd like to monitor for unusual (potentially nefarious) activity (e.g. DOS attack) and be able to shut-down the threat
- As an attacker I'd like to snoop or modify communication traffic between components, e.g. zeromq traffic
- As an insider with rudimentary Cylc access who wishes to sabotage operations, I'd like to trick the role-based permissions model into providing me escalated rights


> Please let me know if you have any questions and/or want to talk further!
