# Cylc Clean

Work on the new `cylc clean` is already underway.

This command is able to locate workflow installations on different platforms
and delete them.

Ideally we would want `cylc clean` to be able to do a lot more.

One, simple example is re-running workflows, which requires moving/removing the
log dir and database. As per the
[rose suite-run migration proposal](https://cylc.github.io/cylc-admin/proposal-rose-suite-run.html#cli-changes)
this shall be done by cylc clean.

> you will be able to use cylc clean to clean out the log directory
> (and whatever other directories you want removing)

This means that we need an interface in `cylc clean` to perform this.

We could give users a pre-defined option, however, the exact requirements for
this niche usage are hard to predict.

This proposal suggests developing more generic functionality in `cylc clean`
for this and other purposes. It also highlights some viable possibilities for
future functionality which could see it largely surpass `rose_prune` and
`rose_arch`. Not of top importance now but worth considering for interface
design reasons.

related https://github.com/cylc/cylc-flow/issues/3887
closes https://github.com/cylc/cylc-flow/issues/1159

(For reference `rose_arch` is ~500 lines of Python and does some things we
might not want to do in Cylc).

## Examples

Control over which installations of a flow are cleaned:

```
# remove a workflow installation on the scheduler host
cylc clean myflow --local

# remove a workflow installation on any remote task hosts
cylc clean myflow --remote

# remove a workflow installation on the scheduler host and
# on any remote task hosts
cylc clean myflow
```

Control over what is cleaned:

```
# delete a directory in a workflow installation
cylc clean myflow --rm share/cycle/1234

# delete two directores in a workflow installation
cylc clean myflow --rm share --rm work
cylc clean myflow --rm share:work  # equivalent
```

Basic glob support:

```
# delete all job log files from 2020
cylc clean myflow --rm 'share/cycle/2020*'

# delete everything
cylc clean myflow --rm '*'
cylc clean myflow  # equivalent
```

Listing rather than removing things:

```
# list stuff in share/cycle for cycles in 2020
cylc clean --ls 'share/cycle/2020*'
```

Tools for moving and tar'ing things:

```
# rename a directory with a timestamp
cylc clean myflow --mv share/cycle/1234

# rename the log dir AND suite DB
cylc clean myflow --mv log

# tar a directory in a workflow installation
cylc clean myflow --tar share/cycle/1234
```

Group the `log` dir and `.service/db` together implicitly
(use case `cylc play --re-run`):

```
# remove the log dir AND suite DB
cylc clean myflow --rm log

# rename the log dir AND suite DB
cylc clean myflow --mv log

# tar the log dir AND suite DB
cylc clean myflow --tar log

# tar two directories together
cylc clean myflow --tar work:share

# --tar is shorthand for --compress <path> <format>
cylc clean --compress work:share archive.tar
```

Cycle aware housekeeping (`rose_prune` style):

```
# remove job logs in the range 2020-2021
cylc clean myflow --rm log/job/<cycle> --cycle-range=2021/-P1Y

# tar job logs for "foo" in the range 2020-2021
cylc clean myflow --tar log/job/<cycle>/foo --cycle-range=2021/-P1Y

# tar job logs for "foo" for the year before $CYLC_TASK_CYCLE_POINT inclusive
cylc clean myflow --tar log/job/<cycle>/foo --cycle-range=<cycle>/-P1Y

# delete the work dor for the task being run
cylc clean myflow --rm work/<cycle>/<task>
```

Archiving (`rose_arch` style):

```
# rsync the work directory to myhost
# NOTE: --rsync has two arguments
cylc clean myflow --rsync work myhost:archived-work

# tar the work directory and scp it to myhost
# NOTE: use MIME types to map compression to file extension
cylc clean myflow --scp work myhost:archived-work.tar

# tar two cycle directories together and scp them to myhost
cylc clean myflow --scp work/1234:work/2345 myhost:archived-work.tar
```

Extendable interface with MIME types:

```
# plugins can add new clean options
cylc clean myflow --[tool] [input:[input:...] [[location:]output[.format]]]

# e.g. scp
cylc clean myflow --scp work myhost:archived-work-dir

# use MIME types to detect output format e.g. tar
cylc clean myflow --scp work myhost:archived-work-dir.tar

# plugins can add support for new MIME types
cylc clean myflow --mytool input output.pax
```

Mix and match:

```
cylc clean \
    --local --mv log/job \
    -- \
    --remote --tar log/job \
    -- \
    --rm work/<cycle>:share/<cycle> --cycle-range=2021/-P1Y
```

Call from Python:

```python
from cylc.flow.clean import clean, cleanitem

clean(
    ['myflow'],
    cleanitem(mv='['log/job']),
    cleanitem(tar='['log/job']),
    cleanitem(rm='['work/<cycle>', 'share/<cycle>'], cycle_range='2021/-P1Y')
)
```

Build into a trivial main-loop plugin:

```
[scheduler]
    [[main loop]]
        # calls cylc-clean from the main loop via an async subprocess
        # so it doesn't cause any load on the scheduler process
        [[[clean]]]
            args = """
                # tar log files on the scheduler host that are P1Y behind the
                # oldest active cycle point
                --local
                --tar log/job/<cycle>
                --cycle-range=<cycles[0]>/-P1Y
                --
                # delete work dir and log files for cycles that are P1Y behind
                # the oldest active cycle point on all platforms
                --remote
                --rm work/<cycle>
                --cycle-range=<cycles[0]>/-P1Y
            """
            # run every 12 hours
            interval = P12H

        # can potentially allow multiple calls of the same plugin with
        # different options like so
        [[[clean(model-output)]]]
            # remove output files in a targeted manner
            args = """
                --rm work/<cycle>/model*/bigfile
                --cycle-range=<cycles[0]>/-P1D
            """

        # we could add a main-loop event which gets fired whenever a cycle
        # "closes" which would be good for housekeep purposes

        # can always use the same options configured here on the command line
```

```python
# to illistrate how trivial this plugin would be ...

from cylc.flow.clean import clean
from cylc.flow.scripts.clean import get_option_parser

@main_loop.startup
def startup(_, conf)
    args = re.sub('^\s*#.*$', '', conf['args'])
    args.replace('\n', ' ')
    conf['args'] = get_option_parser().parse_args(args)

@main_loop.periodic
def periodic(_, conf):
    clean(conf['args'])
```

## Suggested Timeframe

* 8.0b0
  * Remove entire workflow installations locally (done).
  * Remove entire workflow installations on remote platforms (in progress).
* 8.0b1
  * Add `--rm` and `--mv` options.
* 8.x
  * Add `--cycle-range` and `<cycle>` string substitution.
  * Add main-loop plugin (periodic only).

The rest according to future ideas and priorities.
