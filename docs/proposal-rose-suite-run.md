# Proposal - rose suite-run functionality migration to cylc

## Goals

Complete [cylc/cylc#1885](https://github.com/cylc/cylc/issues/1885)

## Synopsis

The work described in this document aims to:
* Deprecate `rose suite-run`.
* Provide a new cylc command to replace `rose suite-run`.
* Change the way hosts for jobs are selected to improve support for clusters.
* Rationalize the formats of settings `*.rc` files.


## Background

In the early days of Rose, there was the desire for Rose to be an umbrella
system that would drive multiple workflow engines. The command `rose suite-run`
was written as a top level utility that was supposed to work with Cylc as well
as other workflow engines. It was designed with extra functionalities that
would appeal to users who also use other parts of Rose. It was thought that
the design would allow Cylc to maintain its independence from these
functionalities of Rose.

In reality, Cylc has become the centre of the Rose application suites, and the
extra functionalities provided by Rose are equally desirable on sites who
choose not to use Rose.

The logical step is therefore to move these extra functionalities provided by
Rose into Cylc - and retire `rose suite-run` and related utilities - so that
all suites will start up with only Cylc commands in the future.

## Work Plan

- [ ] Agree future form of `cylc-flow.rc` [Cylc-Admin #43](https://github.com/cylc/cylc-admin/issues/43)
  - [x] Create new issue against cylc-admin
  - [ ] Create a template `flow.rc`, listing all the options
    to be available in the combined `flow.rc`. [Link to it](example_files/flow.rc)
  - [ ] Deprecate settings for job host?
  - [ ] Add sections for `[job platforms]`


- [ ] Implement new `cylc-flow.rc` schema. [Cylc-flow #3260](https://github.com/cylc/cylc-flow/issues/3260)
  - [ ] Check old tests for `global.rc` & `suite.rc` to ensure that functionality
    is not lost.
  - [ ] Devise tests for the new `cylc-flow.rc`
  - [ ] Create new config schema module, called `cylc.flow.config_schema`
  - [ ] Remove the `cylc.flow.cfgspec` folder.


- [ ] Implement Cluster support functionality. [Cylc-flow #2199](https://github.com/cylc/cylc-flow/issues/2199)
  - [ ] Modify modules to use the new variables:
    - [ ]  `task_job_mgr.py`
    - [ ] `task_remote_mgr`    
  - [ ] Randomize login host used


- [ ] Agree names and functionality of future `cylc flow`(working name)
  command and command args.[#1030](https://github.com/cylc/cylc-flow/issues/1030)
  - [ ] Summarize debate on ticket.
  - [ ] Create a PR against cylc-admin creating a specification for `cylc flow`


- [ ] Create a `cylc flow` CLI.
  - [ ] Include suite validation in `cylc flow` CLI.


- [ ] Replace `rose-suite.conf`.
  - [ ] Create an alternative python config file and tests for
    these format.
  - [ ] Create a cylc plugin to maintain back compatibility by parsing
    rose-suite.conf


## Functionalities to Consider

These functionalities are currently provided by Rose, but should really be part
of Cylc:

* On start up and reload, install suite on suite server cluster (cylc servers).
* On start up and reload, validate suite.
* On start up, archive old log directory.
* On start up and reload, install suite on all task job remotes (such as Cray,
  Spice, remote-desktop, raspi, suite-origin-desktop).
* Utility to clean locations that are known to be created by the suite.
* On starting up a new run of the suite assign a new run name:
  ```
  mi-aa001/
    run1
    run2
  ```

While we consider the above, we may also want to consider the following:
* Introduce configuration and logic to recognise suite clusters and task hosts
  as logical units (e.g. pools of identical servers, HPCs, compute clusters).
  Instead of working with remote hosts, we'll work with local and remote
  compute clusters/platforms.
* Rationalise run/restart/reload CLI/API? E.g. A single command to unify
  `cylc run` and `cylc restart`, with a cleaner set of options and arguments.
* Rationalise related settings from `suite.rc`, `global.rc`, `rose-suite.conf`:
  * Sections and keys should be idenitical between `flow.rc`
    (`global-flow.rc`?) in a global location and a suite `flow.rc` (`suite-
      flow.rc`?) in the suite folder.
  * This combined file should prefer to maintain compatibility with `suite.rc`
    for ease of end user upgrade. Users of `global.rc` are generally
    administrators.
  * Although the aim is to make the the two files the same, it is not likely
    that normal users will use all the available settings in both. It is
    expected that in most cases local and global rc have partially overlapping
    subsets of settings:
    ![Venn Diagram showing expected common usage](img/flow_rc_settings_locs.svg)
  * Migrate relevant settings from `rose.conf` and `rose-suite.conf`.
    __UPDATE THIS__
  * Settings such as `run directory` and `work directory` may need better names
    (users think of "work" as a sub-directory of the run directory, but `run
    directory` and `work directory` are configured separately, and the latter
    (work directory) is actually the top level (run directory) location under
    which to put the work directory ... and it may confuse users that a
    non-standard run directory does bring the work directory with it).

### Clusters instead of Hosts as Logical Units

Modern compute resources come in clusters of one or more serving host(s).
A basic assumption is that the serving hosts of a typical cluster
share the same storage units and a single workload management system.

The current configuration in Cylc (and Rose) allows us to configure settings by
host names, but not by the clusters. There are some immediate issues:
* For clusters with multiple login hosts, remote task host management logic
  would consider the multiple login hosts separately - when it only
  really needs to consider one of the available login hosts.
* Suite hosts may happen to be the access hosts for a compute cluster - so it
  is currently difficult to configure separate logic for such a compute cluster
  without affecting the suite cluster.
* Multiple clusters may share the same set of access hosts. (This creates a
  similar problem as above.)
* The `localhost` settings in the current code restrict what we can do when we
  have a cluster of suite hosts.

The most logical way to support installation of suites on suite clusters and
job hosts is to migrate our host-based configuration logic to a cluster-based
configuration logic. Some points to consider:
* Deprecate `global.rc` `[hosts]` section in favour of a `[job platforms]`
  section. The new section should allow us to configure:
  * Login hosts.
  * Batch system or workload manager, and related settings.
  * Directory locations for various usages by suites.
  * Communication protocols:
    * Forward (install, submit, poll, kill, view log, log retrieval, etc).
    * Backward (message and manipulation commands).
* Deprecate `suite.rc` relevant settings under `[runtime][__MANY__]`, but have
  a top level `[job platforms]` section that simply mirrors that of `global.rc` to allow individual suites to override the site/user settings.
* Users will configure tasks to run on clusters instead of hosts/batch systems.
* If relevant, improve alignment with DRMAA Open Grid Forum API?


So, for example, a suite `suite-flow.rc` might look like this:
(Although a detailed specification should also be created)
```ini
[cylc]

    ...

[scheduling]
    ...

[job platforms]
    [[spice]]
        [[[batch system]]]
            class = slurm
            before command = sshare -u %(user_id)           # Blue Sky thinking
            after command = sacct -j %(jobid)s --format=""
    [[xcs]]
        login hosts = xcslr0, xcslr1
        retrieve job logs = True
        [[[batch system]]]
            class = pbs
        [[[default directives]]]
            --some-directive="directive here!"
    [[desktop]]
        login hosts = vld392

[runtime]
    [[Alice]]
        ...

    [[Bob]]
        ...
        [[[job platform]]]
            platform = desktop

    [[Charlie]]
        ...
        [[[job platform]]]
            platform = spice
        [[[directives]]]
            --mem=5
            --ntasks=1

    [[Dai]]
        ...
        [[[job platform]]]
            platform = cray
```

### Clusters Login Hosts Awareness

While we are at it, if a cluster platform has multiple login hosts, Cylc should
be aware so that if a login host is unavailable, it can try another one. To
implement this, the task job management system will:
* Randomise the list of login hosts for each cluster during run time, and
  rotate the usage of login hosts. This should help balance the load.
* If a login host becomes unavailable, try the next one in the list, until
  exhausted.

Other things to consider:
* Job management commands should allow cluster-wide actions. E.g. kill all jobs
  currently submitted to or running on a cluster.
* New setting to configure a command to run to retrieve metrics of a completed
  job that ran on a cluster.

### Installation of Suites on Platforms

We already have logic to install service files on remote platforms just before
the first set of jobs is submitted to each platform or on suite reload. This
system can be extended to other items of the suite. Things to consider:
* There is no need to install suite on a platform, until just before the
  the submission of the first remote job on that platform.
* Install:
  * Create directories on physical locations that are used by task jobs.
  * Mirror suite items from suite platform to job platform.
  * Make the above configurable.
* Add simple command to remove suite information from installed locations on
  demand.

### Installation of Suites on Suite Cluster

This is the functionality to create the run time directory structure, taking
care of the configuration for alternate physical locations, and to install a
suite from source to the run time location. Items can be pulled from different
sources on installation. Other things to consider:
* Configurable settings for directory structure and extra source locations.
* Add ability to housekeep logs and other items generated by the suite.
* Add equivelent functionality to `rose-suite.conf` in a native format,
  probably a jinja2 file.
* Continue to handle `rose-suite.conf` using a plugin preprocessor.
* Automatic variables.
* Add simple command to remove installed locations on demand.

### Suite Validation

The `rose suite-run` command calls `cylc validate --strict` by default.

Automatic suite validation should become the default behaviour for the new
command, as well as for `cylc reload`.


### Rationalise Suite Start Up Commands

This is raised in [#1030](https://github.com/cylc/cylc/issues/1030). Consider
a single command (e.g. `cylc flow`, `cylc go`?) to with
appropriate command switches should be agreed on in a pull
request to this repository modifying [this document](rose-suite-run-proposal/future-cli-conventions.md)
