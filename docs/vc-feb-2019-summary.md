# Feb 7 Cylc VC Summary

See [Agenda](vc-feb-2019-agenda.md).

__general__

[MS] 
The VC was good today. Technology worked fairly well. My brain dump (some
thoughts are post-VC). And so sorry if it is not very
organised.

[MS] I have already added a reply to your Google Groups 7.8.1 post to notify
   users of upcoming 7.8.2 release. Looking at the list of issues. I now think
   both Martin's PR for BOM  and Sadie's Cylc Review PR should both go in.

[HO] Thanks for the forum post; agreed on the PRs.

__cylc-7.8.x__

[MO] As I'll just patch our site deployment of 7.8.1 for our users, so 7.8.2
should not be urgent for us. I'll see where we are tomorrow - and prepare a
cherry-picked-from-master PR for the 7.8.x branch.

__cylc-8 progress__

[MS] We should have a fully functional system (w/o GUIs) including all of Cylc
and Rose in about a month time. CherryPy requirement should be replaced with
Tornado or 0MQ.

[HO] Yeah, that's brilliant, being able to build on a working core system will
help us a lot.

[MS] We should then get on with packaging. (MO team & Bruno?)

[HO] Yes; this should be a quick job once the Python 3 port (/ working first
hack!) is done.

[MS] UI data model + UI server front end work are moving forward. Bruno and
David S: can we have more exposure of your work (to avoid wasted effort)?
 
[HO] (we need to subscribe to notifications from developer repos; and/or
move some to the Cylc Org on GitHub)

[BK] It would be great to have somebody else working on the project now.
Especially if s/he has experience with another framework like Angular/React/etc
(the one I have better knowledge is Backbone.js, with is a bit different). All
my code is going under https://github.com/kinow/cylc-web/, but I could transfer
ownership to the cylc organisation. Or we can start a new project there.
Whichever works best. 

[MS] Perhaps we have set up some new projects under the Github Cylc
organisation. We can also set up some Github project boards (or Trello, see
later) to explain the progress and decision points.

[HO] (see below)
 
[MS] We can also do with some extra support for Bruno and David S. (Should that
be Hilary or someone from the MO team?)

[HO] I'm going to do my best to get stuck into first JupyterHub then the other
bits, but bear in mind that [project management etc.] takes quite a lot of time
... need more hours in the day!

[MS] JupyterHub appears to still be a big unknown. I am under the impression that
the choice of the spawner will limit what you can or cannot do spawn as well as
connect to? But we are not sure if we understand why. And can we write our own
spawner as a plugin? Do we need a separate VC just to discuss this? Either way,
I believe Hilary has agreed to work closer with Martin on this.

[HO] Yes, from Monday I will attempt to understand this. Have also talked
to Martin today about it.  We can have a VC on the topic at some point after
this.

__TypeScript__

[MS] was mentioned (by Bruno?) What's the conclusion?

[HO] I have no strong opinion. I see Bruno addressed this already in his reply.
 
[BK]
- far every web developer I spoke with recommended to move to TypeScript.
The current project is based on Vuetify [1]. Vuetify is a Vue.js framework
based on Google Material Design.  
- What we use, is actually a free MIT licensed theme based on Vuetify [2]. This
theme is written in JavaScript, and contains more than just Vue.js and Vuetify.
It also has support to i18n, validation, themes, etc. So that makes it a bit
harder to just start using TypeScript immediately, but that's still doable.
- I am finishing the page for signing in. Once that's done, I will check what's
the effort to move everything to TypeScript. Vue.js supports it, and the next
major release includes re-writing the project in TypeScript [3].

__Testing framework (UI)__

[HO]
- I don't think we have any conclusion on web UI testing framework either.
Some names were mentioned, e.g. Selenium, Karma, etc.
- I think we did all agree that we need to use this, plus full unit testing
throughout, from the start on all new code. Except perhaps quick POC work.

[BK]
- That's correct. Vue.js comes with Jest & Mocha for unit testing [4]. That's the easy - but still important - part. Writing tests for things like the data model used by the GraphQL Apollo client to communicate with the backend, the classes used for authentication, and some tests for the components and the data state of the application.
- The other tests that were mentioned are normally called functional tests, but also called e2e or end-to-end tests in web frameworks. Selenium is probably the grandfather of these frameworks, but there are other options like Karma, Nightwatch, Cypress, ... There is a ticket for that in the cylc-web repository, but I have not looked into that yet.

__Python 3__

[MS] MO team will continue to invest in some effort to replace logic with
Python 3 technology, e.g. argparse, asyncio, etc, but noting that other
activities may have higher priorities.

[HO] All good.  As I said in the VC, I'd prefer we get a basic end-to-end
system in place before putting a lot of time into this, because at that point
we will have some confidence of getting across the finish line, and can
reassure all stakeholders of that.


__project management tools__

[MS} I have tried using Github project. The main problem is lack of
notification. Trello appears to be doing better in this front, but is obviously
less integrated with Github. We are also uncertainly as to how much free Trello
allows us to do. We should investigate.

[MS] MO staff are now blocked from accessing Riot from a networked work device.
Matt is happy to continue to access via personal device. It is up to individual
team members to decide whether they will do this or not.
We are waiting for Met Office IT security to confirm that Discourse is an OK
technology to be used on a networked work device.

[HO] We need a comms medium that everyone can participate in, or else we
(particularly me, perhaps) is faced with constant uncertainty about who has
seen what.  So I reckon, much as I've come to like Riot.im, we should do this:
- Free Trello
- Free Slack
- Google Hangouts or Skype VC
Free Slack has no integrated VC, but I imagine we can create "groups" in G
Hangouts so it won't be too inconvenient.  Every needs a Google account though
(I think?).

[HO] Free Slack loses our info after 10k posts, BUT a) we seem to have no
viable alternative that everyone can use; b) it's only chat, the online
equivalent of talking at morning tea - so just make sure that record important
outcomes elsewhere.


__next VC date?_

[MS] Did we decide when would be our next VC?

[HO] Let's aim for the first week of every month (with actual date by Doodle
Poll) and potentially one mid-month (all of us, or a subset) if a significant
fraction of us think it's needed. OK?

