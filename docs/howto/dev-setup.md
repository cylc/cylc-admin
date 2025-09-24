<script type="module">
	import mermaid from 'https://cdn.jsdelivr.net/npm/mermaid@11/dist/mermaid.esm.min.mjs';
	mermaid.initialize({
		startOnLoad: true,
	});
</script>

# Getting Started with Cylc Development

## Development Setup

1. Say Hi On The [Cylc Developers Chat](https://matrix.to/#/#cylc-general:matrix.org).

2. Install System Dependencies:

   > **Note**: We recommend developing in a Conda environment, although it
     is possible, though less flexible, to install the required system
     dependencies and work in Python virtual environments.

     Use whatever Conda client you prefer, `micromamba` is the most performant.

   This command will give you a Conda environment containing all the things you
   might need for Cylc development:

   ```bash
   conda create -n cylc-dev python pip configurable-http-proxy graphviz nodejs yarn shellcheck
   ```

   To ensure your Conda environment is automatically activated across a
   distributed network, create a pair of activate/deactivate files:

   ```console
   $ conda activate cylc-dev
   $ echo "export CYLC_ENV_NAME=$CONDA_DEFAULT_ENV" > $CONDA_PREFIX/etc/conda/activate.d/cylc.sh
   $ echo 'unset CYLC_ENV_NAME' > $CONDA_PREFIX/etc/conda/deactivate.d/cylc.sh
   ```

3. Fork & Clone

   On GitHub, create a fork of any of the Cylc repositories you want to work
   on and create a local clone of them
   (i.e, `git clone git@github.com/<yourname>/<repo>.git`).

   Please star https://github.com/cylc/cylc-flow and
   https://github.com/metomi/rose/ :)


4. Install Python Projects

   Install any Python projects you have just cloned into your Conda
   environment.
   > Note: `cylc-ui` is installed separately below via a different process.

   Install [cylc/cylc-flow](https://github.com/cylc/cylc-flow/) and optionally:

   * [cylc/cylc-uiserver](https://github.com/cylc/cylc-uiserver/) (for UIS work)
   * [metomi/rose](https://github.com/metomi/rose/) (for Rose work)
   * [cylc/cylc-rose](https://github.com/cylc/cylc-rose/) (for Rose work)
   * [cylc/cylc-doc](https://github.com/cylc/cylc-doc/) (for docs changes)
   > Note: This requires all above repo sorties to be installed via:

   ```bash
   pip install -e "path/to/repo[all]"
   ```
   Dependencies are best installed in this order to avoid conflicts:
   * cylc-rose
   * cylc-uiserver
   * cylc-flow
   * rose
   * cylc-doc
  
   > Note: For a fuller description see [project dependencies](#project-dependencies)

   You only need to repeat this `pip install` command when certain project
   files are modified:
   * `setup.py`
   * `setup.cfg`
   * `pyproject.toml`
   * `MANIFEST.in`
   > Note: You can use `uv pip` as a stand-in for `pip`.

5. Install and configure [cylc/cylc-ui](https://github.com/cylc/cylc-ui/) (optional)

   ```bash
   cd path/to/cylc-ui
   conda activate cylc-dev
   yarn install
   yarn run build  # create your first build
   ```

   The Cylc UI Server comes with a bundled version of Cylc UI. To use your
   personal build add the following to your Jupyter configuration:

   ```python
   # ~/.cylc/hub/jupyter_config.py
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

6. Contributor License Agreement

   Read the `CONTRIBUTING.md` file and add your name to it with your first
   commit.

   You will have to do this for each Cylc / Rose repository. Note
   repositories may use different licenses.


## Running Your First Workflow

```bash
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

<pre class="mermaid">
---
config:
    look: handDrawn
---
flowchart RL
    rose -->|requires| isodatetime["metomi/isodatetime"]
    CR[cylc-rose] -->|requires| CF[cylc-flow]
    CR -->|requires| rose["metomi/rose"]
    uis[cylc uiserver] -->|requires| CF
    uis .->|bundles| ui["cylc ui (JavaScript)"]

</pre>
