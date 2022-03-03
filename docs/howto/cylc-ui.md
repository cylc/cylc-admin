# Getting Started with Cylc 8

Say hi on the
[Cylc developers chat](https://matrix.to/#/#cylc-general:matrix.org).


## Development Installation

> **Note**: We recommend developing in a Conda environment as this provides a
  more standard setup though if you ensure the non-python dependencies
  are installed correctly you can bypass this.

1. Install Conda Deps:

   The cylc-flow Conda dependencies are in the cylc-flow repository
   https://github.com/cylc/cylc-flow/blob/master/conda-environment.yml

   (note optional dependencies are commented out)

   Create a new Conda environment containing the cylc-flow deps
   (and `configurable-http-proxy` if working with JupyterHub):

   ```bash
   conda create -n cylc-<version> python=3.7 configurable-http-proxy -f conda-environment.yml
   ```

   Where `<version>` is the cylc-flow version e.g. `8.0.1`.

2. Fork & Clone the repos you want to develop:

   * [cylc/cylc-flow](https://github.com/cylc/cylc-flow/)

   [optional] For Isodatetime development:

   * [metomi/metomi-isodatetime](https://github.com/metomi/metomi-isodatetime/)

   [optional] For ui development:

   * [cylc/cylc-ui](https://github.com/cylc/cylc-uiserver/)
   * [cylc/cylc-uiserver](https://github.com/cylc/cylc-uiserver/)

   [optional] For Rose development:

   * [metomi/metomi-rose](https://github.com/metomi/metomi-rose/)
   * [cylc/cylc-rose](https://github.com/cylc/cylc-rose/)

3. Install Python Projects

   Pip-install any Python projects you want to work with in editable mode
   in the following order:

   ```bash
   pip install -e "metomi-isodatetime[all]"
   pip install -e "cylc-flow[all]"
   pip install -e "cylc-uiserver[all]"
   pip install -e "metomi-rose[all]"
   pip install -e "cylc-rose[all]"
   ```

   > **Note:** If you install the projects in the wrong order you might
   > end up installing released versions from PyPi, use `pip list` to check.

4. [optional] Configure Cylc Flow for distributed development

   > Required to work with cylc-flow on multiple hosts in a network.

   You need to ensure that Cylc is installed and accessible on all of the hosts
   you want to work with.

   You will need to synchronise your environments to any hosts that have an
   independent `$HOME` filesystem.

   If you are working on a site which installs Cylc for you ensure the Cylc
   [wrapper script
   ](https://github.com/cylc/cylc-flow/blob/master/cylc/flow/etc/cylc)
   is in your "$PATH". Otherwise install it yourself (read comments in file).

   Export the `CYLC_HOME_ROOT_ALT` varaible to point at your Conda "envs"
   directory in your `~/.bash_profile` file e.g:

   ```bash
   # add the wrapper script to $PATH
   export PATH="/path/to/wrapper/script:$PATH"
   # set the path to your development environments
   export CYLC_HOME_ROOT_ALT=/home/me/mambaforge/envs/
   ```

   The same wrapper script can be used for Rose & Isodatetime

5. [optional] Install [cylc/cylc-ui](https://github.com/cylc/cylc-ui/).

   > Required for cylc-ui development

   > **Note:** We prefer `yarn`.

   > **Note:** The UI must currently be built with Node<=16 (2022-03-03)

   Install Node dependencies:

   ```bash
   cd path/to/cylc-ui
   yarn install
   ```

   Build the UI with either:

   ```bash
   yarn run build
   yarn run build:watch
   ```

   To use your development build point the UIS at the build directory:

   ```python
   # ~/.cylc/uiserver/jupyter_config.py
   # use a development UI build rather than the build bundled with the UIS
   c.CylcUIServer.ui_build_dir = '~/cylc-ui/dist'  # path to build
   ```

   The version number should appear in the bottom-right of the UI along with the
   word development.

   (see the cylc-uiserver README for more information)


## Running Your First Workflow

1. Create a basic Cylc workflow:

   ```bash
   mkdir -p ~/cylc-run/foo
   cat > ~/cylc-run/foo/flow.cylc <<__HERE__
   [scheduling]
       # a cycling workflow where the cycles are numbered as integers
       cycling mode = integer
       initial cycle point = 1
       [[graph]]
           # tasks which run in each cycle until you stop the workflow
           P1 = """
               # a => b means "b should wait until a has run"
               # [-P1] means "from the previous cycle"
               b[-P1] => a => b => c
           """
   __HERE__
   ```

2. Activate your environment:

   ```bash
   conda activate cylc-<version>
   ```

3. Run the workflow:

   ```bash
   cylc play foo
   ```

4. Watch it run on the CLI:

   ```bash
   cylc tui foo
   ```

5. Watch it run from the GUI:

   ```bash
   cylc gui
   ```

6. Stop it:

   (otherwise it will just keep running forever)

   ```bash
   cylc stop foo
   ```


## Working On Python Projects

You will need your Conda environment activated in order to run the tests.

```bash
conda activate cylc-<version>
```

The Cylc wrapper script will automatically activate your environment on other
hosts around the network.

The `CYLC_VERSION` and `CYLC_ENV_NAME` environment variables make this happen:

```
CYLC_VERSION=8.0.0 cylc version --long
8.0.0 ...
```

We use `flake8` for linting and `pytest` for testing, check the README files in
each repo you work with.

You will need to re-pip install after any changes to packaging
(i.e. setup.cfg files), good idea to do this out of habit.
