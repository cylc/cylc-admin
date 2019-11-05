# Proposal - initial support for platforms

This will close [cylc-flow #3421](https://github.com/cylc/cylc-flow/issues/3421)
and partially address
[cylc-flow #2199](https://github.com/cylc/cylc-flow/issues/2199).

## Key functionality

* Introduce configuration and logic to recognise compute platforms in place of
  task hosts. This will resolve a number of issues.
  * In particular, this allows a compute cluster which can be accessed via
    multiple login nodes (hosts) to be treated as logical unit.
  * We will be able to configure multiple platforms which share the same hosts
    (including `localhost`).

* If a platform has a list of hosts then the host used for each operation on that
  platform (job submission, polling, log retrieval, etc) will be randomised. This
  should help balance the load. If a host becomes unavailable, try another random
  host from the list, until exhausted. This will handle hosts becoming
  unavailable after a job is submitted which is a major limitation of the current
  logic.

* It will be possible to specify the platform for a task as a platform alias
  which defines a list of platforms. For example, you may have 2 different HPC
  systems (configured as separate "platforms") and you are happy for a task to
  run on either. The list will be randomised and the platforms tried in turn
  until successful job submission.
  * Note that the platform will be selected each time a job is run. If you
    specify a platform alias which defines a list of platforms, tasks must not
    assume that retries of that task will use the same platform.

* It will be possible to specify the platform as a command in the same way as we
  can currently specify hosts as a command. The command must return a valid
  platform (or platform alias). We hope that this functionality will seldom be
  needed in the future (the main use at present is to select from a list of login
  nodes). However, there are still cases where this may be needed. For example
  running a command which:
  * Returns the platform which has the shortest queue time.
  * Chooses the platform according to the day of the week (perhaps you have a
    platform which can only be used at weekends?).

* Note: it will **not** be possible to specify the platform as an environment
  variable in the same way as we can currently specify remote hosts as an
  environment variable (e.g. `platform = $ROSE_ORIG_HOST`).
  * Unclear why we need this given the Jinja2 variables can be used?
  * Using `$ROSE_ORIG_HOST` relies on Rose specifying extra options to set this
    in `[cylc][environment]` which we should avoid if possible.

## Example platform configurations

```ini
[platforms]
    [[desktop\d\d|laptop\d\d]]
        # hosts = platform name (default)
        # Note: "desktop01" and "desktop02" are both valid and distinct platforms
    [[sugar]]
        hosts = localhost
        batch system = slurm
    [[hpc]]
        hosts = hpcl1, hpcl2
        retrieve job logs = True
        batch system = pbs
    [[hpcl1-bg]]
        hosts = hpcl1
        retrieve job logs = True
        batch system = background
    [[hpcl2-bg]]
        hosts = hpcl2
        retrieve job logs = True
        batch system = background
[platform aliases]
    [[hpc-bg]]
        platforms = hpcl1-bg, hpcl2-bg
```

Note:

* The default for `hosts` is the platform name and platform section headings can
  be regular expressions to match multiple platform names. This allows us to
  define many platforms in a single section and handles the case where you have
  a large number of separate servers which are identical in terms of their
  platform properties other than their host name.

* The current logic for finding matching host settings is complicated and not
  fully
  [documented](https://cylc.github.io/doc/built-sphinx/appendices/site-user-config-ref.html#hosts-host)
  (sections defining individual host names take precedence over pattern matches
  regardless of where they come in the order). We propose a different logic for
  finding matching platform settings:
  * The pattern will be matched against the full name using
    [re.fullmatch](https://docs.python.org/3/library/re.html#re.fullmatch)
    (rather then `re.match` which is currently used for matching hosts).
  * A hierarchy of platform match expressions from general to specific can be
    used because config items are processed in the reverse order specified in the
    file (i.e. bottom up). This is opposite to the method currently used for
    matching hosts but is required because any additional platforms defined in
    user config files get added to the end of the list. Currently it is
    impossible to add hosts defined as regular expressions in the user config if
    there is already a more generic regular expression defined in the global
    config.
  * There will be no special treatment of sections which define a single
    platform. The first matching section found will be used.
  * Note that the first reference to a section in the global or user config
    determines its position in the list. You can repeat a section later in the
    config and alter some settings but this doesn't change its position.

* The names of all new and existing settings are subject to change until we
  release cylc 8.0. Initially names will be chosen which match current usage
  wherever possible. We intend to review all the config settings once we have
  the key functionality in place.

* The default platform is `localhost`. At cylc 7, any settings for `localhost`
  act as defaults for all other hosts but we do **not** propose doing this for
  platforms. The downside is more duplication if you need to override a
  particular setting for every platform but we think this is clearer and safer.

* If `hosts` is defined then the platform section heading should specify a single
  platform name - otherwise you are defining lots of identical platforms which
  makes no sense (although it is relatively harmless so we probably won't try to
  prevent this or give any warnings if this happens).
  * As a future enhancement we could support the use of string substitution using
    [re.sub](https://docs.python.org/3/library/re.html#re.sub) (this would make
    the deprecation logic more complicated so we don't propose doing this at
    first). e.g. something like:
```ini
[platforms]
    [[hpcl[12]-bg]]
        hosts = ("-bg$","") # or "hosts = s/-bg$//" ?
        retrieve job logs = True
        batch system = background
```

## Example platform usage

These are examples of how the platform may be assigned in the `[runtime]`
section for a job. They refer to the example platforms shown above.

`platform = desktop01`
* The simplest example. The job will be run on the host `desktop01`.

`platform = special`
* There is no platform named `special` defined so this will fail validation.

`platform = hpc`
* The host used will be chosen at random.

`platform = hpc-bg`
* The platform used will be chosen at random ("hpcl1" or "hpcl2").

`platform = $(rose host-select linux)`
* No checking can be performed at validation so if the command does not return a
  valid platform this will result in a submit failure.

## Deprecation method

* You can't mix old with new. If `platform` is defined for **any** task then any
  settings in the `[remote]` or `[job]` sections of **any** task result in a
  validation failure.

* Any settings in the `[remote]` or `[job]` sections which have equivalent
  settings in the new runtime section are converted using the normal deprecate
  method. e.g. `[job] execution time limit`.

* The remaining `[remote]` & `[job]` settings are converted by searched for a
  matching platform. If there is none the conversion fails resulting in an
  error.

* For fixed remote hosts the conversion happens at load time and the lack of a
  matching platform results in a validation failure.

* For remote hosts defined as a command or an environment variable the
  remaining `[remote]` & `[job]` settings are stored and the conversion happens
  at job submission. In this case the lack of a matching platform results in a
  submission failure.

* The `[remote] host` setting is matched first by checking the platforms
  `hosts` setting. If this is not defined then the match is made against
  the valid platform names.
  * We may need special logic so that if `[remote] host` is set to the workflow
    server hostname then this is converted to `localhost` before matching
    occurs.

## Deprecation examples

```ini
[runtime]
    [[alpha]]
        [[[remote]]]
            host = localhost
        [[[job]]]
            batch system = background
        # => platform = localhost (set at load time)

    [[beta]]
        [[[job]]]
            batch system = slurm
        # => platform = sugar (set at load time)

    [[gamma]]
        [[[remote]]]
            host = $(rose host-select hpc)
            # assuming this returns "hpcl1" or "hpcl2"
        [[[job]]]
            batch system = pbs
        # => platform = hpc (set at job submission time)

    [[delta]]
        [[[remote]]]
            host = hpcl1
        [[[job]]]
            batch system = background
        # => platform = hpcl1-bg (set at load time)
```

## Enhancements to support rose suite-run functionality

Several additions to the platform support will be needed to support rose
suite-run functionality:

1. Installation of suites on platforms
   * We already have logic to install service files on remote platforms just
     before the first set of jobs is submitted to each platform after suite
     start-up or on suite reload. This system needs to be extended to other
     items of the suite.
   * Consider whether we can make this work on a filesystem rather than a
     platform basis?
     * Configure the id of the filesystem used on a particular platform
       (defaults to the platform name).
     * If multiple platforms share the same root filesystem then use a common
       id.
     * Would need to consider the case where the root filesystem is shared but
       log, work or share differ.

2. Support for moving the share, work & log directories to different locations
   with optional symlinks to the root directory. Similarly support for moving
   the root directory with symlink to the `cylc-run` directory.

## Further enhancements

There are a number of ideas for enhancements (some of which are referenced in
[cylc-flow #2199](https://github.com/cylc/cylc-flow/issues/2199)) which we will
not attempt to address in the initial implementation. These include:

* Specify whether the root filesystem on a platform is shared with the workflow
  server. Currently it is assumed to be separate so installation is attempted on
  all remote hosts. There is logic to detect that it is shared but it would be
  more efficient to configure this. Note: already covered if we install on a
  filesystem basis as discussed above.

* Define default directives for all jobs on a platform.

* Support task management commands (kill, poll) by platform.

* Hold all tasks that target a particular platform.
  * How would this work with platform aliases and platforms specified via
    commands?

* Limit the number of jobs submitted to a platform at any one time.

* Custom logic to invoke for collecting job accounting information when a job
  completes.

* If a platform has a list of hosts, support other methods for choosing which
  host to use (rather than just random) using the functionality already
  implemented for selecting the workflow server host.
  * Since we are not running resource intensive commands on these hosts it's not
    clear whether this functionality is really needed?
  * Note that we can't simply use the existing funtionality for the random host
    selection since we need to try each host in turn until the job submission
    suceeeds (rather than choose a single host to attempt the job submission).

* If a platform alias has a list of platforms, support other methods for choosing
  which platform to use (rather than just random).
  * Platform aliases can then replace all the current use cases for
    `rose host-select`.
  * We could simply allow `platforms` to be specified as a command? This would
    provide an alternative to specifying a platform as a command in the suite
    (configure a platform alias instead).

* The `[suite servers]` section is configuring something very similar to a
  platform alias. Should we try to unify the approach (i.e specify the suite
  servers as a platform)?

* Support the use of string substitution to derive the host from the platform
  name (discussed above).

* Support platform inheritance so that you can define a platform based on an
   existing platform to reduce duplication. e.g.
```ini
[platforms]
    [[hpcl1-bg]]
        inherit = hpc
        hosts = hpcl1
        batch system = background
```

Note that these are just ideas for possible enhancements - no assumptions are
made at this stage as to which ones are worth implementing.
