<script type="module">
	import mermaid from 'https://cdn.jsdelivr.net/npm/mermaid@11/dist/mermaid.esm.min.mjs';
	mermaid.initialize({
		startOnLoad: true,
	});
</script>

# Getting Started with Cylc Development

## Development Setup

#. **Say Hi On The [Cylc Developers Chat](https://matrix.to/#/#cylc-general:matrix.org)!**


#. **Install System Dependencies**

   > [!NOTE]
   > We recommend developing in a Conda environment, although it
   > is possible, though less flexible, to install the required system
   > dependencies and work in Python virtual environments.
   >
   > Use whatever Conda client you prefer, `micromamba` is the most performant.

   This command will give you a Conda environment containing all the things you
   might need for Cylc/Rose development:

   ```bash
   conda create -y -n cylc-dev \
      python \
      pip \
      configurable-http-proxy \
      graphviz \
      nodejs \
      yarn \
      shellcheck \
      pygraphviz \
      pygobject \
      gtk3
   ```


#. **Configure For Distributed Development**

   Follow these steps if you're running on a network with a shared filesystem.

   [Install the Cylc wrapper script](https://cylc.github.io/cylc-doc/stable/html/installation.html#managing-environments).

   > [!NOTE]
   > Your site may have already done this for you.

   Then configure the path to your Conda environment directory in your shell
   profile file (e.g, `.bash_profile`), e.g:

   ```bash
   # ~/.bash_profile
   export CYLC_HOME_ROOT_ALT="/path/to/conda/environment/directory"
   ```

   To locate your environment directory, run:

   ```bash
   conda activate cylc-dev
   dirname $CONDA_PREFIX
   ```

   Then get Conda to set the `CYLC_ENV_NAME` environment variable automatically
   when you active the environment (this tells the wrapper script which Conda
   environment to use for remote command invocations):

   ```console
   $ conda activate cylc-dev
   $ echo "export CYLC_ENV_NAME=$CONDA_DEFAULT_ENV" > $CONDA_PREFIX/etc/conda/activate.d/cylc.sh
   $ echo 'unset CYLC_ENV_NAME' > $CONDA_PREFIX/etc/conda/deactivate.d/cylc.sh
   ```


#. **Fork & Clone**

   On GitHub, create a fork of any of the Cylc repositories you want to work
   on and create a local clone of them
   (i.e, `git clone git@github.com/<yourname>/<repo>.git`).

   Please star [cylc-flow](https://github.com/cylc/cylc-flow) and
   [metomi-rose](https://github.com/metomi/rose/) :)


#. **Install Python Projects**

   Install any Python projects you have just cloned into your Conda
   environment.

   Install [cylc/cylc-flow](https://github.com/cylc/cylc-flow/) and optionally:

   * [cylc/cylc-uiserver](https://github.com/cylc/cylc-uiserver/) (for web server work)
   * [metomi/rose](https://github.com/metomi/rose/) (for Rose work)
   * [cylc/cylc-rose](https://github.com/cylc/cylc-rose/) (for Cylc / Rose integration)
   * [cylc/cylc-doc](https://github.com/cylc/cylc-doc/) (for Cylc docs changes)

   Install the `[all]` optional dependency to pick up test, developer and
   documentation extras, i.e:

   ```bash
   pip install -e "path/to/repo[all]"
   ```

   When doing this for the first time it's better to install them all in one
   go, e.g:

   ```bash
   pip install -e ./cylc-flow[all] \
               -e ./cylc-uiserver[all] \
               -e ./rose[all] \
               -e ./cylc-rose[all] \
               -e ./cylc-doc[all]
   ```

   > [!NOTE]
   > There are dependencies between projects, so they may need to be installed
   > in order, see [project dependencies](#project-dependencies)

   > [!NOTE]
   > Installing in "editable" mode (the `-e` above) means you only need to
   > repeat this `pip install` command when certain project files are modified:
   > * `setup.py`
   > * `setup.cfg`
   > * `pyproject.toml`
   > * `MANIFEST.in`

   > [!NOTE]
   > You can use `uv pip` as a stand-in for `pip`.


#. **Install and configure [cylc/cylc-ui](https://github.com/cylc/cylc-ui/) (optional)**

   ```bash
   cd path/to/cylc-ui
   conda activate cylc-dev
   yarn install
   yarn run build  # create your first production build
   ```

   The Cylc UI Server comes with a bundled version of Cylc UI. To use your
   personal build add the following to your Jupyter configuration:

   ```python
   # ~/.cylc/hub/jupyter_config.py
   # NOTE: some of us use an environment variable to toggle this on/off
   c.CylcUIServer.ui_build_dir = '~/cylc-ui/dist'  # path to build
   ```

   * See the
     [cylc-uiserver README](https://github.com/cylc/cylc-uiserver?tab=readme-ov-file#developing)
     for more information on `ui_build_dir`.
   * See the
     [cylc-ui README](https://github.com/cylc/cylc-ui?tab=readme-ov-file#development)
     for more information on building/developing the UI.

   Then launch the UI:

   ```bash
   # standalone (single-user mode, good for development)
   cylc gui

   # via Jupyter Hub (multi-user mode)
   cylc hub
   # You will be asked to log in with your desktop credentials if you have not
   # done so before.
   ```


#. **Contributor License Agreement**

   Read the `CONTRIBUTING.md` file and add your name to it with your first
   commit.

   You will have to do this for each Cylc / Rose repository. Note
   repositories may use different licenses.


## Running Your First Workflow

```bash
conda acitvate cylc-dev

# clone the workflow into ~/cylc-src
cylc get-resources tutorial/cylc-forecasting-workflow

# install the workflow into ~/cylc-run and start it
cylc vip cylc-forecasting-workflow

# watch it run
cylc tui  # in-terminal
# OR
cylc gui  # in-browser

# stop it
cylc stop cylc-forecasting-workflow

# uninstall it from ~/cylc-run
cylc clean cylc-forecasting-workflow
```


## Versions & Branches

Cylc projects use semantic versioning, e.g: for the version 8.1.2:

* 8 = **major** release (implies breaking changes)
* 1 = **minor** release (no breaking changes permitted unless forewarning provided)
* 2 = **bugfix** release (bugfixes and UI/UX issues only, no features or refactors)

Cylc projects are all pinned to the minor version of cylc-flow, e.g:

* cylc-uiserver 1.6.x goes with cylc-flow 8.4.x
* cylc-uiserver 1.7.x goes with cylc-flow 8.5.x
* cylc-rose 1.5.x goes with cylc-flow 8.4.x
* etc

This tight coupling prevents unintended combinations of Cylc components being
installed into the same environment whilst allowing them to be installed in
a modular fashion.

### Bugfixes

When we make a minor release, we create a branch with a `.x` suffix for
bugfixes.

E.g. `8.1.x` was for bugfixes to `8.1.0`. We merge these `.x` branches back
into master.

Raise any bugfixes against the `.x` branch, the "sync" PR will be automatically
created after it is merged.

### Supported Versions

The currently supported (i.e. actively developed) versions are documented
[here](https://cylc.github.io/cylc-admin/status/status.html).

### Project Dependencies

There are dependencies between our projects, here's a summary:

<pre class="mermaid">
---
config:
    look: handDrawn
---
flowchart RL
    isodatetime -> rose
    isodatetime -> cylc-flow
    rose -> cylc-rose
    cylc-flow -> cylc-rose
    cylc-flow -> cylc-uiserver
    cylc-ui ->|bundled| cylc-uiserver
</pre>
