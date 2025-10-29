# Proposal: add declarative scroll commands to [HTMLButtonElement](https://html.spec.whatwg.org/multipage/form-elements.html#attr-button-command)

### Summary

This proposal introduces a set of new `command` values for the `HTMLButtonElement` to declaratively control the scroll position of a target element. This allows authors to create custom scrolling UI (e.g., for carousels or galleries) without requiring JavaScript, improving accessibility and simplifying implementation.

### Problem & use cases

Authors frequently build custom components that contain a scrollable region, such as:

* A horizontal carousel of products or images.
* A vertically scrolling list of items within a modal.
* A 2D scrolling area for a map or canvas.

These components almost always require custom "Next," "Previous," "Up," or "Down" buttons. Currently, authors must write JavaScript to wire these buttons up. This proposal moves this common, repetitive behavior into the platform, making it declarative, robust, and accessible by default.

### Proposed solution

The proposal is to add eight new enumerated values to the `command` attribute on `HTMLButtonElement`. These commands would be paired with the `commandfor` attribute, which targets the `id` of the scroll container.

#### Proposed command values

* **Physical directions:**
    * `page-up`
    * `page-down`
    * `page-left`
    * `page-right`
* **Logical directions:**
    * `page-block-start`
    * `page-block-end`
    * `page-inline-start`
    * `page-inline-end`

#### Example usage

Here is how an author could implement a common horizontal carousel:

```html
<style>
  #my-carousel {
    display: flex;
    overflow-x: auto;
    width: 300px;
    scroll-snap-type: x mandatory;
  }
  #my-carousel > div {
    min-width: 300px;
    scroll-snap-align: center;
  }
</style>

<div id="my-carousel">
  <div>Item 1</div>
  <div>Item 2</div>
  <div>Item 3</div>
</div>

<button command="page-inline-start" commandfor="my-carousel">previous</button>
<button command="page-inline-end" commandfor="my-carousel">next</button>
```

### Proposed behavior

* **Action:** When a button with a `page-*` command is invoked, it scrolls the element referenced by its `commandfor` attribute.
* **Scroll amount:** The command scrolls the container by **one "page"** in the specified direction. This behavior should mimic the platform's default action for a PageUp/PageDown keypress on that container (i.e., scroll by the visible viewport dimension, minus a small overlap).
* **Scroll snapping:** The scrolling action **must** respect the target element's `scroll-snap` properties. The expected behavior is that the command scrolls by approximately one page and then settles on the nearest valid snap point in the direction of the scroll. It should *not* be defined as "scroll to the next/previous snap point," as this would be problematic for scrollers with many small snap items.
* **Targeting:** In this initial proposal, the `commandfor` attribute is **required** and must reference the `id` of the element to be scrolled.

### Design rationale (summary from [Open UI CG discussion](https://github.com/openui/open-ui/issues/1220))

* **Why new command names instead of a functional syntax?**
    An initial idea, `command="scroll(down)"`, was rejected. The group felt this "looks like code" and preferred simple, declarative keywords.
* **Why not `command="scroll"` with a `value` attribute (e.g., `value="down"`)?**
    This was deemed semantically incorrect. The `value` attribute (as used in `dialog.close(value)`) is intended to pass "raw data," not to define the *type* of action being performed.
* **Why "page" scrolling?**
    This maps to a well-understood, existing keyboard interaction (PageUp/PageDown). Other magnitudes (e.g., "scroll-to-top") were deferred to keep the initial proposal simple.
* **Why include both logical and physical directions?**
    Logical directions (`block-start`, `inline-end`, etc.) are essential for internationalization, RTL layouts, and vertical writing modes. Physical directions are included for consistency with CSS and to handle simple cases.

### Future considerations (out of scope for this initial proposal)

1.  **Implicit targeting:** A mechanism for a command invoker to automatically target its "nearest ancestor scroll container" if the `commandfor` attribute is omitted.
2.  **Automatic disabled state:** A mechanism for the button to automatically reflect the state of the scroller (e.g., the "next" button becoming `disabled` when at the end of the scroller).
3.  **More functionality:** More commands (e.g. scroll-to-top) + value-based scrolling.
