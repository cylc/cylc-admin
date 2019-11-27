# Work on Platforms

The description of work to be done collected here rather than based on more
transitory tickets.

Based on notes originally created by @OSanders.

## Approaches:
1. Add the new configurations but don't use them at first, do all the the
   upgrader stuff then change over towards the end.
   Means you have two sets of configuration hanging about.

2. Remove the old configurations then add the upgrader stuff towards the
   Will break master, so either you will need to work on a "feature"
   branch or else others will have to maintain their own "feature"

3. Hybrid approach - Work mainly on master, but create a feature branch for
   PR's related to the "wireing" ticket, which _will_ break master.

Do what you think best, there are pros and cons to both approaches.

If we can avoid breaking master that would make life easier for everyone.
If we do break master it will effectively halt all Cylc Flow work until
its done, in which case it should probably become a more international
effort so that we get Cylc Flow development back on track quickly. Otherwise
you will have to work on a feature branch and accept any rebase mess which

I think Cylc Flow might actually be heading towards a quiet patch as we
move our focus downstream to the UI Server and UI. There may be changes
to the GraphQL stuff in Cylc Flow but that wont create conflicts. However,

## ​Blocking PRs:

(these things block the "wireing" portion of the work):
* Python Endpoints
* ZMQ Curve Authentication
* Incremental Data Store Updates

## Overview

<svg id="mermaid-1574848281498" width="1162" xmlns="http://www.w3.org/2000/svg" height="239" viewBox="0 0 1162 239"><style>



#mermaid-1574848281498 .label {
  font-family: 'trebuchet ms', verdana, arial;
  font-family: var(--mermaid-font-family);
  color: #333; }

#mermaid-1574848281498 .label text {
  fill: #333; }

#mermaid-1574848281498 .node rect,
#mermaid-1574848281498 .node circle,
#mermaid-1574848281498 .node ellipse,
#mermaid-1574848281498 .node polygon {
  fill: #ECECFF;
  stroke: #9370DB;
  stroke-width: 1px; }

#mermaid-1574848281498 .node .label {
  text-align: center; }

#mermaid-1574848281498 .node.clickable {
  cursor: pointer; }

#mermaid-1574848281498 .arrowheadPath {
  fill: #333333; }

#mermaid-1574848281498 .edgePath .path {
  stroke: #333333;
  stroke-width: 1.5px; }

#mermaid-1574848281498 .edgeLabel {
  background-color: #e8e8e8;
  text-align: center; }

#mermaid-1574848281498 .cluster rect {
  fill: #ffffde;
  stroke: #aaaa33;
  stroke-width: 1px; }

#mermaid-1574848281498 .cluster text {
  fill: #333; }

#mermaid-1574848281498 div.mermaidTooltip {
  position: absolute;
  text-align: center;
  max-width: 200px;
  padding: 2px;
  font-family: 'trebuchet ms', verdana, arial;
  font-family: var(--mermaid-font-family);
  font-size: 12px;
  background: #ffffde;
  border: 1px solid #aaaa33;
  border-radius: 2px;
  pointer-events: none;
  z-index: 100; }

#mermaid-1574848281498 .actor {
  stroke: #CCCCFF;
  fill: #ECECFF; }

#mermaid-1574848281498 text.actor {
  fill: black;
  stroke: none; }

#mermaid-1574848281498 .actor-line {
  stroke: grey; }

#mermaid-1574848281498 .messageLine0 {
  stroke-width: 1.5;
  stroke-dasharray: '2 2';
  stroke: #333; }

#mermaid-1574848281498 .messageLine1 {
  stroke-width: 1.5;
  stroke-dasharray: '2 2';
  stroke: #333; }

#mermaid-1574848281498 #arrowhead {
  fill: #333; }

#mermaid-1574848281498 .sequenceNumber {
  fill: white; }

#mermaid-1574848281498 #sequencenumber {
  fill: #333; }

#mermaid-1574848281498 #crosshead path {
  fill: #333 !important;
  stroke: #333 !important; }

#mermaid-1574848281498 .messageText {
  fill: #333;
  stroke: none; }

#mermaid-1574848281498 .labelBox {
  stroke: #CCCCFF;
  fill: #ECECFF; }

#mermaid-1574848281498 .labelText {
  fill: black;
  stroke: none; }

#mermaid-1574848281498 .loopText {
  fill: black;
  stroke: none; }

#mermaid-1574848281498 .loopLine {
  stroke-width: 2;
  stroke-dasharray: '2 2';
  stroke: #CCCCFF; }

#mermaid-1574848281498 .note {
  stroke: #aaaa33;
  fill: #fff5ad; }

#mermaid-1574848281498 .noteText {
  fill: black;
  stroke: none;
  font-family: 'trebuchet ms', verdana, arial;
  font-family: var(--mermaid-font-family);
  font-size: 14px; }

#mermaid-1574848281498 .activation0 {
  fill: #f4f4f4;
  stroke: #666; }

#mermaid-1574848281498 .activation1 {
  fill: #f4f4f4;
  stroke: #666; }

#mermaid-1574848281498 .activation2 {
  fill: #f4f4f4;
  stroke: #666; }


#mermaid-1574848281498 .mermaid-main-font {
  font-family: "trebuchet ms", verdana, arial;
  font-family: var(--mermaid-font-family); }

#mermaid-1574848281498 .section {
  stroke: none;
  opacity: 0.2; }

#mermaid-1574848281498 .section0 {
  fill: rgba(102, 102, 255, 0.49); }

#mermaid-1574848281498 .section2 {
  fill: #fff400; }

#mermaid-1574848281498 .section1,
#mermaid-1574848281498 .section3 {
  fill: white;
  opacity: 0.2; }

#mermaid-1574848281498 .sectionTitle0 {
  fill: #333; }

#mermaid-1574848281498 .sectionTitle1 {
  fill: #333; }

#mermaid-1574848281498 .sectionTitle2 {
  fill: #333; }

#mermaid-1574848281498 .sectionTitle3 {
  fill: #333; }

#mermaid-1574848281498 .sectionTitle {
  text-anchor: start;
  font-size: 11px;
  text-height: 14px;
  font-family: 'trebuchet ms', verdana, arial;
  font-family: var(--mermaid-font-family); }


#mermaid-1574848281498 .grid .tick {
  stroke: lightgrey;
  opacity: 0.3;
  shape-rendering: crispEdges; }
#mermaid-1574848281498   .grid .tick text {
    font-family: 'trebuchet ms', verdana, arial;
    font-family: var(--mermaid-font-family); }

#mermaid-1574848281498 .grid path {
  stroke-width: 0; }


#mermaid-1574848281498 .today {
  fill: none;
  stroke: red;
  stroke-width: 2px; }



#mermaid-1574848281498 .task {
  stroke-width: 2; }

#mermaid-1574848281498 .taskText {
  text-anchor: middle;
  font-family: 'trebuchet ms', verdana, arial;
  font-family: var(--mermaid-font-family); }

#mermaid-1574848281498 .taskText:not([font-size]) {
  font-size: 11px; }

#mermaid-1574848281498 .taskTextOutsideRight {
  fill: black;
  text-anchor: start;
  font-size: 11px;
  font-family: 'trebuchet ms', verdana, arial;
  font-family: var(--mermaid-font-family); }

#mermaid-1574848281498 .taskTextOutsideLeft {
  fill: black;
  text-anchor: end;
  font-size: 11px; }


#mermaid-1574848281498 .task.clickable {
  cursor: pointer; }

#mermaid-1574848281498 .taskText.clickable {
  cursor: pointer;
  fill: #003163 !important;
  font-weight: bold; }

#mermaid-1574848281498 .taskTextOutsideLeft.clickable {
  cursor: pointer;
  fill: #003163 !important;
  font-weight: bold; }

#mermaid-1574848281498 .taskTextOutsideRight.clickable {
  cursor: pointer;
  fill: #003163 !important;
  font-weight: bold; }


#mermaid-1574848281498 .taskText0,
#mermaid-1574848281498 .taskText1,
#mermaid-1574848281498 .taskText2,
#mermaid-1574848281498 .taskText3 {
  fill: white; }

#mermaid-1574848281498 .task0,
#mermaid-1574848281498 .task1,
#mermaid-1574848281498 .task2,
#mermaid-1574848281498 .task3 {
  fill: #8a90dd;
  stroke: #534fbc; }

#mermaid-1574848281498 .taskTextOutside0,
#mermaid-1574848281498 .taskTextOutside2 {
  fill: black; }

#mermaid-1574848281498 .taskTextOutside1,
#mermaid-1574848281498 .taskTextOutside3 {
  fill: black; }


#mermaid-1574848281498 .active0,
#mermaid-1574848281498 .active1,
#mermaid-1574848281498 .active2,
#mermaid-1574848281498 .active3 {
  fill: #bfc7ff;
  stroke: #534fbc; }

#mermaid-1574848281498 .activeText0,
#mermaid-1574848281498 .activeText1,
#mermaid-1574848281498 .activeText2,
#mermaid-1574848281498 .activeText3 {
  fill: black !important; }


#mermaid-1574848281498 .done0,
#mermaid-1574848281498 .done1,
#mermaid-1574848281498 .done2,
#mermaid-1574848281498 .done3 {
  stroke: grey;
  fill: lightgrey;
  stroke-width: 2; }

#mermaid-1574848281498 .doneText0,
#mermaid-1574848281498 .doneText1,
#mermaid-1574848281498 .doneText2,
#mermaid-1574848281498 .doneText3 {
  fill: black !important; }


#mermaid-1574848281498 .crit0,
#mermaid-1574848281498 .crit1,
#mermaid-1574848281498 .crit2,
#mermaid-1574848281498 .crit3 {
  stroke: #ff8888;
  fill: red;
  stroke-width: 2; }

#mermaid-1574848281498 .activeCrit0,
#mermaid-1574848281498 .activeCrit1,
#mermaid-1574848281498 .activeCrit2,
#mermaid-1574848281498 .activeCrit3 {
  stroke: #ff8888;
  fill: #bfc7ff;
  stroke-width: 2; }

#mermaid-1574848281498 .doneCrit0,
#mermaid-1574848281498 .doneCrit1,
#mermaid-1574848281498 .doneCrit2,
#mermaid-1574848281498 .doneCrit3 {
  stroke: #ff8888;
  fill: lightgrey;
  stroke-width: 2;
  cursor: pointer;
  shape-rendering: crispEdges; }

#mermaid-1574848281498 .milestone {
  transform: rotate(45deg) scale(0.8, 0.8); }

#mermaid-1574848281498 .milestoneText {
  font-style: italic; }

#mermaid-1574848281498 .doneCritText0,
#mermaid-1574848281498 .doneCritText1,
#mermaid-1574848281498 .doneCritText2,
#mermaid-1574848281498 .doneCritText3 {
  fill: black !important; }

#mermaid-1574848281498 .activeCritText0,
#mermaid-1574848281498 .activeCritText1,
#mermaid-1574848281498 .activeCritText2,
#mermaid-1574848281498 .activeCritText3 {
  fill: black !important; }

#mermaid-1574848281498 .titleText {
  text-anchor: middle;
  font-size: 18px;
  fill: black;
  font-family: 'trebuchet ms', verdana, arial;
  font-family: var(--mermaid-font-family); }

#mermaid-1574848281498 g.classGroup text {
  fill: #9370DB;
  stroke: none;
  font-family: 'trebuchet ms', verdana, arial;
  font-family: var(--mermaid-font-family);
  font-size: 10px; }
#mermaid-1574848281498   g.classGroup text .title {
    font-weight: bolder; }

#mermaid-1574848281498 g.classGroup rect {
  fill: #ECECFF;
  stroke: #9370DB; }

#mermaid-1574848281498 g.classGroup line {
  stroke: #9370DB;
  stroke-width: 1; }

#mermaid-1574848281498 .classLabel .box {
  stroke: none;
  stroke-width: 0;
  fill: #ECECFF;
  opacity: 0.5; }

#mermaid-1574848281498 .classLabel .label {
  fill: #9370DB;
  font-size: 10px; }

#mermaid-1574848281498 .relation {
  stroke: #9370DB;
  stroke-width: 1;
  fill: none; }

#mermaid-1574848281498 #compositionStart {
  fill: #9370DB;
  stroke: #9370DB;
  stroke-width: 1; }

#mermaid-1574848281498 #compositionEnd {
  fill: #9370DB;
  stroke: #9370DB;
  stroke-width: 1; }

#mermaid-1574848281498 #aggregationStart {
  fill: #ECECFF;
  stroke: #9370DB;
  stroke-width: 1; }

#mermaid-1574848281498 #aggregationEnd {
  fill: #ECECFF;
  stroke: #9370DB;
  stroke-width: 1; }

#mermaid-1574848281498 #dependencyStart {
  fill: #9370DB;
  stroke: #9370DB;
  stroke-width: 1; }

#mermaid-1574848281498 #dependencyEnd {
  fill: #9370DB;
  stroke: #9370DB;
  stroke-width: 1; }

#mermaid-1574848281498 #extensionStart {
  fill: #9370DB;
  stroke: #9370DB;
  stroke-width: 1; }

#mermaid-1574848281498 #extensionEnd {
  fill: #9370DB;
  stroke: #9370DB;
  stroke-width: 1; }

#mermaid-1574848281498 .commit-id,
#mermaid-1574848281498 .commit-msg,
#mermaid-1574848281498 .branch-label {
  fill: lightgrey;
  color: lightgrey;
  font-family: 'trebuchet ms', verdana, arial;
  font-family: var(--mermaid-font-family); }

#mermaid-1574848281498 .pieTitleText {
  text-anchor: middle;
  font-size: 25px;
  fill: black;
  font-family: 'trebuchet ms', verdana, arial;
  font-family: var(--mermaid-font-family); }

#mermaid-1574848281498 .slice {
  font-family: 'trebuchet ms', verdana, arial;
  font-family: var(--mermaid-font-family); }

#mermaid-1574848281498 g.stateGroup text {
  fill: #9370DB;
  stroke: none;
  font-size: 10px;
  font-family: 'trebuchet ms', verdana, arial;
  font-family: var(--mermaid-font-family); }

#mermaid-1574848281498 g.stateGroup text {
  fill: #9370DB;
  stroke: none;
  font-size: 10px; }

#mermaid-1574848281498 g.stateGroup .state-title {
  font-weight: bolder;
  fill: black; }

#mermaid-1574848281498 g.stateGroup rect {
  fill: #ECECFF;
  stroke: #9370DB; }

#mermaid-1574848281498 g.stateGroup line {
  stroke: #9370DB;
  stroke-width: 1; }

#mermaid-1574848281498 .transition {
  stroke: #9370DB;
  stroke-width: 1;
  fill: none; }

#mermaid-1574848281498 .stateGroup .composit {
  fill: white;
  border-bottom: 1px; }

#mermaid-1574848281498 .state-note {
  stroke: #aaaa33;
  fill: #fff5ad; }
#mermaid-1574848281498   .state-note text {
    fill: black;
    stroke: none;
    font-size: 10px; }

#mermaid-1574848281498 .stateLabel .box {
  stroke: none;
  stroke-width: 0;
  fill: #ECECFF;
  opacity: 0.5; }

#mermaid-1574848281498 .stateLabel text {
  fill: black;
  font-size: 10px;
  font-weight: bold;
  font-family: 'trebuchet ms', verdana, arial;
  font-family: var(--mermaid-font-family); }

:root {
  --mermaid-font-family: '"trebuchet ms", verdana, arial';
  --mermaid-font-family: "Comic Sans MS", "Comic Sans", cursive; }

:root { --mermaid-font-family: "trebuchet ms", verdana, arial;}</style><style>#mermaid-1574848281498 {
    color: rgba(0, 0, 0, 0.65);
    font: ;
  }</style><g transform="translate(0, 0)"><g class="output"><g class="clusters"></g><g class="edgePaths"><g class="edgePath" style="opacity: 1;"><path class="path" d="M463.25,35.178899082568805L68,74L68,99" marker-end="url(https://mermaidjs.github.io/mermaid-live-editor/#arrowhead6952)" style="fill:none"></path><defs><marker id="arrowhead6952" viewBox="0 0 10 10" refX="9" refY="5" markerUnits="strokeWidth" markerWidth="8" markerHeight="6" orient="auto"><path d="M 0 0 L 10 5 L 0 10 z" class="arrowheadPath" style="stroke-width: 1px; stroke-dasharray: 1px, 0px;"></path></marker></defs></g><g class="edgePath" style="opacity: 1;"><path class="path" d="M599.25,35.178899082568805L994.5,74L994.5,99" marker-end="url(https://mermaidjs.github.io/mermaid-live-editor/#arrowhead6953)" style="fill:none"></path><defs><marker id="arrowhead6953" viewBox="0 0 10 10" refX="9" refY="5" markerUnits="strokeWidth" markerWidth="8" markerHeight="6" orient="auto"><path d="M 0 0 L 10 5 L 0 10 z" class="arrowheadPath" style="stroke-width: 1px; stroke-dasharray: 1px, 0px;"></path></marker></defs></g><g class="edgePath" style="opacity: 1;"><path class="path" d="M953.9505494505495,140L904.5,165L904.5,190" marker-end="url(https://mermaidjs.github.io/mermaid-live-editor/#arrowhead6954)" style="fill:none"></path><defs><marker id="arrowhead6954" viewBox="0 0 10 10" refX="9" refY="5" markerUnits="strokeWidth" markerWidth="8" markerHeight="6" orient="auto"><path d="M 0 0 L 10 5 L 0 10 z" class="arrowheadPath" style="stroke-width: 1px; stroke-dasharray: 1px, 0px;"></path></marker></defs></g><g class="edgePath" style="opacity: 1;"><path class="path" d="M1035.0494505494505,140L1084.5,165L1084.5,190" marker-end="url(https://mermaidjs.github.io/mermaid-live-editor/#arrowhead6955)" style="fill:none"></path><defs><marker id="arrowhead6955" viewBox="0 0 10 10" refX="9" refY="5" markerUnits="strokeWidth" markerWidth="8" markerHeight="6" orient="auto"><path d="M 0 0 L 10 5 L 0 10 z" class="arrowheadPath" style="stroke-width: 1px; stroke-dasharray: 1px, 0px;"></path></marker></defs></g><g class="edgePath" style="opacity: 1;"><path class="path" d="M68,140L68,165L334,204.61702127659575" marker-end="url(https://mermaidjs.github.io/mermaid-live-editor/#arrowhead6956)" style="fill:none"></path><defs><marker id="arrowhead6956" viewBox="0 0 10 10" refX="9" refY="5" markerUnits="strokeWidth" markerWidth="8" markerHeight="6" orient="auto"><path d="M 0 0 L 10 5 L 0 10 z" class="arrowheadPath" style="stroke-width: 1px; stroke-dasharray: 1px, 0px;"></path></marker></defs></g><g class="edgePath" style="opacity: 1;"><path class="path" d="M217.5,140L217.5,165L334,198.97916666666666" marker-end="url(https://mermaidjs.github.io/mermaid-live-editor/#arrowhead6957)" style="fill:none"></path><defs><marker id="arrowhead6957" viewBox="0 0 10 10" refX="9" refY="5" markerUnits="strokeWidth" markerWidth="8" markerHeight="6" orient="auto"><path d="M 0 0 L 10 5 L 0 10 z" class="arrowheadPath" style="stroke-width: 1px; stroke-dasharray: 1px, 0px;"></path></marker></defs></g><g class="edgePath" style="opacity: 1;"><path class="path" d="M373.5,140L373.5,165L373.5,190" marker-end="url(https://mermaidjs.github.io/mermaid-live-editor/#arrowhead6958)" style="fill:none"></path><defs><marker id="arrowhead6958" viewBox="0 0 10 10" refX="9" refY="5" markerUnits="strokeWidth" markerWidth="8" markerHeight="6" orient="auto"><path d="M 0 0 L 10 5 L 0 10 z" class="arrowheadPath" style="stroke-width: 1px; stroke-dasharray: 1px, 0px;"></path></marker></defs></g><g class="edgePath" style="opacity: 1;"><path class="path" d="M550.5,140L550.5,165L413,200.34604519774012" marker-end="url(https://mermaidjs.github.io/mermaid-live-editor/#arrowhead6959)" style="fill:none"></path><defs><marker id="arrowhead6959" viewBox="0 0 10 10" refX="9" refY="5" markerUnits="strokeWidth" markerWidth="8" markerHeight="6" orient="auto"><path d="M 0 0 L 10 5 L 0 10 z" class="arrowheadPath" style="stroke-width: 1px; stroke-dasharray: 1px, 0px;"></path></marker></defs></g><g class="edgePath" style="opacity: 1;"><path class="path" d="M773,140L773,165L413,206.00125156445557" marker-end="url(https://mermaidjs.github.io/mermaid-live-editor/#arrowhead6960)" style="fill:none"></path><defs><marker id="arrowhead6960" viewBox="0 0 10 10" refX="9" refY="5" markerUnits="strokeWidth" markerWidth="8" markerHeight="6" orient="auto"><path d="M 0 0 L 10 5 L 0 10 z" class="arrowheadPath" style="stroke-width: 1px; stroke-dasharray: 1px, 0px;"></path></marker></defs></g></g><g class="edgeLabels"><g class="edgeLabel" style="opacity: 1;" transform=""><g transform="translate(0,0)" class="label"><foreignObject width="0" height="0"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="edgeLabel"></span></div></foreignObject></g></g><g class="edgeLabel" style="opacity: 1;" transform=""><g transform="translate(0,0)" class="label"><foreignObject width="0" height="0"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="edgeLabel"></span></div></foreignObject></g></g><g class="edgeLabel" style="opacity: 1;" transform=""><g transform="translate(0,0)" class="label"><foreignObject width="0" height="0"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="edgeLabel"></span></div></foreignObject></g></g><g class="edgeLabel" style="opacity: 1;" transform=""><g transform="translate(0,0)" class="label"><foreignObject width="0" height="0"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="edgeLabel"></span></div></foreignObject></g></g><g class="edgeLabel" style="opacity: 1;" transform=""><g transform="translate(0,0)" class="label"><foreignObject width="0" height="0"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="edgeLabel"></span></div></foreignObject></g></g><g class="edgeLabel" style="opacity: 1;" transform=""><g transform="translate(0,0)" class="label"><foreignObject width="0" height="0"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="edgeLabel"></span></div></foreignObject></g></g><g class="edgeLabel" style="opacity: 1;" transform=""><g transform="translate(0,0)" class="label"><foreignObject width="0" height="0"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="edgeLabel"></span></div></foreignObject></g></g><g class="edgeLabel" style="opacity: 1;" transform=""><g transform="translate(0,0)" class="label"><foreignObject width="0" height="0"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="edgeLabel"></span></div></foreignObject></g></g><g class="edgeLabel" style="opacity: 1;" transform=""><g transform="translate(0,0)" class="label"><foreignObject width="0" height="0"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="edgeLabel"></span></div></foreignObject></g></g></g><g class="nodes"><g class="node" style="opacity: 1;" id="A" transform="translate(531.25,28.5)"><rect rx="5" ry="5" x="-68" y="-20.5" width="136" height="41" class="label-container"></rect><g class="label" transform="translate(0,0)"><g transform="translate(-58,-10.5)"><foreignObject width="116" height="21"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;">Job Platform Spec</div></foreignObject></g></g></g><g class="node" style="opacity: 1;" id="B" transform="translate(68,119.5)"><rect rx="5" ry="5" x="-60" y="-20.5" width="120" height="41" class="label-container"></rect><g class="label" transform="translate(0,0)"><g transform="translate(-50,-10.5)"><foreignObject width="100" height="21"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;">Forward Lookup</div></foreignObject></g></g></g><g class="node" style="opacity: 1;" id="C" transform="translate(994.5,119.5)"><rect rx="5" ry="5" x="-59.5" y="-20.5" width="119" height="41" class="label-container"></rect><g class="label" transform="translate(0,0)"><g transform="translate(-49.5,-10.5)"><foreignObject width="99" height="21"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;">Reverse Lookup</div></foreignObject></g></g></g><g class="node" style="opacity: 1;" id="D" transform="translate(904.5,210.5)"><rect rx="5" ry="5" x="-60.5" y="-20.5" width="121" height="41" class="label-container"></rect><g class="label" transform="translate(0,0)"><g transform="translate(-50.5,-10.5)"><foreignObject width="101" height="21"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;">Config Upgrader</div></foreignObject></g></g></g><g class="node" style="opacity: 1;" id="E" transform="translate(1084.5,210.5)"><rect rx="5" ry="5" x="-69.5" y="-20.5" width="139" height="41" class="label-container"></rect><g class="label" transform="translate(0,0)"><g transform="translate(-59.5,-10.5)"><foreignObject width="119" height="21"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;">Database Upgrader</div></foreignObject></g></g></g><g class="node" style="opacity: 1;" id="F" transform="translate(373.5,210.5)"><rect rx="5" ry="5" x="-39.5" y="-20.5" width="79" height="41" class="label-container"></rect><g class="label" transform="translate(0,0)"><g transform="translate(-29.5,-10.5)"><foreignObject width="59" height="21"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;">Wireing 2</div></foreignObject></g></g></g><g class="node" style="opacity: 1;" id="G" transform="translate(217.5,119.5)"><rect rx="5" ry="5" x="-39.5" y="-20.5" width="79" height="41" class="label-container"></rect><g class="label" transform="translate(0,0)"><g transform="translate(-29.5,-10.5)"><foreignObject width="59" height="21"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;">Wireing 1</div></foreignObject></g></g></g><g class="node" style="opacity: 1;" id="H" transform="translate(373.5,119.5)"><rect rx="0" ry="0" x="-66.5" y="-20.5" width="133" height="41" class="label-container" style="fill:#ccc;"></rect><g class="label" transform="translate(0,0)"><g transform="translate(-56.5,-10.5)"><foreignObject width="113" height="21"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;">Python Endpoints</div></foreignObject></g></g></g><g class="node" style="opacity: 1;" id="J" transform="translate(550.5,119.5)"><rect rx="0" ry="0" x="-60.5" y="-20.5" width="121" height="41" class="label-container" style="fill:#ccc;"></rect><g class="label" transform="translate(0,0)"><g transform="translate(-50.5,-10.5)"><foreignObject width="101" height="21"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;">ZMQ Curve Auth</div></foreignObject></g></g></g><g class="node" style="opacity: 1;" id="K" transform="translate(773,119.5)"><rect rx="0" ry="0" x="-112" y="-20.5" width="224" height="41" class="label-container" style="fill:#ccc;"></rect><g class="label" transform="translate(0,0)"><g transform="translate(-102,-10.5)"><foreignObject width="204" height="21"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;">Incremental Data Store Updates</div></foreignObject></g></g></g></g></g></g></svg>

## The tasks:

### Job Platform Spec

Add job platforms section to `cylc/flow/cfgspec/global_config.rc`.
* Include the content currently included in the `[hosts][many]` section.
* Do not include the `[hosts][localhost]`

Add platform setting to the task section of `cylc/flow/cfgspec/suite.rc`.

Move Configs from `suite.rc[runtime][TASK][job/remote]` to
`flow.rc[platforms][__MANY__]`. [The diff for PR #3348](https://github.com/cylc/cylc-flow/pull/3348/files/cb83a8ac04ac567488ae97061ee0f934ffc46bbd..ae70eafc03e045ce6255a8bb5910ca208991fb65) should provide further details.

Remove old configurations.

### Forward Lookup
PREREQS: "job platform spec"

How Cylc 8 will pick a job platform using `[runtime][TASK]platform=MyPlatform`

Write a function which takes the "platform" field from the task config
forward(platform_name_or_function) -> job platform dict

This will probably end up in `cfgspec/global_config.py` in the same way that
`def get_host_item()` currently is.

Note the platform_function (i.e. rose host-select) functionality. This may not
be fully testable within the scope of this ticket, since in some cases it may
not be evaluated until job submission time.

This item will need unit testing but will not be wired into cylc as part of
this ticket.

This code can be written and tested without breaking master since it will not
yet be used outside its own tests.

### Reverse Lookup
PREREQS: "job platform spec"

How Cylc 8 will pick a job platform using a Cylc 7 compatible `suite.rc`

Write a function which returns a job platform from available information

This will probably end up in `cfgspec/global_config.py` in the same way that
`def get_host_item()` currently is.

For context this information could come from the task configuration...
...or the database, where it comes from doesn't matter right now.

reverse(host, batch_sys, user=None) -> platform_name

This should return the platform name if there is exactly one possible
...or raise an error otherwise.

This work should not break master.

### Config Upgrader

PREREQS: "reverse lookup"

Write a test battery to show how the old settings should be converted
Write a function which actually does that (but for the moment doesn't
remove the old configurations).

You will probably want to consider existing tests in `tests/deprecations`
as templates for the new test or tests.

### DB Upgrader

PREREQS: "reverse lookup"

Much the same as the previous but for the database.

The database stores the host and batch_system, we would need to upgrade
This could be done towards the end, you might need to put off upgrading
the DB but instead upgrade the information comming out of the database.

### Wireing 1: Understanding Where hosts are used

PREREQS: None

Examine the code base for references to use of settings based on hosts and
record them, probably by editing the ticket below - "Wireing 2".
Replace all references to the old settings with the new ones.

​This needs to be done right at the start before upgraders etc come

### Wireing 2: Replacing the use of all settings under hosts.

PREREQS: "forward lookup", "Wireing 1"

Replace all the references to hosts with platforms.

This work is complete when you can deprecate all the old configurations

**This ticket will break master, and should almost certainly be carried out
in a feature branch**.
