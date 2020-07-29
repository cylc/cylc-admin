Running Cylc Tests
================================

## How to run tests

All tests are in the folder `cylc-flow/tests`. Instructions here, unless
stated otherwise assume that you are working from your cylc-flow repo
dir.

| Test Type | Command |
|---|---|
| Unit Tests | `pytest` |
| Integration tests | `pytest tests/integration` |
| Functional Tests | `etc/bin/run-functional-tests` |


## flow-test.rc

Many tests rely on an alternative user config file called
`flow-tests.rc` usually found at
`~/.cylc/flow/<cylc-version>/flow-tests.rc`. `flow-tests.rc` is used
instead of `flow.rc` for tests.

Many tests (especially those which rely on non-local computers which
vary from developer to developer) use the `skip_all` function from
`test_header` and will return a message saying all tests have been
skipped if the correct remote settings have not been included in
`flow-tests.rc`.

You may find the following idiom useful in a `flow-test.rc` file:

<!-- note added raw tags to prevent Jinja2 being interpreted by Jekyll: -->

```
{% raw %}
#!jinja2
# You only have to change 1 line:
{% set MYTESTBRANCH = False %}

{% if MYTESTBRANCH == True %}
   [settings]
      [[for]]
         your = test branch
{% elif %}
   [settings]
      [[for]]
         master = branch
{% endif %}
{% endraw %}
```


A brief guide to debugging Rose & Cylc (functional) tests
=========================================================

## Running a single test

You can run a single test using the following command:

```bash
cd ~/my/cylc/dev/copy

# run all the tests - this may take quite a long time
etc/bin/run-functional-tests.sh

# run a folder of tests
etc/bin/run-functional-tests.sh tests/functional/validate/

# run a single test
etc/bin/run-functional-tests.sh tests/functional/validate/00-multi.t

# run a single test in verbose mode
etc/bin/run-functional-tests.sh tests/functional/validate/00-multi.t -v

# maximum verbosity for supported tests
CYLC_TEST_DEBUG=true etc/bin/run-functional-tests.sh tests/validate/00-multi.t -v
```

> **note**
>
> adding `-v` to the test command is recommended if you are dealing with
> a failing test.

## Recording tests that fail

When running many tests you can record the tests that fail using
`--state=save`. You can then re-run the only the failed tests using
`--state=failed`.

For more details see `man prove`.

## Understanding the tests

If a test is failing the first step is to look at its test file. All
test files (should) contain:

1.  A description such as
    `# Test validating simple multi-inheritance suites.`
2.  They will add the functions in the test\_header file to the shell:
    `. "$(dirname "$0")/test_header"`.

    > **hint**
    >
    > The test header file is a symlink in each test directory to
    > `cylc.git/tests/lib/bash/test_header`. This file has a useful
    > docstring. If you see an unfamiliar command in a test script it
    > may well be defined `test_header`.

3.  The test\_header function `set_test_number 1` which allows the test
    script to tell how many tests to expect.
4.  One or more testing functions from `test_header`. Common examples
    include `run_ok`, `run_fail`, `cmp_ok`, `grep_ok` and `reftest`.

Many of these functions take a `TEST_NAME` argument which sets the name.
Often this is set by `TEST_NAME=${TEST_NAME_BASE}-some-extra-label`
where `${TEST_NAME_BASE}` is the name of the test file.

## Working out the cause of failures

The tests use the bash shell so you can use bash commands to see what
they are doing, with the following caveat: To see the output you need to
redirect it to standard error using `>&2`. If, for example, you wanted
to understand the directory structure of the temporary directory that
many tests create, you might add `tree >&2` to a test before re-running
it.

Other useful commands include `ls >&2`, `cat ${TEST_NAME}.stdout >&2`
and `cat ${TEST_NAME}.stderr >&2`. These last will show you the standard
error and output of the last test, which may give you more information
about what went wrong.

In many cases you may get a test in the form
`run_ok "${TEST_NAME}" cylc validate "${SUITE_NAME}"`. In these cases it
may be instructive to add a `--debug` switch and examine the stderr
thus:

```bash
run_ok ${TEST_NAME} cylc validate --debug "${SUITE_NAME}"
cat ${TEST_NAME}.stderr >&2
cat ${TEST_NAME}.stdout >&2
```

Many tests also create and run a suite: You can find this in your
cylc-run folder under the name
\`cylctb-\<date-time-string\>: Investigating this suite may often point you to the
cause of failure.

Using Python Debugger
=====================
You can't use a Python debugger straightforwardly inside the test framework -
it will hang as it wait for interaction with the debugger that never comes.
What you can do is place the following in the test before the cylc or rose
command you wish to debug:

```bash
   DUMPDIR=\$(mktemp -d)
   echo "Your test setup stuff is at \${DUMPDIR}" \>&2
   cp -r \* \${DUMPDIR}
   exit 0
```

This will dump all the test setup in a temporary directory for you to play with.
If you want to debug a python program you can add
Python debug statements
\<<https://docs.python.org/3/library/pdb.html>\>\_ to
the code and run it:

These are:

```python

   breakpoint()
   \# at Python ≥ 3.7

   \# or
   import pdb; pdb.set\_trace()
   \# at Python \< 3.7
```

Alternatively you may find that you can run the suite from your test.
Functional tests of form path/to/test.t\` often come with a suite in the
form `path/to/test/suite.rc`. You will need to remember to manually
provide any environment variables the workflow needs, and may need to
alter your `flow.rc` to match `flow-tests.rc`.

Traps for the unwary
====================

#### `grep_ok` vs `comp_ok`

Tests that use comp\_ok generally compare `${TEST_NAME}.stdout` or
`${TEST_NAME}.stderr` against either a reference or against `/dev/null`.
They expect the entire output to be **exactly** the same as the
reference and are therefore very unforgiving.

`grep_ok` is much less sensitive only requiring the reference output to
be present **somewhere** in the test output.

Further Reading
===============

## Heredocs

If you see code that looks like this:

```bash
cat >'hello.py' <<'__HELLO_PY__'
print("Hello World")
__HELLO_PY__
```

You are looking at an "heredoc" and you may wish to read about heredocs:
[A modern looking bloggy guide](https://linuxize.com/post/bash-heredoc/)
[A web 1.0 manual](http://tldp.org/LDP/abs/html/here-docs.html)
