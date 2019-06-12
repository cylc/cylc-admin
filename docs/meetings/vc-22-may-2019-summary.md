# Cylc Project VC 22 May 2019 Notes and Actions

## cylc-7
- no major new issues reported
- no urgent need for 7.8.3 any time soon

## cylc-8 Overall Progress

- somewhat behind, but more or less on-track.
- ACTION (HO) - update Projectt Gantt Chartt

## Packaging

- pip done (cylc/cylc-flow)
- (ACTION (??) - conda, rpm, Docker etc.: as need later on)

## Discourse

We are good to go on the switch-over from Google Groups
 
- ACTION (HO) - post final Google Groups announcement and freeze the forum
- ACTION (HO) - ask UM Partner sites (via the TAG) to use the Discourse forum
- ACTION (ALL) - encourage users at your own sites to do the same
- ACTION (ALL) - start making announcements and "non-chat" posts on Discourse

## Cylc Hub

- MR has been writing code to avoid importing JupyterHub Python libs at the remote end
  - almost working, but quite difficult and trouble with debugging and logging.
  - SB to help out with Tornado aspects of this
  - HO still hoping to look in detail too

- ACTION (MR) - park the custom code and just import JupyterHub libs for now; 
  - push this code to a branch on cylc/cylc-ui and comment in an associated
    issue, so we do not lose it (may want to come back to it in future)

- ACTION (MR) - show that ssh remote spawn works with JupyterHub installed at
  both ends, and write up how to do it

## WS, UI Server, UI

- GraphQL data model is agreed to be solid

- OS argued for GraphQL in the WS, and possible CLI direct to WS (w/o the UI Server)
  - DS can do this if necessary, but is not convinced
  - ACTION (OS, DS, and others): discuss in relevant Issues and make a decision
   - arrange a sub-group meeting once participants have understood the issues

- ACTION (BK) - mock some WS data to serve as a basis for UI development and
  testing, and work on the initial text tree view. (Or dot view).


## END

- ACTION (HO) - next meeting date (before June 14, when HO and BK travel to Exeter?)
