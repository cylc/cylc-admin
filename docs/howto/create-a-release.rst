###########################
Creating a release for Cylc
###########################


Before you start (First time)
=============================
* Create a GPG key for verification on Github: `Instructions <https://help.github.com/en/articles/generating-a-new-gpg-key>`_
  At step 3 I had to use ``gpg --default-key rsa4096 --gen-key``.

* `Share this key locally with git, as well as Github: <https://help.github.com/en/articles/telling-git-about-your-signing-key>`_

* Create a PyPI account for yourself if you don't already have one.
  `Sign up page <https://pypi.org/account/register/>`_
  Get an owner to add you as a maintainer or owner.


Create pre-release Pull requests
================================

* Ensure that version numbers are consistent. These are likely to be set in
  files such as  ``cylc-flow/cylc/flow/__init__`` or ``__version__.py`` and
  ``setup.py`` files, usually at the top level of a repository.
* Examine Pull Requests made since the last release for:
  * cylc-flow
  * cylc-conda
  * cylc-ui
  * cylc-uiserver
* Add to ``CHANGES.md`` text in each repository.
* Check the ``CONTRIBUTING.md``. Ensure that all contributors are listed.
  You can do this using Github, or locally with ``git shortlog -sn``
* Create PRs for any changes you have made and have these reviewed and merged
  to master.
* Make sure any other Pending PR's for the milestone are merged or closed.

Test the build
==============

* Check locally that you have the updated version of master. Ensure that you
  local copy is up to date, and that it has the correct tags attached.
* Use ``git log`` to ensure that you have the correct version and tag.
* Check that you can buid your distribution locally:

.. code-block:: bash

   cd my/repository/

   # This may seem slow as this will build the wheel as well as the source
   # distributions
   python3 setup.py bdist_wheel sdist

   # check that you've successfully created wheel and source files
   ls dist/

   # Tidy up
   rm -r dist


Tag the version
===============

.. code-block:: bash

    # Examine previous tags using
    git tag -ln

    # Create a new tag  "tag_name" is usually a version number.
    git tag -a -s <tag_name>

    # Push tags:
    git push --tags upstream <tag_name>

Create a release on Github
==========================

* On Github navigate to the repository for which you are creating a release
  for and click on the relases tab. You should see your tag at the top.
* [Optional] You may wish to open the edit page of a previous release in a
  new tab so that you can copy and paste its data.
* Click "Draft a new release" button.
* Add your tag to the release. Edit the other fields to give appropriate
  information.


Upload build to PyPI
====================

.. code-block:: bash

    # Build your distribution inside the repository folder this time
    cd my/repository

    # Create your build
    python3 setup.py bdist_wheel sdist

    # Check that the dist folder contains the right artifacts:
    ls dist/

    # Upload your build to PyPI. n.b. This will not work if your build has the
    # same version number as one already on PyPI.
    twine upload dist/*

Check PyPI for your upload.

Upload to Conda
===============

NOTE: the GitHub and PYPI releases are prerequisites for releasing to
Conda Forge!

The Conda Forge release process is mostly automated by bots. These bots
are set up during the initial project bootstrap, which has been done for
all the projects that are part of Cylc 8.

As there are inter-dependencies amongst the Cylc 8 parts, you should
know the dependency tree, and start by the leaf nodes (i.e. a module
A may have multiple dependants, but no dependency to other modules).

For example, Cylc UI Server depends on Cylc Flow. So unless you are
releasing only Cylc UI Server, you should release Cylc Flow first.

* On GitHub, navigate to the project repository on GitHub, e.g.

- https://github.com/conda-forge/metomi-isodatetime-feedstock
- https://github.com/conda-forge/cylc-uiserver-feedstock
- https://github.com/conda-forge/cylc-ui-feedstock
- https://github.com/conda-forge/cylc-flow-feedstock
- https://github.com/conda-forge/cylc-feedstock

On each of these repositories, the release process should be pretty
much the same. Your work will be mainly (if not exclusive) on
the ``recipe/meta.yaml`` file.

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
in a browser. Then update the ``sha256`` value (you can get this value
from PYPI, or use some tool like ``sha256sum``).
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

To test packages locally, first you should make sure that Conda is
configured to avoid automatically upload the package. Open you ``~/.condarc``
and check that you have something similar to:

.. code-block:: yaml

   channels:
     - defaults
     - conda-forge
   ssl_verify: true
   anaconda_upload: false

Now create a Conda environment for your tests, e.g.: ``conda create -n cylc1``,
and then activate it ``conda activate cylc1``. Then to build and install
locally:

.. code-block:: bash

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
   conda install $PACKAGE_NAME

At this point you should be good to go. Test the package with commands
such as ``which $PACKAGE_NAME``, or ``$PACKAGE_NAME --version``, etc.
Of if you are testing the metapackage, try running the complete system
with a workflow and the UI or tui, and check if there is nothing wrong.

If you found problems while testing locally, try troubleshooting locally,
and either mark the pull request as draft, and close it. Merging the pull
request will create a release.

To undo a release, you will have to liaise with the maintainers via Gitter
or Github. Or, alternatively, bump up the build number +1, and release it
again.

Announce on Riot and Discourse?
===============================
Bask in the glory of having created a release, if appropriate.
