### Stylesheet loading behavior

_Last updated: May 2018_
_Chris Harrelson (chrishtr@google.com)_
*Based in part on research by others, including esprehn@google.com and
pmeenan@google.com).*

This document explores the current/past behavior of browsers, use-cases and
desired semantics of style sheet loading. Note that the HTML spec's
[rendering section](https://html.spec.whatwg.org/multipage/webappapis.html#event-loop-processing-model) speak about what happens when rendering (drawing
to the screen, plus some script callbacks) occurs, but do not say how this may depend on external resource load timing with respect to the DOM and the HTML parser. In particular, it is not specified whether rendering or parsing should "block" on loads of external stylesheets, and how "blocking" works if so.
(This is as opposed to script, where the spec specifies that certain scripts block on not-yet-loaded style sheets, and parsing and rendering block on those scripts.)

The following section outline use cases, proposed spec changes, and existing behavior of each major implementation.

## Use cases

Web site developers wish to have web pages whose load characteristics are:
a. Fast
b. Reliable and understandable
c. Deterministic, when desired
d. Compatible across browser implementations

All major browser implementations today attempt to achieve (a) through some application of parallelism. In terms of style sheeets and rendering, each browser's implementation of loading parallelism is somewhat unique, leading to deficiencies in achieving the other three desired characteristics. For example, (b) is hampered at the very least by having to understand four differnet approaches; (c) is affected because parallelism can lead to non-determinism, and (d) is self-evident since browser implementations differ.

Web best practices also encourage achieving (a) via progressive rendering techniques. In terms of style sheets and rendering, this usually means showing some HTML content before all style sheets have loaded ("obtained" in HTML spec language), then potentially updating rendering once more are available over time, under the assumption that the network is the loading bottleneck. (In addition, as the network becomes less of the bottleneck to loading pages, the problems outlined in this document become more important to address, not less, since reliability, determinism and compatibility begin to dominate in importance.)

Parallelism and progressive rendering techniques can also result in a "flash of unstyled content" (FOUC) if the parsed DOM for some HTML must be rendered before its corresponding style sheet(s) are obtained. Hard-to-control FOUC is a form of failure to achieve (b).

## Proposed behavior

A proposed set of edits to the HTML spec are [here](stylesheet-loading-proposal.md).

## Existing behavior

The following is a best understanding of the current behavior of each major browser implementation based on past research, and may be partially incorrect.

Definitions:
  * *FOUC*: Flash of Unstyled Content. Rendering of DOM content given currently-loaded resources (i.e. not blocking rendering on still-pending style sheets).
  * *Forced Style Recalc*: the situation when a script reads computed style on any DOM element, therefore forcing its computed style to be computed. (Note that Forced Style Recalcs always give "FOUC answers" to the computed style property queried, meaning that the result does not block on any style sheet not yet obtained. This is because the APIs to access computed styles are synchronous.)
  * *Painting a DOM element*: drawing the content of the element to the screen. Conceptually the same thing as the painting phases of the element in the
  [painting algorithm](https://www.w3.org/TR/CSS21/zindex.html).
  * *rAF callbacks*: these are [animation frame callbacks](https://html.spec.whatwg.org/#list-of-animation-frame-callbacks).
  * *HTML Parser*: [this](https://html.spec.whatwg.org/#parsing).
  * *Rendering*: step 7 of the event loop. Includes Painting DOM Elements and rAF callbacks, plus other steps such as Resize Observers.

# Gecko
  * Blocks Painting DOM Elements on obtaining external style sheets declared as a descendant of `<head>`, except for those declared via [`@import`](https://developer.mozilla.org/en-US/docs/Web/CSS/@import) within another style sheet.
  * Any Forced Style Recalc results in FOUC, even if the parser is still proceessing the `<head>`.
  * rAF callbacks begin running immediately once the page starts loading, even before Painting DOM Elements occurs.
  * FOUC-avoidance strategy: put all critical style sheets in the `<head>`; don't use `@import`; dont force style recalc during load. Non-critical style sheets should be put in `<body>` and before all DOM that they apply to, and can be followed by an empty script block to force the parser to block on style sheet load.

# WebKit
  * Blocks Painting DOM Elements that are parser-inserted on obtaining any style sheets that are not yet obtained at the time of insertion. This is true regadless of whether the element was a descendant of `<head>` or
    `<body>`, and includes style sheets declared in `@import` from another style sheet.
  * Forced Style Recalc does not result in Painting DOM Elements, but there are some buggy edge cases.
  * rAF callbacks begin running immediately once the page starts loading, even before Painting DOM Elements starts.
  * FOUC-avoidance strategy: put all critical style sheets in the `<head>`; don't force style recalc during load. Non-critical style sheets should be put in `<body>`, and before all DOM that they apply to. To ensure maximum progressiveness of rendering, they can be followed by an empty script block to force the parser to block on style sheet load.

# Edge
  * Blocks Painting DOM Elements on obtaining all style sheets declared as a descendant of `<head>`, including those declared via `@import` within another style sheet.
  * Blocks the HTML Parser for all style sheets, even descendants of `<head>`, except for style sheets declared via `@import` within another style sheet.
  * `@import`-declared style sheets are applied to Painting DOM Elements as they come in, and are not applied atomically, even if they are @imported from the same style sheet parent.
  * Forced Style Recalc does not result in Painting DOM Elements if the parser is still in `<head>`; otherwise Painting DOM Elements is forced and a FOUC results.
  * rAF callbacks begin running immediately once the page starts loading, even before Painting DOM Elements starts.
  * FOUC-avoidance strategy: put all critical style sheets in the `<head>`; do not use `@import`; don't force style recalc during load. Non-critical style sheets should be put in `<body>`, and before all DOM that they apply to.

# Blink (before launch of [this intent-to-ship](https://groups.google.com/a/chromium.org/forum/#!topic/blink-dev/QC5iefctcag))
  * Block Painting DOM Elements of *all elements* (not just those preceded by a sheet) for any parser-inserted style sheet, regardless of where it is in the DOM, and even for those declared via `@import` within another style sheet.
  * Forced Style Recalc does not result in Painting DOM Elements, but there are some buggy edge cases.
  * rAF callbacks begin once all style sheets, including those declared inside `@import` of another style sheet, which are present before `<body>`, are loaded.
  * FOUC-avoidance strategy: put all critical style sheets in the `<head>`; don't force style recalc during load. Non-critical style sheets should be put in `<body>`. To ensure maximum progressiveness of rendering, they can be followed by an empty script block to force the parser to block on style sheet load.

# Blink (after launch)
  * Start Rendering after observing the first `<body>` tag and all parser-inserted style sheets before that time have obtained, including those declared via `@import` from another style sheet.
  * Block the HTML Parser for all style sheets that are parser-inserted after the first `<body>` tag, including those declared via `@import` from another style sheet.
  * FOUC-avoidance strategy: put all critical style sheets in the `<head>`. Non-critical style sheets should be put in `<body>` and before all DOM that they apply to.

# Blink (longer term) (!)

  * Start Rendering after observing the first `<body>` tag and and all parser-inserted style sheets before that time have obtained, including those declared via `@import` from another style sheet.
  * Block the HTML Parser for all style sheets, including those declared via
  `@import` from another style sheet, regardless of location in DOM.
  * FOUC-avoidance strategy: put style sheets before DOM that they apply to.


 (!) This can happen once [HTML Imports](https://developer.mozilla.org/en-US/docs/Web/Web_Components/HTML_Imports) are gone, since the Blink preload scanner doesn't understand HTML Imports. At this point Blink will be fully compliant with the [proposed spec change](stylesheet-loading-proposal.md).