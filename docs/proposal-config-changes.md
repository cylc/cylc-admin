# Proposal - changes to global and workflow configs

These notes mainly address [cylc-flow #3422](https://github.com/cylc/cylc-flow/issues/3422).

## General points

### Terminology
   * We should remove all references to `suite` in the settings (partly because
     we're trying stop referring to "suites" and partly because some affected
     settings are actually referring to the workflow server which is unclear).
     * Note: "suite" is referred to through-out the code, tests and
       documentation.
   * **NOTE** "server" changed to "scheduler" below in light of the recent
     component name decision - *we still need to check each instance carefully*

### Deprecation
   * The global config will be changing drastically for the platforms support.
     Furthermore, it doesn't affect most end users.
     * Proposal: Don't support any deprecated or obsolete settings in the global
       config.
   * For the workflow config the code to support deprecated and obsolete settings
     isn't complicated so there is no big overhead for us. On the other hand we
     don't want users using old syntax for too long. Some settings only became
     deprecated or obsolete very recently - it seems harsh to remove support for
     these.
     * Proposal: Remove everything from 7.6.0 (Feb 2018) and before. That will
       just leaves things from 7.8.x.

### Config file naming

See [cylc-flow #3170](https://github.com/cylc/cylc-flow/issues/3170).
   * Suite: Currently `suite.rc` (specified in
     [suite_files.py](https://github.com/cylc/cylc-flow/blob/master/cylc/flow/suite_files.py#L123)).
     Since we're trying to move away from referring to "suites" we should change
     this (with backwards compatibility).
     * Proposal: `flow.rc` or `cylc-flow.rc`
   * Global
     * Currently `flow.rc` (specified in
       [globalcfg.py](https://github.com/cylc/cylc-flow/blob/master/cylc/flow/cfgspec/globalcfg.py#L325)).
       * Proposal: `config.rc`?
     * Read from `/etc/cylc/flow/8.0a1/flow.rc` and
       `~/.cylc/cylc/flow/8.0a1/flow.rc` unless `$CYLC_CONF_PATH` set in which
       case it is read only from this path.
     * We need the ability to override where the site config file is read from,
       e.g. by setting `$CYLC_SITE_CONF_PATH`?
     * Should we include the version in the path? Users should not have to
       create a new config file every time we deploy a new version. Perhaps
       major version only (with backwards compatibility assumed for minor
       versions)?

*******************************************************************************

## Proposed global config changes

These are proposed changes to the
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

### `[cylc]task event mail interval` -> `[scheduler][mail]task event interval`

See
[discussion](https://github.com/cylc/cylc-flow/pull/3348#discussion_r332937797).

### `[cylc][scheduler][environment]` <- new

We can define handlers in the global config but not configure the handler
environment - this doesn't make sense.

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

### `[suite servers]` -> `[scheduler platforms]`

See
[discussion](https://github.com/cylc/cylc-flow/pull/3348#discussion_r332985001).
Would `[scheduler hosts]` be better or `[scheduler][hosts]`?
Difficult to plan for the future methods (e.g. running via kubernetes?).

Note: alternative proposal later in this document.

### `[suite host self-identification]` -> `[scheduler platforms][host self-identification]`

See
[discussion](https://github.com/cylc/cylc-flow/pull/3348#discussion_r333243416).

### `[job platforms][]task communication method` -> `[job platforms][]communication method`

See
[discussion](https://github.com/cylc/cylc-flow/pull/3348#discussion_r332814930).

### Updated global config

Below is shown the global config as it will be if all the proposed changes are
agreed (not including deprecated settings):
```python
SPEC = {
    'disable interactive command prompts': [VDR.V_BOOLEAN, True],
    'cylc': {
        'UTC mode': [VDR.V_BOOLEAN],
    },
    'scheduler': {
        'health check interval': [VDR.V_INTERVAL, DurationFloat(600)],
        'process pool size': [VDR.V_INTEGER, 4],
        'process pool timeout': [VDR.V_INTERVAL, DurationFloat(600)],
        'run directory rolling archive length': [VDR.V_INTEGER, -1],
        'environment': {
            '__MANY__': [VDR.V_STRING],
        },
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
            'task event interval': [VDR.V_INTERVAL, DurationFloat(300)],
        },
        'logging': {
            'rolling archive length': [VDR.V_INTEGER, 5],
            'maximum size in bytes': [VDR.V_INTEGER, 1000000],
        },
    },
    'scheduler platforms': {
        'run hosts': [VDR.V_SPACELESS_STRING_LIST],
        'run ports': [VDR.V_INTEGER_LIST, list(range(43001, 43101))],
        'condemned hosts': [VDR.V_ABSOLUTE_HOST_LIST],
        'auto restart delay': [VDR.V_INTERVAL],
        'run host select': {
            'rank': [VDR.V_STRING, 'random', 'load:1', 'load:5', 'load:15',
                     'memory', 'disk-space'],
            'thresholds': [VDR.V_STRING],
        },
        'host self-identification': {
            'method': [VDR.V_STRING, 'name', 'address', 'hardwired'],
            'target': [VDR.V_STRING, 'google.com'],
            'host': [VDR.V_STRING],
        },
    },
    'editors': {
        'terminal': [VDR.V_STRING, 'vim'],
        'gui': [VDR.V_STRING, 'gvim -f'],
    },
    'monitor': {
        'sort order': [VDR.V_STRING, 'definition', 'alphanumeric'],
    },
    'job platforms': {
        '__MANY__': {
            'batch system': [VDR.V_STRING, 'background'],
            'batch submit command template': [VDR.V_STRING],
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
            'scp command': [
                VDR.V_STRING, 'scp -oBatchMode=yes -oConnectTimeout=10'],
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
            'task event handler retry delays': [VDR.V_INTERVAL_LIST],
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
    'test battery': {
        'remote host with shared fs': [VDR.V_STRING],
        'remote host': [VDR.V_STRING],
        'remote owner': [VDR.V_STRING],
        'batch systems': {
            '__MANY__': {
                'host': [VDR.V_STRING],
                'out viewer': [VDR.V_STRING],
                'err viewer': [VDR.V_STRING],
                'directives': {
                    '__MANY__': [VDR.V_STRING],
                },
            },
        },
    },
}
```

*******************************************************************************

## Proposed workflow config changes

These are proposed changes to the
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

Question: don't we also need `stop after cycle point`?

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

### `[scheduler][events]abort if any task fails` -> `[scheduler][events]abort if any unexpected task fails`

See change above (different name proposed).

### `[cylc][environment]` -> `[scheduler][environment]`

Makes sense to move this.

Rename to `handler environment`?
(think this was proposed but can't find discussion)

### `[cylc]health check interval` -> `[scheduler]health check interval`

Matches global config change.

### Updated workflow config

Below is shown the workflow config as it will be if all the proposed changes are
agreed (not including deprecated settings):
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
        'UTC mode': [VDR.V_BOOLEAN, False],
        'cycle point format': [VDR.V_CYCLE_POINT_FORMAT],
        'cycle point num expanded year digits': [VDR.V_INTEGER, 0],
        'cycle point time zone': [VDR.V_CYCLE_POINT_TIME_ZONE],
        'simulation': {
            'disable suite event handlers': [VDR.V_BOOLEAN, True],
        },
    },
    'scheduler': {
        'health check interval': [VDR.V_INTERVAL],
        'environment': {
            '__MANY__': [VDR.V_STRING],
        },
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
            'abort if any unexpected task fails': [VDR.V_BOOLEAN],
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
            'extra log files': [VDR.V_STRING_LIST],
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

### `[scheduler platforms]` alternative proposal

```python
SPEC = {
    'scheduler': {
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
            'selection': {
                'rank': [VDR.V_STRING, 'random', 'load:1', 'load:5', 'load:15',
                         'memory', 'disk-space'],
                'thresholds': [VDR.V_STRING],
            },
        },
    },
```

### Separate client config file

See
[discussion](https://github.com/cylc/cylc-flow/pull/3348#discussion_r324181769).

### `[cylc]`

Rename (global and workflow config)? See
[discussion](https://github.com/cylc/cylc-flow/pull/3348#issuecomment-541482565).
Many settings have moved so we should look at the remaining settings in this
section to see where they would fit best.

### `[editors]`

Do we still need these? Should they default to `$EDITOR` & `$GEDITOR` rather
than have fixed defaults?

### Use `None` as default value rather than empty string where appropriate

See
[discussion](https://github.com/cylc/cylc-flow/pull/3348#discussion_r324183612).
This would apply to the workflow config as well.

### `[cylc]UTC mode`

Does it make sense to be able to set `UTC mode` globally but not
`cycle point time zone`? Is the difference between these settings clear?
Which makes more sense to set globally?
See [cylc-flow#2324](https://github.com/cylc/cylc-flow/issues/2324).

### `disable interactive command prompts`

Does anyone use this? It seems dangerous to rely on it being set (or not).
Should we make this obsolete and replace the `--force` option in the affected
commands with a new `--interactive` option (which would be a useful thing to
add anyway).

### `[job platforms][]batch system`

This is not a good name when using `background` and may not fit well with
possible future method (e.g. `kubernetes`?). Change to `job runner` or
`run method`?

### `[job platforms][]batch submit command template`

Is this really needed? If so rename? Also need equivalents for poll & kill?

### `[job platforms][]work directory`

Poor name (contains work & share). It is likely this setting will be replaced
(or re-defined) as part of the work to support rose suite-run functionality for
sym-linking to different file-systems (also `run directory`).

### `[job platforms][]task event handler retry delays`

Do we really need a platform setting for this?

### `[job platforms][]scp command`

No longer used?

Note: setting not used by `cylc [util] scp-transfer` - do we need this command?

### `[platform groups]` -> `[job platform groups]`

For consistency?

### `[task events]` -> `[runtime][root][events]`

Do we want global and workflow config names to match where applicable?

### `[test battery]`

This was made obsolete in the (aborted) unified config branch. Not clear why?

*******************************************************************************

## Potential workflow config changes

Possible changes not currently included in this proposal.

### `[runtime][]extra log files`

Currently used with gcylc but will it be needed for the new UI? See
[discussion](https://github.com/cylc/cylc-flow/pull/3348#discussion_r334219315).

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
