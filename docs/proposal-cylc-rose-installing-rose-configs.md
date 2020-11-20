# Cylc-Rose - Installing Rose Configs

## Rose2019 & Cylc7

The `rose suite-run` command loads in the `rose-suite.conf` file and performs
certain actions based on its configuration.

There are various ways to meddle with the configuration as it is written. Here
are the different methods for defining the Rose configuration along with their
order or precedence:

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

### Optional Conf Keys

#### 1. Any specified opt confs should be appended to `rose-suite.conf|opts`

Example:

```
# ~/roses/FLOW/rose-suite.conf
opts=bar
```

```
$ cylc install -o baz
```

```
# ~/cylc-run/FLOW/rose-suite.conf
opts=bar baz
```

> **Note:** Be aware that the `opts` config may have to be altered by
> string manipulation as the Rose code will
> [fight your changes](https://github.com/metomi/rose/blob/99ad9e766571212c6c3afdbe3915c75ba05e6c68/metomi/rose/config.py#L1280).

#### 2. The `cylc re-install` command should prepend the source `rose-suite.conf|opts` to the run configuration

This is the nastiest bit, necessary as the alternative would require
recording which opts were defined in which places at which times then. This
approach still wouldn't be perfect as there is no way to know the order the
user intended (if indeed they know themselves).

Since changing `rose-suite.conf|opts` should be rare this should be an
acceptable hack (because of the `--rebuild-opt-conf` option in [3]).

Example:

```
# ~/roses/FLOW/rose-suite.conf
opts=foo bar
```

```
$ cylc reinstall
```

```
# ~/cylc-run/FLOW/rose-suite.conf
opts=foo bar baz
```

Or to put it in other words, reinstallation can only add optional configs, or
reorder them, but not remove them without (3)...

#### 3. The `cylc re-install` command should support options for adding, removing and rebuilding optional configuration lists

> **Note:** If an opt conf key occurs more than once then the earlier
> occurrence(s) should be removed.

Example:

```
# ~/roses/FLOW/rose-suite.conf
opts=foo bar baz
```

```
$ cylc install
$ cylc reinstall --remove-opt-conf=foo --add-opt-conf=pub --add-opt-conf=bar
```

```
# ~/cylc-run/FLOW/rose-suite.conf

# new opt confs get appended onto the end of the list
# pre-existing opt confs (bar in this case) are moved to the end of the list
# see (4)
opts=baz pub bar
```

The rebuild option ignores the run directory and functions as `cylc install`
would if it were installing the flow for the first time:

```
$ cylc reinstall --rebuild-opt-conf --add-opt-conf=qux
```

```
# ~/cylc-run/FLOW/rose-suite.conf

# forget about baz,bar,pub and start from scratch
opts=foo qux
```

This would be the recommended way to handle complex opt conf key changes.

### Defines

#### 4. Any new "defines" should be written to a new optional configuration with a special protected name

Defines (`-S`, `-D`) cannot be treated in the same way as opt confs (`-O`)
because they must take Precedence over optional configurations.

Using an optional configuration makes good sense as it would work natively with
`cylc ui` and `rose config` with no extra work required whilst making this
functionality transparent to users in a format they already understand.

Configuration would be written to an optional configuration called
`cylc-install`. The `cylc install` command would exclude this file from the
`rsync` in order to protect it.

Example:

```
# ~/roses/FLOW/rose-suite.conf
opts=foo
```

```
$ cylc install -D '[env]FOO=1' -S 'X=Y'
```

```
# ~/cylc-run/FLOW/rose-suite.conf

opts=foo (cylc-install)

# NOTE: (opt) means an "optional" opt conf
# (i.e. if the cylc-install conf isn't there it doesn't cause an error)
# this means we can apply this opt conf by default, even if no overrides
# are provided on the CLI
```

```
# ~/cylc-run/FLOW/opts/rose-suite-cylc-install.conf
[env]
FOO=1

[template vars]
X=Y
```

#### 5. The `cylc reinstall` command should always move the `cylc-install` opt conf to the end of the list

Example:

```
# ~/roses/FLOW/rose-suite.conf
opts=foo
```

```
$ cylc install
$ cylc reinstall --add-opt-conf bar
```

```
# ~/cylc-run/FLOW/rose-suite.conf
opts=foo bar (cylc-install)
```

#### 6. The `cylc reinstall` command should have an option to delete the `cylc-install` opt conf

Example:

```
# ~/roses/FLOW/rose-suite.conf
opts=foo
```

```
$ cylc install -D '[env]FOO=1
$ cylc reinstall --rebuild-defines -D '[env]BAR=2'
```

```
# ~/cylc-run/FLOW/opt/rose-suite-cylc-install.conf
[env]
BAR=2
```


### Proposed Interfaces

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

```
usage: cylc reinstall ID [opts]

Plugins:
  cylc-rose:
    Opts:
      -S, --define-suite, (--define-template-var)
        Define a Cylc Jinja2/EmPy template variable.
      -D, --define
        Define a Rose configuration.
      --rebuild-defines
        Ignore all defines previously specified to `cylc install`.
        (i.e. delete the `cylc-install` optional configuration).
      -O, --opt-conf-key, --add-opt-conf-key
        Define a Rose optional configuration key.
      --remove-opt-conf-key
        Remove a previously defined opt conf key.
      --rebuild-opt-conf-keys
        Ignore all opt conf keys previously specified to `cylc install`.
    Env:
      ROSE_SUITE_OPT_CONF_KEYS
        Define a set of Rose optional configuration keys to append to the list.
```
