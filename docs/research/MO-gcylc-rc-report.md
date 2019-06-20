# Usage Of `gcylc.rc` At The MetOffice

## Overview

* Users:
  * ~700 users in total.
  * ~150-200 simultaneous users at any one time.
  * 76 users have gcylc.rc files.
  * There isn't much customisation of `gcylc` at present.
  * A lot of people are hard-coding the default settings.

* Views:
  * The most changed setting is `initial views` people like to customise this.
  * People who work with large suites often disable the graph setting.
  * There isn't much love for the `dot` view.

* Visual impairment:
  * Two users increased the size of the task icons.
  * Three users choose the `high-contrast` theme.
  * Four users chose the `color-blind` theme.
  * There are 6 custom task/job state colour themes these appear to be driven by
    preference rather than visibility issues.

## Results

Here are the values users have set for the `gcylc.rc` settings along with
the number of times that value was used:


### `dot icon size`
* 6 `medium`
* 5 `small`
* 2 `large`

### `initial side-by-side views`
* 9 `True`
* 6 `False`
* 1 `true`

### `initial views`

* 17 `text, graph`
* 12 `text`
* 10 `graph, text`
* 7 `dot`
* 6 `text, dot`
* 5 `graph`
* 2 `graph,text`
* 1 `text,graph`
* 1 `dot, text`
* 1 `dot, none`

That boils down to:

* 48 `text`
* 34 `graph`
* 15 `dot`
* 1 `none`

### `maximum update interval`

### `sort by definition order`
* 3 `True`
* 1 `false`

### `sort column`
* 3 `state`
* 1 `none`

### `sort column ascending`
* 1 `True`

### `sub-graphs on`

### `task filter highlight color`
* 2 `PowderBlue`

### `task states to filter out`
* 1 `waiting`
* 1 `succeeded`
* 1 `runahead`
* 1 `held`

### `transpose dot`
* 1 `True`
* 1 `False`

### `transpose graph`
* 3 `True`
* 1 `true`
* 1 `False`

### `ungrouped views`
* 20 `graph`
* 10 `text`
* 10 `dot`

### `use theme`
* 7 `default`
* 5 `solid`
* 3 `high-contrast`
* 3 `color-blind`
* 2 `SMS`
* 2 `PinkRun`
* 1 `multi`
* 1 `custom`
* 1 `adam`
* 1 `Color-blind`
* 1 `Anette`
* 1 ``

### `window size`
* 3 `1080, 900`
* 2 `1250, 800`
* 1 `800, 500`
* 1 `600, 550`
* 1 `1920, 1200`
