# Virtualization

In the contenxt of rendering on the web, *virtualization* is the technique of only rendering what is on-screen or near it, and thereby saving time and memory. It is used ubiquitously within modern browsers as a way to scale to large amounts of content, and is often used on web sites to improve scrolling performance.

Browsers use virtualization to save on rendering costs. They can do so because of the declarative nature of DOM, which the browser takes advantage of to avoid work. For example, browsers can compute which parts of content are far enough off-screen to avoid painting or rastering. Avoiding layout and style work is possible with some forms of [isolation](https://github.com/chrishtr/rendering/blob/master/isolation.md).

A good example of a commonly-used real-world javascript library that implements virtualization for scrolling is [react-window](https://github.com/bvaughn/react-window). [This blog post](https://developers.google.com/web/updates/2016/07/infinite-scroller) describes many of the techniques typically applied in detail. The [virtual-scroller](https://github.com/WICG/virtual-scroller) spec proposal aims to build such capabilities into the browser.

