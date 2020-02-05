# Proposal (Feb 2020)

## Cylc 8 System Component Naming

Cylc is now a complex multi-component system. It is important that we settle on
consistent, unambiguous, and where possible standard, component names so that
users and developers can communicate without confusion. (This was not easy even
before Cylc 8!).

## Proposed Final Names

Note we have already agreed to drop **suite** for **workflow** ("suite" is
non-standard outside of the NWP community).

| Component Name               | Code Repository   | Config File                                          | Description |
| --                           |----               | ---                                                  |---          |
|:boom: **Cylc Scheduler**     | cylc-flow         | :boom: `cylc-flow.conf` and `cylc-flow-global.conf`  | an instance of the cylc-flow scheduler program, to manage a single workflow | 
|**Cylc Hub**                  | (Jupyter Hub)     | :boom: `<cylc-location>/jupyterhub_config.py`               | the single point of access that users log in to |
|**Cylc UI Server**            | cylc-uiserver     | N/A? (or :boom: `cylc-uiserver.conf`)                | an instance of the cylc-uiserver program, that serves the Cylc Web UI and CLI |
|:boom: **Cylc Web UI**        | :boom: cylc-webui | N/A?                                                 | an instance of the Cylc Web UI, served to your browser by the Cylc UI Server |


## Explanatory Comments

Cylc Scheduler
- "workflow server" is ambiguous - it can refer to the host server that the workflows run on
- "workflow server program" is clearer, but too much of a mouthful.
- "workflow daemon" is not strictly correct (`--no-detach`) and is very
  Unix-specific (should we ever port to another OS)
- "workflow service" - a cylc-flow instance really stretches the definition of a "service"
- "cylc-flow instance" (as in, an instance of the workflow manager program) was
  considered but not terribly popular, and compared to "scheduler" it is another very generic term.
- "scheduler" is correct, specific, and obvious. Plus, the program is actually
  implemented via an instance of the `Scheduler` class in `cylc/flow/scheduler.py`.

Cylc config files: `cylc-flow.conf`, `cylc-flow-global.conf`
- (we're dropping the term "suite" - see above)
- the `.rc` suffix is archaic, rarely used today 
- the old `global.rc` is too generic
- :question: we don't need to distinguish between user and site global configs - 
  unless there are different config items allowed in each? (In which case
  we should drop the `global` and use `cylc-flow-user.conf` and `cylc-flow-site.conf`?)
- :question: `flow.conf` is an option
  - (but `cylc-flow.conf` is clearer without being too verbose)

Cylc Hub
- A vanilla (so far) instance of JupyterHub, BUT
  - Configured to show Cylc branding and launch the Cylc UI Server
- To avoid confusion with other JupyterHub on the system we need to call it
  "Cylc Hub", but we should be clear that it is just an instance of JupyterHub
- :question: do we need to launch it with a special command: `cylc hub`? (which
  automatically picks up the Cylc-specific `jupyterhub_config.py`?)

Cylc UI Server
- The word "Server" is not so confusing here (as it could be for the scheduler
  program) because the UI Server exists only to literally "serve the UI"

Cylc Web UI
- The "Cylc UI" is not just the web interface, it's also the CLI
- No one uses the term "GUI" for web interfaces
