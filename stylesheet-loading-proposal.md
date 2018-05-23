The following is a summary proposal for changes to the HTML spec regarding stylesheet loading, with the goal of improving reliability, understandability, determinism and browser compatibility, while not sacrificing performance.

# Pending parsing blocking style sheets

The HTML spec currently specifies the concept of a [pending parsing blocking script](https://html.spec.whatwg.org/multipage/scripting.html#pending-parsing-blocking-script). This
will be extended to include the definition of a _pending parsing blocking
style sheet_.

A _pending parsing blocking style sheet_ is any [style sheet that is blocking script](https://html.spec.whatwg.org/multipage/semantics.html#interactions-of-styling-and-scripting) that is parser-inserted, or declared in a parser-inserted style sheet via `@import`, regardless of its location in the HTML.

# HTML parser blocks in all phases

The HTML parser is changed to be blocked by any pending parsing blocking style sheets, via the same mechanism as that for _pending parsing blocking script_: the [text insertion mode](https://html.spec.whatwg.org/#parsing-main-incdata:pending-parsing-blocking-script) of the HTML parser.

# Rendering starts at <body>

Step 7 of the [event loop processing model](https://html.spec.whatwg.org/multipage/webappapis.html#event-loop-processing-model), _update the rendering_
will only begin happening once:
* A parser-inserted `<body>` element has been added to the DOM

Once begun, step 7 will always happen in the future during the lifetime
of the document, subject to throttling or stopping notes mentioned in the Note
after step 7.3.

Further, step 7 will be specified to always use all style sheets
which [have beeen added](https://drafts.csswg.org/cssom/#add-a-css-style-sheet) to the document.

# Preload scanning defined

Add a definition of a preload scanner. The preload scanner may parse and
analyze the HTML document, or any external resources, in order to pre-load
network resources.

Note: the pre-load scanner may affect the order in which resources are obtained, and may induce side-effects in cases where a server providing the resource over the network maintain states, and may be observable to Service Workers.
