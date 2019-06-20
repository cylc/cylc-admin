# Getting Started with Cylc 8

1. Create a clean conda install:
  ```bash
  conda env create -n my-example-env python=3.7
  conda activate my-example-env
  ```

2. Ensure that you install node-js using conda first*. If you allow pip to
  install it as a dependency of cylc & cylc-uiserver it you will end up with
  a very old version.
  ```bash
  conda install nodejs
  node --version
  ```
  For comparison devs have used node at 10.13.x and 12.x.x.

3. Install development copies of cylc and cylc-uiserver:
  ```bash
  pip install -e $DEVDIR/cylc
  pip install -e $DEVDIR/cylc-uiserver
  ```
4. Install npm in cylc-ui client.
  ```bash
  cd $DEVDIR/cylc-ui
  npm install
  npm install -g configurable-http-proxy
  ```

5. in `$DEVDIR/cylc-uiserver/jupyterhub_config.py` change
```diff
-c.Spawner.args = ['-s', '../cylc-ui/dist/']
+c.Spawner.args = ['-s', '/path/to/my/home/metomi/cylc-ui/dist/']
```

6. Set `jupyterhub` running.

7. Open a new terminal and ensure that you have the correct conda environment.
  ```bash
  conda activate my-example-env
  cd $DEVDIR/cylc-ui
  ```

8. Run the UI
  ```bash
  npm run build:watch
  ```

* Open a browser and navigate to `localhost:8000`. You will be asked to log in
  with your desktop credentials if you have not done so before.
  





*It is important that jupyterhub is not installed on conda after nodejs, or it
will fatally downgrade nodejs.
