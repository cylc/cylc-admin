# Cylc GUI discussion @ Suites Guild
2019 06 06
A discussion with a group of (mainly power) users about what they like/dislike
about using the current cylc GUIs.
Hosts: Tim Pillinger (Editor), Oliver Sanders

[Editor's Notes Attempting to add context in square brackets]

These are:
* Cylc GUI (gcylc)
* Cylc Gscan
* Cylc Graph
* Cylc Graph-diff
* Cylc Gpanel
* Cylc Namespace Grapher (cylc graph -n)

---

## Usage of GUIs & GUI vs CLI comments
#### Table
| cylc...| All the time | Most Days | Most Weeks | Infreqently | Never|Honestly, I didn't know it existed |
| --- | --- | --- | --- | --- | --- |--|
|gui (gcylc)|16|4||2|||
|gscan|8|1| |6|6||
|graph|||1|11 |8|1|
|graph-diff||||4|10 |7|
|gpanel|3|1|| |2|11|
|graph -n (namespace grapher)|||| |5|16||

#### Are there tasks that you do a lot either in the gui or the command line?

* Starting the suite - Slight majority use CLI, one person mentions wrapping
  CLI in scripts or suites[?!]
* Stop a suite - about 50-50
* Create pictures for presentations - lines too thin.
* GUI good for inserting tasks, holding suites, triggering tasks

#### Other comments
* Advertise Plug Button!
* Difference between nudging and polling tasks [is unclear?]
* Namespace graph too big/complex for many suites. Needs better navigation.
* Lots of love for gpanel (when it worked). Request for RHEL7 version.
* Graph diff is rarely used, but very useful when it is.

---

### Usage of Cylc GUI
#### Are there common gotchas in the cylc guis?
* [I find it confusing that] task states are ordererd opposite in Gcylc and
  Gscan
* Controls that behave unexpectedly
  * All the time on state and name
  * Could it filter differently for different views?
* Triggering task inheritance sometimes does wierd things

#### Are there capabilities in the Cylc GUI's that made your life easier whe you found out about them?
* Simulation mode
* Using Wildcard to insert tasks
* Being able to manually set the state of a task is useful for development
  and debugging.
* Triggering with multiple tasks is good.
* Multiple selection really helpful in gcylc
* Useful feature - stop suite after named cycle.

#### Do you use task filtering? If you do, what do you use it for?

#### Do you use hierachically named suites?
* What are hierachically named suites?
  * Those that know what they are don't use them [at least in this room, if you
    read this on Sharepoint and disagree...].
* Not many of us [use them] but seems useful.
* Not well known.
* Iteration over experiments is desirable in suites.

#### How do you find out more about a Cylc GUI if you need to?
* Metomi team
* Yammer
* Neighbour
* [is it just me, or are the docs notably absent from this answer? ]

#### Other
* Editing job script to test things without editing the suite. [is this good,
  or a gotcha?]
* Menu item that does what cylc-trigger does is useful.

---

### Appearance & Usability
#### Do you have issues with GUI perfomance (question implied from answers)?
* Slow start up with complex graphs.
* Occasional gui/gscan dropouts.
* Suite and/or GUI dissapears
* Scrolling through large graphs/lists cna be slow/freeze
* Viewing job log files sometimes fails repeatedly.

#### Do you have any difficulty reading/seeing any part of the GUI? If so why?

* Can be tricky to see what you want from gcylc graphs
* Helpful to be able to change the colour of the task states.

#### Do you theme the task states? Did you know you could?
* Customizing the GUI would be good. (perhaps on a per-suite basis)
* Theming task states - Some people have never heard of it, some knew but
  haven't used them, and one or two people use them.

#### How many controls in a cylc gui do you use regularly, and why?
* "I _never_ use the graph view".
  * This looks contested - someone else wrote "I _ALWAYS_ use this feature"
  * Probably depends on type of suite
  * Suites like UM rose-stem have too many tasks and too few cycles.

#### Are there items/buttons/features in the menu you see but never use?

#### Does it take too many mouse/keyboard clicks/strokes to carry out some tasks?
* Inserting tasks is labourious
* More keyboard shortcuts.
* More obvious keyboard shortcuts - make it clearer what ctrl + .... does
* Customizable keyboard shortcuts.

#### Other Responses
* Popup "stop-suite" panel is well designed and informative.
* "KILL NOW" Useful
* Multiple task selection is good
* Double click functionality is not [always?] obvious?
* Screenshot functionality (bit like cylc graph) might be handy.

---

### Notes on future Cylc GUI Responses
* Some did not like the loss of the separate app window.
  * --- suggests that the gui could be run in a separate browser
    profile.
  * We might want to look fro a browser => native app conversion tool.
* --- suggested the ability to customize menus might be good.
* --- asked if there would still be a colour blind theme. Was
  told that there would be a colour blind safe theme.
* Nesting suites are a "monster of jinja2". 👹
* Would be good to fiter suites by cycle.
* More control of graph view, esp without a mouse 🖱️
* Can we stagger Alignment of task names in graph view:
* Can we filter by tasks prequesite to a given task?
* Can we focus on tasks < N links from task?
* Can we have interfaces with other software such as NIWA's Paraview?
  Talk to NIWA contacts to see if this is possible. What about Jupyter
  Notebook, HPC status page, Yammer....?
* Are there alternative visualizations being considered? Yes, the most
  prominent in our current thinking is the Gantt view, although having got the
  data into a rendering engine new views might be fairly ok to implement.


### Other (From MS notes)
We may need to have a tool to collect and analyse usage behind the scene.
Nesting suite reload not copying files? Bug?
