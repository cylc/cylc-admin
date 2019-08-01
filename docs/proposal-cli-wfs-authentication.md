# Proposal for Authentication between the CLI & the Workflow Service

##### Acronyms used:

* CLI == command line interface;
* WFS == workflow service (in Cylc <=7 terminology, the suite daemon or server
  program);
* UIS == UI (user interface) server.


## The Problem

### Introduction

We need authentication between the CLI & the WFS in Cylc 8.


#### Client cases

There are various client cases to cover, ultimately, some of which we will treat
in the same way, & some differently (see the next section for requirements):

1. **User (interactive) CLI** => WFS, for user commands:
    1. what I will call **"the direct, or full-privilege full-access (FPFA),
       case"**:
        * user is logged in (authenticated);
        * user is the workflow owner;
        * user has full access to the file system where the WFS is running;
        * the WFS is running as this user.
    2. the user CLI where any of the conditions in (1i) are not satisfied, as
       I will call **the "non-direct or non-FPFA case"**.
2. **Task-job CLI** <=> WFS, for job status messaging etc.:
    1. remote task-jobs that are **on a non-shared file system**;
    2. **any task-jobs not covered by (2i)**, i.e. local ones, or ones running
       remotely on a host where the file system is shared.
3. **Involving the UIS**:
    1. **UIS <=> WFS**:
        * This should be in scope, since this case should be completely
          equivalent to the 'CLI <=> WFS' case, i.e. to the cases in (1) (see
          Q2 under 'Open Questions')?
    2. **CLI => UIS**:
        * This is out-of-scope.


#### Token-based approaches

Discussions on the topic have converged (see e.g.
[here](cylc-8-architecture#command-line-interface) &
[here](cylc-8-tasks#general-authentication-issues-sujata-hilary--damian-and-martin-)
towards using some form of *non-permanent (transient) token* to achieve this.
These forms have been distinguished as options:

* **"timed" tokens**: tokens that *expire*, & *are replaced* with a different
  token, after a set *time period* is up;
* **event-based/driven "one-time" tokens**: "single-use" tokens that are
  *created* & *deleted* to cover the duration of set *events*, only being
  vaild for one such event instance.


### The Old (Cylc 7) Approach

The approach for authentication between the CLI & the suite server program
(name for the Cylc 7 WFS equivalent) is
[described here](cylc-7-architecture#authentication). It is ultimately
a token written out on-disk in plain text, that relies on Linux file
permissions to be secure & only usable by the suite owner.

Note the phrase "passphrase" is used historically, but it is not actually
a user-specified mnemonic passphrase (though users can override it with one,
but we believe this is done rarely), it is a *random token* that is generated
on the first run of a suite. Since the token ("passphrase") is valid for
the lifetime of that suite, it is *effectively a one-time suite-level token*,
where the creation event is the first suite run & the deletion event is the
deletion of the suite (or its ``.service`` directory).


### New challenges for Cylc 8

There are complications relative to the Cylc 7 approach:

* in Cylc 7, jobs authenticate automatically as the user, but for a web UI
  that is not possible, so task-job clients as previous case (2ii) must be
  explicitly dealt with;
* a [core principle of the new architecture](cylc-8-roadmap.md#core-principles--motivation-for-the-new-architecture)
  is that it will "enable users to view and interact with suites on remote
  platforms without requiring shared file system or SSH access" so case (2i)
  is a new case to accommodate.


### Aims: what do we want?

We want to provide & solution that will, by means of, & on top of, being
functional:

* **address the complications** outlined in the previous section;
* **offer increased security**, rather than just being a "rebrand" accounting
  for the above;
* **not preclude, or make difficult, any future work** that we need to or want
  to do, e.g:
  * later work on authorisation, which may rely on a mapping against tokens;
  * work on out-of-scope WFS authentication i.e. (cases 3ii & possibly 3i);
  * perhaps providing the means to restrict what remote jobs are able to do
    on certain platforms, to account for the fact that certain ones have
    different security levels.
* **not be incompatible with any future infrastructure changes** that we can
  foresee, e.g:
  *  jobs running on cloud platforms.


### Open Questions

#### On client cases (c.f. 'Client cases' section above)

1. What **cases** shall we manage through the **same approach**?
2. What is **in scope** for authentication **involving the UIS**
   (see cases under 3)?


#### On token choice (c.f. 'Token-based approaches' section above)

3. What **token-based approach** should be used in each client case group as
   in Q1?


#### On token management

4. For timed tokens: how do we deal with the **changeover** of tokens? E.g:
    * will both tokens be valid over the changeover period?
5. For one-time tokens: what **events** do we set as those for which one-time
   tokens are created & then deleted?
     * We should consider the *granularity*, e.g. to make sure there will
       not be a performance overhead from too much interaction with the
       filesystem etc., but not so coarse-grained as to compromise security.
       How long-lived should the events be, along the scale of covering the
       duration (longest to shortest, where the extreme cases are clearly not
       good choices!) of a whole:
         * workflow (as in Cylc 7)?
         * cycle-point?
         * task (i.e. for all of its task-jobs)?
         * job;
         * set of defined (workflow-based) commands?
         * (single) command?


#### On the nature & location of the tokens

6. What **algorithms &/or standards** to use (see also Q9)? Notably for:
    * tokenisation (signing, verification, etc.);
    * encryption, if used;
7. How shall we provide any **related information required to identify e.g.
   jobs**:
    * incorporated it into the token?
    * just have it alongside the token (exposing it, but does that matter)?
8. Which **location** shall we use **to store** the token(s)?
    * Somewhere under each workflow's ``.service`` as before?


#### On implementation

9. What **module(s)** to use (see also Q6). Should it/they be:
    * Python *built-in*? Notably e.g:
        * [``hashlib``](https://docs.python.org/3/library/hashlib.html): "a
          common interface to many different secure hash and message digest
          algorithms";
        * [``secrets``](https://docs.python.org/3/library/secrets.html): "used
          for generating cryptographically strong random numbers".
    * *Third-party*? Notably e.g:
        * [``pyjwt``](https://github.com/jpadilla/pyjwt): JSON Web Tokens
          implemented in Python;
        * [``PyOTP``](https://github.com/pyauth/pyotp): a Python library for
          generating and verifying one-time passwords.


## Working Proposal for a Solution

This is the **current status of the plan** to address defined aspects (else
stated as 'out of scope' meaning to be addressed in the future, after the
rest) for the problem outlined in 'The Problem' section above.


### Timeline of major updates to the working proposal

* 25.07.19: initial PR up, with a very basic proposal.
* 28.07.19: proposal extended in response to PR feedback from MS, HO & BK.
* 29.07.19: extended again to include UML Sequence diagrams created by MS.
* 01.08.19: proposal updated in response to feedback from DM, & reformatted
  to accommodate much higher level of detail than was originally intended.


### Case-by-case Outline

**For command-to-workflow-service (CLI-to-WFS) authentication, use:**

(The numbers below refer to the cases outlined in the 'Client cases' section
above, so please cross-reference with that.)

* (1i) **Direct timed tokens**.
* (1ii) Out of scope (in the future this may be supported to go through the
  UIS).
* (2i) **Event-based one-time tokens, which each last for the duration of
  single job only (call them "one-job" tokens)**. They would expire when the
  job finishes (note this means they would apply to multiple commands). These
  would require one of the following:
    * another file per job, which is not ideal as it adds to the file count;
    * a shared combined file containing tokens for all jobs per workflow;
    * extending an existing job file such as the 'job.status' so that such
      files also carry the token information for each job, & preventing that
      information from being displayed in Cylc Review.
* (2ii) **Direct timed tokens, the same ones from (1i)** which will be
  accessible via the local file system.
* (3i) Equivalent to cases under (1i), hence **direct timed tokens**.
* (3ii) Out of scope (see note for 1ii).


### Case-by-case UML Sequence Diagrams

<!---
These diagrams are generated using the tool at https://sequencediagram.org/

The code to generate these directly from the tool are included in comments
below, so anyone that would like to make edits can do so easily.
-->

#### Key for interaction arrow colours:

* Black: *initiating* interaction
* Red: *token-related* interaction or process
* Green: *en- or de-cryption* process
* Blue: generic message or communication
* Purple: *batch scheduling* interaction or process


### Direct timed tokens for cases (1i) & (2ii)

This diagram outlines interactions for the *creation, changeover & deletion*
of valid timed tokens:

<!---
DIAGRAM CODE HERE
-->

[DIAGRAM TO BE ADDED SHORTLY]

This diagram outlines interactions for the *usage* of the valid timed tokens
(which are created, replaced & deleted as in the diagram above):

<!---
title Authentication: CLI => WFS: a) Token Usage

actor User
participantgroup #lightblue **WFS Host**
entity Local job
participant CLI
alt case 1
User->CLI: workflow command
else case 2
Local job->CLI: command
end
database File System
participant WFS
activate CLI
CLI-#red>File System: put token
activate File System
CLI-#green>CLI: encrypt message
CLI-#blue>WFS: send encrypted message
activate WFS
WFS-#red>File System: get token
File System--#red>WFS: token
WFS-#green>WFS: decrypt message
WFS-#green>WFS: encrypt reply
WFS--#blue>CLI: reply
deactivate WFS
CLI-#green>CLI: decrypt reply
deactivate File System
deactivate CLI
destroyafter CLI
end
-->

![Authentication: CLI => WFS](img/wfs_auth_local_usage.png)


### "One-job" (one-time over the lifetime of a single job) tokens for case (2i)

This diagram outlines the interactions for *both* the *creation & deletion*, &
for the *usage*, of valid one-job tokens:

<!---
title Authentication: WFS <=> Remote Job

participantgroup #lightblue **WFS Host**
participant WFS
end
participantgroup #lightyellow **Remote Job Host**
participant Job Submit
database File System
participant Batch System
entity Remote Job
end
WFS-#blue>Job Submit: SSH
activate WFS
activate Job Submit
Job Submit-#red>Job Submit: create token
Job Submit-#red>File System: put token
activate File System
File System--#blue>Job Submit: write OK
Job Submit-#purple>Batch System: submit
activate Batch System
Batch System--#purple>Job Submit: return
Job Submit--#red>WFS: return with token
deactivate Job Submit
destroyafter Job Submit
Batch System-#purple>Remote Job: start job
deactivate Batch System
activate Remote Job
Remote Job-#red>File System: get token
File System--#blue>Remote Job: read OK
Remote Job-#blue>WFS: message started/completed
deactivate WFS
Remote Job-#red>File System: delete token
deactivate File System
deactivate Remote Job
destroyafter Remote Job
-->

![Authentication: WFS <=> Remote Job](img/wfs_auth_remote.png)
