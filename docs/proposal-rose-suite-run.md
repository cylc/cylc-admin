# Proposal - rose suite-run functionality migration to cylc

The work described in this document aims to:
* Change the way hosts for jobs are selected to improve support for clusters.
* Deprecate `rose suite-run` and provide a new cylc command to replace it.

This will complete [cylc-flow #1885](https://github.com/cylc/cylc-flow/issues/1885).

## Work Plan

1. Implement basic platforms support.
   See [platforms proposal](proposal-platforms.md).

2. Add new platforms features required to support rose suite-run functionality.
   See [platforms proposal](proposal-platforms.md#enhancements-to-support-rose-suite-run-functionality).

3. Define new cylc command to replace rose suite-run (and adapt rose suite-run
   to provide backwards compatibility). See [below](#replacing-rose-suite-run).

The remaining points can happen any time after 1) is complete although note that
2) and 3) are higher priority.

5. Support generation of cylc config reference documentation from the code and
   ensure all settings are documented.
   See [cylc-doc #11](https://github.com/cylc/cylc-doc/issues/11).
   Alternatively, review and update the existing documentation.

6. Implement further enhancements to the platforms support.
   See [platforms proposal](proposal-platforms.md#further-enhancements).
   Note that these are not essential for Cylc 8.0.

7. Review all config settings and make any agreed changes.
   See [cylc-flow #3422](https://github.com/cylc/cylc-flow/issues/3422).
   Note that these are not essential for Cylc 8.0 although we should try to
   address any non-controversial changes.

## Replacing rose suite-run

### Key Functionality

1. Installation of suites on platforms and support for moving the share, work &
   log directories to different locations. This will be included in the
   [platforms support](proposal-platforms.md#enhancements-to-support-rose-suite-run-functionality).

2. Support installation of a suite from the source location (working copy) to
   the run location (`cylc-run`).
   * This functionality assumes the source location (working copy) and the run
     location (`cylc-run`) are both accessible on the host where the command is
     run.
   * The installation needs to create the run time directory structure which has
     been configured for the `localhost` platform.
   * Need to be able to update from the source location as part of reload.

3. On start up, archive any previous logs in a tar.gz.
   * Propose to stop supporting this since, in most cases, we will use a
     separate run directory each time a suite is installed.
   * Continue to support a timestamped log dir for the special occasions where
     we re-use an existing run directory (see the discussion in the CLI changes
     below).

4. On start up and reload, validate suite (using `--strict`).
   * Hopefully this can become a standard part of suite start-up rather than a
     separate step. This was difficult previously because cylc loads the suite
     after it has been detached?
   * Consider making all validation strict?
     * If we really want to allow naked dummy tasks in some circumstances then
       we would be better to add a suite configuration for this?
     * Note: we don't allow you to inherit from an undeclared ("naked") family
       so why do we allow you to run an undeclared task?

5. Handle
   [rose-suite.conf](https://metomi.github.io/rose/doc/html/api/configuration/suite.html#rose:file:rose-suite.conf)
   functionality including optional configurations. See
   [cylc-flow #3819](https://github.com/cylc/cylc-flow/issues/3819)
   and [rose #2412](https://github.com/metomi/rose/pull/2412).

6. Utility to remove installed locations (on all platforms) on demand
   (equivalent to `rose suite-clean`).

7. Use a different run directory for each installation of a suite:
   ```
   u-ab123/
     run1
     run2
   ```
   * This is big change in working practice but Cylc 8 is a good time to
     introduce it.
   * Use a symbolic link to identify the latest run which is the default name
     when performing operations on a suite?

8. Consider `rose prune` functionality.
   * Need to make sure it will continue to work correctly.
   * Longer term we should build housekeeping into Cylc, see
     [cylc-flow #1159](https://github.com/cylc/cylc-flow/issues/1159).

9. Record version control information when you install a suite, see
   [cylc-flow #3849](https://github.com/cylc/cylc-flow/issues/3849).
   * `rose suite-run` current supports this for Subversion only
     (`svn info` + `svn status` + `svn diff`).
   * Extend this to support equivalent information for git (and preferably make
     it easy to plugin support for other systems as well).

### CLI changes

`cylc install` - new command to install workflows.
* By default, `cylc install` will install the workflow found in `$PWD` into
  `~/cylc-run/$(basename $PWD)/runN` (where `runN` = `run1`, `run2` ...).
* If `--run-name` is not specified and `run1` already exists it will install
  into `run2` (and so on).
  Create a symlink `runN` pointing at the latest run.
* `--run-name=my-run` implies install into `~/cylc-run/$(basename $PWD)/my-run`.
  If the target directory already exists then fail.
* `--no-run-name` implies install into `~/cylc-run/$(basename $PWD)`.
* `--flow-name=my-flow` implies install into `~/cylc-run/my-flow/runN`.
* `--directory=/path/to/flow` (`-C ...`) implies install the workflow found in
  `/path/to/flow` (rather than `$PWD`).
* Installation will involved copying over the files found in the source
  directory.
  * This will exclude any `.git`, `.svn`, `log`, share` or `work` directories.
    Everything else will be copied.
    * If needed we could make this configurable via a `.cylcignore` file (future
      enhancement). Note that `log`, share` & `work` would always be excluded.
  * The installation should fail if neither `suite.rc` nor `cylc.flow` exist.
  * The installation should fail if the target directory is not valid.
    e.g. you cannot install into `~/cylc-run/my-flow/runN` if `~/cylc-run/my-flow`
    already contains an installed suite.
* Version control information will be recorded where relevant (svn/git).
* The `log, `share`, share/cycle` and `work` directories  will be created
  following whatever symlink rules are defined for localhost
  (over-ridable via command line options).
  * This will continue to support "symlinked alternate run directories"
    (see [cylc-flow #2935](https://github.com/cylc/cylc-flow/pull/2935)).

`cylc play` - new command to run an installed workflow
(replacing `cylc run|start`).
* Options will look very similar to current `cylc run` command but with no
  `[START_POINT]` argument (already supported as an option).
* Fail if the workflow is already running or has completed
  (need to be able to detect completed run).
  * If the workflow is already running the alternative is to treat this as an
    alias to `cylc unpause`.
* Continue running the workflow if it was stopped before completion
  (replaces `cylc restart`).
* If, for some reason, you want to re-run a workflow from the beginning you will
  be able to use `cylc clean` to clean out the log directory (and whatever other
  directories you want removing) after which `cylc play` can be used.
  Note that this means that `cylc play` must be able to use the command
  line options from the previous run as stored in the private database.
  There are 2 options:
  * Require the use of a `--re-run` option to enable this
    (otherwise `cylc play` will fail if it finds an existing private database).
  * Do this automatically if no existing `log` directory is found.
* Note that `cylc play` will always load the latest workflow definition found in
  the run directory and will respect any command line options (which may alter
  the workflow definition). Therefore, continuing a workflow using `cylc play`
  effectively implies a reload.
  * The processed `cylc.flow` files will be kept so there will be a record of
    any changes
    (see also [cylc-flow #3763](https://github.com/cylc/cylc-flow/issues/3763)).
* Support Rose optional configuration via the environment variable
  `ROSE_SUITE_OPT_CONF_KEYS` or the option `--rose-opt-conf-key=KEY`.

`cylc reinstall` - new command to re-install workflows.
* Very similar to the `cylc install` command except that this applies to a
  previously installed workflow (i.e. you run this command in the directory of
  the installed workflow, not in the source directory).
* The rsync will use `--delete` to ensure that any previously installed files
  which have been removed from the source directory also get removed from the
  installed workflow.
  * Note that this will have the effect of removing any files installed via
    `rose-suite.conf`. They will get recreated by a subsequent reload or play but
    could affect a running workflow.
* Any options specified as part of the original install will be retained unless
  overridden on the command line.
* Consider a `--dry-run` option to report what would be changed?
  * Does this replace `rose suite-cmp-vc`?

`cylc reload` - no change from existing command.

`cylc validate` - as per existing command with the following changes:
* If validating a workflow installed with `cylc install` then use any relevant
  command line options specified as part of the install unless overridden on the
  command line.
* Will work fine on source directories which include `rose-suite.conf` files.
* No `--strict` mode
  (naked dummy tasks no longer allowed - all tasks must be declared).
  * This one is controversial and requires further discussion!
* Cyclic graph validation can be quite slow and is currently included as part of
  `--strict`. Add a `--no-cyclic-graph-validation` option to disable this?

Several combinations of the above commands will be commonly used.
Propose to support these as separate commands:
* `cylc install-play` (`cylc ip` for short?)
* `cylc install-validate`
* `cylc reinstall-reload`

`cylc pause|unpause` - new commands replacing `cylc hold|release|unhold`
* Note that when you use `cylc stop` the scheduler will be modified to unpause
  the workflow before stopping.
  This means that a stopped workflow will not be paused when it is continued via
  `cylc play`. On the other, a workflow which is continued for any other reason
  (e.g. the server died) will retain its paused status.

`cylc register` - remove: functionality no longer required.
* `cylc install` will now perform the task of associating the source and run
  directories via a symlink.
* `cylc play` has no need to do this since the workflow definition will now
  always be found in the run directory.

`cylc clean` - new command to replace `rose suite-clean`.
* This should support options which allow you to just clean subsets of the data.
  For example, if you clean the `log` directory this allows you to rerun a
  workflow (keeping the existing data in `share` if that is what you want).
  The default will be to remove everything including the top level directory.
* Support an option to clean all install targets other than localhost?
* We may also want an option to archive the log directory rather than remove it.
  This could simply imply renaming the directory, perhaps with a timestamp
  (similar to what `rose suite-run` does but without tar gzipping the
  directory). This would allow a workflow to be warm started without completely
  losing the previous log files (this is an operational requirement at present
  although we hope that re-flows will reduce or remove the need for this).
  Note that we would not provide any further support for these archived log
  directories (e.g. no support for accessing the logs via the UI).
* The file `log/rose-suite-run.locs` is currently required by `rose suite-clean`
  but `cylc clean` should probably get this information from the database?

Note that many commands currently have REG / SUITE as a required argument
(e.g. `cylc validate`). Where possible these will be changed to be optional
arguments with the default being `$PWD`.
* We could also consider using an option rather than an argument.

For all commands which have been replaced (run, restart, hold, register,etc) we
will replace them with (hidden, not listed in CLI help) commands which simply
report what commands should be used in their place.

Once the main CLI changes are in place we should do a complete review of the
entire cylc command set to see whether there are further commands which should
be retired or altered.

[Rose commands](https://metomi.github.io/rose/doc/html/api/command-reference.html#rose-commands):
* Propose to retire (i.e. no backwards compatibility support):
  * `rose suite-clean`
  * `rose suite-cmp-vc` - check whether we have a replacement for this in cylc first
  * `rose suite-gcontrol`
  * `rose suite-hook` - has been deprecated for some time now
  * `rose suite-log` - need to check whether it is still in use with the `--update` or `--archive` options
  * `rose suite-restart`
  * `rose suite-run`
  * `rose suite-scan`
  * `rose suite-shutdown`
* As with cylc, we will provide hidden commands which simply report what
  commands should be used in their place.
* Assume we will still need to support `rose stem`.

Related issues / notes:
* [cylc-flow #1030](https://github.com/cylc/cylc-flow/issues/1030)
* [Workshop notes](https://cylc.github.io/cylc-admin/feb2020-workshop-notes#wednesday).
  Note that this proposal differs in a number of ways from what was discussed at the workshop:
  * no git version control of run dir (future enhancement?)
  * "in place warm start" now effectively supported via `cylc clean`
  * proposing to make old commands obsolete rather than deprecated (too hard to do safely)
  * probably several other differences as well

### Log file changes

Proposed contents of the `log` directory in the run directory.

* `db` - no change
* `job/` - no change
* `scheduler/` - replaces `suite/`
* `conf/` - replaces `suiterc/` and `rose-conf/*.conf`
  * see also [cylc-flow #3763](https://github.com/cylc/cylc-flow/issues/3763)
* `install/<timestamp>-install.log` (replaces `rose-suite-run.log`)
* `install/<timestamp>-reload.log` (currently appended to `rose-suite-run.log`)
* `install/<timestamp>-vc.log` (replaces rose-conf/*.version)
