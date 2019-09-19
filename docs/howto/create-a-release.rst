###########################
Creating a release for Cylc
###########################


Before you start (First time)
=============================
* Create a GPG key for verification on Github: `Instructions <https://help.github.com/en/articles/generating-a-new-gpg-key>`_
  At step 3 I had to use ``gpg --default-key rsa4096 --gen-key``.

* `Share this key locally with git, as well as Github: <https://help.github.com/en/articles/telling-git-about-your-signing-key>`_

* Create a PyPI account for yourself if you don't already have one.
  `Sign up page <https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=1&cad=rja&uact=8&ved=2ahUKEwjouIq44MrjAhUKTsAKHeeoCyQQFjAAegQIAxAB&url=https%3A%2F%2Fpypi.org%2Faccount%2Fregister%2F&usg=AOvVaw2ZnHHcXEbkC1c5UJPPMg5a>`_
  Get an owner to add you as a maintainer or owner.


Create pre-release Pull requests
================================

* Ensure that version numbers are consistent in the following files:
  * ``cylc-flow/cylc/flow/__init__``, variable ``__version__ = "8.?.?"``.
  * ``cylc-flow/setup.py``
  * ``cylc-uiserver/setup.py``
* Examine Pull Requests made since the last release for:
  * cylc-flow
  * cylc-conda
  * cylc-ui
  * cylc-uiserver
* Add to ``CHANGES.md`` text in each repository.
* Check the ``CONTRIBUTING.md``. Ensure that all contributors are listed.
  You can do this using Github, or locally with ``git shortlog -sn``
* Get these PRs reviewed and merged to master.


Test the build
==============

* Check locally that you have the updated version of master. Ensure that you
  use git fetch as well as git pull to get the tags.
* Use ``git log`` to ensure that you have the correct version and tag.
* Check that you can buid your distribution locally:

.. code-block:: bash

  # deliberately leave your repository
  # so that the build is a more thorough check
  cd ~/

  # This seems to be really slow
  python3 setup.py bdist_wheel sdist

  # check that you've sucessfully created the build
  ls dist/

  # Tidy up
  rm -r dist


Tag the build
=============

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
  new tab so that you can copy and paste it's data.
* Click "Draft a new release" button.
* Add your tag to the release. Edit the other fields to give appropriate
  information.


Upload build to PyPI
====================

.. code-block:: bash

    # Build your distribution inside the repository folder this time
    cd metomi/isodatetime.git

    # Create your build
    python3 setup.py bdist_wheel sdist

    # Upload your build to PyPI. n.b. This will not work if your build has the
    # same version number as one already on PyPI.
    twine upload dist/*

Check PyPI for your upload.

Announce on Riot and Discourse?
===============================
Bask in the glory of having created a release, if appropriate.
