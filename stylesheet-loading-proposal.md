* The following is a summary proposal for changes to the HTML spec regarding stylesheet loading, with the goal of improving reliability, understandability, determinism and browser compatibility, while not sacrificing performance.

** Pending parsing blocking style sheets

The HTML spec currently specifies the concept of a [pending parsing blocking script](https://html.spec.whatwg.org/#pending-parsing-blocking-script). This
will be extended to include the definition of a _pending parsing blocking
style sheet_.

A _pending parsing blocking style sheet_ is any style sheet that is parser-inserted, or declared in a parser-inserted style sheet via @import, regardless of its location in the HTML.

** HTML parser blocks in all phases

The HTML parser is changed to be blocked by any pending parsing blocking style sheets, via the same mechanism as that for _pending parsing blocking script_: the [text insertion mode](https://html.spec.whatwg.org/#parsing-main-incdata:pending-parsing-blocking-script) of the HTML parser.

** Rendering starts at <body>

Step 7 of the [event loop processing model](https://html.spec.whatwg.org/multipage/webappapis.html#event-loop-processing-model), _update the rendering_
will only begin happening once:
* A parser-inserted <body> element has been added to the DOM, and
* All style sheets specified in any <head> element inserted before that <body>
have obtained.

Once begun, step 7 will always happen in the future during the lifetime
of the document, subject to throttling or stopping notes mentioned in the Note
after step 7.3.

Further, step 7 will be specified to always use all style sheets
which [have beeen added](https://drafts.csswg.org/cssom/#add-a-css-style-sheet) to the document.

** Preload scanning defined

Add a definition of a preload scanner. The preload scanner may parse and
analyze the HTML document, or any external resources, in order to pre-load
network resources. However, any external resource specified within a style
sheet resource may only be pre-loaded if the resource is *guaranteed* to be needed to style the HTML document.

Note: the pre-load scanner may affect the order in which resources are obtained, and may induce side-effects in cases where a server providing the resource over the network maintain states, and may be observable to Service Workers. The requirement to only load resources which are guaranteed to be needed is in order to avoid (a) excessive network resource use, and (b) unexpected side-effects due to style sheet rules which have no rendering effect.

Example: if a style sheet has the rule:

.class-name {
  background-image: url("https://example.com/example.png");
}

then example.png would not be pre-loaded, because it is not guaranteed that
there will be a DOM element with class name "class-name".

However, in the following example, example.png could be pre-loaded if it the preload scanner could verify that the media query applied to the document.

@media (640x480)
body {
  background-image: url("https://example.com/example.png");
}
