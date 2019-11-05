# Proposal - example of a new style `cylc-flow.rc` file

**WARNING: this document does not reflect the latest discussions from
[PR3348](https://github.com/cylc/cylc-flow/pull/3348) nor the latest proposals
for [supporting platforms](../proposal-platforms.md).**

## References
[cylc-admin PR #40](https://github.com/cylc/cylc-admin/pull/40)
[cylc-admin work plan](../proposal-rose-suite-run.md)

## Purpose
As part of the work to transfer functionality from `rose suite run` to cylc
it is proposed that the options available in global, user and suite config
files are brought into alignment. This file sets out a specification for the
combined `cylc-flow.rc` file, although for the foreseeable future `suite.rc`
will also be read in the same way.

## Preamble

To reduce changes required for end users this file format is based on
`suite.rc`: Where aspects of the file are described as unchanged this implies
that file contents are exactly the same as they would be in a `suite.rc`.
Users of the `global.rc` are usually admins or power users and thus changes
to the format of this file are preferable.

It should be noted that although it will be possible to modify all settings in
all contexts, that some settings are more likely to be used in global contexts
and some are more likely to be used in suite configs. It has been proposed
that it ought to be possible for sysadmins to lock some settings in site
configurations.

****

## specification for `cylc-flow.rc`

### 1. Top level sections to come from from `suite.rc`

The new file will be based on `suite.rc` and so items in the new file
will be the same as in `suite.rc` unless explicitly stated.

### 2. Top level sections from global and user configs

It is likely that most users will continue to have these set by site admins.

#### 2.1 Simple changes

* `[authentication]` becomes `[authorization]`

* The `[suite servers]` to be renamed `[suite run platforms]` for consistency
  with job platforms.

* `[test battery]` will be removed entirely.

* `[task events]` will be moved to `[runtime][[root]][[[events]]]`

#### 2.2 `[suite run platforms]`
This is the dictionary key formerly known as ``[suite servers]``. Changed only
for the purpose of keeping the name "platforms" conistent. This is expected to
be set only by system administrators. It should include the former top-level
section `[[suite host self-identification]]`.

#### 2.3 `[cylc]` -> `[general]`

`[cylc]` is be renamed `[general]`.

`global.rc[cylc]` at present contains a subset of the items available in
`suite.rc[cylc]` for this section so it is proposed that the new fill just has
the larger set. These items are:
```ini
[cylc]
  health check interval = 600
  task event mail interval = 300
  [[events]]
```

#### 2.4 New `email` top level section
All config items in the form `[global][event]mail *` will be moved to a new
top level section `[email]`

### 3 Job Platforms and the deprecation of `[runtime][[TASK]][[[job]]]host`

#### 3.1 `[job platforms]`
Many of the options in this section will be very similar to items formerly in 
`[hosts]`, `[runtime][TASK][remote]` and `[runtime][TASK][job]`
It is expected that these will mainly be set at site level, but that
small numbers of power users may wish to over-ride them.

__The key items in this config are `batch system` and `login hosts`
as these between them define way the cluster will function.__

Cases for this include
- Batch system run on local host.
- Job run without batch system on login node of a cluster.
- Batch system run from login node of a cluster.

```ini
[job platforms]
  [[example platform]]
    run directory =
    work directory =
    task communication method =
    submission polling intervals =
    execution polling intervals =
    scp command =
    ssh command =
    use login shell =
    login hosts =                         # list of possible login hosts
    batch system =                        # name of batch system
    cylc executable =
    global init-script =
    retrieve job logs =
    retrieve job logs command =
    retrieve job logs max size =
    retrieve job logs retry delays =
    task event handler retry delays =
    tail command template =
    [[batch systems]]
        err tailer =
        out tailer =
        err viewer =
        out viewer =
        job name length maximum =
        execution time limit polling intervals =


    [[default directives]]                # This is probably something to do
      --some-directive="directive here!"  # sometime after cylc8
```

The `job platform` to use will be set for a task in the top level of the
`[runtime][task]` definition using the key `platform = <name of platform>`

#### 3.2 Legacy `[runtime][TASK][job]` & `[runtime][TASK][remote]` behaviour

All the items in `[runtime][TASK][remote]` will be deprecated in favour of equivelent
items in a `job platform`. The following items from `[runtime][TASK][job]` will also
be deprecated in favour of items in `[job platforms]`:
- `batch system`
- `batch submit command template`
- `submission polling intervals`
- `submission retry delays`


`[runtime][TASK][remote]hosts` will be deprecated but we need to ensure that 
we can handle deprecated useage in a sophisticated way. For back compatibility 
if host `host` is a login node defined in `[job platforms]` then that job 
will use that platform. If a user wishes to over-ride this they can over-ride the
`[job-platforms][[PLATFORM]]` section.

The other settings to be moved to `job platforms` could be subject to these
strategies:
1. Raise a warning, ignore the deprecated keys, and use the `job platform` 
   selected using the value in `hosts`.
2. Raise a warning and over-ride the values in `job platform`
3. Raise an error and stop everything dead.

### 4. Locking down global settings
There should be a mechanism by which system administators can lock global
settings for their sites. This should probably be a `lock=True/False` switches
within those settings that we wish to make lockable. If unset these will
default to `False`. If set to `True` a user who tries to over-ride that setting
will see a warning explaining that the setting has been over-ridden by a site
admin.
