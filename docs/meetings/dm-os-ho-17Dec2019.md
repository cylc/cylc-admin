Meeting at start of OS visit to NIWA

Present: OS, HO, DM

Topics discussed and outcomes:

- OS to work on initially at NIWA:
  - Primarily: [API-on-the-fly](https://github.com/cylc/cylc-ui/issues/339)
  - Secondarily: work with BK on completing treeview design, mutations, and
    performance

- Graph View:
  - Agreed to come back to this once we are sure the most-important treeview is
    more or less complete and performant

- Hub multi-user access (i.e. hub-authenticated user to access other-user UIS)
  - We tested this in June by manual insertion of other-user name in URL?
  - TODO: how to expose this via the Hub UI?
  - TODO: show the UIS user clearly when viewing workflows
  - TODO: confirm authenticated user info passed to the UIS for use in
      authorization
  - reasonably confident this can be done quite easily
  - [DM] We will need this to allow "anonymous" access for users who do
    not have an account (to replace the existing functionality of cylc review).
    We expect to do this via a guest account.

- Remote job CLI over HTTPS (TW@NRL has properly remote job sites)
  - API-on-the-fly will allow CLI to easily switch (or fall-back) from
    direct ZMQ to WFS, to indirect GraphQL over WebSocket to UIS
  - TODO: confirm this solves the problem (WebSocket is initiated over HTTPS
    and stays on the same channel/port)
  - TODO: How to keep a headless (hubless) UIS alive indefinitely? Do we need to
    support this, or leave it to sites with this requirement?
  - TODO: non-interactive CLI authentication to UIS without the Hub?
