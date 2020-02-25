 

## Hub remote spawning of UI Servers
 

The Cylc Hub is an unmodified Jupyter Hub. For those who don't know, Jupyter
Hub runs with elevated privileged (eg root or as a user using sudo access -
https://github.com/jupyterhub/jupyterhub/wiki/Using-sudo-to-run-JupyterHub-without-root-privileges).
Jupyter Hub runs Jupyter notebook servers as real system users on a shared
system, and thus needs to be able to spawn processes as those other users.
Jupyter Notebooks exist to allow system users to write (and share) arbitrary
python into a web interface and execute it in a sandbox on the server hosting
the notebook. A lot of work has been done to ensure that the code users write
is self-contained and unable to break out of its sandbox, and that the Hub is
as unassailable as possible.

In the default Cylc-8 architecture, the Hub runs as a privileged user and
spawns cylc UI Servers when required in the same way as Notebooks are spawned.
Like Jupyter Notebooks, the UI Servers will run as real system users and
although they don't exist to enable the composition of code, they enable code
to be edited and executed as the real system user (who the UI Server is running
as). The UI Server doesn't grant the system user any additional access to the
system than it already has.^

UI Servers should time out and die if no one is looking at them, and be
respawned upon interaction. (This may require keep-alives being sent if a
workflow is going to be kept open for a very long time in an inactive web
browser tab/on a big screen with no interaction).

The primary focus of this discussion was that the Bureau has some concerns
about running Jupyter Hub as a privileged user. As per the link above it's
possible to run with elevated, but not full privileges, but it is also possible
(although not ideal) to run one UI Server per user. (This may remove access to
gscan-like functionality*). We are likely to be breaking our own ground in this
area.

## Authorisation

^ - One of the key requirements of the UI Servers is to allow others users (who
have appropriate authorisation) to act upon work flows owned by another user
via acting on their UI Server.

E.g. I (jacintar) authenticate to the Hub, and want to interact with the
atmosphere_stage user's suites. The atmosphere_stage's UI Server has timed out
and died. My desire to interact it causes the Hub to spawn a UI Server owned by
the atmosphere_stage user. Depending on my specific authorisations, I can then
interact with the atmosphere_stage user's work flows (starting, stopping,
restarting, pausing and other interactions). All *actions* are carried out by
the atmosphere_stage user, but I (jacintar) am causing them to happen. My
actions are recorded eg "timestamp User jacintar stopped
atmosphere_stage.suitename.task.cycle_point".

Authorisation needs site, user and workflow config, with permissions
over-ridden in that order. For example the site config might say that everyone
has read and stop-start on all workflows by default. However a particular user
might be more sensitive, and thus they might only allow read-only to a
particular role and stop-start to no one but themselves and further this user
may have a particular workflow that is so sensitive that only the owner can
read it.

Roles could be defined by unix groups (which is what we do at the Bureau) or by
LDAP groups.

Actions were nominally broken into three concepts.

- Read – see logs, look at the workflow, read the code, read the suite.rc (or
whatever it gets renamed to), etc.

- Execute – stop, start, hold tasks and workflows.

- Write – edit runs/edit triggers, changes to the suite.rc etc.


## Raising privileges to do more stuff

I proposed that authentication tokens (for non-development) should expire after
a short period of time which was configurable. The others suggested that this
wouldn't work for operators who have workflow views open on big screens as
something to keep an eye on, or in several tabs. I suggested that an
alternative would be to have (subject to authorisation for any given user) user
access be read-only by default. In order to interact with the suite (stop,
start or hold tasks and workflows, perform edit-triggers) the user would have
to turn on higher authorisations (by a toggle or similar) that would invalidate
their current WSS session causing them to need to re-authenticate. This would
then give them elevated privileges for a fixed time (perhaps 15 minutes) after
which the session would silently down-grade itself back to read-only. This
behaviour is the same as using sudo on a unix system or temporarily increasing
privileges on OSX and Windows.

E.g. I (jacintar) authenticate to the Hub, and want to interact with the atmosphere_stage user's suites. The atmosphere_stage's UI Server has timed out and died. My desire to interact it causes the Hub to spawn a UI Server owned by the atmosphere_stage user. I am authorized to read, and execute the clouds and winds workflows. So I see both the clouds and winds workflows in the UI Server. I can watch them cycle, I can see their logs and outputs and their code. I want to stop the clouds.climb task. I toggle on execute permissions. My session is invalided and I have to re-authenticate. I re-authenticate and I now have execute permissions on both the clouds and winds workflows (as well as on any other user's workflows I have the appropriate authorisation for). I stop the clouds.climb task. My action is recorded (see possible example below). I want to perform an edit trigger on the clouds.climb task to change what it will do before I restart it. I don't have authorisation to perform an edit-trigger so the option isn't even available to me. I restart the clouds.climb task.

- timestamp User jacintar raised privileges to execute
- timestamp User jacintar stopped atmosphere_stage.clouds.climb.1234 task
- timestamp User jacintar started atmosphere_stage.clouds.climb.1234 task

See also https://github.com/cylc/cylc-uiserver/issues/121

## (Back-end) authentication between workflows and their jobs

Currently the workflow host generates keys for the job and sends the job those
keys including its private key to the execution host it is about to run a job
on. The suggestion is to have the workflow host generate its own public and
private key, and send the job with a copy of the workflow's public key, then
the job creates its own public and private key and returns its job public key
back to the workflow.

We discussed possible MITM and other attacks. We agreed that the main goal is
that the goal is to not allow the job host any extra capacity to affect the
workflow host than it would have if cylc was not being used. We can reduce this
need by
a. keeping workflow's keys in-memory and never writing them to disk
b. only allowing certain message types from jobs

c. maintaining the ssh connection (used to copy and deploy the job to the
remote host) until the job's keys have been generated and returned.

There remains a chance that a malicious job could overwrite the keys between
generation and return.... however that would require the malicious job to be
running with sufficient privileges that it could overwrite the behaviour of the
job at any other time, and still sign things with the agreed public keys, so
there's no change in risk from this small window of attack versus the general
timeframe of the running of that job. 

Cylc-8 will be CurveZMQ for the messaging between jobs and their workflows.
CurveMQ prevents replay-attacks and various other combinations of network
capture and private key theft.

## Cylc gscan

Cylc scan and gscan haven't been solved yet. In cylc 7 these tools allow you to
find out the statuses of all the currently executing workflows for your current
host or all the hosts you've specified. This is performed by a port scan, but
cylc-8 has a different set up. Cylc scan was a command line tool and gscan was
a GUI equivalent.

## Given UI Servers may not be running when they're not in use, and given that
identifying stopped workflows is also not solved, obtaining the date for cylc-8
is more tricky. Having the hub have access to all the relevant information may
solve this. Then (for gscan at least) a distinct UI can be crafted to display
this information eg:

```
Atmosphere_stage

    Clouds  < status >
    Winds   < status >

Oceans_stage

    Waves < status >
    Currents < status >
```

* - This would require the same Hub to be used for multiple users, which violates the Bureau's desire to have unprivileged hubs.


## Cylc-7 penetration test and Cylc 8 thread modelling

See https://cylc.github.io/cylc-admin/threats.html

I'm also writing a document covering the security model in more detail which
I'll be sending through soon.

Cylc Review – This tool (previously called rose bush) takes cylc job, error and
output logs (as well as some other files) and serves them as HTML.

At this point, the goal is to serve historical job logs via the cylc UI just as
it will serve current job logs.

Scrolling back to a recently finished cycle point is easy, but I requested
search functionality as well because scrolling back 3 or more months for a
frequently cycling suite could be very unpleasant. I'm not sure this particular
point was resolved.  There was discussion about resurrecting cylc review
functionality (it has currently been deleted from the repo) if search
functionality is not provided.

## Load balancing of UI Servers

We discussed that the ssh-spawner (used by the Hub to spawn the UI Servers)
needs to handle load balancing for the situation where the UI Servers are not
all running on the same server. There was a proposal to use the existing load
balancing already written for cylc-flow and integrated it in with the spawners.
A suggestion was made to make this a plugin so that others can also write their
own remote spawner plugins.

## Command Line Interface (CLI) versus UI Servers

See https://cylc.github.io/cylc-admin/proposal-cli-wfs-authentication.html and
Hilary's day 1 notes.

Cylc scan (and related, cylc gscan) need to know which work flows are running.
They're going to leverage the UI Servers for this (but what if the UI Servers
are inactive, and what about UI Servers on other hosts).   Somewhat of an
unfinished problem.

## Spawn on Demand

Currently in cylc, a task spawns its own next-cycle successor when it is
submitted. This means that both the task and its successor can run in parallel
if dependencies are met, but you don't get infinite many copies of those tasks.
However this means that your workflow engine needs to have a pool containing at
least one object per task. So if a suite has 1000 tasks, the workflow engine
needs to have a minimum of 1000 objects. This also means that tasks cannot run
out of order, which is bad if a task can run if ONE of its dependencies are
met. Having tasks create their next task on submit means that tasks could have
their dependencies met BUT not appear in the task pool because the previous
task hasn't yet been submitted.

This makes the successful submission of a previous task be a secret dependency
of a task.

The new solution is spawn on demand. So a task essentially notifies tasks
dependent on it that it's complete. This way there is no "task pool" but
instead only a collection of the currently running tasks and their children.
The graph view then only needs to show this subset. (The full graph view will
still exist, but not necessarily as a running view given large suites)

Spawn on demand is likely to reduce the number of objects held in memory by up
to 90%, given their profiles of large MetOffice suites only having 10% of tasks
actually active versus the rest waiting. This will improve runtime and reduce
memory usage AND improve the performance of the GUI

But there are lots of things to still be sorted. Related:
https://cylc.github.io/cylc-admin/proposal-spawn-on-d.html
