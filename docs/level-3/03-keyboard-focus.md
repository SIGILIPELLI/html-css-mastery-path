---
description: "Make everything work from the keyboard: focusable elements, tab order, tabindex, :focus-visible, inert, and managing focus in dynamic UI."
---

# 03 · Keyboard Access & Focus Management

Everything a mouse user can do, a keyboard user must be able to do. That single rule
(WCAG 2.1.1 *Keyboard*) covers a wide range of people: screen-reader users (who drive
the page from the keyboard), people with tremors or limited mobility, switch and
voice-control users (whose tools emulate keyboard input), and power users who simply
prefer it. It's also the easiest accessibility requirement to test yourself — put the
mouse away and try.

## The keys

| Key | Does |
|-----|------|
| <kbd>Tab</kbd> / <kbd>Shift</kbd>+<kbd>Tab</kbd> | Move focus to the next / previous focusable element |
| <kbd>Enter</kbd> | Follow a link; activate a button; submit a form from a text field |
| <kbd>Space</kbd> | Activate a button; toggle a checkbox; scroll the page when nothing is focused |
| Arrow keys | Move within a group: radio buttons, select options, tabs, menus; scroll |
| <kbd>Escape</kbd> | Close dialogs, popovers, menus |
| <kbd>Home</kbd> / <kbd>End</kbd> | Start/end of a text field or list |

Links activate with Enter only; buttons with Enter **and** Space. That's one of the ways
users (and assistive tech) tell them apart, and a reason not to fake one with the other.

## What's focusable

Natively focusable: `<a href>`, `<button>`, `<input>`, `<select>`, `<textarea>`,
`<summary>`, `<iframe>`, elements with `contenteditable`, `<audio>`/`<video>` with
`controls`, and anything with a `tabindex`. **Not** focusable: `<a>` without `href`,
disabled controls, and every `div` and `span`.

We built a page with a mix of elements and pressed Tab seven times in Chromium, recording
where focus went:

```html
<a id="a1" href="#">A</a>
<div id="div-click" onclick="">div</div>
<span id="span-t0" tabindex="0">span</span>
<button id="b1">B</button>
<input id="i-pos" tabindex="3">
<a id="no-href">no href</a>
<button id="b-dis" disabled>dis</button>
<div id="neg" tabindex="-1">neg</div>
<button id="b2">C</button>
```

```text
["i-pos", "a1", "span-t0", "b1", "b2", "body", "i-pos"]
```

- `i-pos` came **first**, despite being fifth in the source, because of `tabindex="3"`.
- The `div` with a click handler was **skipped** — adding `onclick` doesn't make anything
  focusable.
- `span-t0` was reached because of `tabindex="0"`.
- `<a>` without `href`, the disabled button and the `tabindex="-1"` div were skipped.
- After the last element, focus left the page (`body` — in a real browser it goes to the
  address bar and other browser UI) and then cycled back.

### `tabindex` values

| Value | Meaning |
|-------|---------|
| `0` | Focusable, in normal source order |
| `-1` | Focusable **only by script** (`element.focus()`), never by Tab |
| positive (`1`, `3`…) | Focusable, and **jumps ahead** of all `0` elements, in numeric order |

**Never use positive `tabindex`.** As the test shows, it rips one element out of the
natural order, and once one exists, every other element's order becomes hard to reason
about. If the tab order is wrong, fix the source order.

Use `tabindex="0"` sparingly too: it makes something focusable, but not *operable* — it
won't respond to Enter or Space and has no role. Usually the right fix is a real
`<button>`. Legitimate uses include scrollable regions (Level 1 · 05) and custom widgets
that also get a role and key handlers (lesson 04).

## Visible focus

Focus must be visible (WCAG 2.4.7), and WCAG 2.2 adds that the focused element must not
be entirely hidden behind sticky headers or overlays (2.4.11).

```css
:focus-visible {
  outline: 3px solid var(--focus, #1a5fb4);
  outline-offset: 2px;
}
```

`:focus-visible` matches when the browser judges a visible indicator useful. We tested
Chromium's heuristics:

```text
button focused by mouse click:  :focus-visible = false
button focused by keyboard:     :focus-visible = true
text input focused by mouse:    :focus-visible = true
```

Clicking a button doesn't show a ring (the user knows where they clicked), keyboard focus
always does, and text inputs always do because you need to see where typing will go.
That's why the old advice to write `:focus { outline: none }` to "remove ugly rings for
mouse users" is obsolete and harmful — `:focus-visible` already solves it.

Guidelines for focus styles:

- **Outline, not just colour.** `outline` doesn't affect layout, follows `border-radius`,
  and survives Windows High Contrast mode. A background-colour-only change often doesn't.
- **3:1 contrast** against the adjacent colours.
- **`outline-offset`** keeps the ring clear of the element's own border.
- **Keep it inside the viewport** under sticky headers: `scroll-padding-top` on the
  scrolling element prevents focused elements from being scrolled underneath:

```css
html { scroll-padding-top: 5rem; }   /* height of the sticky header */
```

## Removing whole regions: `inert`

`inert` makes an element and everything inside it non-focusable, non-clickable and
hidden from assistive technology — while leaving it visible:

```html
<button id="before">Before</button>
<div inert>
  <button>Inert 1</button>
  <a href="#">Inert link</a>
</div>
<button id="after">After</button>
```

Tabbing from the top visited `["before", "after", "body"]` — both elements inside the
inert region were skipped. Use it for content behind an overlay, an off-canvas menu when
closed, or a form section that doesn't apply yet (with a visual cue that it's disabled).

## Focus management

When your page changes without a page load, **you** are responsible for where focus goes.
The rules of thumb:

| Situation | Move focus to |
|-----------|---------------|
| Open a dialog | The first focusable element inside it (or the dialog heading) |
| Close a dialog | The control that opened it |
| Delete the focused item in a list | The next item, or the list's heading |
| Show inline form errors on submit | An error summary, or the first invalid field |
| Client-side route change | The new page's `<h1>` (with `tabindex="-1"`) |
| Load more results | The first new result |

If focus is on an element that gets removed from the DOM, it falls back to `<body>`, and
a keyboard user has to start tabbing from the top of the page.

### The native dialog does it for you

We built a modal with the native `<dialog>` element:

```html
<button id="opener">Open</button>
<dialog id="d">
  <h2>Title</h2>
  <button id="close" onclick="this.closest('dialog').close()">Close</button>
  <input id="field">
</dialog>
<button id="behind">Behind</button>
<script>
  document.getElementById('opener').onclick = () => document.getElementById('d').showModal();
</script>
```

Tabbing to "Open", pressing Enter and observing in Chromium:

```text
focus after showModal():               close
Tab ×4 inside the modal:               field, body, close, field
what's on top at the "Behind" button:  DIALOG (the backdrop)
after pressing Escape:                 open = false, focus = opener
```

`showModal()` moved focus to the first focusable element, kept Tab cycling within the
dialog (plus the browser's own UI), made everything behind it inert, closed on Escape,
and **returned focus to the opener** — all without extra script. Level 3 · 09 covers the
dialog in full.

One thing that went wrong while we wrote that test: the first version gave the button
`id="open"` and wrote `open.onclick = ...`. Nothing happened, because `open` resolved to
the built-in `window.open` function rather than the element (browsers expose elements by
id as globals only when the name isn't already taken). Our second attempt used
`id="opener"` — which collides with `window.opener`. Use `getElementById` rather than the
id-as-global shortcut.

## Worked example: a roving tabindex toolbar

Composite widgets — toolbars, tab lists, menus, grids — should be **one Tab stop**, with
arrow keys moving within them. Otherwise a 12-button toolbar costs 12 Tab presses to get
past. The standard technique is a **roving tabindex**: only the active item has
`tabindex="0"`; the rest have `-1`.

```html
<div role="toolbar" aria-label="Text formatting" class="toolbar">
  <button type="button" tabindex="0" aria-pressed="false">Bold</button>
  <button type="button" tabindex="-1" aria-pressed="false">Italic</button>
  <button type="button" tabindex="-1" aria-pressed="false">Underline</button>
</div>
```

```js
const toolbar = document.querySelector('.toolbar');
const items = [...toolbar.querySelectorAll('button')];

toolbar.addEventListener('keydown', (event) => {
  const current = items.indexOf(document.activeElement);
  let next = null;
  if (event.key === 'ArrowRight') next = (current + 1) % items.length;
  if (event.key === 'ArrowLeft') next = (current - 1 + items.length) % items.length;
  if (event.key === 'Home') next = 0;
  if (event.key === 'End') next = items.length - 1;
  if (next === null) return;
  event.preventDefault();
  items[current].tabIndex = -1;
  items[next].tabIndex = 0;
  items[next].focus();
});
```

Tab enters the toolbar at whichever button was last active, arrows move within it, and
Tab leaves it. This is the one place where keyboard behaviour needs JavaScript — and
where you should follow the WAI-ARIA Authoring Practices patterns exactly, because users
expect every toolbar to behave the same way.

## How It Actually Works

The browser keeps a **sequential focus navigation order**: first all elements with a
positive `tabindex` in ascending order (ties in source order), then all elements with
`tabindex="0"` or natively focusable, in source (DOM) order — **not visual order**. CSS
that reorders things visually (`order`, `flex-direction: row-reverse`, grid placement,
absolute positioning) doesn't change it. The newer CSS `reading-flow` property is
designed to let the focus order follow a flex or grid layout's visual order; it was
supported in the Chromium build we tested but not in WebKit, so don't rely on it yet.

Focus is tracked per document in `document.activeElement`. Keyboard events are dispatched
to the focused element first and bubble up, which is how a toolbar's single `keydown`
listener handles all its buttons. Native elements have built-in **activation behaviour**:
a button's Enter/Space fires `click`, a link's Enter follows it. A `div` has none, so a
`div` with only a click handler ignores the keyboard entirely (we confirmed this in
lesson 04).

`inert` and a modal `<dialog>` work by marking nodes as inert in the browser's internal
tree: inert nodes are skipped by focus navigation, hit testing (clicks) and the
accessibility tree construction.

## Common mistakes

- **Clickable `div`s and `span`s**, unreachable and inoperable by keyboard.
- **Positive `tabindex`.**
- **`outline: none`** with no replacement.
- **Focus lost to `<body>`** after deleting, closing or re-rendering.
- **Custom modals that don't trap focus**, so Tab wanders into the page behind.
- **Keyboard traps** — the opposite problem: a widget you can Tab into but not out of
  (embedded players and editors are common culprits).
- **Tab order that jumps around** because of visual reordering.

## Exercise

1. Unplug the mouse and complete a task on your product page: choose a size, change the
   quantity, add to basket, follow a related-product link. Note every problem.
2. Style `:focus-visible` for your whole site and check it against your dark theme.
3. Add `scroll-padding-top` so focused elements aren't hidden under your sticky header.
   Test by tabbing down the page.
4. Build the roving-tabindex toolbar and make each button toggle `aria-pressed`.
5. Convert a custom modal (or build one from `div`s), and then replace it with `<dialog>`
   and `showModal()`. Compare how much script you deleted.
