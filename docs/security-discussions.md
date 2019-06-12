# Security Discussions

This document is used for discussion around security in Cylc repositories.

## CVE's in Cylc

At the moment we are working to add a `SECURITY.md` file to `cylc/cylc-flow`.
This document includes information on how Cylc manages security issues, and
how users can report it safely. Other repositories can link to that document.

However, in case of a security bug, the only way of knowing if a version of
our projects is safe or not, is via its change log.

There are global CVE databases, maintained by organizations that work
to collect these issues and make them available to interested parties via
a simple web interface, with a search form.

This way security professionals are able to quickly assess if a version of
some software has ever had a security issue, and whether that version can
be used in his/her environment safely.

Perhaps later we could also get one or two developers familiarised with
the process to request CVE's, and start filling (maybe even retroactively)
requests whenever we find security issues in Cylc projects.

Right now there are no CVE's for Cylc in these databases, even though we have
old versions that had reports of security issues. Which means users are not able
to check which versions are safe to use, or whether they need to update their
environments due to a security incident.

_References:_

- https://cve.mitre.org/cve/request_id.html
- https://github.com/RedHatProductSecurity/CVE-HOWTO
- https://thoughtbot.com/blog/handling-security-issues-in-open-source-projects
- https://www.csoonline.com/article/3157377/application-development/open-source-software-security-challenges-persist.html
