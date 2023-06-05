# Explainer: improved CSS-triggered entry and exit animations

## Motivation

Appropriate animations are a [great way](https://www.nngroup.com/articles/animation-purpose-ux/) to help users build an accurate mental model of how a web page UI works and therefore increase usability of a web site, and are important for a UI to feel polished and fluid to users. A common example is elements being visible or modal on the page only when they are “open”. Entry and exit animations for such elements are types of animations where an element enters or exits (respectively) the visible screen, or substantially changes their rendering in some other way; examples include dismissible [dialogs](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/dialog), [popovers](https://html.spec.whatwg.org/multipage/popover.html#the-popover-attribute), or similar UI patterns that do not use those built-in elements.

There are currently several gaps in declarative web platform APIs for the entry and exit animations in particular:

* Animations or transitions involving the [display](https://developer.mozilla.org/en-US/docs/Web/CSS/display) CSS property do not work.

* Some use cases work with [CSS animations](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Animations/Using_CSS_animations) but not [CSS transitions](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Transitions).

* Animations and transitions are impossible to coordinate with entry or exit from the [top layer](https://developer.chrome.com/blog/what-is-the-top-layer/).

We propose to close those gaps.

## Proposed solution

1. Support CSS transitions for [discrete](https://drafts.csswg.org/web-animations-1/#discrete) properties, and for discrete states of continuous properties (e.g. the `auto` value of `width`). Currently they can be CSS animated but not CSS transitioned. It’s tracked in [this CSSWG issue](https://github.com/w3c/csswg-drafts/issues/4441) for standardization and [this chromestatus entry](https://chromestatus.com/feature/5071230636392448). This is a prerequisite for 2 and 4 below.

2. Support animating and transitioning the display css property. Currently it is impossible to use display in an animation or transition. It’s tracked in [this CSSWG issue](https://github.com/w3c/csswg-drafts/issues/6429) for standardization and [this chromestatus entry](https://chromestatus.com/feature/5154958272364544).

3. Support for CSS transitions on entry animations from a display:none state by adding the `@initial` syntax. (Currently this use case is only possible via CSS animations.) It’s tracked in [this CSSWG issue](https://github.com/w3c/csswg-drafts/issues/8174) for standardization.

4. Support for specifying a CSS transition delay on entry to or exit from the top layer via an `overlay` CSS property (that is in effect restricted to CSS transitions). It’s tracked in [this CSSWG issue](https://github.com/w3c/csswg-drafts/issues/8189) for standardization and [this chromestatus entry](https://chromestatus.com/feature/5138724910792704).

5. Ensuring exit animations are inert to user input, to avoid accidental clicks against content that no longer conceptually visible, by making animations to `display:none` automatically inert. It's tracked in [this CSSWG issue](https://github.com/w3c/csswg-drafts/issues/8389) for standardization.

With these five changes, web developers will be able to use CSS transitions and CSS animations for discrete-animatable properties, the display CSS property, and a CSS transition (but not CSS animations; see Alternatives Considered for why) for the top layer.

## Example code

Instructions for a complete working example of an exit animation for a `<dialog>` element is [here](https://github.com/w3c/csswg-drafts/issues/8189#issuecomment-1464556841).

Inline:

```
<dialog id=dialog>
Content here
<button onclick="dialog.close()">close</button>
</dialog>

<button onclick="dialog.showModal()">Show Modal</button>

<div id="high">
 
</div>
<style>

dialog {
  transition: overlay 5s, opacity 5s, display 5s step-end; /* New! */

    /* duplicate UA styles during the animation */
    position: fixed;
    inset-block-start: 0px;
    inset-block-end: 0px;
    max-width: calc((100% - 6px) - 2em);
    max-height: calc((100% - 6px) - 2em);
    user-select: text;
    visibility: visible;
    overlay: browser !important;
    overflow: auto;
}

dialog:not(:modal) {
  opacity: 0;
}

#high {
  z-index: 999999999;
  background: green;
  position: fixed;
  height: 100%;
  width: 100%;
}
</style>

```

And a demo of an entry animation for an element with `popover` that uses CSS transitions is [here](https://jsbin.com/tigicuj/edit?html,output).

Inline:

```
<button popovertoggletarget=f popovertarget=f>Toggle the popover</button>
<div popover=auto id=f>I'm a Popover! I should animate.</div>

<style>
  [popover],
  [popover]:initial {
    opacity: 0;
    transition: display 0.5s, overlay 0.5s, opacity 0.5;
  }
  [popover]:open {
    opacity: 1;
  }
  [popover] {
    top: 100px;
    left:100px;
    right:auto;
    bottom:auto;
  }
</style>

```

## Alternatives Considered

In the use case addressed by solution 3 above, it’s already possible to use CSS animations instead of CSS transitions, so in that sense this solution is not strictly required. However, this alternative is more verbose and confusing to developers already used to CSS transitions.

For solution 4, an alternative could be for the user agent to automatically detect animations when entering or exiting the top layer. This was prototyped in Chromium and discussed in detail [here](https://github.com/whatwg/html/issues/7785) (see comments towards the end). The proposal in this document is more explicit and simple than the alternative described there. Another alternative could be to directly expose the `overlay` CSS property in all cases, but that would result in circularity and loss of UA-guaranteed accessibility for existing top-layer elements (which are: fullscreen, dialog and popover, all of which are UA controlled to ensure accessibility).

Similarly, the `overlay` CSS property could be exposed to CSS animations in addition to CSS transitions. On top of introducing the problems of directly exposing an top-layer CSS property, this would also: allow unbounded-in-time animations (and therefore lead to top layer elements failing to ever open or close); and lead to other implementation complications in the implementation of the [CSS cascade](https://developer.mozilla.org/en-US/docs/Web/CSS/Cascade).
