# Development & working practices: discussion prompts

**Prompts to discuss for the Monday afternoon session of the
  Cylc Development Workshop (December 2018), aiming to complement & extend
  points under the 'Goals' section of the [agenda](dec-workshop-agenda.md).**

(Questions are numbered, while relevant considerations are bulleted.)

_NB. for future thoughts on this general topic:_

- There are many established & tried-and-tested
  methodologies e.g. flavours of Agile & common traditional approaches we
  can take inspiration & learn from.
- We are not tied to any one, therefore free to 'pick-and-choose' practices
  that work well for us.
- We should consider our approach carefully at this early stage, but be
  flexible to adapt to what ultimately works best.


## Work assignment

1. How do we assign tasks to a specific team member?
  - We should aim to distribute work/tasks by topic to:
    - minimise silo-ing of knowledge;
    - accommodate everyone's strengths & weaknesses & preferences;
    but...
    - not overwhelm/over-burden anyone, given the range of new technologies
      & theory etc. to learn to use effectively & adapt to.
  - Distinguish between the in-depth &/or broader knowledge attained in
    code development with the surface-level &/or partial knowledge attained
    in code review.
    
2. (On that note) do we want to change our Pull Request reviewing policy?
  - i. with a larger team should we extend the merge threshold at 2 reviews?
  - ii. should we create agreed terms (or similar) to clarify review
    thoroughness, e.g. 'sanity check', 'functionality', 'good enough'
    (where it is not clear exactly what is needed e.g. UI elements)?
  - iii. could we make use of joint e.g. pair reviews to maximise knowledge
    sharing? Would that constitute one or multiple (e.g. for pairs two)
    GitHub reviews?


## Tracking

Without awareness of the current project state, & propagating of key
information influencing other people's work, we risk wasted effort (&
therefore time) due to misunderstandings.

1. How (what tools & format) & to what precision will we track:
  - i. components/features status e.g. in development (code & tests & docs),
    in review, merged (& on user feedback & response stages if appropriate)?
  - ii. what everyone is assigned to do & what they are actually doing?
  - iii. progress?

We have a target completion date, & ultimately deadlines with the PyGTK
platform freezing & then the Python 2 EOL, so we need to understand if we
are at least roughly on-track.

2. Retrospectives &/or progress reviews:
   - i. How often (or upon achievement of what milestones) to have these?
   - ii. What format should these be in (e.g. discussion or formal document)?
   - iii. How to 'measure' progress?
     - time & effort estimates are notoriously difficult in the software
       industry, & mainstream e.g. 'burn-down' charts etc would probably
       (?) be excessive for our context.


## Priorities

1. How do we assign priority to general tasks to minimise blockers?
   - there's not just development & review etc. to consider, but also
     necessary planning & research etc.


## Communication tools

#### Current channels

GitHub is our development & issue-tracking home. We also communicate
online/remotely with each other, & enable users to contact us,
via email (chains), internal site tools, & the Google Group.

#### A discussion channel

It is apparent that in many cases detailed dialogue is better moved & placed
outside of GitHub issues. There are no shortage of platforms available
for messaging in these cases.

1. But which do we choose?
   - Current candidates in discussion are:
     - Riot.im
       - disadvantages: it does not have threaded conversations & Met Office
         security will only allow it to be used by the Metomi team, so
         Met Office Cylc users would not be able to view or contribute there.
     - Slack
       - a prominent, popular alternative
       - is only free up-to 10,000 messages, which given some GitHub PRs
         & Issues have order-of-magnitude 100 messages, would not take us
         long to exhaust; the crux of messaging freely is hence not possible.
     - Discourse
       - a free alternative with a nice interface, threaded conversations,
         & intuitive 'forum-like' conversation arrangement by topic.
     - We could still investigate other tools e.g. Gitter etc.
   - The platform chosen needs to:
     - be free to the extent we will use it
     - display code easily & effectively
     - interface well with GitHub
     - (ideally) allow users to submit questions or issues etc without having
       to create an account (which will put many off bothering), so that we
       can retire the Google Group.


#### GitHub

2. Should we make further, extended, use of GitHub? It:
   - is already our 'base', so it would not add inspection overhead to use
     more of the various features it offers.
   - is version-controlled, so project planning/organisational arrangements
     would be logged & notifications sent to relevant people with any changes.
   - i. Should we use GitHubs 'Projects' feature, & if so for what purpose?
      - its agreed domain should be clear-cut & not overlap with
        that of the selected conversational platform (e.g. Riot).
   - ii. Or could we put up tracking &/or planning documents as files under
       'cylc/admin'?
       - these could be basic text markdown files, or we could
         investigate simple libraries to implement nicer tables & logs, etc.


#### Overall resource strategy

Since we will then have another communication platform to manage:

2. Can we agree a plan on which tool should be used in what cases, for
   consistency & to utilise resources for their designed best-use cases?
   - want to avoid having too many tools; we have to check each one
     regularly (but...)
   - do not want to have tools being used outside their scope e.g. GitHub
     single issues for very broad-ranging topics with lots of comments
     obscuring key information.
   - proposal to discuss:
     - as always, GitHub for code & registering issues with the essential
       information include only.
     - if issue or PR review comments become too involved or numerous, agree
       to move them to the chosen conversation platform (e.g. Riot).
     - only use emails when there is a clear need to e.g. for notifying
       external people, administrative reasons, etc. Inboxes are usually
       cluttered & arrangement by topic is difficult.


## Coordination

Wider team not co-located, & the majority working under towo flipped timezones.

1. How can we use this to our advantage?
  - Almost round-the-(UTC)-clock development / eyes on the codebase; want
    to leverage this to streamline hand-over for review & testing stages...
  - ...not allow to be a drawback via adding out-of-hours waiting time or via
    miscommunication.
  - we should also aim to benefit from the different software environments
    e.g. OSs & software stacks at each site for testing purposes.


## Decision making

There will be decisions to make along the roadmap, both which we could
& couldn't anticipate at this early stage. In order to make these:

1. What communication tool(s) will we use?
   - necessary since after the workshop we will rarely (if that) get to meet
     face-to-face.
2. What processes?
   - everyone needs to get a chance to contribute their thoughts & ideas.


## User involvement

It is clear that users will need to involved in some capacity to provide
feedback & direction. But:

1. At what stages (when)?
   - operational users will need to be given use guidance or
     training in advance
2. ...and in what way? Feedback is crucial to successful UI & UX, but:
   - i. users have varying levels of knowledge about the purpose &
     technicalities of Cylc, so how can we extract useful, rather than
     unrealistic/idealistic, feedback & manage expectations?
   - ii. in general, people do not like change; how do we convince users of
     our general vision?
   - iii. we cannot please everyone, so how do we approach maximising the
     number of users who are happy (& their happiness) with our new GUI?

Critical i.e. operational users must to be able to smoothly change over to
using the new GUI from the old.

3. Can we provide guidance &/or training in the later stages, before the
   official version release, to (at least) these users?
