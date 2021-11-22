# Getting Started with Cylc 8

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
   conda create -n cylc-8-dev python=3.7 configurable-http-proxy -f conda-environment.yml
   ```

2. Install Python Projects

   Install [cylc/cylc-flow](https://github.com/cylc/cylc-flow/) and optionally:

   * [cylc/cylc-uiserver](https://github.com/cylc/cylc-uiserver/) (for UIS work)
   * [metomi/metomi-rose](https://github.com/metomi/metomi-rose/) (for Rose work)
   * [cylc/cylc-rose](https://github.com/cylc/cylc-rose/) (for Rose work)

   ```bash
   # clone the git repository locally then...
   pip install -e "path/to/repo[all]"
   ```

3. Install [cylc/cylc-ui](https://github.com/cylc/cylc-ui/).

   > **Note:** We prefer `yarn`.

   ```bash
   cd path/to/cylc-ui
   yarn install
   ```

4. Point Cylc Hub at your UI build

   The Cylc UI comes bundled with the UI Server.

   If you want to develop the UI you will need to point the UI Server at
   your local UI build:

   ```python
   # ~/.cylc/hub/jupyter_config.py
   c.CylcUIServer.ui_build_dir = '~/cylc-ui/dist'  # path to build
   ```

   (see the cylc-uiserver README for more information)

5. Build The UI

   ```bash
   yarn run build:watch
   ```

   (see the cylc-ui README for more information)

6. Launch the UI

   ```bash
   # standalone
   cylc gui

   # via Jupyter Hub
   cylc hub
   # You will be asked to log in with your desktop credentials if you have not
   # done so before.
   ```

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

2. Run the workflow:

   ```bash
   cylc play foo
   ```

3. Watch it run on the CLI:

   ```bash
   cylc tui foo
   ```

4. Stop it:

   (otherwise it will just keep running forever)

   ```bash
   cylc stop foo
   ```
