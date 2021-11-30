---
name: release: <version>
about: Steps to check off, to make a new release.
title: ''
labels: 'release'
assignees: ''

---

### Release Progress

Issue to track the coordinated release of multiple Cylc components.

**Required for all minor releases of cylc-flow.**

Se [the release docs](https://github.com/cylc/cylc-admin/blob/master/docs/howto/create-a-release.md)
for first time instructions and more info.

#### Prep:

* [ ] Ensure all milestones complete.
* [ ] Test cylc-doc (run a test build, perform any required fixes).
* [ ] Delete entries below for packages you are not releasing.

#### PyPi / GitHub releases:

> Ensure all Cylc components are pinned to the correct version of cylc-flow.

> Trigger releases via GitHub actions.

* [ ] metomi-isodatetime
* [ ] cylc-flow (bump metomi-isodatetime if required)
* [ ] cylc-ui
* [ ] cylc-uiserver (update the ui version before releasing)
* [ ] metomi-rose (bump metomi-isodatetime if required)
* [ ] cylc-rose

#### Forge (check dependencies match):

> Pull requests will be automatically opened on the conda-forge feedstocks
> after the pypi releases.
>
> If not, create a new branch, change the version, reset the build number and
> update the hash from the PyPi website.
> Finally trigger a rerender in a comment.

> Ensure dependencies are up to date and follow instructions on the PR. Some
> repos may maintain a list of conda dependencies locally.

* [ ] metomi-isodatetime
* [ ] cylc-flow
* [ ] cylc-ui
* [ ] cylc-uiserver
* [ ] metomi-rose
* [ ] cylc-rose

> It make take a couple of hours for a release to become available.
> Use `conda search <package>` to determine when it's ready.

#### Misc (after the above has been completed):

* cylc-doc
  * [ ] bump instersphinx versions if required
  * [ ] review installation instructions
  * [ ] deploy (run the "deploy" workflow on GitHub Actions) (can be re-deployed later if necessary)
* [ ] discourse post

#### Metadata:

> Update project versions to the next milestone
> AND pin downstream components to the next cylc-flow dev release.

* [ ] cylc-flow
* [ ] cylc-uiserver (pin to latest cylc-flow)
* [ ] cylc-rose (pin to latest cylc-flow)

#### Finally:

* [ ] close this issue :rocket:
