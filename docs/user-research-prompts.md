# User research for Cylc 8 & Rose 2: discussion prompts

**Key questions** we should contemplate. I also briefly outline some of my
thoughts, mostly to *foster discussion*.

Shorthands used throughout:

* **UR**: user research
* **UX**, **UI**: user experience, interface
* **C&R**: Cylc & Rose
* **C8 & R2**, **C7 & R1**: Cylc 8 & Rose 2 (i.e. the version of Rose to accompany Cylc 8), Cylc 7 & Rose 1 (current Rose in use)
* **MO**: Met Office


****************

### 1. Do we conduct UR for C8 & R2 largely together or independently?

#### Key considerations:

* Novice (& even some experienced) users will *not know the difference*
  between our systems.
* Users are *strongly influenced* by what they have *already experienced*.
* The definitions of C&R are *static*...
* ...but the line between C&R is *dynamic* & indeed has changed significantly
  recently, e.g. through migrations to Cylc.
* Using Cylc with Rose is our main use case, but both C&R are also (designed
  to be potentially) used standalone.

#### My proposals:

* We should ultimately consider the *full package of configuring & running
  applications by workflow*, rather than C&R as systems in themselves, as
  this is the *user-centric* view.
* Therefore UR on C&R *together* is necessary, but we should make sure we
  gather input from those who use Cylc, & Rose, standalone, as well as
  C&R in combination (see my proposals for Q3).
* We should refer to the overall experience & problems therein, not explicit
  C &/ R usage.


### 2. What categories of user would define a minimal representative sample?

#### Key considerations:

* C&R are used in a wide range of contexts & by a diverse user base.
* Whilst categorising users is flawed in many respects (e.g. it is
  pigeonholing), we need to ensure we collect data from a varied
  cross-section of users. This will involve distinguishing some user groups
  from which input must be captured (unless anyone can think of another way
  to do it?)
* Ideally, we could collect input from a cross-site user base, but given time
  & resource constraints, in practice it seems likely input will come
  largely or wholly from MO users.

#### My proposals:

* Firstly categorise by *domain* (or *role*) & *C&R expertise level*,
  as in the table below, where we should collect input from each
  category not marked :x:. Input would *not* have to be gathered
  *proportionately*, as that seems both unnecessary & too difficult to achieve
  in practice. I do think certain categories are most are more important to
  research (see :bulb: & :rocket: tags), however given that they are those
  that are most common, they should naturally be the most sampled.

  | Domain: <br>  Example Role: | Research <br> Scientist  | Operations <br> PSA | Software <br> SSE | Optimisation <br> HPC Analyst | Other <br> n/a |
  |---------|-------|-------|-------|--------|------|
  | **Expert or "power" user** | :bulb: | :bulb: | | :bulb: | :x: |
  | **Standard & (fairly) frequent user** | :rocket: | :rocket: | :bulb: | | |
  | **Beginner or occasional user** | :bulb:  | :x: | | :x: | |

  ##### Key:

  User group is...

  :x:: ...non-existent/rare/insignificant
  :bulb:: ...important (relative to blank box)
  :rocket:: ...especially important (ditto)

* Outside of that "phase space" (i.e. not as extra axes to the category table,
  given the categories not numbered "1" will be rare in our sample) we should
  also make sure we hear from at least some of those who:
  * use 1) C&R, 2) Cylc *standalone* & 3) Rose *standalone*;
  * are 1) internal (MO) but also 2) *external* (i.e. based at another site)
  * do 1) not have... 2) have... & special *accessibility* requirements,
    e.g. are colour-blind.


### 3. How can we encourage users to provide input for a good\* sample?

\*i.e. decently-sized & representative (as above)

#### Key considerations:

* Lack of *incentives* will likely mean a small user input sample size & also
  maximise a
  [bias by inclination & availability](https://measuringu.com/random-sample/)).
* Users cannot join in with something they are not aware of, so *advertising*
  for UR participation is crucial.
* We cannot, like specialised UX or private firms etc, offer renumeration
  for feedback, but that is not to say we cannot be creative to offer
  *non-monetary* incentives.


#### My proposals:

* Advertise, advertise, advertise! Via all MO channels we can get items on, &
  via word of mouth.
* *Emphasise strongly* that:
  * any input will be a worthwhile investment, since it will (likely)
    *influence* the nature of C8 & R2, so that by conveying needs, problems &
    ideals users will *make the systems better suited to their own needs* &
    therefore *make their lives (jobs) easier* in the (near) future.
  * *now* (i.e. the early development stage) is the time to provide input;
    any input after will probably come *too late* given UR analysis &
    development plans. One (major) chance only!
* As *concrete* incentives:
  * See if we can set-up & promote an IRA & have a "random winner" prize
    draw to which any user providing feedback will be entered into, a solution
    that is low-cost but offers high-ish potential reward per user.
  * Perhaps (despite being gimmicky) have some title (e.g. "C&R UX Champion")
    to offer to users who are particularly helpful, which may appeal as it
    is something they could e.g. put down against their performance &/or
    personal development evidence.
  * For e.g. drop-in sessions, provide (& stress in all advertisements) sweet
    treats like *cake*. Cheap but effective!


### 4. (How) should we describe the context &/or our vision?

#### Key considerations:

* Some context must be provided so users are aware of the need for the UR
  they are participating in & to direct it...
* ... but the context used to initiate discussion (etc.) will influence the
  responses users provide.
* Beware of the general introduction of bias. Notably, if we mention
  general feedback (themes) we have heard from (many) other users, that will
  influence the user who could easily go on to provide an inaccurate account
  of their views & thoughts e.g. by mimicking what was conveyed by others.

#### My proposals:

* For all UR events, outline the following, consistently to ensure the context
  is a controlled variable (at least as far as it can be, as of course some
  users will already know more background than this) by writing into a
  reference document:
  * The *major scope of C8 & R2, in neutral statements*, e.g: it will be in
    Python 3 not Python 2 as previous; we are changing the nature of
    the GUIs from those build upon PyGTK to solutions for the web browser
    & doing so means we have to make major architectural changes to Cylc; for
    the web GUIs we plan to unify a number of the previously separate GUI
    components/services of C&R, such as 'gcylc', 'cylc review' &
    'rose config-edit', ...
  * An overview of the *generic potential for change* e.g. the web-based
    nature opens up a wide-range of possibilities for enhanced usability with
    respect to functionality, aesthetics, integration, flexibility &
    speed, ...
* Ensure final said reference document is peer-reviewed to check for
  potential biases.
* To help use analyse data the collected, make notes during UR on any extra
  context outlined (while we should try to avoid doing so, it may be
  unavoidable or natural to disclose given a user's responses or background.)


### 5. What is the core information we want to extract?

#### Key considerations:

* One key piece of advice from the UX course attended by members of
  the MO team was "*don't research customers to ask them for a solution,
  research them to better understand the problem at hand*".
* C&R are software systems, & therefore there may be ways to extract useful
  information from existing stored text files e.g. logs, command history &
  C&R configuration files.
* Common themes for data to collect in the UX field include problems
  ("pain points") & work-arounds for them, "flows" (the step-by-step path
  followed through the UI), habits, opinions, needs, ideals.
* We do not know what collected UR data (& "metadata" i.e. background
  information) might be useful going forward.

#### My proposals:

* Aim to gather (largely) qualitative information on (at least) the following:
  * *User background*: their job, their area, why they use C &/or R & how
    long they have been using them for, if they needed help in some respect
    what they would do in the first instance (documentation, come to see us in
    person, etc.), how often they read the documentation, if they have
    attended the MO training course.
  * General *working preferences*: GNOME theme (dark/light), general window
    placement & use of tabs, etc.
  * The nine *GUI components of C&R* [listed in turn]: which users use (most),
    what they use each one for, for how long at a time, & what their ideal
    would be for each. If they do not use a GUI component, why they do not.
  * *Component-specific questions*: e.g. for a general component which buttons
    & widgets they do or do not use, for gcylc which views they use (most) &
    again if they don't use one why not, for Cylc Review what information
    (what log or table, etc.) they tend to view & for what reason & at what
    points they open a new tab ...
  * How they *interact* with C&R when GUIs are open i.e. when there is a choice,
    whether they use the the GUI controls & widgets, or use a command, &/or
    interact directly with the relevant configuration text file.
  * *Devices*: which kinds they would like to be able to view the C&R UIs on
    (looking to find out if they might like a mobile/tablet version without
    asking that straight which would be a leading question).
  * Their "*flow*" (path) through any GUI components they use, where for any
    apparent divergences from an ideal efficient flow, what they did as a
    work-around.
  * *General input* e.g. what the user considers: the best, & worst, things
    about the GUIs, what improvements they think are important or would like
    to see, etc.
* Aim to gather evidence/data-based information on the following:
  * *Command-line usage*: ``history | grep cylc`` & ``history | grep rose``
  * A reference (or zipped directory copy for an external user) to what the
    user considers a *"typical" suite* they would work with (so we can e.g.
    see how it appears under the various GUI components e.g. as graphed,
    in the Tree View, within 'config-edit' if applicable...)
  * Their *default configuration* to customise GUI components: i.e. their
    ``~/.cylc/gcylc.rc`` & ``~/.cylc/gscan.rc`` if they have these defined,
    else go through all config items possible for these & ask what settings
    they would prefer.
  * If internal (i.e. MO), the user's suite listing page on Cylc Review.
* Record everything! Write as much as possible down even if it seems
  irrelevant. Information can always be ignored but will be lost (even if
  confined to memory as that is not a reliable source) if not not written down.


### 6. By what means do we extract the information (from Q5)?

#### Key considerations:

* Beware of asking *leading questions*! e.g. "*Do you find the GUIs slow?*".
  The best way to avoid this is by asking questions that are
  [direct](https://www.usertesting.com/blog/direct-questions/) i.e. open-ended.
  Direct questions are also wider in scope so users may share more information.
* Examples of common UR methods from the UX field that don't require
  specialist resources we don't have, & that are applicable to the pre- or
  early-development stage we are at, are:
  * interviews
  * focus groups
  * surveys
  * card sorting
* An important distinction in the UX field is between what people *say* (their
  attitude) and what they *do* (behaviour). Both are important. Some UR
  techniques investigate the former, some the latter, & some both to various
  extents. A good summary can be found [here](https://www.nngroup.com/articles/which-ux-research-methods/).
* As well as collecting new information, we can draw upon existing or stored
  past information to use for our UR (see Q7). However, since our problem is
  a unique & niche one, there is probably not any existing UR that will be
  useful to us, so all UR must be new ("primary"), not "secondary" research.
* There are user groups e.g. the MO Suites Guild which we can utilise as a
  platform & a source of users (to try to recruit) for UR.

#### My proposals:

* I think we should use UR methods which probe both *individual* users &
  *groups* of users.
* For individuals, use the method of *ethnographic interview*,
  which is a combination of interviewing (asking questions) & observing for
  the user in their natural environment & context. We would take a (perhaps)
  30-minute slot to sit with the user at a set time where they are using
  C &/ R & we observe them as they undertake their daily tasks, asking them
  to verbalise what they are doing & why, as well as asking questions &/or
  taking notes on the item from Q5 at appropriate points.
* For groups of users, such as the group attending a Suites Guild session, use
  a *focus group* (possibly including card sorting exercises) which revolves
  around a volunteer user working on a specific pre-arranged suite
  problem (e.g. relating to configuration, scheduling, runtime or design of)
  taken from their day-to-day work on their laptop, as shared on the screens,
  & describing some brief background & how they use C&R to go about solving it.
* As well as the above, invite & encourage *random pieces of user feedback*,
  via the usual channels but also by setting up & advertisting a form for that
  purpose only. Users may think of something important to convey during their
  day-to-day work, & this would allow them to note it down easily at the
  time, where it otherwise may be forgotten.


### 7. Should we consider historical user feedback, & if so how?

#### Key considerations:

* From user support over the years, we have heard & observed a lot,
  including information that could be very useful.
* However, the context in which it was said is really important. For example,
  a user might have conveyed a more negative opinion than they really hold
  at a stressful instance (a deadline or when encourntering a bug) or have
  made a comment about an aspect of C &/ R that has since been
  modified so that it is no longer applicable.

#### My proposals:

* Consider signifcant past feedback only where it is evidenced visibly,
  (typically in writing) & not just recalled from someone's memory, & only
  where there is enough background evidenced to provide context to record.
* To uncover & expose useful feedback conveyed in the past in this way,
  we should systematically trawl (i.e. skim read) through our written
  communications channels, including the Google Groups & Metomi team inboxes
  & relevant MO Yammer groups, & copy & paste into the UR records any relevant
  feedback *alongside brief written metadata notes about the context
  it was made in* (the date conveyed on, the channel, the user, the
  immediate circumstances, etc).
* Make reasonable effort to check whether any historical comments are still
  (likely) valid, e.g. by looking at the codebase to see if the utility/aspect
  in question has not since been changed in a relevant way.


### 8. Where do we record all user research?

#### Key considerations:

* Information is useless unless it can be *seen*.
* It is much easier to organise & analyse information if it is all *kept in one
  location*.
* It would be helpful if we could all freely add to & edit documents holding
  UR, with knowledge that we won't undo or block other people's amendments.
* It is *not wise to disclose the store of UR with any users* we are collecting
  input from, as they will be influenced & biased by other people's comments.

#### My proposals:

* Use a contained location (repo or directory) in GitHub to record all UR as
  it is accumulated.
* Either make the repository private to the core Cylc team *or* do not mention
  that such a UR log exists or indeed its location (it is very unlikely users
  would look for or find & then read it).


****************
