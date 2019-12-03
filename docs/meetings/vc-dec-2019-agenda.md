# Cylc VC 3 Dec 2019 Agenda and Notes

- Project admin - 15 min (HO)
  - OS visit to NIWA
    - computer and office sorted!
    - staying across from NIWA, first week
  - SC19 update
  - 7.8.5?
    - agreed we will make another release soon
  - 8.0a2?
    - late Jan 2020? (would help UK milestone) with:
      - mutations (and some useful commands hooked up)
      - websocket and subscriptions (not 5 sec polling)
      - (maybe: view and start stopped suites)
  - Cylc FTE (etc.) TAG meeting
     - Cylc 8 is a high priority and trying various channels to get more
       resource, but it will take time, so we just have to accept delays.
       Not too much concern about sticking with 7.8.x in production for a bit
       longer at least.
  - Feb Workshop 
     - time to start making bookings
     - discuss and implement config name changes, and "unification"
     - revisit the Gantt chart 
     - finally decide Cylc 8.0 MVP priorities and best route to it

- Platforms support (TP)  - 10 min
  - plan: submit a bunch of small PRs for quick merge, which won't break
    anything but also won't do anything until the final PR. HO needs to
    follow closely

- Back-end authentication (DM, OS?) - 10 min
  - SB's work merged, and incorporated into DS's incremental update PR (just
    merged)
  - TODO:
    - file locations
    - managing remote job hosts
    - keys/tokens should be per suite, not per user
  
- Data provision:
  - Data sync now pretty much nailed
  - TODO:
    - CLI to GraphQL
    - authorization (not too urgent)
    - event driven subscriptions (GraphQL over WebSocket UIS-to-UI) 

- UI: mutations PR related to subscriptions PR (which also adds WebSocket)
   - then can work on adding all the specific commands to the UI

   
- UI views etc. (BK, OS) - 10 min
  - status and next steps
  - mutations

- Current cylc-flow PRs - 5 min
  - CLI: entry points
   - enables extending the Cylc CLI without touching cylc-flow
   - last problem is trivial (test calls a CLI alias instead of full command
     name)
   - should be able to get it in very soon
   - `cylc subscribe` command (just merged) will need to be done
  - sqlalchemy
   - not a high priority, and a couple of small details to finish
   - but agreed let's get it in before too long
  - retry as xtriggers (OS absent)
  - job scripts: user scripts in subshell
    - DM flagged concerns that testing is needed (and interaction with current
      polling logic for tasks that resurrect?)
    - but MS and HO think its a good change
    - let's put it in Cylc 8 but flag the fact that extensive testing should be
      done
 
- Misc
  - "Tip of the Day" Discourse category (OS)
    - Let's start with 15 or 20 pre-prepared tips
  - GitHub Actions (BK)
    - Agreed, let's do it (GitHub just talked to MO about this!)

- AOB?
  - None
