# Cylc-Rose - Installation of Rose Configs

## Rose2019 & Cylc7

The `rose suite-run` command loads in the `rose-suite.conf` file and performs
certain actions based on its configuration.

There are various ways to meddle with the configuration as it is written. Here
are the different methods for defining the Rose configuration in the order
in which they are applied:

1. The "base" `rose-suite.conf` configuration.
2. Optional configurations defined by:
   * The `rose-suite.conf|opts` configuration.
   * The `ROSE_SUITE_OPT_CONF_KEYS` environment variable.
   * The `-O, --opt-conf-key` CLI option.
   Note these are all additive so can be used in combination
3. CLI "defines"
   * Suite template defines via the `-S, --define-suite` CLI option.
   * Any Rose configuration via the `-D, --define` CLI option.

> **Note:** The order in which opt conf keys are applied appears to be
> undocumented but the answer lies in the code
> [here](https://github.com/metomi/rose/blob/99ad9e766571212c6c3afdbe3915c75ba05e6c68/metomi/rose/run.py#L149-L155)
> and
> [here](https://github.com/metomi/rose/blob/99ad9e766571212c6c3afdbe3915c75ba05e6c68/metomi/rose/config.py#L1280-L1286).


`rose suite-run` copies source files into the run directory *excluding* the
`rose-suite.conf` file.

The parsed Rose config is dumped in the run directory into
`log/rose-suite.conf` for logging reasons, however, it is not read by Rose
from this location for any further purpose.

Any configuration set by either (2) or (3) above must be provided
in any subsequent `rose suite-run` command or it will be lost. This includes
`rose suite-run --reload` and `rose suite-run --restart` making these
commands especially dangerous.

## Rose2 & Cylc8

The `rose suite-run` command is better thought of as `rose suite-install --run`.
By default it runs the suite but can also `--reload` or `--restart`
on request.

In the new system `rose suite-run` has been replaced by `cylc install`
which will call out to the new `cylc-rose` plugin for Rose specific
functionality.

This new command will be responsible for the installation of the Rose
configuration into the run directory as well as supporting the command line
options and environment variables that can change the Rose configuration
(as outlined above).

```
usage: cylc install [opts]

Plugins:
  cylc-rose:
    Opts:
      -S, --define-suite, (--define-template-var)
        Define a Cylc Jinja2/EmPy template variable.
      -D, --define
        Define a Rose configuration.
      -O, --opt-conf-key
        Define a Rose optional configuration key.
    Env:
      ROSE_SUITE_OPT_CONF_KEYS
        Define a set of Rose optional configuration keys to append to the list.
```

## Proposed Changes

1. The `rose-suite.conf` file should be installed into the run directory.
   * For consistency and clarity.
   * To enable the configuration to be inspected with standard tooling
     (`rose config`).
2. Configuration alterations (via CLI or environment) should be written
   to the filesystem so that subsequent Cylc commands
   (e.g. `play`, `get-config`, `validate`)
   have access to the required information (i.e. template variables).
2. `cylc reinstall` should remember previously defined configuration
   alterations.
   * This means we can reload and restart flows safely without having
     to remember all previously specified options.

## Proposed Implementation

Meddling with the Rose configuration during re-installation is likely to be
a rare occurrence and what follows is somewhat overblown for purpose, however,
should reduce to far less code than text in this proposal.

### 1. Any new "defines" should be written to a new optional configuration with a special protected name

Using an optional configuration makes good sense as it would work natively with
`cylc ui` and `rose config` with no extra work required whilst making this
functionality transparent to users in a format they already understand.

Configuration would be written to an optional configuration called
`cylc-install`. The `cylc install` command would exclude this file from the
`rsync` in order to protect it.

Example:

```
# ~/roses/FLOW/rose-suite.conf
opts=foo
```

```
$ cylc install -D '[env]FOO=1' -S 'X=Y'
```

```
# ~/cylc-run/FLOW/rose-suite.conf

opts=foo (cylc-install)

# NOTE: (opt) means an "optional" opt conf
# (i.e. if the cylc-install conf isn't there it doesn't cause an error)
# this means we can apply this opt conf by default, even if no overrides
# are provided on the CLI
```

```
# ~/cylc-run/FLOW/opts/rose-suite-cylc-install.conf
[env]
FOO=1

[template vars]
X=Y
```

### 2. The `cylc reinstall` command should always move the `cylc-install` opt conf to the end of the list

Example:

```
# ~/roses/FLOW/rose-suite.conf
opts=foo
```

```
$ cylc install
$ cylc reinstall --opt-conf-key bar
```

```
# ~/cylc-run/FLOW/rose-suite.conf
opts=foo bar (cylc-install)
```

> **Note:** Be aware that the `opts` config may have to be altered by
> string manipulation as the Rose code will
> [fight your changes](https://github.com/metomi/rose/blob/99ad9e766571212c6c3afdbe3915c75ba05e6c68/metomi/rose/config.py#L1280).

### 3. The `cylc reinstall` command should have an option to delete the `cylc-install` opt conf

Example:

```
# ~/roses/FLOW/rose-suite.conf
opts=foo
```

```
$ cylc install -O bar -D '[env]FOO=1'
$ cylc reinstall --clear-rose-install-options -O baz -D '[env]BAR=2'
```

```
# ~/cylc-run/FLOW/opt/rose-suite-cylc-install.conf
!opts=baz

[env]
BAR=2
```

### 4. `cylc install` should record optional configs to the `cylc-install` optional config

Any optional configurations specified via the command line option or
environment variable should be recorded in the optional configuration
using the `opts` setting.

This setting should be ignored (`!`) as it is not "functional", that is to
say, editing this value will make no difference to `cylc validate`,
`cylc play`, etc.

However, `cylc reinstall` will read this value (ignored vars can still be
read) and append it to `rose-suite.conf|opts` so as to preserve the memory
of previously specified opt confs.

`cylc reinstall` may also append any newly specified opt confs to the end of
this list.

Example:

```
# ~/roses/flow/rose-suite.conf
opts=a
```

```
$ ROSE_SUITE_OPT_CONF_KEYS=b cylc install -O c
```

```
# ~/cylc-run/FLOW/rose-suite.conf
opts=a (cylc install)
```

```
# ~/cylc-run/FLOW/opt/rose-suite-cylc-install.conf
!opts=b c
```

```
$ cylc reinstall -O d
```

```
# ~/cylc-run/FLOW/opt/rose-suite-cylc-install.conf
!opts=b c d
```

```
# ~/roses/FLOW/rose-suite.conf
# with each reinstall the cylc-install|opts are appended to the end
# of the rose-suite.conf|opts setting
opts=a (cylc-install)
```

Because the `cylc-install|opts` are appended to the `rose-suite.conf|opts`
setting with each reinstallation the base `rose-suite.conf|opts` setting
is permitted to change:

```
# ~/roses/flow/rose-suite.conf
opts=z
```

```
$ cylc reinstall
```

```
# ~/roses/FLOW/rose-suite.conf
# NOTE: opt a has been "removed"
# NOTE: opts b c d have been "remembered"
opts=z (cylc-install)
```

## Proposed Interfaces

```
usage: cylc install [opts]

Plugins:
  cylc-rose:
    Opts:
      -S, --define-suite, (--define-template-var)
        Define a Cylc Jinja2/EmPy template variable.
      -D, --define
        Define a Rose configuration.
      -O, --opt-conf-key
        Add a Rose optional configuration key.
    Env:
      ROSE_SUITE_OPT_CONF_KEYS
        Define a set of Rose optional configuration keys to append to the list.
```

```
usage: cylc reinstall ID [opts]

Plugins:
  cylc-rose:
    Opts:
      -S, --define-suite, (--define-template-var)
        Define a Cylc Jinja2/EmPy template variable.
      -D, --define
        Define a Rose configuration.
      -O, --opt-conf-key
        Add a Rose optional configuration key.
      --clear-rose-install-options
        Delete the cylc-install optional configuration which preserves
        Rose options previously specified to cylc (re)install.
    Env:
      ROSE_SUITE_OPT_CONF_KEYS
        Define a set of Rose optional configuration keys to append to the list.
```
