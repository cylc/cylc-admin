# Getting Started with Cylc 8

## Development Installation

> **Note**: We recommend developing in a Conda environment as this provides a
  more standard setup though if you ensure the non-python dependencies
  are installed correctly you can bypass this.

1. Create a clean conda install:

   ```bash
   conda create -n my-example-env python=3.7 configurable-http-proxy --yes
   conda activate my-example-env
   ```

2. Ensure that you install `nodejs` using conda first.

   If you allow pip to
   install it as a dependency of cylc & cylc-uiserver it you will end up with
   a very old version.

   ```bash
   conda install nodejs --yes
   node --version
   npm install yarn -g
   ```

   For comparison devs have used node at 10.13.x and 12.x.x.

3. Install
   [cylc/cylc-flow](https://github.com/cylc/cylc-flow/) and
   [cylc/cylc-uiserver](https://github.com/cylc/cylc-uiserver/):

   ```bash
   pip install -e "${DEVDIR}/cylc-flow"[all]
   pip install -e "${DEVDIR}/cylc-uiserver"[all]
   ```

   Where `${DEVDIR}/cylc-flow` and `${DEVDIR}/cylc-uiserver` are the locations
   of your local (git) worktrees of the two projects.

4. Install [cylc/cylc-ui](https://github.com/cylc/cylc-ui/).

   > **Note:** We prefer `yarn`.

   ```bash
   cd $DEVDIR/cylc-ui
   yarn install
   ```

   Where `${DEVDIR}/cylc-ui` is the location
   of your local (git) worktree of the project.

5. Point Cylc Hub at your UI build

   You may need to hardcode the path to your UI build in the Cylc Hub config:

   ```python
   #  ~/.cylc/hub/config.py
   c.Spawner.cmd = ['cylc', 'uiserver', '-s', '<path/to/dist>']
   ```

6. Run `jupyterhub`.

   ```bash
   conda activate my-example-env
   cylc hub
   ```

7. Run UI.
   ```bash
   conda activate my-example-env
   yarn run build:watch
   ```

8. Open a browser and navigate to `http://localhost:8000/`.

   You will be asked to log in with your desktop credentials if you have not
   done so before.

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
   conda activate my-example-env
   cylc run foo
   ```

3. Watch it run on the CLI:

   ```bash
   cylc tui foo
   ```

4. Watch it run in the UI:

   Fire up jupyterhub as before, open a browser and navigate to
   `http://localhost:8000/`.

5. Stop it:

   (otherwise it will just keep running forever)

   ```bash
   cylc stop foo
   ```
