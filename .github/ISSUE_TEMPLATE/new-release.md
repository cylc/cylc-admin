---
name: 'release: <version>'
about: Steps to check off to make a new minor release.
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
* [ ] List the milestones for release below (delete entries as appropriate).
* [ ] All bugfix branches should be merged into master.
* [ ] Ensure all milestones complete.
* [ ] Ensure major changes are listed in cylc-doc (`reference/changes`).

#### Testing:

> Some testing is not fully automated and must be actioned by hand. Make sure
> the tests for downstream components have been run against the latest
> version of upstream repositories.

* [ ] Run cylc-flow functional tests against locally available platforms.
* [ ] cylc-doc (run a [test build](https://github.com/cylc/cylc-doc/actions/workflows/test.yml)) - (link workflow run).
* [ ] cylc-rose (run the ["tests" action](https://github.com/cylc/cylc-rose/actions/workflows/tests.yml)) - (link workflow run).
* [ ] cylc-uiserver (run the ["test" action](https://github.com/cylc/cylc-uiserver/actions/workflows/test.yml)) - (link workflow run).

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
>    metomi_isodatetime => cylc_flow & metomi_rose => cylc_rose
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
>   <br />Open a new issue on the feedstock repository, select the
>   "bot command" issue type and set the title to
>   `@conda-forge-admin, please update version`.
> </details>

> Ensure dependencies are up to date by running:
> ```
> $ git diff <previous-release> <new-release> -- setup.cfg setup.py pyproject.toml conda-environment.yml
> ```
> And checking these changes against the `recipe/meta.yaml` file.
>
> <details>
>   <summary>Info on outputs:</summary>
>   <br />Some repositories have multiple "outputs", e.g. `cylc-flow-base`
>   is a cut-down release (or output in Conda speak), whereas `cylc-flow` is
>   the full release including optional dependnecies. You will need to check
>   the dependencies in all of the places they appear, including each of the
>   outputs.
> </details>

> If you need to make changes, remember to re-render the feedstock
> by commenting `@conda-forge-admin, please rerender` on the PR.

* [ ] metomi-isodatetime
* [ ] cylc-flow
* [ ] cylc-uiserver
* [ ] metomi-rose
* [ ] cylc-rose

> It make take a couple of hours for a release to become available.
> Use `conda search <package>` to determine when it's ready.

#### Misc (after the above has been completed):

* metomi-rose
  * [ ] build & deploy documentation (manual process ATM)
* cylc-doc
  * [ ] bump intersphinx versions if required (`cylc-doc/src/conf.py`)
  * [ ] review [deployment instructions](https://github.com/cylc/cylc-doc#deploying)
  * [ ] deploy (run the "deploy" workflow on GitHub Actions) (can be re-deployed later if necessary)
* Discourse
  * [ ] announce the release
  * [ ] scan through the [major changes page](https://cylc.github.io/cylc-doc/stable/html/reference/changes.html)
    and create "tip" posts (linking back to the changes page) to announce any new features.

#### Metadata:

GH Actions should automatically open PRs that bump the dev version of the
projects. Check and merge them (can push alterations to PR branch if needed).

Downstream components will need to have their dependencies bumped:

* [ ] cylc-uisever (pin to next minor cylc-flow version)
* [ ] cylc-rose (pin to next minor cylc-flow and metomi-rose versions)
* [ ] cylc-admin (update the [meta-release config](https://github.com/cylc/cylc-admin/blob/master/docs/status/branches.json))

#### Finally:

* [ ] close this issue :rocket:
