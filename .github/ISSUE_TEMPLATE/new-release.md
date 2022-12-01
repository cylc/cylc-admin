---
name: 'release: <version>'
about: Steps to check off to make a new release.
title: ''
labels: ['release']
assignees: ''

---

### Release Progress

Issue to track the coordinated release of multiple Cylc components.

**Required for all minor releases of cylc-flow.**

See [the release docs](https://github.com/cylc/cylc-admin/blob/master/docs/howto/create-a-release.md) for first time instructions and more info.

#### Prep:

* [ ] The release lead should be assigned to this issue.
* [ ] Ensure all milestones complete.
* [ ] Ensure major changes are listed in cylc-doc (`reference/changes`).
* [ ] Test cylc-doc (run a test build, perform any required fixes).
* [ ] Run cylc-flow functional tests against locally available platforms.
* [ ] List the milestones for release below (delete entries as appropriate).

#### Milestones for release:

> The release actions close the milestones for you automatically.

<!--
    Replace `<number>` with the milestone for each package to release.
    Delete lines as appropriate.
    (you can get the milestone number from the milestone URL)
-->

- metomi-isodatetime: [![](
  https://img.shields.io/github/milestones/issues-open/metomi/isodatetime/<number>)](
  https://github.com/metomi/isodatetime/milestone/<number>)
- cylc-flow: [![](
  https://img.shields.io/github/milestones/issues-open/cylc/cylc-flow/<number>)](
  https://github.com/cylc/cylc-flow/milestone/<number>)
- cylc-ui: [![](
  https://img.shields.io/github/milestones/issues-open/cylc/cylc-ui/<number>)](
  https://github.com/cylc/cylc-ui/milestone/<number>)
- cylc-uiserver: [![](
  https://img.shields.io/github/milestones/issues-open/cylc/cylc-uiserver/<number>)](
  https://github.com/cylc/cylc-uiserver/milestone/<number>)
- metomi/rose: [![](
  https://img.shields.io/github/milestones/issues-open/metomi/rose/<number>)](
  https://github.com/metomi/rose/milestone/<number>)
- cylc-rose: [![](
  https://img.shields.io/github/milestones/issues-open/cylc/cylc-rose/<number>)](
  https://github.com/cylc/cylc-rose/milestone/<number>)
- cylc-doc: [![](
  https://img.shields.io/github/milestones/issues-open/cylc/cylc-doc/<number>)](
  https://github.com/cylc/cylc-doc/milestone/<number>)

#### PyPi / GitHub releases:

> Ensure all Cylc components are pinned to the correct version of cylc-flow.

> Trigger releases via GitHub actions.
>
> <details>
>   <summary>(logical release order)</summary>
> <pre>R1 = """
>    cylc_flow & metomi_rose => cylc_rose
>    cylc_flow & cylc_ui => cylc_uis
> """</pre>
> </details>

> <details>
>   <summary>Info on version pinning</summary>
>   <br />
>   Cylc plugins (i.e. cylc-rose and cylc-uiserver) are "pinned" to the minor version
>   of cylc-flow. E.G. if the cylc-flow version is 8.1.2 the plugins should be pinned to 8.1.
>   <br /><br />
>   <a href="https://github.com/cylc/cylc-admin/issues/130">More Information</a>
> </details>

* [ ] metomi-isodatetime
* [ ] cylc-flow (bump metomi-isodatetime if required)
* [ ] cylc-ui
* [ ] cylc-uiserver ([update the ui [via GH action](https://github.com/cylc/cylc-uiserver/actions/workflows/update_ui.yml) first)
* [ ] metomi-rose (bump metomi-isodatetime if required)
* [ ] cylc-rose

#### Forge (check dependencies match):

> Pull requests will be automatically opened on the conda-forge feedstocks
> after the pypi releases.
>
> <details>
>   <summary>If the PR doesn't get opened automatically</summary>
>   <br />Create a new branch, change the version, reset the build number and
>   update the hash from the PyPi website.
>   <br />Finally trigger a rerender in a comment.
> </details>

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
  * [ ] bump intersphinx versions if required (`cylc-doc/src/conf.py`)
  * [ ] review [deployment instructions](https://github.com/cylc/cylc-doc#deploying)
  * [ ] deploy (run the "deploy" workflow on GitHub Actions) (can be re-deployed later if necessary)
* metomi-rose
  * [ ] build & deploy documentation (manual process ATM)
* [ ] discourse post

#### Metadata:

GH Actions should automatically open PRs that bump the dev version of the projects. Check and merge them (can push alterations to PR branch if needed).
    
Pin downstream components to the next cylc-flow dev release:
* [ ] cylc-uiserver (pin cylc-flow to next upcoming version)
* [ ] cylc-rose (pin cylc-flow to next upcoming version)
    
Pin components in GH Actions tests:
* [ ] cylc-doc (update versions in `.github/workflows/test.yml` "install libs to document" step(s) as appropriate)
* [ ] metomi-rose (update versions in `.github/actions/install-libs` as appropriate)
    
#### Finally:

* [ ] close this issue :rocket:
