# Proposal for Workflow Authentication

## Proposal summary

**For workflow-to-workflow-service authentication, use:**

* **for jobs: one-time per-job tokens**, i.e. a single one-time token for
  every job;
* **for the user/interactive CLI: timed tokens**.

Per-job tokens would require one of the following (or another alternative):

  * another file per job, which is not ideal as it adds to the file count;
  * a shared combined file containing tokens for all jobs per workflow, also
    not ideal as it means that jobs would needlessly know about other jobs'
    tokens;
  * extending an existing job file such as the 'job.status' so that these files
    also carry the token information for each job, & preventing that
    information from being displayed in Cylc Review.

Decisions on implementation details, such as modules & standards to use,
await the go-ahead on an approach, the above one or otherwise.


## Further information

### Context

Cylc 8 discussions have converged (as described briefly
[here](cylc-8-architecture#command-line-interface) &
[here](cylc-8-tasks#general-authentication-issues-sujata-hilary--damian-and-martin-)
on usage of tokens for authentication between any workflow & the workflow
service. Both of the following token approaches have been raised as potential
solutions:

* "one-time" tokens: single-use tokens i.e. ones that are created then deleted
  once they are no longer required, with fine (process) granularity for
  creation/deletion;
* "timed" tokens: tokens that expire after a set time period.

Due e.g. to the new architecture, job clients must be distinguished & treated
differently from user (UI & interactive CLI) clients, unlike for the
[simple passphrase-based approach of Cylc 7](cylc-7-architecture#authentication).

We also want to make sure the new approach offers increased security, rather
than just being a "rebrand" accounting for the above, to the Cylc 7 approach.


### Difficulties & complications

We need to (sooner or later) consider the following points:

* Jobs can run where there is a non-shared filesystem;
* We should consider the longer-term future, where jobs are likely to run on
  cloud platforms;
* Later work on authorisation may rely on a mapping against tokens, so
  we should not preclude that;
* There will be an overlap between the new one-time & timed token approach &
  the old passphrase approach that needs to be managed.


### An alternative to consider

Another idea that was raised was to instead have one token *per host* for
job clients. Different platforms have different security levels & a
per-platform approach could benefit from that.

We could do some preliminary investigation to understand whether having
one token *per job* or one *per host* would be better for job clients?
