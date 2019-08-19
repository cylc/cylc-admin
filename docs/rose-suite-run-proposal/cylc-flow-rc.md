# Proposal example of a new style `cylc-flow.rc` file

## References
[cylc-admin PR #40](https://github.com/cylc/cylc-admin/pull/40)
[cylc-admin work plan](../proposal-rose-suite-run.md)

## Purpose
As part of the work to transfer functionality from `rose suite run` to cylc
it is proposed that the options available in global, user and suite config
files are brought into alignment. This file sets out a specification for the
combined `cylc-flow.rc` file.

## Preabamble

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

### Top level sections to copy from `suite.rc` to `cylc-flow.rc`

Most sites will leave these to users (Although I could imagine adding a copy-
right message to meta by default, for example, and one user has suggested a
very simple runtime might be added too, for training and debugging purposes.)

```ini
[meta]
  ...
[scheduling]
  ...
[vizualization]
  ...
```

### Top level sections to copy from `global.rc` to `cylc-flow.rc`

It is likely that most users will continue to have these set by site admins.

```ini
[task events]
  ...
[test battery]
  ...
[suite host self-identification]
  ...
[authentication]
  ...
```
The `[suite servers]` to be deprecated in favour of suite platforms.


### Entirely new top level sections

#### `[job platforms]`
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

#### `[suite platforms]`
This is the dictionary key formerly known as ``[suite servers]``. Changed only
for the purpose of keeping the name "platforms" conistent. This is expected to
be set only by system administrators.

```ini
[suite platforms]
```


### Top level sections to merge in a more complex way
#### `[cylc]`
`global.rc` at present contains a subset of the items available in `suite.rc`
for this section so it is proposed that the new fill just has the larger set.
```ini
[cylc]
  health check interval = 600
  task event mail interval = 300
  [[events]]
```

`[hosts]` will be deprecated but we need to keep many of its settings
in `[job platforms]`. Old `[[job]]` & `[[remote]]` sections to be merged and rationalized, being replaced by a new `[[job platform]]` section.

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
```
We could do some quite sophisticated logic here:


if "host" in ``[job platforms][[platform]]->login hosts``:
  pick any host in ``[job platforms][[platform]]->login hosts``

This means that users are forced onto the new behaviour by default.
Users who want a specific host can over-write
``[job platforms][[platform]]->login hosts`` in their suite `flow.rc`

```ini
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
