# Proposal - rose suite-run functionality migration to cylc

## Goals

Complete [cylc/cylc#1885](https://github.com/cylc/cylc/issues/1885)
with consideration of
[cylc/cylc#2199](https://github.com/cylc/cylc/issues/2199).

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

## Functionalities to Consider

These functionalities are currently provided by Rose, but should really be part
of Cylc:
* On start up and reload, install suite on suite cluster.
* On start up and reload, validate suite.
* On start up, archive old log directory.
* On start up and reload, install suite on all task remotes.
* Utility to clean locations that are known to be created by the suite.

While we consider the above, we may also want to consider the following:
* Introduce configuration and logic to recognise suite cluster and task hosts
  as logical units (e.g. pool of identical servers, HPC, compute clusters).
  Instead of working with remote hosts, we'll work with local and remote
  compute clusters/platforms.
* Rationalise run/restart/reload CLI/API? E.g. A single command to unify
  `cylc run` and `cylc restart`, with a cleaner set of options and arguments.
* Rationalise related settings from `suite.rc`, `global.rc`, `rose-suite.conf`:
  * Where relevant, sections and keys should be idenitical between `suite.rc`
    and `global.rc`. (Or a unified replacement called `cylc-flow.rc`?)
  * Migrate relevant settings from `rose.conf` and `rose-suite.conf`.
  * Settings such as `run directory` and `work directory` may need better names.

### Clusters instead of Hosts as Logical Units

Modern days compute resources come in clusters of one or more serving host(s).
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
* The `localhost` settings restrict what we can do when we have a cluster of
  suite hosts.

The most logical way to support installation of suites on suite clusters and
task hosts is to migrate our host-based configuration logic to a cluster-based
configuration logic. Some points to consider:
* Deprecate `global.rc` `[hosts]` section in favour of a `[clusters]` section.
  (Or `[platforms]`?) The new section should allow us to configure:
  * Login hosts.
  * Batch system or workload manager, and related settings.
  * Directory locations for various usages by suites.
  * Communication protocols:
    * Forward (install, submit, poll, kill, view log, log retrieval, etc).
    * Backward (message and manipulation commands).
* A special section for configuring the suite cluster?
* Deprecate `suite.rc` relevant settings under `[runtime][__MANY__]`, but have
  a top level `[clusters]` section that simply mirror that of `global.rc` to
  allow individual suites to override the site/user settings.
* Users will configure tasks to run on clusters instead of hosts/batch systems.
* If relevant, improve alignment with DRMAA Open Grid Forum API?

### Clusters Login Hosts Awareness

While we are at it, if a cluster has multiple login hosts, Cylc should be aware
so that if a login host is unavailable, it can try another one. To implement,
the task job management system will:
* Randomise the list of login hosts for each cluster during run time, and
  rotate the usage of login hosts. This should help balance the load.
* If a login host becomes unavailable, try the next one in the list, until
  exhausted.

Other things to consider:
* Job management commands should allow cluster-wide actions. E.g. kill all jobs
  currently submitted to or running on a cluster.
* New setting to configure a command to run to retrieve metrics of a completed
  job that ran on a cluster.

### Installation of Suites on Task Remotes

We already have logic to install service files on task remotes just before the
first set of tasks is submitted to each task remote or on suite reload. This
system can be extended to other items of the suite. Things to consider:
* There is no need to install suite on a task remote, until just before the
  the submission of the first remote task on that task remote.
* Install:
  * Create directories on physical locations that are used by task jobs.
  * Mirror suite items from suite cluster to task remote.
  * Make the above configurable.
* Add simple command to remove installed locations on demand.

### Installation of Suites on Suite Cluster

This is the functionality to create the run time directory structure, taking
care of the configuration for alternate physical locations, and to install a
suite from source to the run time location. Items can be pulled from different
sources on installation. Other things to consider:
* Configurable settings for directory structure and extra source locations.
* Add ability to housekeep logs and other items generated by the suite.
* Continue to handle `rose-suite.conf` using a plugin preprocessor or
  otherwise.
* Automatic variables.
* Add simple command to remove installed locations on demand.

### Suite Validation

The `rose suite-run` command calls `cylc validate --strict` by default.
Automatic suite validation should become the default behaviour for `cylc run`
and `cylc reload`. Things to consider:
* Is the *strict* mode necessary by default?
* `rose suite-run` does not currently validate Rose items. This should be
  changed to ensure that all aspects of the suite are validated uniformly.

### Rationalise Suite Start Up Commands

This is raised in [#1030](https://github.com/cylc/cylc/issues/1030). Consider
a single command (e.g. `cylc flow`?) to:
* Restart a suite.
* Restart a suite at a given checkpoint.
* Warm start a suite.
* Cold start a suite.
* Clean start a suite.
