<!-- This file is auto-generated, please do not edit it directly.

Please edit `branches.json` and raise a pull request, this file will be automatically regenerated on merge. -->














# Status

Cylc is a federated project made up of multiple separate components each
developed in its own repository. This page provides an overview dashboard to
make it easier to track the status of branches/releases/issues across these
repositories.

> [!NOTE]
> This file is auto-generated, to update the releases, branches or workflows
> edit the [branches.json file](https://github.com/cylc/cylc-admin/blob/master/docs/status/branches.json)
> and the page will be automatically rebuilt.

#https://github.com/cylc/cylc-admin/blob/master/docs/status/branches.json

<a href="https://github.com/cylc/cylc-admin/actions/workflows/status.yml?query=branch%3Amaster">
  <img src="https://github.com/cylc/cylc-admin/actions/workflows/status.yml/badge.svg?branch=master" />
</a>

## Nightly Builds

<table>
  <tr>
    <th>repo</th>
    <th>workflows</th>
  </tr>

  <tr>
    <td><a href="https://github.com/cylc/cylc-doc">
  <b>cylc/cylc-doc</b>
</a></td>
    <td><a href="https://github.com/cylc/cylc-doc/actions/workflows/nightly.yml?query=branch%3Amaster">
  <img src="https://github.com/cylc/cylc-doc/actions/workflows/nightly.yml/badge.svg?branch=master" />
</a></td>
  </tr>
  <tr>
    <td><a href="https://github.com/cylc/cylc-admin">
  <b>cylc/cylc-admin</b>
</a></td>
    <td><a href="https://github.com/cylc/cylc-admin/actions/workflows/system.yml?query=branch%3Amaster">
  <img src="https://github.com/cylc/cylc-admin/actions/workflows/system.yml/badge.svg?branch=master" />
</a></td>
  </tr>
</table>

## Branches

This section lists the Cylc meta releases (8.0, 8.1, 8.2, etc) that are
currently under active development (including bugfixes), and which branches
this development takes place on in their respective repositories.

The `branches.json` file that this page is built from is used in our CI to
determine which branch to install. For example, the cylc-uiserver CI tests need
to install cylc-flow (because it is a dependency of cylc-uiserver), it uses
the `branches.json` file to work out which cylc-flow branch to use.


### 8.5

<table>
  <tr>
    <th>repo</th>
    <th>branch</th>
    <th>workflows</th>
  </tr>
<tr>
    <td><a href="https://github.com/cylc/cylc-flow">
  <b>cylc/cylc-flow</b>
</a></td>
    <td><a href="https://github.com/cylc/cylc-flow/tree/master">
  master
</a></td>
    <td><a href="https://github.com/cylc/cylc-flow/actions/workflows/bash.yml?query=branch%3Amaster">
  <img src="https://github.com/cylc/cylc-flow/actions/workflows/bash.yml/badge.svg?branch=master" />
</a> <a href="https://github.com/cylc/cylc-flow/actions/workflows/build.yml?query=branch%3Amaster">
  <img src="https://github.com/cylc/cylc-flow/actions/workflows/build.yml/badge.svg?branch=master" />
</a> <a href="https://github.com/cylc/cylc-flow/actions/workflows/shortlog.yml?query=branch%3Amaster">
  <img src="https://github.com/cylc/cylc-flow/actions/workflows/shortlog.yml/badge.svg?branch=master" />
</a> <a href="https://github.com/cylc/cylc-flow/actions/workflows/test_conda-build.yml?query=branch%3Amaster">
  <img src="https://github.com/cylc/cylc-flow/actions/workflows/test_conda-build.yml/badge.svg?branch=master" />
</a> <a href="https://github.com/cylc/cylc-flow/actions/workflows/test_fast.yml?query=branch%3Amaster">
  <img src="https://github.com/cylc/cylc-flow/actions/workflows/test_fast.yml/badge.svg?branch=master" />
</a> <a href="https://github.com/cylc/cylc-flow/actions/workflows/test_functional.yml?query=branch%3Amaster">
  <img src="https://github.com/cylc/cylc-flow/actions/workflows/test_functional.yml/badge.svg?branch=master" />
</a> <a href="https://github.com/cylc/cylc-flow/actions/workflows/test_tutorial_workflow.yml?query=branch%3Amaster">
  <img src="https://github.com/cylc/cylc-flow/actions/workflows/test_tutorial_workflow.yml/badge.svg?branch=master" />
</a></td>
  </tr><tr>
    <td><a href="https://github.com/cylc/cylc-doc">
  <b>cylc/cylc-doc</b>
</a></td>
    <td><a href="https://github.com/cylc/cylc-doc/tree/master">
  master
</a></td>
    <td><a href="https://github.com/cylc/cylc-doc/actions/workflows/shortlog.yml?query=branch%3Amaster">
  <img src="https://github.com/cylc/cylc-doc/actions/workflows/shortlog.yml/badge.svg?branch=master" />
</a> <a href="https://github.com/cylc/cylc-doc/actions/workflows/test.yml?query=branch%3Amaster">
  <img src="https://github.com/cylc/cylc-doc/actions/workflows/test.yml/badge.svg?branch=master" />
</a></td>
  </tr><tr>
    <td><a href="https://github.com/cylc/cylc-rose">
  <b>cylc/cylc-rose</b>
</a></td>
    <td><a href="https://github.com/cylc/cylc-rose/tree/master">
  master
</a></td>
    <td><a href="https://github.com/cylc/cylc-rose/actions/workflows/shortlog.yml?query=branch%3Amaster">
  <img src="https://github.com/cylc/cylc-rose/actions/workflows/shortlog.yml/badge.svg?branch=master" />
</a> <a href="https://github.com/cylc/cylc-rose/actions/workflows/tests.yml?query=branch%3Amaster">
  <img src="https://github.com/cylc/cylc-rose/actions/workflows/tests.yml/badge.svg?branch=master" />
</a></td>
  </tr><tr>
    <td><a href="https://github.com/cylc/cylc-uiserver">
  <b>cylc/cylc-uiserver</b>
</a></td>
    <td><a href="https://github.com/cylc/cylc-uiserver/tree/master">
  master
</a></td>
    <td><a href="https://github.com/cylc/cylc-uiserver/actions/workflows/build.yml?query=branch%3Amaster">
  <img src="https://github.com/cylc/cylc-uiserver/actions/workflows/build.yml/badge.svg?branch=master" />
</a> <a href="https://github.com/cylc/cylc-uiserver/actions/workflows/shortlog.yml?query=branch%3Amaster">
  <img src="https://github.com/cylc/cylc-uiserver/actions/workflows/shortlog.yml/badge.svg?branch=master" />
</a> <a href="https://github.com/cylc/cylc-uiserver/actions/workflows/test.yml?query=branch%3Amaster">
  <img src="https://github.com/cylc/cylc-uiserver/actions/workflows/test.yml/badge.svg?branch=master" />
</a></td>
  </tr><tr>
    <td><a href="https://github.com/metomi/rose">
  <b>metomi/rose</b>
</a></td>
    <td><a href="https://github.com/metomi/rose/tree/master">
  master
</a></td>
    <td><a href="https://github.com/metomi/rose/actions/workflows/build.yml?query=branch%3Amaster">
  <img src="https://github.com/metomi/rose/actions/workflows/build.yml/badge.svg?branch=master" />
</a> <a href="https://github.com/metomi/rose/actions/workflows/shortlog.yml?query=branch%3Amaster">
  <img src="https://github.com/metomi/rose/actions/workflows/shortlog.yml/badge.svg?branch=master" />
</a> <a href="https://github.com/metomi/rose/actions/workflows/test.yml?query=branch%3Amaster">
  <img src="https://github.com/metomi/rose/actions/workflows/test.yml/badge.svg?branch=master" />
</a></td>
  </tr>
</table>

### 8.4

<table>
  <tr>
    <th>repo</th>
    <th>branch</th>
    <th>workflows</th>
  </tr>
<tr>
    <td><a href="https://github.com/cylc/cylc-flow">
  <b>cylc/cylc-flow</b>
</a></td>
    <td><a href="https://github.com/cylc/cylc-flow/tree/8.4.x">
  8.4.x
</a></td>
    <td><a href="https://github.com/cylc/cylc-flow/actions/workflows/bash.yml?query=branch%3A8.4.x">
  <img src="https://github.com/cylc/cylc-flow/actions/workflows/bash.yml/badge.svg?branch=8.4.x" />
</a> <a href="https://github.com/cylc/cylc-flow/actions/workflows/build.yml?query=branch%3A8.4.x">
  <img src="https://github.com/cylc/cylc-flow/actions/workflows/build.yml/badge.svg?branch=8.4.x" />
</a> <a href="https://github.com/cylc/cylc-flow/actions/workflows/shortlog.yml?query=branch%3A8.4.x">
  <img src="https://github.com/cylc/cylc-flow/actions/workflows/shortlog.yml/badge.svg?branch=8.4.x" />
</a> <a href="https://github.com/cylc/cylc-flow/actions/workflows/test_conda-build.yml?query=branch%3A8.4.x">
  <img src="https://github.com/cylc/cylc-flow/actions/workflows/test_conda-build.yml/badge.svg?branch=8.4.x" />
</a> <a href="https://github.com/cylc/cylc-flow/actions/workflows/test_fast.yml?query=branch%3A8.4.x">
  <img src="https://github.com/cylc/cylc-flow/actions/workflows/test_fast.yml/badge.svg?branch=8.4.x" />
</a> <a href="https://github.com/cylc/cylc-flow/actions/workflows/test_functional.yml?query=branch%3A8.4.x">
  <img src="https://github.com/cylc/cylc-flow/actions/workflows/test_functional.yml/badge.svg?branch=8.4.x" />
</a> <a href="https://github.com/cylc/cylc-flow/actions/workflows/test_tutorial_workflow.yml?query=branch%3A8.4.x">
  <img src="https://github.com/cylc/cylc-flow/actions/workflows/test_tutorial_workflow.yml/badge.svg?branch=8.4.x" />
</a></td>
  </tr><tr>
    <td><a href="https://github.com/cylc/cylc-doc">
  <b>cylc/cylc-doc</b>
</a></td>
    <td><a href="https://github.com/cylc/cylc-doc/tree/8.4.x">
  8.4.x
</a></td>
    <td><a href="https://github.com/cylc/cylc-doc/actions/workflows/shortlog.yml?query=branch%3A8.4.x">
  <img src="https://github.com/cylc/cylc-doc/actions/workflows/shortlog.yml/badge.svg?branch=8.4.x" />
</a> <a href="https://github.com/cylc/cylc-doc/actions/workflows/test.yml?query=branch%3A8.4.x">
  <img src="https://github.com/cylc/cylc-doc/actions/workflows/test.yml/badge.svg?branch=8.4.x" />
</a></td>
  </tr><tr>
    <td><a href="https://github.com/cylc/cylc-rose">
  <b>cylc/cylc-rose</b>
</a></td>
    <td><a href="https://github.com/cylc/cylc-rose/tree/1.5.x">
  1.5.x
</a></td>
    <td><a href="https://github.com/cylc/cylc-rose/actions/workflows/shortlog.yml?query=branch%3A1.5.x">
  <img src="https://github.com/cylc/cylc-rose/actions/workflows/shortlog.yml/badge.svg?branch=1.5.x" />
</a> <a href="https://github.com/cylc/cylc-rose/actions/workflows/tests.yml?query=branch%3A1.5.x">
  <img src="https://github.com/cylc/cylc-rose/actions/workflows/tests.yml/badge.svg?branch=1.5.x" />
</a></td>
  </tr><tr>
    <td><a href="https://github.com/cylc/cylc-uiserver">
  <b>cylc/cylc-uiserver</b>
</a></td>
    <td><a href="https://github.com/cylc/cylc-uiserver/tree/1.6.x">
  1.6.x
</a></td>
    <td><a href="https://github.com/cylc/cylc-uiserver/actions/workflows/build.yml?query=branch%3A1.6.x">
  <img src="https://github.com/cylc/cylc-uiserver/actions/workflows/build.yml/badge.svg?branch=1.6.x" />
</a> <a href="https://github.com/cylc/cylc-uiserver/actions/workflows/shortlog.yml?query=branch%3A1.6.x">
  <img src="https://github.com/cylc/cylc-uiserver/actions/workflows/shortlog.yml/badge.svg?branch=1.6.x" />
</a> <a href="https://github.com/cylc/cylc-uiserver/actions/workflows/test.yml?query=branch%3A1.6.x">
  <img src="https://github.com/cylc/cylc-uiserver/actions/workflows/test.yml/badge.svg?branch=1.6.x" />
</a></td>
  </tr><tr>
    <td><a href="https://github.com/metomi/rose">
  <b>metomi/rose</b>
</a></td>
    <td><a href="https://github.com/metomi/rose/tree/2.4.x">
  2.4.x
</a></td>
    <td><a href="https://github.com/metomi/rose/actions/workflows/build.yml?query=branch%3A2.4.x">
  <img src="https://github.com/metomi/rose/actions/workflows/build.yml/badge.svg?branch=2.4.x" />
</a> <a href="https://github.com/metomi/rose/actions/workflows/shortlog.yml?query=branch%3A2.4.x">
  <img src="https://github.com/metomi/rose/actions/workflows/shortlog.yml/badge.svg?branch=2.4.x" />
</a> <a href="https://github.com/metomi/rose/actions/workflows/test.yml?query=branch%3A2.4.x">
  <img src="https://github.com/metomi/rose/actions/workflows/test.yml/badge.svg?branch=2.4.x" />
</a></td>
  </tr>
</table>


## Issues


### cylc/cylc-flow

<a href="https://github.com/cylc/cylc-flow/issues/?q=is%3Aopen+is%3Aissue">
  <img src="https://img.shields.io/github/issues-raw/cylc/cylc-flow" />
</a> <a href="https://github.com/cylc/cylc-flow/pulls">
  <img src="https://img.shields.io/github/issues-pr/cylc/cylc-flow" />
</a> <a href="https://github.com/cylc/cylc-flow/issues?q=is%3Aissue+is%3Aopen+label%3Aquestion">
  <img src="https://img.shields.io/github/issues/cylc/cylc-flow/question" />
</a> <a href="https://github.com/cylc/cylc-flow/issues?q=is%3Aissue+is%3Aopen+label%3Abug">
  <img src="https://img.shields.io/github/issues/cylc/cylc-flow/bug" />
</a> <a href="https://github.com/cylc/cylc-flow/issues?q=is%3Aopen+no%3Amilestone">
  <img src="https://img.shields.io/github/issues-search/cylc/cylc-flow?query=is%3Aopen%20no%3Amilestone&label=no%20milestone" />
</a>

<table style="margin-left:20px; margin-top:10px">

<tr>
  <td><a href="https://github.com/cylc/cylc-flow/milestone/82?q=is%3Aopen">
  <img src="https://img.shields.io/github/milestones/issues-open/cylc/cylc-flow/82" />
</a></td>
  <td><a href="https://github.com/cylc/cylc-flow/milestone/82?q=is%3Aclosed">
  <img src="https://img.shields.io/github/milestones/issues-closed/cylc/cylc-flow/82" />
</a></td>
</tr>

<tr>
  <td><a href="https://github.com/cylc/cylc-flow/milestone/129?q=is%3Aopen">
  <img src="https://img.shields.io/github/milestones/issues-open/cylc/cylc-flow/129" />
</a></td>
  <td><a href="https://github.com/cylc/cylc-flow/milestone/129?q=is%3Aclosed">
  <img src="https://img.shields.io/github/milestones/issues-closed/cylc/cylc-flow/129" />
</a></td>
</tr>

<tr>
  <td><a href="https://github.com/cylc/cylc-flow/milestone/146?q=is%3Aopen">
  <img src="https://img.shields.io/github/milestones/issues-open/cylc/cylc-flow/146" />
</a></td>
  <td><a href="https://github.com/cylc/cylc-flow/milestone/146?q=is%3Aclosed">
  <img src="https://img.shields.io/github/milestones/issues-closed/cylc/cylc-flow/146" />
</a></td>
</tr>

<tr>
  <td><a href="https://github.com/cylc/cylc-flow/milestone/144?q=is%3Aopen">
  <img src="https://img.shields.io/github/milestones/issues-open/cylc/cylc-flow/144" />
</a></td>
  <td><a href="https://github.com/cylc/cylc-flow/milestone/144?q=is%3Aclosed">
  <img src="https://img.shields.io/github/milestones/issues-closed/cylc/cylc-flow/144" />
</a></td>
</tr>

<tr>
  <td><a href="https://github.com/cylc/cylc-flow/milestone/143?q=is%3Aopen">
  <img src="https://img.shields.io/github/milestones/issues-open/cylc/cylc-flow/143" />
</a></td>
  <td><a href="https://github.com/cylc/cylc-flow/milestone/143?q=is%3Aclosed">
  <img src="https://img.shields.io/github/milestones/issues-closed/cylc/cylc-flow/143" />
</a></td>
</tr>

<tr>
  <td><a href="https://github.com/cylc/cylc-flow/milestone/145?q=is%3Aopen">
  <img src="https://img.shields.io/github/milestones/issues-open/cylc/cylc-flow/145" />
</a></td>
  <td><a href="https://github.com/cylc/cylc-flow/milestone/145?q=is%3Aclosed">
  <img src="https://img.shields.io/github/milestones/issues-closed/cylc/cylc-flow/145" />
</a></td>
</tr>

<tr>
  <td><a href="https://github.com/cylc/cylc-flow/milestone/89?q=is%3Aopen">
  <img src="https://img.shields.io/github/milestones/issues-open/cylc/cylc-flow/89" />
</a></td>
  <td><a href="https://github.com/cylc/cylc-flow/milestone/89?q=is%3Aclosed">
  <img src="https://img.shields.io/github/milestones/issues-closed/cylc/cylc-flow/89" />
</a></td>
</tr>

<tr>
  <td><a href="https://github.com/cylc/cylc-flow/milestone/139?q=is%3Aopen">
  <img src="https://img.shields.io/github/milestones/issues-open/cylc/cylc-flow/139" />
</a></td>
  <td><a href="https://github.com/cylc/cylc-flow/milestone/139?q=is%3Aclosed">
  <img src="https://img.shields.io/github/milestones/issues-closed/cylc/cylc-flow/139" />
</a></td>
</tr>

<tr>
  <td><a href="https://github.com/cylc/cylc-flow/milestone/76?q=is%3Aopen">
  <img src="https://img.shields.io/github/milestones/issues-open/cylc/cylc-flow/76" />
</a></td>
  <td><a href="https://github.com/cylc/cylc-flow/milestone/76?q=is%3Aclosed">
  <img src="https://img.shields.io/github/milestones/issues-closed/cylc/cylc-flow/76" />
</a></td>
</tr>

<tr>
  <td><a href="https://github.com/cylc/cylc-flow/milestone/7?q=is%3Aopen">
  <img src="https://img.shields.io/github/milestones/issues-open/cylc/cylc-flow/7" />
</a></td>
  <td><a href="https://github.com/cylc/cylc-flow/milestone/7?q=is%3Aclosed">
  <img src="https://img.shields.io/github/milestones/issues-closed/cylc/cylc-flow/7" />
</a></td>
</tr>

</table>


### cylc/cylc-doc

<a href="https://github.com/cylc/cylc-doc/issues/?q=is%3Aopen+is%3Aissue">
  <img src="https://img.shields.io/github/issues-raw/cylc/cylc-doc" />
</a> <a href="https://github.com/cylc/cylc-doc/pulls">
  <img src="https://img.shields.io/github/issues-pr/cylc/cylc-doc" />
</a> <a href="https://github.com/cylc/cylc-doc/issues?q=is%3Aissue+is%3Aopen+label%3Aquestion">
  <img src="https://img.shields.io/github/issues/cylc/cylc-doc/question" />
</a> <a href="https://github.com/cylc/cylc-doc/issues?q=is%3Aissue+is%3Aopen+label%3Abug">
  <img src="https://img.shields.io/github/issues/cylc/cylc-doc/bug" />
</a> <a href="https://github.com/cylc/cylc-doc/issues?q=is%3Aopen+no%3Amilestone">
  <img src="https://img.shields.io/github/issues-search/cylc/cylc-doc?query=is%3Aopen%20no%3Amilestone&label=no%20milestone" />
</a>

<table style="margin-left:20px; margin-top:10px">

<tr>
  <td><a href="https://github.com/cylc/cylc-doc/milestone/20?q=is%3Aopen">
  <img src="https://img.shields.io/github/milestones/issues-open/cylc/cylc-doc/20" />
</a></td>
  <td><a href="https://github.com/cylc/cylc-doc/milestone/20?q=is%3Aclosed">
  <img src="https://img.shields.io/github/milestones/issues-closed/cylc/cylc-doc/20" />
</a></td>
</tr>

<tr>
  <td><a href="https://github.com/cylc/cylc-doc/milestone/19?q=is%3Aopen">
  <img src="https://img.shields.io/github/milestones/issues-open/cylc/cylc-doc/19" />
</a></td>
  <td><a href="https://github.com/cylc/cylc-doc/milestone/19?q=is%3Aclosed">
  <img src="https://img.shields.io/github/milestones/issues-closed/cylc/cylc-doc/19" />
</a></td>
</tr>

<tr>
  <td><a href="https://github.com/cylc/cylc-doc/milestone/22?q=is%3Aopen">
  <img src="https://img.shields.io/github/milestones/issues-open/cylc/cylc-doc/22" />
</a></td>
  <td><a href="https://github.com/cylc/cylc-doc/milestone/22?q=is%3Aclosed">
  <img src="https://img.shields.io/github/milestones/issues-closed/cylc/cylc-doc/22" />
</a></td>
</tr>

<tr>
  <td><a href="https://github.com/cylc/cylc-doc/milestone/23?q=is%3Aopen">
  <img src="https://img.shields.io/github/milestones/issues-open/cylc/cylc-doc/23" />
</a></td>
  <td><a href="https://github.com/cylc/cylc-doc/milestone/23?q=is%3Aclosed">
  <img src="https://img.shields.io/github/milestones/issues-closed/cylc/cylc-doc/23" />
</a></td>
</tr>

<tr>
  <td><a href="https://github.com/cylc/cylc-doc/milestone/3?q=is%3Aopen">
  <img src="https://img.shields.io/github/milestones/issues-open/cylc/cylc-doc/3" />
</a></td>
  <td><a href="https://github.com/cylc/cylc-doc/milestone/3?q=is%3Aclosed">
  <img src="https://img.shields.io/github/milestones/issues-closed/cylc/cylc-doc/3" />
</a></td>
</tr>

</table>


### cylc/cylc-rose

<a href="https://github.com/cylc/cylc-rose/issues/?q=is%3Aopen+is%3Aissue">
  <img src="https://img.shields.io/github/issues-raw/cylc/cylc-rose" />
</a> <a href="https://github.com/cylc/cylc-rose/pulls">
  <img src="https://img.shields.io/github/issues-pr/cylc/cylc-rose" />
</a> <a href="https://github.com/cylc/cylc-rose/issues?q=is%3Aissue+is%3Aopen+label%3Aquestion">
  <img src="https://img.shields.io/github/issues/cylc/cylc-rose/question" />
</a> <a href="https://github.com/cylc/cylc-rose/issues?q=is%3Aissue+is%3Aopen+label%3Abug">
  <img src="https://img.shields.io/github/issues/cylc/cylc-rose/bug" />
</a> <a href="https://github.com/cylc/cylc-rose/issues?q=is%3Aopen+no%3Amilestone">
  <img src="https://img.shields.io/github/issues-search/cylc/cylc-rose?query=is%3Aopen%20no%3Amilestone&label=no%20milestone" />
</a>

<table style="margin-left:20px; margin-top:10px">

<tr>
  <td><a href="https://github.com/cylc/cylc-rose/milestone/29?q=is%3Aopen">
  <img src="https://img.shields.io/github/milestones/issues-open/cylc/cylc-rose/29" />
</a></td>
  <td><a href="https://github.com/cylc/cylc-rose/milestone/29?q=is%3Aclosed">
  <img src="https://img.shields.io/github/milestones/issues-closed/cylc/cylc-rose/29" />
</a></td>
</tr>

<tr>
  <td><a href="https://github.com/cylc/cylc-rose/milestone/30?q=is%3Aopen">
  <img src="https://img.shields.io/github/milestones/issues-open/cylc/cylc-rose/30" />
</a></td>
  <td><a href="https://github.com/cylc/cylc-rose/milestone/30?q=is%3Aclosed">
  <img src="https://img.shields.io/github/milestones/issues-closed/cylc/cylc-rose/30" />
</a></td>
</tr>

<tr>
  <td><a href="https://github.com/cylc/cylc-rose/milestone/2?q=is%3Aopen">
  <img src="https://img.shields.io/github/milestones/issues-open/cylc/cylc-rose/2" />
</a></td>
  <td><a href="https://github.com/cylc/cylc-rose/milestone/2?q=is%3Aclosed">
  <img src="https://img.shields.io/github/milestones/issues-closed/cylc/cylc-rose/2" />
</a></td>
</tr>

</table>


### cylc/cylc-uiserver

<a href="https://github.com/cylc/cylc-uiserver/issues/?q=is%3Aopen+is%3Aissue">
  <img src="https://img.shields.io/github/issues-raw/cylc/cylc-uiserver" />
</a> <a href="https://github.com/cylc/cylc-uiserver/pulls">
  <img src="https://img.shields.io/github/issues-pr/cylc/cylc-uiserver" />
</a> <a href="https://github.com/cylc/cylc-uiserver/issues?q=is%3Aissue+is%3Aopen+label%3Aquestion">
  <img src="https://img.shields.io/github/issues/cylc/cylc-uiserver/question" />
</a> <a href="https://github.com/cylc/cylc-uiserver/issues?q=is%3Aissue+is%3Aopen+label%3Abug">
  <img src="https://img.shields.io/github/issues/cylc/cylc-uiserver/bug" />
</a> <a href="https://github.com/cylc/cylc-uiserver/issues?q=is%3Aopen+no%3Amilestone">
  <img src="https://img.shields.io/github/issues-search/cylc/cylc-uiserver?query=is%3Aopen%20no%3Amilestone&label=no%20milestone" />
</a>

<table style="margin-left:20px; margin-top:10px">

<tr>
  <td><a href="https://github.com/cylc/cylc-uiserver/milestone/34?q=is%3Aopen">
  <img src="https://img.shields.io/github/milestones/issues-open/cylc/cylc-uiserver/34" />
</a></td>
  <td><a href="https://github.com/cylc/cylc-uiserver/milestone/34?q=is%3Aclosed">
  <img src="https://img.shields.io/github/milestones/issues-closed/cylc/cylc-uiserver/34" />
</a></td>
</tr>

<tr>
  <td><a href="https://github.com/cylc/cylc-uiserver/milestone/31?q=is%3Aopen">
  <img src="https://img.shields.io/github/milestones/issues-open/cylc/cylc-uiserver/31" />
</a></td>
  <td><a href="https://github.com/cylc/cylc-uiserver/milestone/31?q=is%3Aclosed">
  <img src="https://img.shields.io/github/milestones/issues-closed/cylc/cylc-uiserver/31" />
</a></td>
</tr>

<tr>
  <td><a href="https://github.com/cylc/cylc-uiserver/milestone/36?q=is%3Aopen">
  <img src="https://img.shields.io/github/milestones/issues-open/cylc/cylc-uiserver/36" />
</a></td>
  <td><a href="https://github.com/cylc/cylc-uiserver/milestone/36?q=is%3Aclosed">
  <img src="https://img.shields.io/github/milestones/issues-closed/cylc/cylc-uiserver/36" />
</a></td>
</tr>

<tr>
  <td><a href="https://github.com/cylc/cylc-uiserver/milestone/35?q=is%3Aopen">
  <img src="https://img.shields.io/github/milestones/issues-open/cylc/cylc-uiserver/35" />
</a></td>
  <td><a href="https://github.com/cylc/cylc-uiserver/milestone/35?q=is%3Aclosed">
  <img src="https://img.shields.io/github/milestones/issues-closed/cylc/cylc-uiserver/35" />
</a></td>
</tr>

<tr>
  <td><a href="https://github.com/cylc/cylc-uiserver/milestone/8?q=is%3Aopen">
  <img src="https://img.shields.io/github/milestones/issues-open/cylc/cylc-uiserver/8" />
</a></td>
  <td><a href="https://github.com/cylc/cylc-uiserver/milestone/8?q=is%3Aclosed">
  <img src="https://img.shields.io/github/milestones/issues-closed/cylc/cylc-uiserver/8" />
</a></td>
</tr>

</table>


### metomi/rose

<a href="https://github.com/metomi/rose/issues/?q=is%3Aopen+is%3Aissue">
  <img src="https://img.shields.io/github/issues-raw/metomi/rose" />
</a> <a href="https://github.com/metomi/rose/pulls">
  <img src="https://img.shields.io/github/issues-pr/metomi/rose" />
</a> <a href="https://github.com/metomi/rose/issues?q=is%3Aissue+is%3Aopen+label%3Aquestion">
  <img src="https://img.shields.io/github/issues/metomi/rose/question" />
</a> <a href="https://github.com/metomi/rose/issues?q=is%3Aissue+is%3Aopen+label%3Abug">
  <img src="https://img.shields.io/github/issues/metomi/rose/bug" />
</a> <a href="https://github.com/metomi/rose/issues?q=is%3Aopen+no%3Amilestone">
  <img src="https://img.shields.io/github/issues-search/metomi/rose?query=is%3Aopen%20no%3Amilestone&label=no%20milestone" />
</a>

<table style="margin-left:20px; margin-top:10px">

<tr>
  <td><a href="https://github.com/metomi/rose/milestone/101?q=is%3Aopen">
  <img src="https://img.shields.io/github/milestones/issues-open/metomi/rose/101" />
</a></td>
  <td><a href="https://github.com/metomi/rose/milestone/101?q=is%3Aclosed">
  <img src="https://img.shields.io/github/milestones/issues-closed/metomi/rose/101" />
</a></td>
</tr>

<tr>
  <td><a href="https://github.com/metomi/rose/milestone/104?q=is%3Aopen">
  <img src="https://img.shields.io/github/milestones/issues-open/metomi/rose/104" />
</a></td>
  <td><a href="https://github.com/metomi/rose/milestone/104?q=is%3Aclosed">
  <img src="https://img.shields.io/github/milestones/issues-closed/metomi/rose/104" />
</a></td>
</tr>

<tr>
  <td><a href="https://github.com/metomi/rose/milestone/76?q=is%3Aopen">
  <img src="https://img.shields.io/github/milestones/issues-open/metomi/rose/76" />
</a></td>
  <td><a href="https://github.com/metomi/rose/milestone/76?q=is%3Aclosed">
  <img src="https://img.shields.io/github/milestones/issues-closed/metomi/rose/76" />
</a></td>
</tr>

<tr>
  <td><a href="https://github.com/metomi/rose/milestone/74?q=is%3Aopen">
  <img src="https://img.shields.io/github/milestones/issues-open/metomi/rose/74" />
</a></td>
  <td><a href="https://github.com/metomi/rose/milestone/74?q=is%3Aclosed">
  <img src="https://img.shields.io/github/milestones/issues-closed/metomi/rose/74" />
</a></td>
</tr>

<tr>
  <td><a href="https://github.com/metomi/rose/milestone/87?q=is%3Aopen">
  <img src="https://img.shields.io/github/milestones/issues-open/metomi/rose/87" />
</a></td>
  <td><a href="https://github.com/metomi/rose/milestone/87?q=is%3Aclosed">
  <img src="https://img.shields.io/github/milestones/issues-closed/metomi/rose/87" />
</a></td>
</tr>

<tr>
  <td><a href="https://github.com/metomi/rose/milestone/65?q=is%3Aopen">
  <img src="https://img.shields.io/github/milestones/issues-open/metomi/rose/65" />
</a></td>
  <td><a href="https://github.com/metomi/rose/milestone/65?q=is%3Aclosed">
  <img src="https://img.shields.io/github/milestones/issues-closed/metomi/rose/65" />
</a></td>
</tr>

<tr>
  <td><a href="https://github.com/metomi/rose/milestone/3?q=is%3Aopen">
  <img src="https://img.shields.io/github/milestones/issues-open/metomi/rose/3" />
</a></td>
  <td><a href="https://github.com/metomi/rose/milestone/3?q=is%3Aclosed">
  <img src="https://img.shields.io/github/milestones/issues-closed/metomi/rose/3" />
</a></td>
</tr>

</table>


### cylc/cylc-admin

<a href="https://github.com/cylc/cylc-admin/issues/?q=is%3Aopen+is%3Aissue">
  <img src="https://img.shields.io/github/issues-raw/cylc/cylc-admin" />
</a> <a href="https://github.com/cylc/cylc-admin/pulls">
  <img src="https://img.shields.io/github/issues-pr/cylc/cylc-admin" />
</a> <a href="https://github.com/cylc/cylc-admin/issues?q=is%3Aissue+is%3Aopen+label%3Aquestion">
  <img src="https://img.shields.io/github/issues/cylc/cylc-admin/question" />
</a> <a href="https://github.com/cylc/cylc-admin/issues?q=is%3Aissue+is%3Aopen+label%3Abug">
  <img src="https://img.shields.io/github/issues/cylc/cylc-admin/bug" />
</a> <a href="https://github.com/cylc/cylc-admin/issues?q=is%3Aopen+no%3Amilestone">
  <img src="https://img.shields.io/github/issues-search/cylc/cylc-admin?query=is%3Aopen%20no%3Amilestone&label=no%20milestone" />
</a>

<table style="margin-left:20px; margin-top:10px">

</table>


### cylc/cylc.github.io

<a href="https://github.com/cylc/cylc.github.io/issues/?q=is%3Aopen+is%3Aissue">
  <img src="https://img.shields.io/github/issues-raw/cylc/cylc.github.io" />
</a> <a href="https://github.com/cylc/cylc.github.io/pulls">
  <img src="https://img.shields.io/github/issues-pr/cylc/cylc.github.io" />
</a> <a href="https://github.com/cylc/cylc.github.io/issues?q=is%3Aissue+is%3Aopen+label%3Aquestion">
  <img src="https://img.shields.io/github/issues/cylc/cylc.github.io/question" />
</a> <a href="https://github.com/cylc/cylc.github.io/issues?q=is%3Aissue+is%3Aopen+label%3Abug">
  <img src="https://img.shields.io/github/issues/cylc/cylc.github.io/bug" />
</a> <a href="https://github.com/cylc/cylc.github.io/issues?q=is%3Aopen+no%3Amilestone">
  <img src="https://img.shields.io/github/issues-search/cylc/cylc.github.io?query=is%3Aopen%20no%3Amilestone&label=no%20milestone" />
</a>

<table style="margin-left:20px; margin-top:10px">

</table>


### cylc/cylc-sphinx-extensions

<a href="https://github.com/cylc/cylc-sphinx-extensions/issues/?q=is%3Aopen+is%3Aissue">
  <img src="https://img.shields.io/github/issues-raw/cylc/cylc-sphinx-extensions" />
</a> <a href="https://github.com/cylc/cylc-sphinx-extensions/pulls">
  <img src="https://img.shields.io/github/issues-pr/cylc/cylc-sphinx-extensions" />
</a> <a href="https://github.com/cylc/cylc-sphinx-extensions/issues?q=is%3Aissue+is%3Aopen+label%3Aquestion">
  <img src="https://img.shields.io/github/issues/cylc/cylc-sphinx-extensions/question" />
</a> <a href="https://github.com/cylc/cylc-sphinx-extensions/issues?q=is%3Aissue+is%3Aopen+label%3Abug">
  <img src="https://img.shields.io/github/issues/cylc/cylc-sphinx-extensions/bug" />
</a> <a href="https://github.com/cylc/cylc-sphinx-extensions/issues?q=is%3Aopen+no%3Amilestone">
  <img src="https://img.shields.io/github/issues-search/cylc/cylc-sphinx-extensions?query=is%3Aopen%20no%3Amilestone&label=no%20milestone" />
</a>

<table style="margin-left:20px; margin-top:10px">

<tr>
  <td><a href="https://github.com/cylc/cylc-sphinx-extensions/milestone/15?q=is%3Aopen">
  <img src="https://img.shields.io/github/milestones/issues-open/cylc/cylc-sphinx-extensions/15" />
</a></td>
  <td><a href="https://github.com/cylc/cylc-sphinx-extensions/milestone/15?q=is%3Aclosed">
  <img src="https://img.shields.io/github/milestones/issues-closed/cylc/cylc-sphinx-extensions/15" />
</a></td>
</tr>

<tr>
  <td><a href="https://github.com/cylc/cylc-sphinx-extensions/milestone/3?q=is%3Aopen">
  <img src="https://img.shields.io/github/milestones/issues-open/cylc/cylc-sphinx-extensions/3" />
</a></td>
  <td><a href="https://github.com/cylc/cylc-sphinx-extensions/milestone/3?q=is%3Aclosed">
  <img src="https://img.shields.io/github/milestones/issues-closed/cylc/cylc-sphinx-extensions/3" />
</a></td>
</tr>

</table>


### cylc/release-actions

<a href="https://github.com/cylc/release-actions/issues/?q=is%3Aopen+is%3Aissue">
  <img src="https://img.shields.io/github/issues-raw/cylc/release-actions" />
</a> <a href="https://github.com/cylc/release-actions/pulls">
  <img src="https://img.shields.io/github/issues-pr/cylc/release-actions" />
</a> <a href="https://github.com/cylc/release-actions/issues?q=is%3Aissue+is%3Aopen+label%3Aquestion">
  <img src="https://img.shields.io/github/issues/cylc/release-actions/question" />
</a> <a href="https://github.com/cylc/release-actions/issues?q=is%3Aissue+is%3Aopen+label%3Abug">
  <img src="https://img.shields.io/github/issues/cylc/release-actions/bug" />
</a> <a href="https://github.com/cylc/release-actions/issues?q=is%3Aopen+no%3Amilestone">
  <img src="https://img.shields.io/github/issues-search/cylc/release-actions?query=is%3Aopen%20no%3Amilestone&label=no%20milestone" />
</a>

<table style="margin-left:20px; margin-top:10px">

<tr>
  <td><a href="https://github.com/cylc/release-actions/milestone/1?q=is%3Aopen">
  <img src="https://img.shields.io/github/milestones/issues-open/cylc/release-actions/1" />
</a></td>
  <td><a href="https://github.com/cylc/release-actions/milestone/1?q=is%3Aclosed">
  <img src="https://img.shields.io/github/milestones/issues-closed/cylc/release-actions/1" />
</a></td>
</tr>

<tr>
  <td><a href="https://github.com/cylc/release-actions/milestone/2?q=is%3Aopen">
  <img src="https://img.shields.io/github/milestones/issues-open/cylc/release-actions/2" />
</a></td>
  <td><a href="https://github.com/cylc/release-actions/milestone/2?q=is%3Aclosed">
  <img src="https://img.shields.io/github/milestones/issues-closed/cylc/release-actions/2" />
</a></td>
</tr>

</table>


### cylc/cylc-textmate-grammar

<a href="https://github.com/cylc/cylc-textmate-grammar/issues/?q=is%3Aopen+is%3Aissue">
  <img src="https://img.shields.io/github/issues-raw/cylc/cylc-textmate-grammar" />
</a> <a href="https://github.com/cylc/cylc-textmate-grammar/pulls">
  <img src="https://img.shields.io/github/issues-pr/cylc/cylc-textmate-grammar" />
</a> <a href="https://github.com/cylc/cylc-textmate-grammar/issues?q=is%3Aissue+is%3Aopen+label%3Aquestion">
  <img src="https://img.shields.io/github/issues/cylc/cylc-textmate-grammar/question" />
</a> <a href="https://github.com/cylc/cylc-textmate-grammar/issues?q=is%3Aissue+is%3Aopen+label%3Abug">
  <img src="https://img.shields.io/github/issues/cylc/cylc-textmate-grammar/bug" />
</a> <a href="https://github.com/cylc/cylc-textmate-grammar/issues?q=is%3Aopen+no%3Amilestone">
  <img src="https://img.shields.io/github/issues-search/cylc/cylc-textmate-grammar?query=is%3Aopen%20no%3Amilestone&label=no%20milestone" />
</a>

<table style="margin-left:20px; margin-top:10px">

</table>


### cylc/Cylc.tmbundle

<a href="https://github.com/cylc/Cylc.tmbundle/issues/?q=is%3Aopen+is%3Aissue">
  <img src="https://img.shields.io/github/issues-raw/cylc/Cylc.tmbundle" />
</a> <a href="https://github.com/cylc/Cylc.tmbundle/pulls">
  <img src="https://img.shields.io/github/issues-pr/cylc/Cylc.tmbundle" />
</a> <a href="https://github.com/cylc/Cylc.tmbundle/issues?q=is%3Aissue+is%3Aopen+label%3Aquestion">
  <img src="https://img.shields.io/github/issues/cylc/Cylc.tmbundle/question" />
</a> <a href="https://github.com/cylc/Cylc.tmbundle/issues?q=is%3Aissue+is%3Aopen+label%3Abug">
  <img src="https://img.shields.io/github/issues/cylc/Cylc.tmbundle/bug" />
</a> <a href="https://github.com/cylc/Cylc.tmbundle/issues?q=is%3Aopen+no%3Amilestone">
  <img src="https://img.shields.io/github/issues-search/cylc/Cylc.tmbundle?query=is%3Aopen%20no%3Amilestone&label=no%20milestone" />
</a>

<table style="margin-left:20px; margin-top:10px">

</table>


### cylc/vscode-cylc

<a href="https://github.com/cylc/vscode-cylc/issues/?q=is%3Aopen+is%3Aissue">
  <img src="https://img.shields.io/github/issues-raw/cylc/vscode-cylc" />
</a> <a href="https://github.com/cylc/vscode-cylc/pulls">
  <img src="https://img.shields.io/github/issues-pr/cylc/vscode-cylc" />
</a> <a href="https://github.com/cylc/vscode-cylc/issues?q=is%3Aissue+is%3Aopen+label%3Aquestion">
  <img src="https://img.shields.io/github/issues/cylc/vscode-cylc/question" />
</a> <a href="https://github.com/cylc/vscode-cylc/issues?q=is%3Aissue+is%3Aopen+label%3Abug">
  <img src="https://img.shields.io/github/issues/cylc/vscode-cylc/bug" />
</a> <a href="https://github.com/cylc/vscode-cylc/issues?q=is%3Aopen+no%3Amilestone">
  <img src="https://img.shields.io/github/issues-search/cylc/vscode-cylc?query=is%3Aopen%20no%3Amilestone&label=no%20milestone" />
</a>

<table style="margin-left:20px; margin-top:10px">

</table>


### cylc/language-cylc

<a href="https://github.com/cylc/language-cylc/issues/?q=is%3Aopen+is%3Aissue">
  <img src="https://img.shields.io/github/issues-raw/cylc/language-cylc" />
</a> <a href="https://github.com/cylc/language-cylc/pulls">
  <img src="https://img.shields.io/github/issues-pr/cylc/language-cylc" />
</a> <a href="https://github.com/cylc/language-cylc/issues?q=is%3Aissue+is%3Aopen+label%3Aquestion">
  <img src="https://img.shields.io/github/issues/cylc/language-cylc/question" />
</a> <a href="https://github.com/cylc/language-cylc/issues?q=is%3Aissue+is%3Aopen+label%3Abug">
  <img src="https://img.shields.io/github/issues/cylc/language-cylc/bug" />
</a> <a href="https://github.com/cylc/language-cylc/issues?q=is%3Aopen+no%3Amilestone">
  <img src="https://img.shields.io/github/issues-search/cylc/language-cylc?query=is%3Aopen%20no%3Amilestone&label=no%20milestone" />
</a>

<table style="margin-left:20px; margin-top:10px">

</table>


### cylc/cylc.vim

<a href="https://github.com/cylc/cylc.vim/issues/?q=is%3Aopen+is%3Aissue">
  <img src="https://img.shields.io/github/issues-raw/cylc/cylc.vim" />
</a> <a href="https://github.com/cylc/cylc.vim/pulls">
  <img src="https://img.shields.io/github/issues-pr/cylc/cylc.vim" />
</a> <a href="https://github.com/cylc/cylc.vim/issues?q=is%3Aissue+is%3Aopen+label%3Aquestion">
  <img src="https://img.shields.io/github/issues/cylc/cylc.vim/question" />
</a> <a href="https://github.com/cylc/cylc.vim/issues?q=is%3Aissue+is%3Aopen+label%3Abug">
  <img src="https://img.shields.io/github/issues/cylc/cylc.vim/bug" />
</a> <a href="https://github.com/cylc/cylc.vim/issues?q=is%3Aopen+no%3Amilestone">
  <img src="https://img.shields.io/github/issues-search/cylc/cylc.vim?query=is%3Aopen%20no%3Amilestone&label=no%20milestone" />
</a>

<table style="margin-left:20px; margin-top:10px">

</table>

