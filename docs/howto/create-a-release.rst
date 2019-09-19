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

Announce on Riot and Discourse?
===============================
Bask in the glory of having created a release, if appropriate.
