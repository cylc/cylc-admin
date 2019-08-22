# Proposal example of a new style `cylc-flow.rc` file

## References
[cylc-admin PR #40](https://github.com/cylc/cylc-admin/pull/40)
[cylc-admin work plan](../proposal-rose-suite-run.md)

## Purpose
As part of the work to transfer functionality from `rose suite run` to cylc
it is proposed that the options available in global, user and suite config
files are brought into alignment. This file sets out a specification for the
combined `cylc-flow.rc` file.

## Preamble

To reduce changes required for end users this file format is based on
`suite.rc`: Where aspects of the file are described as unchanged this implies
that file contents are exactly the same as they would be in a `suite.rc`.
Users of the `global.rc` are usually admins or power users and thus changes
to the format of this file are preferable.

It should be noted that although it will be possible to modify all settings in
all contexts, that some settings are more likely to be used in global contexts
and some are more likely to be used in suite configs.

****

## specification for `cylc-flow.rc`

### 1. Top level sections to remain unchanged from `suite.rc`

Most sites will leave these to users (Although I could imagine adding a
copyright message to meta by default, for example, and one user has suggested a
very simple runtime might be added too, for training and debugging purposes.)

```ini
[meta]
  ...
[scheduling]
  ...
[vizualization]
  ...
```

### 2. Top level sections to remain unchanged from `global.rc` to `cylc-flow.rc`

It is likely that most users will continue to have these set by site admins.

```ini
[authorization]
  ...
```
The `[suite servers]` to be renamed `[suite run platforms]`.


### 3 Entirely new top level sections

#### 3.1 `[job platforms]`
Many of the options in this section will be very similar to `[hosts]`
It is expected that these will mainly be set in site `cylc-flow.rc`, but that
small numbers of power users may wish to over-ride them.

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
    copyable environment variables =
    retrieve job logs =
    retrieve job logs command =
    retrieve job logs max size =      [[default directives]]

    retrieve job logs retry delays =
    task event handler retry delays =
    tail command template =
    [[batch systems]]
    # Does this need to be like this? Are there really likely to be more than
    # One batch system per platform.
      [[__MANY__]]
        err tailer =
        out tailer =
        err viewer =
        out viewer =
        job name length maximum =
        execution time limit polling intervals =


    [[default directives]]                # This is probably something to do
      --some-directive="directive here!"  # sometime after cylc8
```



#### 3.2 `[suite run platforms]`
This is the dictionary key formerly known as ``[suite servers]``. Changed only
for the purpose of keeping the name "platforms" conistent. This is expected to
be set only by system administrators.

```ini
[suite run platforms]
# should include formerly free-standing section:
[[suite host self-identification]]
  ...
```


### 4 Top level sections to merge in a more complex way
#### 4.1 `[cylc]` -> `[general]`
This section is to be renamed.

#### 4.2 `[general]` configuration items.
`global.rc` at present contains a subset of the items available in `suite.rc`
for this section so it is proposed that the new fill just has the larger set.
```ini
[cylc]
  health check interval = 600
  task event mail interval = 300
  [[events]]
```

#### 4.3 `[hosts]` deprecation
`[hosts]` will be deprecated but we need to keep many of its settings in
`[job platforms]`. For back compatibility host should re-direct to
`[runtime][[__MANY__]][[[job platform]]]` with a warning and behaviour
described in 4.6.

#### 4.4 `[[task events]]` to be moved to runtime

#### 4.5 `[[[job]]]` & `[[[remote]]]`
Old `[runtime][[__MANY__]][[[job]]]` & `[[[remote]]]`
sections to be merged and rationalized, being replaced by a new
`[runtime][[__MANY__]][[[job platform]]]` section.

We should select the platform defined by `[job platforms]`

```ini
[runtime]

[[job platform]]
  platform =                            
```

I think that we will probably want users to set this in the
[job platforms] section, leaving some of these options here as due-to-be
deprecated back compat over-rides which will give warnings if set?

```ini
    batch system =
    batch submit command template =
    execution polling intervals =
    execution retry delays =
    execution time limit =
    submission polling intervals =
    submission retry delays =
    host =
    owner =
    suite definition directory =
    retrieve job logs =
    retrieve job logs max size =
    retrieve job logs retry delays =
      [[[batch systems]]]
        err tailer =
        out tailer =
        err viewer =
        out viewer =
        job name length maximum =
        execution time limit polling intervals =
```

#### 4.6 Using a single host?
We could do some quite sophisticated logic here:

if "host" in ``[job platforms][[platform]]->login hosts``:
 translate `host=???` into platform = platform containing ??? as a login host.

This means that users are forced onto the new behaviour by default.
Users who want a specific host can over-write
``[job platforms][[platform]]->login hosts``.
