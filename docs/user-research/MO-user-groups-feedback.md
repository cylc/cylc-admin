# Feedback from Met Office User Groups (July '19)

## Context

* TSUG & CRUG are two internal Met Office groups of (mostly) power users that
  meet regularly to discuss matters of collective concern. MS regularly
  attends both meetings, which are held monthly (at least of late).
* Both groups had their July meeting (separately) on 16/07/19, & these were
  attended by SB & OS who requested an invite to discuss Cylc 8 feedback.
* Here we outline significant points from each group as:
  * conveyed in each meeting as above; &
  * noted down (up to now) in dedicated feedback pages set up by each group.
* Initials, as provided with each item, are used instead of names, so
  individuals cannot be recognised for this public record, but full names are
  known to SB & OS. '??' implies a name was not captured in the detailed
  notes, in most cases because the person in question was not known to SB.
* Note all comments are paraphrased, & transcribed as accurately as possible,
  but misinterpretations are plausible.


## Feedback raised

### TSUG (Trials Systems User Group)

#### From the meeting

* MT: What happens when you right click on suites? [We talked about the right
  click menus.]

* MB: Dot view is not shown in the mock ups. Will it still be available? For
  one particular suite of mine, Dot View is by far the most useful view, so
  I would not want it to be removed. [We reassured that it will still be
  there.]

* MB: Would rose bunch invocations show up separately, like for multiple jobs?
  [We said no.]

* MT: Families don't seem to be shown - how are they going to appear? [We
  scrolled down the design document & talked through the 'Family/Subgraph
  Grouping' item.]

* ??: What is the "clock face" indicating? [We determined they are referring
  to the task running state icon, & explained it is a radial progress bar.]

* AA: Whilst jobs are running ("at a click"), will we now be able to capture
  the job.out & job.err? [We said possibly, as we may include some
  functionality to tail the outputs.] Will clicking for the job.out or job.err
  take you a separate page? [We said perhaps, those outputs will be displayed
  within some UI web page & not in a pop-up window as before, & they would be
  shown in some tab, component or perhaps even panel under the new formulation
  of Cylc Review.]

* ??: How does Cylc Review fit into this? Will it be a separate page? [We
  explained that we have not decided yet on how exactly it will fit in, but
  that it will eventually be adsorbed into the UI in some way, rather than
  being a separate utility.]

* ??: The new Tree View (as shown on the mock-up) does not seem to show task
  timings. Will they be available to view? [We said yes, but explained that we
  feel the old/current Tree View contains too much information at once, & are
  looking at ways to make the new version more lightweight to show only the
  information a user cares about at any one time.] Task timings are really
  important to us (their team?), are there are plans for a view to give them
  much more prominence, as that would be very useful to us? For example a
  Gantt chart? [We readily shared that we plan to create a Gantt (chart) view,
  for precisely that reason since it had been expressed by other users, but
  that it would probably not be part of Cylc 8.0.0, only a later version.]

* ??  Will we still be able to run our suites in their current state? [We
  responded 'yes'.] So no upgrade process will be required? ['Correct'.] Will
  anything be deprecated? [We said maybe but emphasised that we would make
  sure users are informed of any deprecations.]

* RR: A new UI is good, but I actually prefer the command line, e.g. because
  I can explicitly see & use the whole "plethora of options". Will I still
  be able to interact with my suites purely by command, as I can do now? [We
  said yes, but stressed that we are not neglecting the CLI for the new UI,
  in fact we have plans including a wide-ranging "CLI refactor" Issue
  (``cylc-flow``, #2972) open for Cylc 8, & towards that have already made
  various improvements to the command line].

* RR: When you interact with a suite via the UI, it would be very useful to
  see the corresponding commands, e.g. in command logs (obviously there will be
  no command equivalent to the action of 'adding in views' etc., but for
  anything with a suite command equivalent). [We agreed that it would indeed be
  useful & we would note it, but also described how back-end changes mean that
  the UI & command line will be synchronised so that most/all commands
  correspond to UI actions & vice versa, & the command equivalents should be
  mostly evident.]

* AA: How will this all work for shared accounts? [We explained we are still
  planning what to do with shared accounts, but probably the shared account
  owner will grant various levels of access (e.g. limited control, read-only,
  etc.) to sets of users as desired, such that when logged-in to the Cylc Hub
  as themselves, users will be able to interact, to the permitted extent, with
  any shared accounts which owners have allowed them to.]

* MT: What are those panels containing the views? [We elaborated on
  the multi-view system in the design mock-up.]

* MB: Will you still be able to have multiple instances open, like multiple
  windows for the old GUIs? I guess it would be multiple tabs now? [We
  confirmed that multiple tabs would be the browser equivalent, & that the UI
  will work across multiple tabs, but explained how we are trying to reduce
  the need for multiple (and especially large numbers of) tabs with the
  multi-view panelling system, which aims to show users as much information
  as they could want to see, all within a single tab.

* AD: We have set up a feedback page under the retooling project pages & we'll
  keep putting recommendations on there [We said thanks.].

* MB: In general, Cylc meets our needs, but Rose does not. [We said this was
  noted.]


#### From feedback pages

Notable points (relating to the Cylc UI) from feedback pages:

* [From the agenda notes:] "TSUG noted that this [the Rose replacement] will
  be of most benefit to users represented here. The *Cylc upgrades are nice
  to have* but the Rose upgrades will be vital to allow us to get the most
  out of our trials."

NB: those pages contain quite a lot of useful feedback, but not much
(just the above) relating to the Cylc UI, mainly to Rose config-edit, plus one
non-UI Cylc matter (which is of relevance to future Python API work):

* Under the section 'User requirements for new suite control system', from MB:
  "Better support for determining more complex aspects of the graph of a
  suite. Some aspects of the suite graph of an ideal version of an evaluation
  suite would be sufficiently complex that using jinja2 is unwieldy, limiting
  and difficult to maintain. It would be useful to have the ability to define
  the suite programmatically either as an initial app, or by some other means.


### CRUG (Climate/Coupled Rose User Group)

#### From the meeting

* DC: For the Tree View, will we still be able to see [task-]job times? [We
  said that they are not shown on the mock-up but will be accessible, yes,
  though the Tree View may be more lightweight such that they (as with other
  column data) are only displayed when requested, rather than by default.]

* JCR: It would be useful if there were tools to analyse the performance of a
  workflow, e.g. a Gantt Chart. [We mentioned our plan to have a Gantt View,
  though said probably only in later versions of Cylc 8, along with other
  components to provide analytics for workflows & their individual tasks.]

* JR: Dot View is not on the mock-up, will it still be there in the new UI?
  I think Dot View is the most useful, for example to get access to the jobs &
  times just from the tasks. [We said this is noted & useful to know.]
* CM: Yes [agreeing with JR previous comment] I find Dot View useful to
  determine "flow" through suites. [Ditto.]

* RG: Will we be able to customise the datetime formats? I find them very
  unwieldy in their full form. [We explained that it would be difficult &
  impractical to change the datetimes used internally.]
* RG: (clarifying previous comment) I don't mean customising them as used &
  processed for the suite internals, just for the UI display, i.e. what we
  will see on the front-end. [We stated that we have considered datetime
  customisation on the UI, & hope to enable some means to make the datetime
  formats clearer there, for example by including the ':' & '-' delimiters even
  if not used in the configuration, to break apart the datetime components.]

* JCR: Will the semantics be changing at all, i.e. will we still use the 
  same terms (e.g. 'insert' c.f. task insertion)? We do face user errors
  occasionally, & too much change to the terminology might make that more
  likely by misinterpretation. [We said that terminology would mostly be
  the same as at present, with a small number of exceptions, giving the
  likely example of ghost nodes, but we emphasised that any such changes
  would be made clear to users.]

* RG: On "annoyances", I find it really frustrating that job files have
  no line wrapping, so you can end up scrolling along e.g. 14 screens wide to
  read a job.out line. [We replied that this was understandable, & we had
  heard others express that as an flaw, so we would look into line wrapping
  on job outputs for the Cylc 8 version of Cylc Review.]

* CM: I'm concerned about non-running suites being displayed on the 'gscan'
  sidebar component, as I have a lot of these. [We explained that suites
  listed by the gscan component will be filterable, so that stopped & un-run
  but registered suites can easily be set (& unset) by the user to not be
  displayed, as desired.]

* JR: Why are jobs and tasks listed separately for each task? And how will
  that work? [We elaborated on task-job separation.]
* JCR: [Responding to JR's comment] I think users may be confused [about
  that]. [We had a brief discussion about that concern, aiming to persuade
  that it may take some readjustment for users to become accustomed, but that
  we believe it will make the nature of tasks as an abstraction clearer to all
  in the longer-term.]

* RG: How do you plan to disseminate information to help users adjust to
  changes [such as that in the above question] & generally communicate
  features? [We said that we haven't decided on a detailed plan yet, but that
  we would definitely be prepared nearer the time of site release to
  communicate en masse, & well in advance of release, key information on Cylc
  8 & to provide training to ease the 7-to-8 transition suggesting drop-in
  sessions or courses as one possible format for that.
* RG: [Continuing on the above] Being able to get textual help at any time
  will be a big help. [We explained that the documentation will be updated
  to be applicable to Cylc 8.]
* CM: [Continuing...] Could you restart doing "tip of the day"? [We conveyed
  that others have expressed that they would like to see it return, &
  that we were considering incorporating it into the UI such that users could
  enable it if they found it useful or disable it otherwise, but we were not
  sure what content would be suitable & how to ensure it would be up-to-date.]
* RG: [Continuing...] I think for tip of the day, you would have to pitch it
  to the right audience, else it would be too specialised or simplistic. [We
  said we agree.]
* ??: It could also be helpful to include answers to frequently asked
  questions (FAQs) via a dedicated page or in a suitable component. [We said
  we thought that was a good idea, & we would consider it, though that
  something of that nature might be better placed in the documentation.]

* JR: The freeing up of colours for family/sub-graph grouping would be really
  useful for ensemble-type models, but in context there could easily be too
  many colours in those cases. [We expressed that it was a valid point, but
  that we hope to also support ways other than colour for distinguishing
  between sub-graphs.]

* JCR: What is the difference between the big(ger) & the small(er) squares
  for jobs in the Graph View? [We explained that the larger square shows
  the most recent job of all the jobs for a given task, to make it more
  prominent.]

* CM: What would happen when you have many more jobs?
* DC: [Agreeing on the above] Yes, then surely they would get too wide to fit
  inside the panel for the view? [We explained that it was not indicated in
  the mock-up which was to focus only upon key features, but that when there
  are a larger number of jobs than is sensible to display, we would have some
  means to summarise the state of them all, probably (in current plans) by
  displaying per task only the most recent job only along with a number to
  indicate the amount of preceding jobs.]

* JCR: What is the "triangle" icon? [We established this was a reference to
  the warning icon in the gscan component (&/or on the views in the
  'Dismissable Warning Symbols' section of the more-comprehensive design
  document) & described how it was to indicate warnings or errors (or task
  failures), & that it was in fact a feature in the current GUIs.]
* JCR: [In response to it being in the Cylc 7 GUIs] We don't tend to use that
  warning icon, in fact we didn't know about it. Do you [question directed
  at CM] use it?
* CM: No, not personally.

* RG: Could you have a terminal emulator? [We conveyed that the idea was very
  interesting & would be nice, but as something that would be nice rather
  than necessary to users, probably unlikely to be added, at least for Cylc
  8 or 9.]

* ??: What is the plan with Rose? [There was much interest in that question.
  We explained that we would soon like to hear feedback relating to Rose, but
  that the current session was specifically to hear Cylc UI feedback, briefly
  providing context for prioritising Cylc feedback.]

* JR: Will we be able to keep a long-running suite (running for a year or so)
  going through the changeover? [We said yes, there will be no breaking
  changes.]

* CM, on behalf of AC: AC wants to point out that both the Suites Guild &
  its Yammer group should contain, & could facilitate, useful feedback. [We
  said indeed, we would extract feedback from those.]

* RH: I have shared the Discourse forum post regarding feedback with my group.
  [We expressed our thanks.]


#### From feedback pages

A dedicated Cylc & Rose UI feedback page was set-up for, & advertised to,
members of CRUG, but (as of 13/08/19) no items of feedback have been
registered regarding the Cylc UI (there is one item pertaining to the Rose UI,
though, so some group members have engaged with the page, apparently not
having any Cylc UI related feedback to note at the time.)
