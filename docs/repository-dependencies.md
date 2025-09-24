### Cylc Repository Dependency Graph

```mermaid
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

```

Current meta-release versions for this dependency tree can be viewed at the [project status page](https://github.com/cylc/cylc-admin/blob/master/docs/status/status.md).
