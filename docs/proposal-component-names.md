# Proposal (Feb 2020)

## Cylc 8 System Component Naming

Cylc is now a complex multi-component system. It is important that we settle on
consistent, unambiguous, and where possible standard, component names so that
users and developers can communicate without confusion. (This was not easy even
before Cylc 8!).

## Proposed Final Names

Firstly, we have already agreed to drop **suite** for **workflow** throughout
("suite" is non-standard outside of the NWP community).

Secondly, we should name the various system components as follows (changes marked with :boom:):

| Component Name               | Code Repository   | Config File                                          | Description |
| --                           |----               | ---                                                  |---          |
|:boom: **Cylc Flow Instance** | cylc-flow         | :boom: `cylc-flow.conf` and `cylc-flow-global.conf`  | an instance of the cylc-flow program, to manage a single workflow | 
|**Cylc Hub**                  | (Jupyter Hub)     | :boom: `cylc-hub.conf`                               | the single point of access that users log in to |
|**Cylc UI Server**            | cylc-uiserver     | N/A? (or :boom: `cylc-uiserver.conf`)                | an instance of the cylc-uiserver program, that serves the Cylc Web UI and CLI |
|:boom: **Cylc Web UI**        | :boom: cylc-webui | N/A?                                                 | an instance of the Cylc Web UI, served to your browser by the Cylc UI Server |


## Explanatory Comments

Cylc Flow Instance
- "workflow server" is ambiguous - it can refer to the host server that the workflows run on
- "workflow server program" is clearer, but too much of a mouthful.
- "workflow daemon" is not strictly correct (`--no-detach`) and is very
  Unix-specific (should we ever port to another OS)
- "workflow service" - a cylc-flow instance really stretches the definition of a "service"
- "cylc-flow instance" is strictly correct and unambiguous, reasonably
  intuitive (a running workflow instance), and its function in the context of the wider system
  can be described more verbosely if needed (see the Description in the table above)
- :question: "Flow Instance" is an option
  - (but "Cylc Flow Instance" is clearer without being too verbose)

Cylc config files: `cylc-flow.conf`, `cylc-flow-global.conf`
- (we're dropping the term "suite" - see above)
- the `.rc` post-fix is archaic, rarely used today 
- the old `global.rc` is too generic
- :question: `flow.conf` is an option
  - (but `cylc-flow.conf` is clearer without being too verbose)

Cylc Hub
- A vanilla (so far) instance of JupyterHub, BUT
  - Configured to show Cylc branding and launch the Cylc UI Server
- To avoid confusion with any standard JupyterHub on the system
  - we should call it "Cylc Hub", and:
  - it should be launched with a special command: `cylc hub`
  - it should have an obvious Cylc-specific config file: `cylc-hub.conf`

Cylc UI Server
- The word "Server" here potentially suffers from the "host server" ambiguity too, but:
  - it is a more appropriate term here in the sense that the UI Server exists entirely to "serve the UI"
  - (for cylc-flow we have to distinguish between the instance and the
    codebase, which includes the CLI etc. as well as the "server program")

Cylc Web UI
- The "Cylc UI" is not just the web interface, it's also the CLI
- No one uses the term "GUI" for web interfaces
