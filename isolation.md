In the context of the web rendering model, rendering *isolation* refers to a property that some part of rendering for one piece of DOM content does not depend on some other piece of content.

For example, if a DOM element A *has style isolation* the `contain: style` [1] CSS property on it, then the rest of the document is *style isolated* from A's subtree. In other words, the style for the rest of the document can be computed without reference to any style on A's subtree. Note that this particular type of isolation is only in one direction. A's style does in fact depend on the rest of the document's style.

See also [this document](https://github.com/chrishtr/rendering/blob/master/rendering-event-loop.md) for a description of the typical lifecyle phases of rendering, such as style, layout, paint, raster and compositing.

Types of one-directional rendering isolation for a DOM subtree rooted at A include:
* Style isolation (achieved via `contain: style`): the computed style of the rest of document does not depend on A or its subtrees computed styles. 
* Layout isolation (achieved via `contain: size layout`): the layout (i.e., size and positioning) of the rest of the document does not depend on the layout of A's subtree (but does depend on A's layout).
* Paint isolation (achieved via `isolation: isolate` [3] or some other stacking context-inducing [4] property): the output of paint (a display list) for the rest of the document does not depend on the paint inputs [5] of A or its subtree, and vice-versa.
* Raster isolation (achieved via `contain: paint`): pixels outside of A's border box do not depend on A or it's subtree's content.
* Compositing isolation (achieved via `will-change: transform`, as one example): triggers a typical browser heuristic by which A and it's subtree's content is rastered into a separate memory buffer, typically via a GPU texture.

Note that isolation types are not additive: just because some rendering phases come after other ones. For example, paint isolation does not imply layout isolation, and layout isolation does not imply style isolation. 

Browsers may have implementation strategies that take advantage of these types of isolation. As of August 2019, Chrome takes advantage of layout, paint and compositing isolation to reduce rendering work. For example, if a DOM mutation occurs only within a subtree of an element A with layout isolation, then 



[1] https://developer.mozilla.org/en-US/docs/Web/CSS/contain
[2] https://developer.mozilla.org/en-US/docs/Web/CSS/will-change
[3] https://developer.mozilla.org/en-US/docs/Web/CSS/isolation
[4] https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Positioning/Understanding_z_index/The_stacking_context
[5] The "paint inputs" of an element are the [z-index order](https://www.w3.org/TR/CSS2/zindex.html) and non-layout-affecting styles, such as color, opacity or filters.
