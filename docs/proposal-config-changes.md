# Proposal - changes to global and workflow configs

These notes mainly address [cylc-flow #3422](https://github.com/cylc/cylc-flow/issues/3422).
They have been updated to reflect the discussion at
[CylcCon 2020](https://cylc.github.io/cylc-admin/feb2020-workshop-notes#wednesday).

## General points

### Terminology
   * We should remove all references to `suite` in the settings (partly because
     we're trying stop referring to "suites" and partly because some affected
     settings are actually referring to the workflow scheduler which is unclear).
   * We will refer to the process which controls the workflow as the "scheduler"
    (previously referred to as either "daemon", "server" or "service").
   * **NOTE**: The old terminology is still used through-out the code, tests and
     documentation - these will need to be updated (not urgent).

### Deprecation
   * The global config will be changing drastically for the platforms support.
     Furthermore, it doesn't affect most end users.
     * Decision: Don't support any deprecated or obsolete settings in the global
       config.
   * For the workflow config the code to support deprecated and obsolete settings
     isn't complicated so there is no big overhead for us. On the other hand we
     don't want users using old syntax for too long. Some settings only became
     deprecated or obsolete very recently - it seems harsh to remove support for
     these.
     * Decision: Remove everything from 7.6.0 (Feb 2018) and before. That will
       just leaves things from 7.8.x.

### Config file naming

See [cylc-flow #3170](https://github.com/cylc/cylc-flow/issues/3170).
   * Suite: Currently `suite.rc` (specified in
     [suite_files.py](https://github.com/cylc/cylc-flow/blob/master/cylc/flow/suite_files.py#L123)).
     Since we're trying to move away from referring to "suites" we should change
     this (with backwards compatibility).
     * Decision: `flow.cylc`
   * Global
     * Currently `flow.rc` (specified in
       [globalcfg.py](https://github.com/cylc/cylc-flow/blob/master/cylc/flow/cfgspec/globalcfg.py#L325)).
       * Decision: change to `global.cylc`
     * Both the site and user config may consist of a number of files. For
       example, for release 8.0.1 the following config files will be read in (if
       they exist) in order from least specific to most specific:
       * `<config-dir>/flow/global.cylc`
       * `<config-dir>/flow/8/global.cylc`
       * `<config-dir>/flow/8.0/global.cylc`
       * `<config-dir>/flow/8.0.1/global.cylc`
     * The user config dir will `~/.cylc`.
     * The site config dir will defined by the environment variable
       `$CYLC_SITE_CONF_PATH` (defaults to `/etc/cylc` if not defined). See
       [cylc-flow #3167](https://github.com/cylc/cylc-flow/issues/3167).
     * If `$CYLC_CONF_PATH` is set then the global config is read from
       `$CYLC_CONF_PATH/global.cylc` (ignoring the site and user configs) - this
       is existing functionality which is used for testing.

*******************************************************************************

## Agreed global config changes

These are agreed changes to the
[specification currently on master](https://github.com/cylc/cylc-flow/blob/master/cylc/flow/cfgspec/globalcfg.py#L37).

### `[authentication]` -> obsolete

See [cylc-admin#77](https://github.com/cylc/cylc-admin/pull/77).

### `[cylc][events]mail *` -> `[scheduler][mail]`

`mail from`, `mail smtp`, `mail to` and `mail footer` can apply to both scheduler
and task events so move them to a separate section and remove `mail` from the
name. See
[discussion](https://github.com/cylc/cylc-flow/pull/3348#discussion_r324580369).

### `[cylc][events]` -> `[scheduler][events]`

See
[discussion](https://github.com/cylc/cylc-flow/pull/3348#discussion_r333229597).

### `[cylc]task event mail interval` -> `[scheduler][mail]task event batch interval`

See
[discussion](https://github.com/cylc/cylc-flow/pull/3348#discussion_r332937797).

### `[cylc]health check interval` -> `[scheduler]health check interval`

Makes sense to move this.

### `process pool *` -> `[scheduler]process pool *`

Makes sense to move these.

### `[documentation]` -> obsolete

See
[discussion](https://github.com/cylc/cylc-flow/pull/3348#discussion_r324581298).

### `[document viewers]` -> obsolete

See
[discussion](https://github.com/cylc/cylc-flow/pull/3348#discussion_r325406770).

### `[suite logging]` -> `[scheduler][logging]`

Makes sense to move these.

### `run directory rolling archive length` -> `[scheduler]run directory rolling archive length`

Makes sense to move this. May well change as part of the rose suite-run
migration work.

### `[task events]mail smtp` -> obsolete

No reason to set this separately for task events.

### `[task events]mail retry delays` -> obsolete

Can't think of a good reason to want to delay the sending of an email
notification or to want to retry it (unlikely to succeed on retry if it fails).

(If we do decide to keep this setting, surely it should apply to scheduler events
as well and should be moved into the `[scheduler][mail]` section.)

### `[suite servers]` -> `[scheduler]`

See
[discussion](https://github.com/cylc/cylc-flow/pull/3348#discussion_r332985001).
Difficult to plan for the future methods (e.g. running via kubernetes?).
* `[suite servers]auto restart delay` -> `[scheduler]auto restart delay`
* `[suite servers]run hosts` -> `[scheduler][run hosts]available`
* `[suite servers]condemned hosts` -> `[scheduler][run hosts]condemned`
* `[suite servers]run ports` -> `[scheduler][run hosts]ports`
* `[suite servers]ranking` -> `[scheduler][run hosts]ranking`

### `[suite host self-identification]` -> `[scheduler][host self-identification]`

See
[discussion](https://github.com/cylc/cylc-flow/pull/3348#discussion_r333243416).

### `[job platforms]` -> `[platforms]`

Agreed at CylcCon.

### `[job platforms][]task communication method` -> `[platforms][]communication method`

See
[discussion](https://github.com/cylc/cylc-flow/pull/3348#discussion_r332814930).

### `[cylc]UTC mode` -> `[scheduler]UTC mode`

Matches workflow config change.

### `[editors]`

Discussed at CylcCon. Agreed to keep these settings but discourage their use.
Also, change them to use to `$EDITOR` & `$GEDITOR` if set (otherwise use current
defaults).

### `disable interactive command prompts` -> obsolete

Agreed at CylcCon to remove support for interactive prompts (and remove the
`--force` option in the affected commands). If there is a requirement for this
in the future it would be better to support it via a `--interactive` option.

### `[test battery]` -> obsolete

Now in `global-tests.cylc` (but need to check this).

### `[job platforms][]batch system` -> `[platforms][]job runner`

This name makes more sense when using `background` and, hopefully, will fit
better with possible future methods (e.g. `kubernetes`?).

### `[job platforms][]batch submit command template` -> `[platforms][]job runner command template`

Matches previous change. However, we should open an issue to consider removing
this (inconsistent as no poll and kill templates) but leave for now as it is
used in the test battery.

### `[job platforms][]task event handler retry delays` -> obsolete

Discussed at CylcCon - no requirement for this.

### `[job platforms][]scp command` -> obsolete

No longer used. Also remove the `cylc [util] scp-transfer` command.

### Updated global config

Below is shown the global config as it will be after all the agreed changes have
been made:
```python
SPEC = {
    'scheduler': {
        'UTC mode': [VDR.V_BOOLEAN],
        'health check interval': [VDR.V_INTERVAL, DurationFloat(600)],
        'process pool size': [VDR.V_INTEGER, 4],
        'process pool timeout': [VDR.V_INTERVAL, DurationFloat(600)],
        'run directory rolling archive length': [VDR.V_INTEGER, -1],
        'events': {
            'handlers': [VDR.V_STRING_LIST],
            'handler events': [VDR.V_STRING_LIST],
            'mail events': [VDR.V_STRING_LIST],
            'startup handler': [VDR.V_STRING_LIST],
            'timeout handler': [VDR.V_STRING_LIST],
            'inactivity handler': [VDR.V_STRING_LIST],
            'shutdown handler': [VDR.V_STRING_LIST],
            'aborted handler': [VDR.V_STRING_LIST],
            'stalled handler': [VDR.V_STRING_LIST],
            'timeout': [VDR.V_INTERVAL],
            'inactivity': [VDR.V_INTERVAL],
            'abort on stalled': [VDR.V_BOOLEAN],
            'abort on timeout': [VDR.V_BOOLEAN],
            'abort on inactivity': [VDR.V_BOOLEAN],
        },
        'mail': {
            'from': [VDR.V_STRING],
            'smtp': [VDR.V_STRING],
            'to': [VDR.V_STRING],
            'footer': [VDR.V_STRING],
            'task event batch interval': [VDR.V_INTERVAL, DurationFloat(300)],
        },
        'logging': {
            'rolling archive length': [VDR.V_INTEGER, 5],
            'maximum size in bytes': [VDR.V_INTEGER, 1000000],
        },
        'auto restart delay': [VDR.V_INTERVAL],
        'host self-identification': {
            'method': [VDR.V_STRING, 'name', 'address', 'hardwired'],
            'target': [VDR.V_STRING, 'google.com'],
            'host': [VDR.V_STRING],
        },
        'run hosts': {
            'available': [VDR.V_SPACELESS_STRING_LIST],
            'condemned': [VDR.V_ABSOLUTE_HOST_LIST],
            'ports': [VDR.V_INTEGER_LIST, list(range(43001, 43101))],
            'ranking': [VDR.V_STRING],
        },
    },
    'editors': {
        'terminal': [VDR.V_STRING, 'vim'],
        'gui': [VDR.V_STRING, 'gvim -f'],
    },
    'monitor': {
        'sort order': [VDR.V_STRING, 'definition', 'alphanumeric'],
    },
    'platforms': {
        '__MANY__': {
            'job runner': [VDR.V_STRING, 'background'],
            'job runner command template': [VDR.V_STRING],
            'shell': [VDR.V_STRING, '/bin/bash'],
            'run directory': [VDR.V_STRING, '$HOME/cylc-run'],
            'work directory': [VDR.V_STRING, '$HOME/cylc-run'],
            'suite definition directory': [VDR.V_STRING],
            'communication method': [
                VDR.V_STRING, 'zmq', 'poll'],
            'submission polling intervals': [VDR.V_INTERVAL_LIST],
            'submission retry delays': [VDR.V_INTERVAL_LIST, None],
            'execution polling intervals': [VDR.V_INTERVAL_LIST],
            'execution time limit polling intervals': [VDR.V_INTERVAL_LIST],
            'ssh command': [
                VDR.V_STRING, 'ssh -oBatchMode=yes -oConnectTimeout=10'],
            'use login shell': [VDR.V_BOOLEAN, True],
            'remote hosts': [VDR.V_STRING_LIST],
            'cylc executable': [VDR.V_STRING, 'cylc'],
            'global init-script': [VDR.V_STRING],
            'copyable environment variables': [VDR.V_STRING_LIST, ''],
            'retrieve job logs': [VDR.V_BOOLEAN],
            'retrieve job logs command': [VDR.V_STRING, 'rsync -a'],
            'retrieve job logs max size': [VDR.V_STRING],
            'retrieve job logs retry delays': [VDR.V_INTERVAL_LIST],
            'tail command template': [
                VDR.V_STRING, 'tail -n +1 -F %(filename)s'],
            'err tailer': [VDR.V_STRING],
            'out tailer': [VDR.V_STRING],
            'err viewer': [VDR.V_STRING],
            'out viewer': [VDR.V_STRING],
            'job name length maximum': [VDR.V_INTEGER],
            'owner': [VDR.V_STRING],
        },
    },
    'platform groups': {
        '__MANY__': {
            'platforms': [VDR.V_STRING_LIST]
        }
    },
    'task events': {
        'execution timeout': [VDR.V_INTERVAL],
        'handlers': [VDR.V_STRING_LIST],
        'handler events': [VDR.V_STRING_LIST],
        'handler retry delays': [VDR.V_INTERVAL_LIST, None],
        'mail events': [VDR.V_STRING_LIST],
        'mail from': [VDR.V_STRING],
        'mail to': [VDR.V_STRING],
        'submission timeout': [VDR.V_INTERVAL],
    },
}
```

*******************************************************************************

## Agreed workflow config changes

These are agreed changes to the
[specification currently on master](https://github.com/cylc/cylc-flow/blob/master/cylc/flow/cfgspec/suite.py#L42).

### `[cylc]force run mode` & `required run mode` -> obsolete

Not required. See
[discussion-1](https://github.com/cylc/cylc-flow/pull/3348#discussion_r331819355)
and
[discussion-2](https://github.com/cylc/cylc-flow/pull/3348#discussion_r332862973)

### `[visualization]` -> obsolete

May return in a different form later. See
[discussion](https://github.com/cylc/cylc-flow/pull/3348#discussion_r332427759).

### `[cylc][authentication]` -> obsolete

See [cylc-admin#77](https://github.com/cylc/cylc-admin/pull/77).

### `[cylc]disable automatic shutdown` -> obsolete

Not required. See
[discussion](https://github.com/cylc/cylc-flow/pull/3348#discussion_r332865172).

### `[cylc][parameters]` -> `[task parameters]`

See
[discussion](https://github.com/cylc/cylc-flow/pull/3348#discussion_r332805028).

### `[cylc][parameter templates]` -> `[task parameters][templates]`

Note this is different to what was proposed in the discussion (above).

### `[scheduling]hold after point` -> `[scheduling]hold after cycle point`

See
[discussion](https://github.com/cylc/cylc-flow/pull/3348#discussion_r332811653).

### `[scheduling]stop after cycle point` <- new

Discussed at CylcCon.

### `[scheduling][special tasks]exclude at start-up` & `include at start-up` -> obsolete

Not required. See
[discussion](https://github.com/cylc/cylc-flow/pull/3348#discussion_r333238144).

### `[runtime][][job]` -> `[runtime][]`

This applies only to the settings which have not been deprecated by the
platforms work: `execution polling intervals`, `execution retry delays`,
`execution time limit`, `submission polling intervals` &
`submission retry delays`. See
[discussion](https://github.com/cylc/cylc-flow/pull/3348#discussion_r333242245)
and [cylc-flow#3480](https://github.com/cylc/cylc-flow/pull/3480).

### `[runtime][][remote]suite definition directory` -> obsolete

We will assume the suite definition is in the cylc-run directory. See
[discussion-1](https://github.com/cylc/cylc-flow/pull/3348#discussion_r324481883)
and
[discussion-2](https://github.com/cylc/cylc-flow/pull/3348#discussion_r333232472).

Note that we also need to deprecate the
[environment variables](https://cylc.github.io/doc/built-sphinx/suite-config.html?highlight=cylc_suite_def_path#task-job-script-variables)
`$CYLC_SUITE_DEF_PATH` & `$CYLC_SUITE_DEF_PATH_ON_SUITE_HOST`.

### `[cylc][events]mail *` -> `[scheduler][mail]`

Matches global config change.

Also make `smtp` obsolete? No reason to set this on a per workflow basis?

### `[cylc][events]` -> `[scheduler][events]`

Matches global config change.

### `[cylc]task event mail interval` -> `[scheduler][mail]task event interval`

Matches global config change.

### `[runtime][][events]mail smtp` -> obsolete

Matches global config change to `[task events]mail smtp`.

### `[runtime][][events]mail retry delays` -> obsolete

Matches global config change to `[task events]mail retry delays`.

### `[cylc][reference test]expected task failures` -> `[scheduler][events]expected task failures`

See
[discussion](https://github.com/cylc/cylc-flow/issues/3404#issuecomment-540190164).

### `[scheduler][events]abort if any task fails`  -> obsolete

Make command line option instead (discussed at CylcCon).

### `[cylc][environment]` -> obsolete

Discussed at CylcCon.

### `[cylc]health check interval` -> `[scheduler]health check interval`

Matches global config change.

### `[cylc]` time settings -> `[scheduler]`

This covers `UTC mode`, `cycle point format`,
`cycle point num expanded year digits` and `cycle point time zone` (discussed at
CylcCon).

### `[cylc][simulation]disable suite event handlers` -> obsolete

Agreed at CylcCon to move this and make it positive. However, now proposing to
remove it and automatically disable event handlers in simulation and dummy
modes. It's easy to test event handlers, if you need to, in a real test suite
written for that purpose and there's no obvious use case for this when running
a real suite in simulation or dummy mode.

### `[runtime][]extra log files` -> obsolete

Instead provide access to all files found in the job log dir in the new UI. See
[discussion](https://github.com/cylc/cylc-flow/pull/3348#discussion_r334219315).

### Empty string defaults

Where default values are currently defined as empty strings, these will be
replaced with `None`. See
[discussion](https://github.com/cylc/cylc-flow/pull/3348#discussion_r324183612).

Note: there are no empty string defaults in the global config.

### Updated workflow config

Below is shown the workflow config as it will be after all the agreed changes
have been made (not including deprecated settings):
```python
SPEC = {
    'meta': {
        'description': [VDR.V_STRING, ''],
        'group': [VDR.V_STRING, ''],
        'title': [VDR.V_STRING, ''],
        'URL': [VDR.V_STRING, ''],
        '__MANY__': [VDR.V_STRING, ''],
    },
    'cylc': {
        'simulation': {
            'disable suite event handlers': [VDR.V_BOOLEAN, True],
        },
    },
    'scheduler': {
        'UTC mode': [VDR.V_BOOLEAN, False],
        'cycle point format': [VDR.V_CYCLE_POINT_FORMAT],
        'cycle point num expanded year digits': [VDR.V_INTEGER, 0],
        'cycle point time zone': [VDR.V_CYCLE_POINT_TIME_ZONE],
        'health check interval': [VDR.V_INTERVAL],
        'events': {
            'handlers': [VDR.V_STRING_LIST, None],
            'handler events': [VDR.V_STRING_LIST, None],
            'mail events': [VDR.V_STRING_LIST, None],
            'startup handler': [VDR.V_STRING_LIST, None],
            'timeout handler': [VDR.V_STRING_LIST, None],
            'inactivity handler': [VDR.V_STRING_LIST, None],
            'shutdown handler': [VDR.V_STRING_LIST, None],
            'aborted handler': [VDR.V_STRING_LIST, None],
            'stalled handler': [VDR.V_STRING_LIST, None],
            'timeout': [VDR.V_INTERVAL],
            'inactivity': [VDR.V_INTERVAL],
            'abort if startup handler fails': [VDR.V_BOOLEAN],
            'abort if shutdown handler fails': [VDR.V_BOOLEAN],
            'abort if timeout handler fails': [VDR.V_BOOLEAN],
            'abort if inactivity handler fails': [VDR.V_BOOLEAN],
            'abort if stalled handler fails': [VDR.V_BOOLEAN],
            'abort on stalled': [VDR.V_BOOLEAN],
            'abort on timeout': [VDR.V_BOOLEAN],
            'abort on inactivity': [VDR.V_BOOLEAN],
            'expected task failures': [VDR.V_STRING_LIST],
        },
        'mail': {
            'from': [VDR.V_STRING],
            'smtp': [VDR.V_STRING],
            'to': [VDR.V_STRING],
            'footer': [VDR.V_STRING],
            'task event interval': [VDR.V_INTERVAL],
        },
    },
    'task parameters': {
        '__MANY__': [VDR.V_PARAMETER_LIST],
        'templates': {
            '__MANY__': [VDR.V_STRING],
        },
    },
    'scheduling': {
        'initial cycle point': [VDR.V_CYCLE_POINT],
        'final cycle point': [VDR.V_STRING],
        'initial cycle point constraints': [VDR.V_STRING_LIST],
        'final cycle point constraints': [VDR.V_STRING_LIST],
        'hold after cycle point': [VDR.V_CYCLE_POINT],
        'stop after cycle point': [VDR.V_CYCLE_POINT],
        'cycling mode': (
            [VDR.V_STRING, Calendar.MODE_GREGORIAN] +
            list(Calendar.MODES) + ["integer"]),
        'runahead limit': [VDR.V_STRING],
        'max active cycle points': [VDR.V_INTEGER, 3],
        'spawn to max active cycle points': [VDR.V_BOOLEAN],
        'queues': {
            'default': {
                'limit': [VDR.V_INTEGER, 0],
                'members': [VDR.V_STRING_LIST],
            },
            '__MANY__': {
                'limit': [VDR.V_INTEGER, 0],
                'members': [VDR.V_STRING_LIST],
            },
        },
        'special tasks': {
            'clock-trigger': [VDR.V_STRING_LIST],
            'external-trigger': [VDR.V_STRING_LIST],
            'clock-expire': [VDR.V_STRING_LIST],
            'sequential': [VDR.V_STRING_LIST],
        },
        'xtriggers': {
            '__MANY__': [VDR.V_XTRIGGER],
        },
        'graph': {
            '__MANY__': [VDR.V_STRING],
        },
    },
    'runtime': {
        '__MANY__': {
            'platform': [VDR.V_STRING],
            'inherit': [VDR.V_STRING_LIST],
            'init-script': [VDR.V_STRING],
            'env-script': [VDR.V_STRING],
            'err-script': [VDR.V_STRING],
            'exit-script': [VDR.V_STRING],
            'pre-script': [VDR.V_STRING],
            'script': [VDR.V_STRING],
            'post-script': [VDR.V_STRING],
            'work sub-directory': [VDR.V_STRING],
            'execution polling intervals': [VDR.V_INTERVAL_LIST, None],
            'execution retry delays': [VDR.V_INTERVAL_LIST, None],
            'execution time limit': [VDR.V_INTERVAL],
            'submission polling intervals': [VDR.V_INTERVAL_LIST, None],
            'submission retry delays': [VDR.V_INTERVAL_LIST, None],
            'meta': {
                'title': [VDR.V_STRING, ''],
                'description': [VDR.V_STRING, ''],
                'URL': [VDR.V_STRING, ''],
                '__MANY__': [VDR.V_STRING, ''],
            },
            'simulation': {
                'default run length': [VDR.V_INTERVAL, DurationFloat(10)],
                'speedup factor': [VDR.V_FLOAT],
                'time limit buffer': [VDR.V_INTERVAL, DurationFloat(30)],
                'fail cycle points': [VDR.V_STRING_LIST],
                'fail try 1 only': [VDR.V_BOOLEAN, True],
                'disable task event handlers': [VDR.V_BOOLEAN, True],
            },
            'environment filter': {
                'include': [VDR.V_STRING_LIST],
                'exclude': [VDR.V_STRING_LIST],
            },
            'events': {
                'execution timeout': [VDR.V_INTERVAL],
                'handlers': [VDR.V_STRING_LIST, None],
                'handler events': [VDR.V_STRING_LIST, None],
                'handler retry delays': [VDR.V_INTERVAL_LIST, None],
                'mail events': [VDR.V_STRING_LIST, None],
                'mail from': [VDR.V_STRING],
                'mail to': [VDR.V_STRING],
                'submission timeout': [VDR.V_INTERVAL],
                'expired handler': [VDR.V_STRING_LIST, None],
                'late offset': [VDR.V_INTERVAL, None],
                'late handler': [VDR.V_STRING_LIST, None],
                'submitted handler': [VDR.V_STRING_LIST, None],
                'started handler': [VDR.V_STRING_LIST, None],
                'succeeded handler': [VDR.V_STRING_LIST, None],
                'failed handler': [VDR.V_STRING_LIST, None],
                'submission failed handler': [VDR.V_STRING_LIST, None],
                'warning handler': [VDR.V_STRING_LIST, None],
                'critical handler': [VDR.V_STRING_LIST, None],
                'retry handler': [VDR.V_STRING_LIST, None],
                'submission retry handler': [VDR.V_STRING_LIST, None],
                'execution timeout handler': [VDR.V_STRING_LIST, None],
                'submission timeout handler': [VDR.V_STRING_LIST, None],
                'custom handler': [VDR.V_STRING_LIST, None],
            },
            'suite state polling': {
                'user': [VDR.V_STRING],
                'host': [VDR.V_STRING],
                'interval': [VDR.V_INTERVAL],
                'max-polls': [VDR.V_INTEGER],
                'message': [VDR.V_STRING],
                'run-dir': [VDR.V_STRING],
                'verbose mode': [VDR.V_BOOLEAN],
            },
            'environment': {
                '__MANY__': [VDR.V_STRING],
            },
            'directives': {
                '__MANY__': [VDR.V_STRING],
            },
            'outputs': {
                '__MANY__': [VDR.V_STRING],
            },
            'parameter environment templates': {
                '__MANY__': [VDR.V_STRING],
            },
        },
    },
}
```

*******************************************************************************

## Potential global config changes

Possible changes not currently included in this proposal.

### Separate client config file

See
[discussion](https://github.com/cylc/cylc-flow/pull/3348#discussion_r324181769).

### `[cylc]UTC mode`

Does it make sense to be able to set `UTC mode` globally but not
`cycle point time zone`? Is the difference between these settings clear?
Which makes more sense to set globally?
See [cylc-flow#2324](https://github.com/cylc/cylc-flow/issues/2324).

### `[job platforms][]work directory`

Poor name (contains work & share). It is likely this setting will be replaced
(or re-defined) as part of the work to support rose suite-run functionality for
sym-linking to different file-systems (also `run directory`).

### `[task events]` -> `[runtime][root][events]`

Do we want global and workflow config names to match where applicable?

*******************************************************************************

## Potential workflow config changes

Possible changes not currently included in this proposal.

### `[scheduling]spawn to max active cycle points`

Presumably this will become obsolete if/when we implement spawn on demand?

### `[runtime]`

Rename? See
[discussion](https://github.com/cylc/cylc-flow/pull/3348#discussion_r333241186).

### `[runtime][]environment`

Rename? See
[discussion](https://github.com/cylc/cylc-flow/pull/3348#discussion_r332985938).

### `[runtime][]outputs`

Rename? See
[discussion](https://github.com/cylc/cylc-flow/pull/3348#discussion_r333242497).

### `[runtime][]parameter environment templates`

Rename? See
[discussion](https://github.com/cylc/cylc-flow/pull/3348#discussion_r334310430).

### `[runtime][]directives`

Rename? See
[discussion](https://github.com/cylc/cylc-flow/pull/3348#discussion_r332986258).

### `[runtime][]events`

Rename? See
[discussion](https://github.com/cylc/cylc-flow/pull/3348#discussion_r332812965).
