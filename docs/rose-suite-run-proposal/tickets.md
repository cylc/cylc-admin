# Work on Platforms

The description of work to be done collected here rather than based on more
transitory tickets.

Based on notes originally created by @OSanders.

## Approaches:
1. Add the new configurations but don't use them at first, do all the the
   upgrader stuff then change over towards the end.
   Means you have two sets of configuration hanging about.

2. Remove the old configurations then add the upgrader stuff towards the
   Will break master, so either you will need to work on a "feature"
   branch or else others will have to maintain their own "feature"

3. Hybrid approach - Work mainly on master, but create a feature branch for
   PR's related to the "wireing" ticket, which _will_ break master.

Do what you think best, there are pros and cons to both approaches.

If we can avoid breaking master that would make life easier for everyone.
If we do break master it will effectively halt all Cylc Flow work until
its done, in which case it should probably become a more international
effort so that we get Cylc Flow development back on track quickly. Otherwise
you will have to work on a feature branch and accept any rebase mess which

I think Cylc Flow might actually be heading towards a quiet patch as we
move our focus downstream to the UI Server and UI. There may be changes
to the GraphQL stuff in Cylc Flow but that wont create conflicts. However,

## ​Blocking PRs:

(these things block the "wireing" portion of the work):
* Python Endpoints
* ZMQ Curve Authentication
* Incremental Data Store Updates

<iframe src=https://mermaidjs.github.io/mermaid-live-editor/#/view/eyJjb2RlIjoiZ3JhcGggVERcblxuICAgIEEoSm9iIFBsYXRmb3JtIFNwZWMpIC0tPiBCKEZvcndhcmQgTG9va3VwKVxuICAgIEEgLS0-IEMoUmV2ZXJzZSBMb29rdXApXG4gICAgQyAtLT4gRChDb25maWcgVXBncmFkZXIpXG4gICBDIC0tPiBFKERhdGFiYXNlIFVwZ3JhZGVyKVxuQiAtLT4gRihXaXJlaW5nIDIpXG5HKFdpcmVpbmcgMSkgLS0-IEZcbkhbUHl0aG9uIEVuZHBvaW50c10gLS0-IEZcbkpbWk1RIEN1cnZlIEF1dGhdIC0tPiBGXG5LW0luY3JlbWVudGFsIERhdGEgU3RvcmUgVXBkYXRlc10gLS0-IEZcblxuc3R5bGUgSCBmaWxsOiNjY2NcbnN0eWxlIEogZmlsbDojY2NjXG5zdHlsZSBLIGZpbGw6I2NjYyIsIm1lcm1haWQiOnsidGhlbWUiOiJkZWZhdWx0In19></iframe>

## The tasks:

### Job Platform Spec

Add job platforms section to `cylc/flow/cfgspec/global_config.rc`.
* Include the content currently included in the `[hosts][many]` section.
* Do not include the `[hosts][localhost]`

Add platform setting to the task section of `cylc/flow/cfgspec/suite.rc`.

Move Configs from `suite.rc[runtime][TASK][job/remote]` to
`flow.rc[platforms][__MANY__]`. [The diff for PR #3348](https://github.com/cylc/cylc-flow/pull/3348/files/cb83a8ac04ac567488ae97061ee0f934ffc46bbd..ae70eafc03e045ce6255a8bb5910ca208991fb65) should provide further details.

Remove old configurations.

### Forward Lookup
PREREQS: "job platform spec"

How Cylc 8 will pick a job platform using `[runtime][TASK]platform=MyPlatform`

Write a function which takes the "platform" field from the task config
forward(platform_name_or_function) -> job platform dict

This will probably end up in `cfgspec/global_config.py` in the same way that
`def get_host_item()` currently is.

Note the platform_function (i.e. rose host-select) functionality. This may not
be fully testable within the scope of this ticket, since in some cases it may
not be evaluated until job submission time.

This item will need unit testing but will not be wired into cylc as part of
this ticket.

This code can be written and tested without breaking master since it will not
yet be used outside its own tests.

### Reverse Lookup
PREREQS: "job platform spec"

How Cylc 8 will pick a job platform using a Cylc 7 compatible `suite.rc`

Write a function which returns a job platform from available information

This will probably end up in `cfgspec/global_config.py` in the same way that
`def get_host_item()` currently is.

For context this information could come from the task configuration...
...or the database, where it comes from doesn't matter right now.

reverse(host, batch_sys, user=None) -> platform_name

This should return the platform name if there is exactly one possible
...or raise an error otherwise.

This work should not break master.

### Config Upgrader

PREREQS: "reverse lookup"

Write a test battery to show how the old settings should be converted
Write a function which actually does that (but for the moment doesn't
remove the old configurations).

You will probably want to consider existing tests in `tests/deprecations`
as templates for the new test or tests.

### DB Upgrader

PREREQS: "reverse lookup"

Much the same as the previous but for the database.

The database stores the host and batch_system, we would need to upgrade
This could be done towards the end, you might need to put off upgrading
the DB but instead upgrade the information comming out of the database.

### Wireing 1: Understanding Where hosts are used

PREREQS: None

Examine the code base for references to use of settings based on hosts and
record them, probably by editing the ticket below - "Wireing 2".
Replace all references to the old settings with the new ones.

​This needs to be done right at the start before upgraders etc come

### Wireing 2: Replacing the use of all settings under hosts.

PREREQS: "forward lookup", "Wireing 1"

Replace all the references to hosts with platforms.

This work is complete when you can deprecate all the old configurations

**This ticket will break master, and should almost certainly be carried out
in a feature branch**.
