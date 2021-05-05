# Multi-User Functionality & Cylc Flow Proposal

## Background

Pre Cylc8 there was some level of multi-user functionality offered by Cylc.

We had the "anonymous" user, the ability to scan other users suites and HTTP(s)
endpoints had authorisation levels.

With the movement to the Cylc8 architecture we are planning to control suites
via a UI Server running as the suite owner for UI purposes meaning that, from
the Cylc Flow perspective we only need to authenticate the suite owner. All
multi-user capability and authorisation happen at the UI Server.

## Introduction

Cylc Flow should be a single-user application with no ability to see or
interact with other users' workflows at all.

All multi-user functionality should be provided by the UI Server and, where
applicable, made accessible to the ``cylc`` CLI tools via the GraphQL
interface.

Work discussed here will be implementing accessing of other users' UI Servers, the orange arrow in the diagram below.

![Cylc-8 Authorisation](img/cylc-8-auth.png "Cylc-8 Authorisation")

## Security Considerations

In line with the guidance provided by [section 4.1 of the OWASP ASVS](https://github.com/OWASP/ASVS/blob/master/4.0/en/0x12-V4-Access-Control.md#v41-general-access-control-design) (Application Security Verification Standard):

* Authorisation will be *fail secure* and *deny by default*. That is, unless explicit authorisation has been granted to the user to perform a given action, UI Servers will be locked down.
* The *principle of least privilege* should be adhered to, whereby users are only granted those privileges that are essential to the work they are doing. To achieve this, UI Server owners will need a sufficient level of granularity for configuring access.

UI Server back-end code should prevent users manipulating client-side code to bypass authorisation. For example, sending a GraphQL query for a different operation to that stated in the operation name parameter.

## Configuration

Two configuration files will be needed, one for site and one for users, which could take the form as discussed in CylcCon2020. This authorisation configuration can be added to the existing UI Server config files.

### Example UI Server User Configuration

```python
c.UIServer.authorisation = {
    "<user1>": {
        "read": True,  # View only with no interaction allowed
        "write": ["pause", "trigger", "message"],  # Specified interactions allowed
        # No execute permissions
    },
    "group:<group1>": {  # Denote a group with `group:`
        "write": True  # Read is implied by Write
    },
    "<user2>": {"execute": True},  # Read and Write implied by Execute
}
```

### Example Site Configuration

```python
c.UIServer.authorisation = {
    {
        "<*>": {                    # For all ui-server owners,
            "<*>": {                # Any authenticated user
                "default": "read",  # Will have default read-only access
                "limit": "execute"  # All server owners are allowed to raise access up to
                                    # a maximum of execute.
            }
        },
        "<server_owner1>": {        # Specific UI Owner
            "<user1>": {            # Specific user1
                "limit": "write"    # Can only be granted a maximum of write by
                                    # server_owner1
            },
            "group:<groupA>": {"default": "write"},  # Denote a group with `group:`
        },
        "group:<group_of_server_users>":{
            "group: groupB": {"limit": "write"},  # Group of users who own UI Servers
    }
}
```

### Site vs. User Config Precedence

At CylcCon2020 is was agreed that a:

> user can ramp up authorization levels as far as the site allows

Site config will takes precedence in the form of an upper boundary set, a maximum `limit`, it will also set a `default` access level. Users cannot raise access levels in their UI Server config for a given user or group, higher than those set in site config.

If the site config does not set access for a given user or group then a UI Server will **not** be limited by site config.

The development team should discuss this to refine it further.

## Defaults and Limits

Unset defaults for both the `limit` and `default` will need consideration, I suggest that, in keeping with the deny by default principle:

* if a limit is not set but a default is, then the default level is used as the limit.
* if a default is not set but a limit is, then the default should be no access.

We will need to consider the desired behaviour if a user appears twice with different defaults and limits set, this is probably most likely to occur when a user appears in either multiple groups or in a group and as a user.

With mutation-level granularity `play` and `stop` could be added under the `write` mutations list instead of having an execute role. Alternatively we provide a well-documented mapping system:

## Possible Access Assignment of Mutations

If running with the read/write/execute configuration, initial assignments will be needed, for the case when users set e.g. `'write' = True`.

Some of the below are not currently available to users but including them here for consideration. This is also a fairly substantial list, which makes the case for read, write, execute pre-set access groups which would be easier for users and sites to configure, rather than defining a long list of mutations per user/group.

As a springboard for discussion, defaults could be assigned as follows:

| Mutation | Read | Write | Exectute |
| :---     |:---: |:---:  |---:
Broadcast|||x|
Cat-log||x|x|
Check-versions||x|x|
Clean||x|x|
Compare||x|x|
Config||x|x|
Diff||x|x|
Dump||x|x|
Edit|||x|
Ext-trigger||x|x|
Get-cylc-version||x|x|
Get-workflow-version||x|x|
Graph||x|x|
Hold||x|x|
Install||x|x|
Kill||x|x|
List||x|x|
Message||x|x|
Pause||x|x|
Ping||x|x|
Play|||x|
Poll||x|x|
Read|x|x|x|
Reinstall||x|x|
Release||x|x|
Reload||x|x|
Remove||x|x|
Report-timings||x|x|
Resume||x|x|
Scan||x|x|
Search||x|x|
SetOutputs||x|x|
SetVerbosity||x|x|
Show||x|x|
Stop|||x|
Terminal Access||x|x|
Trigger||x|x|
Tui||x|x|
Workflow-state||x|x|
Validate||x|x|
View||x|x|

Any mutation without a specified access assignment will be denied by default.
As future features are added, they will also need to be categorised.

The associated arguments for the mutations may need consideration.

We should set a recommended write permissions level for the user config file. It could pose a security risk to have users config files writable by others. Perhaps we could have a warning in the log if the file permissions are not strict enough.

## Specific Use Case for Authorisation

Having a site config that can give users who are members of a group with the same name as the UI server owner write permissions, would be desirable. With the above config, this could be achieved as follows:

```python
c.UIServer.authorisation = {
    {
        "<account*>": {                     # User UI Server
            "<group:account*>": {           # Members in group of same name as UIS owner
                "default": "write"          # Default write permissions
            }
        },

```

## Open Config Questions

* Config design - read/write/exectute vs mutation granularity. `read`, `write` and `execute` mutations could be rolled into one configuration: e.g. `user permissions`
* If running with the read/write/execute defaults (suggested in table above) need confirmation/agreement
* It may be sufficient to have site config set permissions as suggested above (with implied `True`), without mutation granularity. If it is preferential for this mutation level granularity to be set at site level, lists of mutations could be configured as an alternative.

### Access Group Inheritance

A user/group with `execute: True` should be assumed to have `read: True` and `write: True`, similarly, a `write: True` user/group should have `read:True` access.
This would need to be documented for sites and users.

## Ongoing Investigation: Fetching User Groups

I've tried a number of methods for fetching group membership for the authenticated browser users i.e. return the same groups as I see using `groups` on the Linux command line, which should be both local groups and remote via SSSD/LDAP.

One method reliably returns the expected under PAM:

```python
group_ids = os.getgrouplist(username, 0)
group_ids.remove(0)
users_groups = list(map(lambda x: grp.getgrgid(x).gr_name, group_ids))
```

The outstanding issues with this, that I'd like to resolve is the required second `gid` parameter:

> An integer value representing a group id.
> If gid does not belong to the specified user, it will also be included in the return list

I've used 0 for `root` in the proof of concept work but ideally we would use something like `sys.maxint`.

Testing this works on users' systems might present some challenges - this would preferably be heavily beta tested. If we have concerns, a cruder alternative might be to `Popen` out to the command line `groups`.

If an organisation has a level of nesting in their groups, investigation is still needed - does the command pull the nested groups too? If not, we need to document this limitation for users.

The implementation of groups could be computationally expensive, so these should be cached after first call.

![Cylc-8 Authorisation High-Level Diagram](img/cylc-8-auth-high-level.png "Cylc-8 Authorisation HLD")

## Ongoing Investigation: Reading Config Behaviour

If a user changes their config, for example, to reduce permissions, we would expect them to restart their UI Server for those changes to take effect. Restarts are currently required for UI updates.

Having a regular interval reload of the config may be an option, depending on other/future configuration requirements. However this interval time could pose a security risk and effects on performance would need to be considered.

## Work Breakdown

The beginnings of the authorisation work have been started [Proof of Concept PR](https://github.com/cylc/cylc-uiserver/pull/204), this is not production ready but a fair chunk of the work has been completed.
This uses hard coded config, the basic functionality of read/write/execute permissions is in place.
Also included in that PR is the beginnings of the user config work.

Still to be completed, once authorisation fine detail has been agreed upon...

* Clear logging of user interaction.
* Config fine tuning.
* Auth group inheritance.
* UI error handling for 403 - e.g. handle read only users getting 403 on write attempt.
* Ramp up / ramp down logic for site vs. user config.
* UI to display greyed out buttons for mutations without authorisation.
* UI to display currently authenticated user and editable buttons for navigation to another UI Server.
* Documentation for Authorisation
* Set up GH Actions to incorporate multi-user access testing using docker (still at the investigation stage).

## Not supported in this proposal

* Other possible configured attributes such as time-based authorisation. e.g. limit access to UI Server to User_A for 30 minutes.

Whilst this would be useful to help users debug their workflows, screen sharing through Teams, for example, can currently provide access to a users UI Server and workflows. The control can be handed over to another user but the owner/user can remove access control at any time.

* Workflow level granularity

Initially this authorisation work will be implemented on an all workflow basis, i.e. grant User_A access to all my workflows.
Future work could implement authorisation configuration on a per-workflow basis.

* Access level change requiring re-authentication

A change in access level interactions with a workflow could require the user to have to re-authenticate. For example, User A accessing User B's workflows, on an attempted execute operation for a workflow would require User A to login again to re-authenticate themselves. This would need further investigation as may be difficult to implement due to the current reliance on Jupyterhub authentication.
Perhaps a confirmation box may be worth implementing, e.g. You have selected to Pause Workflow_A of User1. Click OK to proceed.

### Command Line

Not directly related to this proposal, although perhaps worth bearing in mind is access on the command line.
e.g.

```console
$ # stop myflow locally
$ cylc stop myflow
$ # stop alice's flow via the UIS
$ cylc stop ~alice/theirflow
$ # stop all flows via the UIS
$ cylc stop '*'
```
