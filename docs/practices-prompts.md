# Development & working practices: discussion prompts

**Prompts to discuss for the Monday afternoon session of the
Cylc Development Workshop, to complement & extend on points under the
'Goals' section of the 'dec-workshop-agenda.md' document.**

NB. for future thoughts on this topic:

Overall, there are many established & tried-and-(thoroughly)-tested
methodologies e.g. flavours of Agile & common tradiational approaches we can
take inspiration & learn from. We are free to 'pick-and-choose'
practices that would work well for us. We should consider our approach
carefully at this early stage, but be flexible to adapt to what
ultimately works best.


## Work assignment

How to assign tasks to a certian team member. Want to distribute work/tasks
by topic to:
- minimise silo-ing of knowledge;
- accomodate everyone's strengths & weaknesses & preferences;
but...
- not overwhelm/over-burdening anyone, given the range of new technologies
  & theory etc. to learn & use & adapt to.

Development provides in-depth knowledge, & review (depending on level)
builds superficial but signficant knowledge & awareness.

Additionally, for reviewing:
- with a larger team should we extend the merge threshold at 2 reviews?
- should we create agreed terms (or similar) for review thoroughness,
  e.g. 'sanity check', 'functionality', 'good enough' (for that point in time,
  for cases where it is not clear exactly what is needed e.g. UI elements)?
- can we arrange pair reviews to maximise knowledge sharing? Would that
  consititute one or two GitHub reviews?


## Tracking

Without awareness of the current situation, & propagating of key information
potentially influencing other people's work, we risk wasted effort due to
misunderstandings. How (what tools & format) & to what precision/detail will
we track:

- components/features status e.g. in development (code & tests & docs),
  in review, merged, user feedback & response stages if appropriate...
- what everyone is assigned to do & what they are doing.
- progress.

Regarding progress, we have a target completion date, & ultimately deadlines
with the Python 2 EOL & PyGTK freeze, so we need to understand if we are at
least roughly on-track.
- how often to have retrospectives &/or progress reviews?
- how to measure progress, given time & effort estimates are notoriously
  difficult in the software industry, & mainstream e.g. 'burn-down' charts etc
  would probably (?) be excessive.


## Priorities

- How do we assign priority to general tasks (not just development & review,
but necessary planning & research etc.) to minimise blockers?


## Tools & resources

GitHub is our development & issue-tracking home. We also communicate
online/remotely via email (chains), internal site tools, & have the
Google Group.

It is apparent that in many cases detailed dialogue is better moved & placed
outside of GitHub issues. There are no shortage of platforms available
 for messaging in these cases, notably:
- 'Riot.im' has been suggested, but has distinct disadvantages: it does not
  have threaded conversations, Met Office security will only allow it to be
  used by the Metomi team, so Met Office Cylc users would not be able to
  view or contribute to discussions.
- Slack: a prominent, popular alternative, but is only free up-to 10,000
  messages, which given some GitHub Pull Requests & Issues have had
  order-of-magnitude ~100 messages, this would not last long. The whole idea
  is we can message freely without constraint.
- Discourse: a free alternative with a nice interface, threaded conversations,
  & intuitive conversation arrangement by topic.
- Another tool e.g. Gitter?

The platform chosen needs to:
- be free to the extent we will use it
- display code easily & effectively
- interface well with GitHub
Also:
- ideally allow general users to submit questions or issues etc without
  having to create an account (which will put many off bothering), so that
  we can retire the Google Group.

Can we agree a plan on which tool should be used in what cases, for
consistency & to utilise resources for their designed use cases? Want:

- to avoid having too many tools; we have to check each one regularly
-it to be clear & simple to navigate to all information on a certain topic.

Should we make use of GitHubs 'Projects' feature? GitHub is already our
'base', so it would not add inspection overhead to use this feature. However
if we use it we should agree a consistent purpose, so as not to overlap
with the domain of the discussion platform (e.g. Riot).

#### Proposal:

- As before, GitHub for code & registering issues with the essential
  information for these.
- If an issue or review comments become too involved, agreed to move them to
  the chosen discussion platform (e.g. Riot).
- Only use emails when there is a clear need to e.g. for notifying people
  external to the team of information, administrative reasons, etc. Inboxes
  are usually difficult to manage without assorted dev emails flying in,
  & it is not simple to arrange by topic.


## Communication & coordination

Wider team not co-located, & working in flipped timezones:

- Almost round-the-(UTC)-clock development / eyes on the codebase;
  want to leverage this to streamline hand-over for review & testing stages, not
  allow to be a drawback via adding out-of-hours waiting time or via
  miscommuication.
- Want to effective use of the different software environments e.g. OSs &
  software stacks at each site for testing purposes, to increase
  coverage of these.


## Decision making

There will be decisions to make along the roadmap, both which we could
& couldn't anticipate at this early stage. After the workshop we will
only rarley, if that, get the chance to meet face-to-face to make these.
- What communication tool(s) to use?
- How to ensure everyone gets a chance to contribute their thoughts & ideas?


## User involvement

At what stage, & in what way, do we get users involved for feedback &
direction?

Feedback is crucial to successful UI & UX, but:
- users have varying levels of knowledge about the purpose &
  technicalities of Cylc, so how can we extract useful, rather than
  unrealistic & idealistic, feedback & manage expectations?
- in general, people do not like change; how do we convince users of our
  general vision?
- you cannot please everyone, so how do we approach maximising the number of
  users who are happy (& their happiness) with our new GUI?
