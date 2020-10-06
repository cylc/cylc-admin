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
   * This is big change in working practise but Cylc 8 is a good time to
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
   * Extend this to support equivalent information for git.

### CLI changes

`cylc install` - new command to install workflows.
* By default, `cylc install` will install the workflow found in `$PWD` into
  `~/cylc-run/$(basename $PWD)/runN` (where `runN` = `run1`, `run2` ...)
  and then run it.
* If `run1` already exists it will install into `run2` (and so on).
  Create a symlink `runN` pointing at the latest run.
* `--run-name=my-run` implies install into `~/cylc-run/$(basename $PWD)/my-run`
  (a "named" run). If the target directory already exists then fail.
* `--no-run-name` implies install into `~/cylc-run/$(basename $PWD)`.
* `--flow-name=my-flow` implies install into `~/cylc-run/my-flow/runN`.
* `--directory=/path/to/flow` implies install the workflow found in
  `/path/to/flow` (rather than `$PWD`).
* All the options accepted by `cylc play` will be accepted by `cylc install` and
  applied when the workflow is run. Relevant options will also apply to other
  cylc commands when operated on the installed suite (e.g. `cylc validate`).
* `--install-only` implies install the workflow, validate it and record what
  options are to be used when the workflow is run but don't actually run it.
  * This will require the creation of a new options file. Note that these
    options will provide the defaults for subsequent commands such as
    `cylc play` but can still be overridden.
* Installation will involved copying over the files found in the source directory
  * This will exclude any `.git` or `.svn` directories.
  * Should it also exclude any `log`, share` or `work` directories?
  * Should we configure what get copied or just copy everything else?
  * The installation should fail if none of the files `rose-suite.conf`,
    `suite.rc` or `cylc.flow` exist.
  * The installation should fail if the target directory is not valid.
    e.g. you cannot install into `~/cylc-run/my-flow/runN` if `~/cylc-run/my-flow`
    already contains an installed suite.
* Version control information will be recorded where relevant (svn/git).
* The `log,`share`, share/cycle` and `work` directories  will be created
  following whatever symlink rules are defined for localhost
  (over-ridable via command line options).
* Support Rose optional configuration via the environment variable
  `ROSE_SUITE_OPT_CONF_KEYS` or the option `--rose-opt-conf-key=KEY`.

`cylc play` - new command to run an installed workflow.
* Will use any relevant options specified as part of the install unless
  overridden on the command line.
* Options will look very similar to current `cylc run` command but with no
  `[REG]` or `[START_POINT]` arguments.
* Fail if the workflow is already running or has completed
  (need to be able to detect completed run).
* Continue running the workflow if it was stopped before completion
  (replaces `cylc restart`).
* `--rerun` implies rerun the installed workflow from the beginning.
  * Not the normal way of working but still required for some use cases
    (e.g. operational warm starts in cases where reflow is not considered
    possible).
  * Implies each new run needs to use a new timestamped log directory
    (similar to current `rose suite-run`).
    However, propose not to support tar gzipping of these directories.
    Also, do not propose to provide access to the old logs via the UI.
* `--clean[=log,share,share/cycle,work] implies clean out the relevant runtime
  directories prior to re-running (implies `--rerun`).
  Note: cleaning of install targets other than localhost should be done at run
  time via remote init to avoid start up delays.
* `--reload` - equivalent to starting the workflow and then immediately
  performing a reload
  * `cylc play` will always load the latest workflow definition found in the run
    directory and will respect any command line options (which may alter the
    workflow definition). Therefore the `--reload` option only applied to
    workflows installed via `cylc install` and implies updating the workflow
    definition from the source.
  * The processed `cylc.flow` files will be kept as at present so there will be
    a record of any changes.

Note that REG / SUITE will be removed as an argument from other relevant
commands (e.g. `cylc validate`, `cylc hold`). Instead, commands will expect to
find the workflow in $PWD unless the workflow location is specified via a
`-C, --directory` option (like tar).

`cylc validate` - as per existing command with the following changes:
* If validating a workflow installed with `cylc install` then use any relevant
  command line options specified as part of the install unless overridden on the
  command line.
* Will work fine on source directories which include `rose-suite.conf` files.
* No `--strict` mode
  (naked dummy tasks no longer allowed - all tasks must be declared).

`cylc reload` - as per existing command with the following changes:
* Update the workflow source files from the source directory (if installed via
  `cylc install`) unless `--no-source` is specified.

`cylc hold|release|unhold` - no change

`cylc register` - remove: functionality should now be performed by
`cylc install` or `cylc play` with no need for a separate command.

`cylc run|start` - remove: replaced by `cylc play`
* Should we stick with `cylc run` rather than `cylc play`?

`cylc restart` - remove: replaced by `cylc play`

`cylc clean` - new command to replace `rose suite-clean`

Once the main CLI changes are in place we should do a complete review of the
entire cylc command set to see whether there are further commands which should
be retired or altered.

[Rose commands](https://metomi.github.io/rose/doc/html/api/command-reference.html#rose-commands):
* Propose to retire (i.e. no backwards compatibility support):
  * `rose suite-clean`
  * `rose suite-cmp-vc` - not proposing a replacement for this at the moment
  * `rose suite-gcontrol`
  * `rose suite-hook` - has been deprecated for some time now
  * `rose suite-log` - need to check whether it is still in use with the `--update` or `--archive` options
  * `rose suite-restart`
  * `rose suite-run`
  * `rose suite-scan`
  * `rose suite-shutdown`
* Assume we will still need to support `rose stem`

Related issues / notes:
* [cylc-flow #1030](https://github.com/cylc/cylc-flow/issues/1030)
* [Workshop notes](https://cylc.github.io/cylc-admin/feb2020-workshop-notes#wednesday).
  Note that this proposal differs in a number of ways from what was discussed at the workshop:

  * no git version control of run dir (future enhancement?)
  * can't use play to unhold (a held suite should restart in held mode)
  * no numbered run dirs for named runs?
  * warm starts continue in same run dir?
  * proposing to remove old commands rather than deprecate (hard to do safely)

### Log file changes

Proposed contents of the `log` directory in the run directory.

* `db` - no change
* `job/` - no change
* `scheduler/` - replaces `suite/`
* `conf/` - replaces `suiterc/`

  * or put in `scheduler/` ?
  * move suite.rc.processed here as well

* `install/<timestamp>-install.log` (replaces `rose-suite-run.log`)
* `install/<timestamp>-reload.log` (currently appended to `rose-suite-run.log`)
* `install/<timestamp>-vc.log` (replaces rose-conf/*.version)
* Items to remove:

  * `rose-suite-run.locs` - not required/useful
  * `rose-conf/*.conf` - recorded in the workflow config so not needed?
