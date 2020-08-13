# Proposal - initial support for platforms

This will close [cylc-flow #3421](https://github.com/cylc/cylc-flow/issues/3421)
and partially address
[cylc-flow #2199](https://github.com/cylc/cylc-flow/issues/2199).

## Key functionality

* Introduce configuration and logic to recognise compute platforms in place of
  task hosts. This will resolve a number of issues.
  * In particular, this allows a compute cluster which can be accessed via
    multiple login nodes (hosts) to be treated as a logical unit.
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
  * Unclear why we need this given that Jinja2 variables can be used?
  * Using `$ROSE_ORIG_HOST` relies on Rose specifying extra options to set this
    in `[cylc][environment]` which we should avoid if possible.

## Example platform configurations

```ini
[platforms]
    [[desktop\d\d,laptop\d\d]]
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
        # other config(s) to be added here for load balancing etc

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
* The platform used will be chosen at random ("hpcl1-bg" or "hpcl2-bg").

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

* The remaining `[remote]` & `[job]` settings are converted by searching for a
  platform which matches **all** the items set. If there is none the conversion
  fails resulting in an error.

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
        [[[remote]]]
            host = desktop01
        [[[job]]]
            batch system = slurm
        # => validation failure (no matching platform)

    [[gamma]]
        [[[job]]]
            batch system = slurm
        # => platform = sugar (set at load time)

    [[delta]]
        [[[remote]]]
            host = $(rose host-select hpc)
            # assuming this returns "hpcl1" or "hpcl2"
        [[[job]]]
            batch system = pbs
        # => platform = hpc (set at job submission time)

    [[epsilon]]
        [[[remote]]]
            host = $(rose host-select hpc)
        [[[job]]]
            batch system = slurm
        # => job submission failure (no matching platform)

    [[zeta]]
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
   * We will modify this to work on an install target basis rather than a
     platform basis.
     * Configure the id of the target used on a particular platform
       (`install target`: defaults to the platform name).
     * If multiple platforms share the same filesystems then use a common
       `install target`.
     * Note that, when we add we support for separate client keys, these will
       now be per `install target` rather than per platform.
     * Cylc 7 creates a `.service/uuid` file to detect whether a remote host
       shares the same filesystem as the scheduler host - we can remove this
       functionality. Instead we should check for the existence of a client key
       and fail the remote-init if one is found for a different `install target`
       (different targets used within the same workflow should not share the
       same run directory).
   * Currently, the files which need to be installed are transferred as a tar
     file via stdin. This has the advantage that the remote init can be done as
     a single ssh. We will change the installation to use rsync which will
     require a second ssh but this will be better for installing more files and
     allow the logging to report which files have been changed after a reload.
     * If we change cylc to re-use ssh connections by default (using
       `ControlPersist`) then this will avoid the ssh overhead (and will
       benefit other ssh connections as well).
     * We will probably need a third ssh connection for transferring the client
       public key.
   * The remote installation will include particular files/directories.
     * Items to include: `.service/server.key app/ bin/ etc/ lib/`. Note that
       `.service/contact` is not included since we may need to be able to modify
       this file on remote hosts depending on the comms method.
     * We will use the `--delete` rsync option so that any files removed from
       from installed directories also get removed from the install targets on
       reload/restart.
     * Does this need to be configurable? (i.e. configure additional items to
       include)? If so, global and/or workflow level?
       * Is there any reason why workflow designers can't just stick to the list
         of supported directories for files that need to be installed?
       * Are there many existing Rose suites which rely on other directories
         being installed? A quick search implies there are quite a few suites
         using a wide range of sub-directories such as `python`, `util`, `data`
         (although it is not clear how many of these would need to be installed
         on remote platforms).
       * If we allow global configuration this is likely to lead to portability
         issues where a suite only works if you modify your global config. If
         additional items need to be installed it's better to force these to be
         configured in the suite. Note that this will require changes to some
         Cylc 7 suites in order for them to work at Cylc 8.
       * Hopefully it should be fine to install the same files to all install
         targets. If there are cases where this is an issue we can recommend the
         use of install tasks as a workaround.
       * `rose suite-run` currently installs everything other than a set list of
         exclusions. However, it has been known for suites to write to top level
         directories (possibly not deliberate) so this is not a good default
         behaviour (especially with the `--delete` rsync option).
       * Proposal:
         * Install the same items to all install targets.
         * Allow the installation items to be configured at the suite level via
           either of the following methods:
           * A new setting `[scheduler]install` (with a default value of None)
             where you can specify extra top level directories and files to be
             installed, e.g. `extra-dir-to-copy/ extra-file-to-copy`
             (directories must have a trailing slash). Patterns are allowed (as
             defined by rsync), e.g `*/ *` would mimic the current
             `rose suite-run` behaviour.
           * Create an extra file in the suite directory named
             `.rsync-filter` where you can specify rsync filter rules. For
             example, to configure an additional directory
             "data" to be copied you would simply create the file
             `.rsync-filter` in the top level directory and add the line
             `+ /data/***`. We can document a simple example but refer users who
             want to do anything complicated to the
             [rsync man page](https://linux.die.net/man/1/rsync). Advantages:
             a) gives users access to the full power of rsync filters;
             b) this file can be added to cylc 7 suites without breaking them.
         * The rsync filter options will be applied in the following order (note
           that the first matching pattern is acted on):
           * Include files required by cylc:
             `--include='/.service/' --include='/.service/server.key'`
           * Exclude directories which should never be copied:
             `--exclude='.service/***' --exclude='log' --exclude='share' --exclude='work'`
           * Apply any rules defined in an `.rsync-filter` file:
             `--filter=': .rsync-filter'`
           * Add the standard set of directories:
             `--include='/app/***' --include='/bin/***' --include='/etc/***' --include='/lib/***'`
           * Add any extra items defined in the `[scheduler]install` suite
             setting. Note that any items ending in `/` will have `***` added to
             the pattern (which matches everything in the directory).
           * Exclude everything else: `--exclude='*'`

2. Support moving some directories to different locations with symlinks to the
   original location.
   * New settings:
     * `[symlink dirs][<install target>]run` (default: `none`):
       Specifies the directory where the workflow run directories are
       created. If specified, the workflow run directory will be created in
       `<run dir>/<workflow-name>` and a symbolic link will be
       created from `$HOME/cylc-run/<workflow-name>`. If not specified the
       workflow run directory will be created in
       `$HOME/cylc-run/<workflow-name>`. All the workflow files and the
       `.service` directory get installed into this directory.
     * `[symlink dirs][<install target>]log` (default: `none`):
       Specifies the directory where log directories are created. If
       specified the workflow log directory will be created in
       `<log dir>/<workflow-name>/log` and a symbolic link will be
       created from `$HOME/cylc-run/<workflow-name>/log`. If not specified
       the workflow log directory will be created in
       `$HOME/cylc-run/<workflow-name>/log`.
     * `[symlink dirs][<install target>]share` (default: `none`):
       Specifies the directory where share directories are created. If
       specified the workflow share directory will be created in
       `<share dir>/<workflow-name>/share` and a symbolic link will
       be created from `<$HOME/cylc-run/<workflow-name>/share`. If not
       specified the workflow share directory will be created in
       `$HOME/cylc-run/<workflow-name>/share`.
     * `[symlink dirs][<install target>]share/cycle` (default: `none`):
       Specifies the directory where share/cycle directories are created.
       If specified the workflow share/cycle directory will be created in
       `<share/cycle dir>/<workflow-name>/share/cycle` and a symbolic link
       will be created from `$HOME/cylc-run/<workflow-name>/share/cycle`.
       If not specified the workflow share/cycle directory will be created in
       `$HOME/cylc-run/<workflow-name>/share/cycle`.
     * `[symlink dirs][<install target>]work` (default: `none`):
       Specifies the directory where work directories are created. If
       specified the workflow work directory will be created in
       `<work dir>/<workflow-name>/work` and a symbolic link will be
       created from `$HOME/cylc-run/<workflow-name>/work`. If not
       specified the workflow work directory will be created in
       `$HOME/cylc-run/<workflow-name>/work`.
   * What happens if you want to be able to have some workflows which use
     different directory configurations on the same "platform"? For
     example, you might have some workflows where you want the run
     directory on $HOME and others on $DATADIR.
     * For platforms other than localhost the answer is to create separate
       platforms and then choose the appropriate platform in the workflow.
     * For localhost it is more complicated since there is only one localhost
       platform and the settings are applied earlier; at workflow install or
       startup time (rather than prior to first job submission). We will need a
       way to override the directory settings for localhost, e.g. on the command
       line.
       * Cylc 8 already has a way to
         [symlink individual run directories](https://github.com/cylc/cylc-flow/pull/2935).
         The interface may change but this proposal will still support this
         functionality.
   * What environment variables can be used in these settings?
     * Cylc currently only allows `$HOME` or `$USER` in the
       [run and work directory settings](https://cylc.github.io/doc/built-sphinx/appendices/site-user-config-ref.html#hosts-host-run-directory).
       With Rose you can use any variable which is available when the
       remote-init command is invoked on the remote platform. Should we adopt a
       similar approach for Cylc?
     * Rose also provides the option of using the login shell for remote
       commands (which may give access to more environment variables). Should we
       add similar support to Cylc for sites that allow it?
   * Symlink directories are specified per install target rather than per
     platform. Any platforms which use the same install target must use the same
     symlink directories. Note that the assumption is that there are no other
     settings which need to be defined per install target.
     * What about comms method? If possible this should be supported per
       platform. For example you could have a case where HPC login nodes can use
       a different method compared with the nodes where jobs are run. This
       implies comms method needs to go back to being specified in the job
       rather rather than in the contact file.
     * If we ever find a need to define other settings per install target we
       should introduce a new `install target` section and move the
       `[symlink dirs]` section.
   * In order to reduce duplication (and configuration errors) we will support
     platform inheritance so that you can define a platform based on an
     existing platform.
     * Note: `inherit` will imply using the same `install target`.
   * The `[symlink dirs]` settings are only applied when a workflow is installed
     (or first run on a target). Therefore, changes to these settings have no
     affect on running or restarting workflows.
   * At the moment this proposal does not provide any way to relocate the
     top level `$HOME/cylc-run` directory (which you can do currently via the
     existing Cylc `run directory` setting). The assumption is that it is
     sufficient to provide a way to move the run directory and there is no
     reason not to use `$HOME/cylc-run`.
   * Note that these new settings replace the existing Cylc `run directory` and
     `work directory` settings.

Example platform configurations:
```ini
[platforms]
    # The localhost platform is defined by default so there is no need to
    # specify it if it just uses default settings
    # [[localhost]]
    [[desktop\d\d,laptop\d\d] # Specify install target
        install target = localhost
    [[desktop\d\d,laptop\d\d]] # Equivalent using inherit
        inherit = localhost
    [[sugar]]
        inherit = localhost
        hosts = localhost
        batch system = slurm
    [[hpc]]
        hosts = hpcl1, hpcl2
        retrieve job logs = True
        batch system = pbs
    [[hpcl1-bg]]
        inherit = hpc
        hosts = hpcl1
        batch system = background
    [[hpcl2-bg]]
        inherit = hpcl1-bg
        hosts = hpcl2
[symlink dirs]
    [[localhost]]
        log = $DATADIR
        share = $DATADIR
        share/cycle = $SCRATCH
        work = $SCRATCH
    [[hpc]]
        run = $DATADIR
        share/cycle = $SCRATCH
        work = $SCRATCH
# Alternative if we are concerned there may be other install target properties
[install targets]
    [[localhost]]
        [[[symlink dir]]]
            log = $DATADIR
            share = $DATADIR
            share/cycle = $SCRATCH
            work = $SCRATCH
```

## Further enhancements

There are a number of ideas for enhancements (some of which are referenced in
[cylc-flow #2199](https://github.com/cylc/cylc-flow/issues/2199)) which we will
not attempt to address in the initial implementation. These include:

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
  * Note that we can't simply use the existing functionality for the random host
    selection since we need to try each host in turn until the job submission
    succeeds (rather than choose a single host to attempt the job submission).

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

Note that these are just ideas for possible enhancements - no assumptions are
made at this stage as to which ones are worth implementing.
