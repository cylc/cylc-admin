# Getting Started with Cylc 8

1. Create a clean conda install:
  ```bash
  conda create -n my-example-env python=3.7 --yes
  conda activate my-example-env
  ```

2. Ensure that you install `nodejs` using conda first*. If you allow pip to
  install it as a dependency of cylc & cylc-uiserver it you will end up with
  a very old version.
  ```bash
  conda install nodejs --yes
  node --version
  ```
  For comparison devs have used node at 10.13.x and 12.x.x.

3. Install
   [cylc/cylc-flow](https://github.com/cylc/cylc-flow/) and
   [cylc/cylc-uiserver](https://github.com/cylc/cylc-uiserver/):
  ```bash
  pip install -e "${DEVDIR}/cylc-flow"[all]
  pip install -e "${DEVDIR}/cylc-uiserver"[all]
  ```
  where `${DEVDIR}/cylc-flow` and `${DEVDIR}/cylc-uiserver` are the locations
  of your local (git) worktrees of the two projects.

4. Install [cylc/cylc-ui](https://github.com/cylc/cylc-ui/).
  ```bash
  cd $DEVDIR/cylc-ui
  npm install --prefix="${DEVDIR}/cylc-ui"
  npm install -g configurable-http-proxy
  ```
  where `${DEVDIR}/cylc-ui` is the location
  of your local (git) worktree of the project.
  

5. Run `jupyterhub`.
  ```bash
  conda activate my-example-env
  jupyterhub -f ${DEVDIR}/cylc-uiserver/jupyterhub_config.py
  ```

6. Run UI.
  ```bash
  conda activate my-example-env
  npm run --prefix="${DEVDIR}/cylc-ui" build:watch
  ```

7. Open a browser and navigate to http://localhost:8000/.
   You will be asked to log in with your desktop credentials if you have not
   done so before.
  





*It is important that jupyterhub is not installed on conda after nodejs, or it
will fatally downgrade nodejs.
