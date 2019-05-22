# Cylc VC 22 May 2019 Agenda

## Kickoff (5 min)

- repositories renamed
- now have Issue and Pull Request templates to help with general development
  and reduce reliance on Hilary at release time
- over-all progress - are we on track?
- is our speed and concurrency of development OK? (Do we need to slow down a
  bit to enable more involvement in more components from more people?)

## Review Previous Meeting Notes and Actions (5 min)

- [April agenda](vc-4-apr-2019-agenda.md)
- [April summary](vc-4-apr-2019-summary.md)

- [Left over actions from previous meetings](left-over-actions.md)

## Site Updates or News? (5 min)

- NIWA?
- Met Office?
 - (note summary email from Matt to Hilary)
- BOM?

## 7.8 maintenance (2 min)

- [7.8.2](https://github.com/cylc/cylc/releases/tag/7.8.2) released 3 weeks ago
- any new issues noted?
- any need for 7.8.3 any time soon?

## cylc-8 cylc-flow packaging (5 min)

- **pip packaging is DONE!**
- global config file location (in `/etc`?) still somewhat controversial?
- we lost a few things that may need to be restored
- (conda etc. TBD but low priority for now?)
- (other components?)

## cylc-8 Hub (10 min)

- status update from Martin
  - ssh spawner, BatchSpawner?
  - any remaining concerns about JupyterHub delivering what we need?
  - what exactly remains to be done here?
  - how can we speed up completion? (Do others need to get involved more?)

## cylc-8 WS, UIS, UI (20 min)

- lots of work by David and Bruno, some review by Oliver
- progress summary?
- any major concerns?
- TBD?: incremental update, multiple workflows etc.?

- [API-on-the-fly?](https://github.com/cylc/cylc/pull/3005#issuecomment-479512438) 
  - CLI? web UI?
  - (GraphQL in WS would impact this? ... but GraphQL in UIS, not WS)

- cylc-8 integration: how close are we to a first end-to-end system?:
  - cylc-ui <-> cylc-uiserver via GraphQL over WebSocket
  - cylc-uiserver <-> cylc-flow via Protobuf over ZeroMQ

## Orthogonal work (2 min)

Probably little progress, and Matt is away
- Rose Python 3
- "rose suite-run" migration?
- cluster awareness
- other?

## End (5 min)
- **anything else?**
- next meeting date
