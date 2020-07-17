# Cylc VC 4/5 April 2019 Summary Notes

[Agenda](vc-4-apr-2019-agenda.md)

(No new issues reported at sites.)

**ACTION: aim to release cylc-7.8.2 as soon as possible, maybe Friday 12 April**

(carried over: cylc-7.8.3 needed by May for MO PS43, hopefully with fix for the
"stuck in ready state" problem)

**ACTION: make a new milestone for 7.8.3, May 1 end date** (Hilary) **DONE**

UI Server: PR w/ Tornado and WebSockets coming soon. Then adding GraphQL should
be straightforward.

As soon as we have GraphQL hooked up, consider creating some static (perhaps
with some limited artificial time evolution) "demo data" - good for standalone
testing (and automated testing too) without the need to hook up to a running
workflow. 


**ACTION: Oliver to write down high-level description of UIS-WS data model update.**

**ACTION: Martin to put WIP code on GitHub and make it available to others** - DONE

Asyncio in WS: not done yet, but not on the critical path (can be done later).
Oliver has concerns about what it means to put the ZMQ server in the same event
loop.

However, asyncio has a *priority queue* so we should be OK.

**ACTION: Matt and Tim to look at this in due course.** (No hurry)

Rose: no work done lately, due to other commitments.  Strategy: 1) better cluster
awareness and global and suite config rationalization; then 2) "rose suite-run"
migration suite installation; and 3) cylc run/restart CLI rationalization.

New name for server program and repository: "cylc-flow", `cylc-flow.rc` (Sadie)

**ACTION: PR Template needed** including description for change log. - DONE

**ACTION: Oliver to put up a PR to do the test battery (minimally at first)
with pytest.**

Workflow definition: We decided not to consider supporting YAML any time soon.
We will have to support parsec/suite.rc for the foreseeable future, and the
future Python API will likely make YAML + templating unnecessary. 

**Next Meeting?** - UK team have limited availability throughout April, and Hilary
is away week of 15-19 April. Let's consider the need for a meeting toward the
end of the month. 
