# Creating a release for Cylc

**Note:** The release process has been substantially automated. For reference,
the old release process is documented at [create-a-release-old.md](create-a-release-old.md).

## Before you start (First time)

- Create a PyPI account for yourself if you don't already have one.
  [Sign up page](https://pypi.org/account/register/).
  Get an owner to add you as a maintainer or owner.
- Go to the [repository secrets](https://github.com/cylc/cylc-flow/settings/secrets)
  (Settings tab of the repo on GitHub and choose "Secrets" on the left).
  Check for the presence of the `PYPI_TOKEN` secret.
  - If it doesn't exist,
    [create an API token](https://pypi.org/help/#apitoken), **making sure to
    limit the scope to the project**. Copy the token, including the `pypi-`
    prefix, and add it to the repo as a secret called `PYPI_TOKEN`. (Don't
    worry about the PyPI instructions for using the token.)

## Test the docs

For any projects which are auto-documented by cylc-doc, currently:

* cylc-flow

Ensure the docs build against master by manually triggering the test workflow
in cylc-doc.

This will catch syntax errors, broken URLs etc which need to be fixed
prior to releasing the project.

## Stage 1: trigger the GitHub Actions workflow

- Go to the [Actions tab](https://github.com/cylc/cylc-flow/actions) of the
  repo on GitHub and choose the "Release stage 1 - create release PR" workflow.
- Click the "Run workflow" dropdown and enter the version number in the input
  field (also the base branch to open the PR against, defaults to master).

  ![screenshot](/docs/img/automated_release.png)

- The workflow will automatically create a pull request with a change to the
  version number in `__init__.py`. It will also give a checklist to follow on
  the pull request.

## Stage 2: merge the release PR

- After completing the checklist on the pull request, and having had the
  pull_request approved and merged, a second GitHub Actions workflow will
  publish the release to PyPI.org and publish a GitHub release.
- It will then comment on the pull request with a link to edit the description
  of the GitHub release to add any information.

### Notes

It is also possible to manually create a release PR with the conditions:
- The branch name must be of the format `prepare-<version_number>`, e.g.
  `prepare-1.0.1` or `prepare-5.0a2`.
- Add the `release` label to the PR. **Warning:** any PR you create with the
  `release` label will trigger publishing to PyPI when merged.

If anything goes wrong, you can also do the whole process manually by
following the [old instructions](create-a-release-old.md).

## Stage 3: release on Conda

> **Note:** The GitHub and PYPI releases are prerequisites for releasing to
> Conda Forge!

> **Note:** The Conda Forge release process is mostly automated by bots. These
> bots are set up during the initial project bootstrap, which has been done for
> all the projects that are part of Cylc 8.

After a Python project has been pushed to PyPi a new PR should be
automatically created on the conda-forge feedstock.

- Follow the instructions on the PR.
- Check the dependencies are upto date, some projects e.g. cylc-flow have
  a Conda environment file in the reop.
- Once approved merge the PR to make the release.

As there are inter-dependencies amongst the Cylc 8 parts, you should
know the dependency tree, and start by the leaf nodes (i.e. a module
A may have multiple dependants, but no dependency to other modules).

For example, Cylc UI Server depends on Cylc Flow. So unless you are
releasing only Cylc UI Server, you should release Cylc Flow first.

### Prepare the Conda Releases

On GitHub, navigate to the project repository on GitHub, e.g.

- https://github.com/conda-forge/cylc-feedstock
- https://github.com/conda-forge/cylc-flow-feedstock
- https://github.com/conda-forge/cylc-rose-feedstock
- https://github.com/conda-forge/cylc-ui-feedstock
- https://github.com/conda-forge/cylc-uiserver-feedstock
- https://github.com/conda-forge/metomi-isodatetime-feedstock
- https://github.com/conda-forge/metomi-rose-feedstock

On each of these repositories, the release process should be pretty
much the same. Your work will be mainly (if not exclusive) on
the `recipe/meta.yaml` file.

This is a possible order of action:

- Check open pull requests—There are bots that alert about dependencies
(N.B. conda packages do not use PYPI for dependencies, they all use
other conda packages as dependencies!.
- Check if you need to update the version in the Jinja2 variable right
at the top of the file.
- If you updated the version, set the build number back to 0.
- If you did not update the version, increase the build number.
- If the conda package contains a source URL, check that it is evaluated
correctly by manually replacing the Jinja variables, and trying this out
in a browser. Then update the `sha256` value (you can get this value
from PYPI, or use some tool like `sha256sum`).
- Check requirements (we may have changed setup.py, or package.json, etc.
ensure we are up to date).
- Ensure the test commands are still valid (i.e. they can be executed
successfully)
- Check if you need to change summary, description, or any other field.
- Create a branch in your clone, commit, and push to your fork. Wait
for the feedback of the Conda Forge CI bots.
- In the meantime, test the package locally (see below).
- If the bots gave you the OK, and testing locally worked fine too,
your pull request should be ready for review.
- If you changed something in the recipe, and you are not sure if
that will affect the Conda Forge package, you can ask Conda Forge
maintainers help reviewing the pull request (via Gitter).
- Otherwise, get others from Cylc Core team to review & approve. Once
your pull request has been approved and merged, the bots will start the
release process.

From past experience, I've seen Conda packages available in Conda Forge
in under 30 minutes, but have also had to wait hours (<8 hours) for it
to show in Conda Forge. For example, if Azure or GitHub infrastructure
have issues (as I experienced during the first releases) it may take days.
So keep that in mind before announcing the Conda packages.

### Testing the Packages

To test packages locally, first you should make sure that Conda is
configured to avoid automatically upload the package. Open your `~/.condarc`
and check that you have something similar to:

```yaml

   channels:
     - defaults
     - conda-forge
   ssl_verify: true
   anaconda_upload: false
```

Now create a Conda environment for your tests, e.g.: `conda create -n cylc1`,
and then activate it `conda activate cylc1`. Then to build and install
locally:

```bash

   # Where $CONDA_FORGE_REPOSITORY could be, for example,
   # cylc-uiserver-feedstock.
   cd $CONDA_FORGE_REPOSITORY
   # Your package should not be listed!
   conda list
   # This will take some minutes and print useful information.
   conda build recipe/
   # The following command will install the locally created package. Before
   # installing it will ask you to confirm. Scroll up and search the
   # package name. The right-side column must show a location like
   # .../anaconda3/conda-bld/linux-64::cylc-uiserver-0.1-py37_1.
   # This confirms you are installing the local build. Here $PACKAGE_NAME
   # could be something like cylc-uiserver.
   conda install --use-local $PACKAGE_NAME
```

At this point you should be good to go. Test the package with commands
such as `which $PACKAGE_NAME`, or `$PACKAGE_NAME --version`, etc.
Of if you are testing the metapackage, try running the complete system
with a workflow and the UI or tui, and check if there is nothing wrong.

### Troubleshooting Conda Forge issues

#### Issues building or installing the package locally with `conda-build` and `conda install`

If you found problems while testing locally, try troubleshooting locally,
and either mark the pull request as draft, and close it. Merging the pull
request will create a release.

#### Marking a release as invalid or broken

To undo a release, you will have to liaise with the maintainers via Gitter
or Github. Or, alternatively, bump up the build number +1, and release it
again.

#### `conda-install` is picking the wrong version or build of a package

If `conda install cylc` returns the incorrect build number, it could be
an issue that is very difficult to debug (a posteriori). In fact it may not even be a bug:

Question to conda Gitter channel: (@hjoliver Jul 21 17:23):
> I have a conda metapackage just published to conda-forge, which (on conda install) does not install the latest build numbers of several of our component packages. The conda docs cite bug fixes as a primary use case for incrementing build numbers, in which case I would have thought the environment solver should never select older builds? If I install the component packages directly, I do get the latest builds, but not via the metapackage. Anyone know what's going on here?

Replies:
> @wolfv:
@hjoliver this article describes how the conda solver works: https://www.anaconda.com/blog/understanding-and-improving-condas-performance. It specifically says that build numbers are only maximized for explicit specs

> @jjhelmus:
@hjoliver changes to the build number can also occur when a package is built against a different version of a library which is incompatible with earlier version. For example, one build of python 3.7.7 might be compiled against libffi 3.2 while the next build number against libffi 3.3. Depending on what other packages are installed in the environment it may not be possible to install the package with the largest built number. In the example if cffi is only available as a package built against libffi 3.2 then the older build of python 3.7.7 would be required.

Interpretation: build number is treated much like version number, if not pinned down exactly conda can choose an older build if that works best for solving all the dependencies in the system. Conda would presumably(?) choose the newest build if it does not affect dependencies; we may just need to check what happens if dependencies get changed by a new build.

If you are sure that you have your new version or build available from
Anaconda.org, and that `conda install --dry-run $package` should
work, then you could have hit a bug in either `conda` client or in
Conda Forge repository data.

You can try the following:

- Debug the issue increasing the logging of the `conda install` command
- Check with the Conda devs in their [Gitter channel](https://gitter.im/conda/conda)
- Publish new builds of your artefacts (just increase build number in a PR)
- Finally, if none of the previous worked, you can set the build string in
the Cylc metapackage

Conda documentation has details about their [package math specification](https://docs.conda.io/projects/conda-build/en/latest/resources/package-spec.html#package-match-specifications).
Basically, there are three parts, where the last one is optional.

- A match specification is a space-separated string of 1, 2, or 3 parts:
- The first part is always the exact name of the package.
- The second part refers to the version and may contain special characters (BUT... further down)
    * When there are 3 parts, the second part must be the exact version.
- The third part is always the exact build string.

So you will have to specify the exact version, and use a glob expression
for the build number. For example, from our alpha2 release:

```yaml
run:
  - python
  - cylc-flow 8.0a2 *_3
  - cylc-uiserver 0.2 *_2
```

In the example above, we are specifying the packages and versions we want.
Plus, we also give it a pattern to the build string, so conda will match
`cylc-flow-8.0a2-py37_2`, or `cylc-flow-8.0a2-py38_2`, or
`cylc-flow-8.0a2-py37h2387c_2` (all valid build strings).

## Announce on Element and Discourse?

Bask in the glory of having created a release.
