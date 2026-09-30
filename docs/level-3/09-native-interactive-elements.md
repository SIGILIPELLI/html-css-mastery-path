---
description: "Native interactive HTML: details/summary and exclusive accordions, modal dialog, the popover attribute, the top layer and anchor positioning."
---

# 09 · Native Interactive Elements: details, dialog, popover

For years, disclosure widgets, modals and dropdowns meant a JavaScript library —
and most of those libraries had keyboard or screen-reader bugs, because getting focus
management, Escape handling, stacking and ARIA state right is hard. HTML now has built-in
elements and attributes for the three most common patterns. They're keyboard-accessible,
screen-reader-friendly and render above everything else on the page, and they need
little or no script. This lesson tests each one.

## `<details>` and `<summary>`

A disclosure widget with zero JavaScript:

```html
<details>
  <summary>Storage tips</summary>
  <p>Keeps for 3 days in the fridge, or 3 months frozen.</p>
</details>
```

The `<summary>` is focusable and toggles with a click, Enter or Space. Chromium's
accessibility tree, closed and then open:

```text
closed:
- group: Storage tips

open:
- group:
  - text: Storage tips
  - paragraph: Keeps 3 days.
```

When closed, the content isn't just hidden visually — it's absent from the accessibility
tree. In Chromium-based browsers, closed content is still searchable with find-in-page,
which opens the `<details>` automatically when it finds a match (check other browsers
before depending on this).

Add the `open` attribute to start expanded. Listen for the `toggle` event if you need to
react in script.

### Exclusive accordions with `name`

Give several `<details>` the same `name` and opening one closes the others:

```html
<details name="faq"><summary>Can I freeze it?</summary><p>Yes, for up to 3 months.</p></details>
<details name="faq"><summary>Is it vegan?</summary><p>Without the cream, yes.</p></details>
```

We opened the first, then the second: afterwards `[first.open, second.open]` was
`[false, true]`. Use this sparingly — people often want to compare two answers, and an
accordion that closes things they were reading is frustrating.

### Styling

```css
details { border: 1px solid var(--border); border-radius: 0.5rem; padding: 0.5rem 1rem; }
summary { cursor: pointer; font-weight: 600; }
summary::marker { color: var(--brand); }            /* the default triangle */
details[open] > summary { margin-bottom: 0.5rem; }

/* or replace the marker entirely */
summary { list-style: none; }
summary::-webkit-details-marker { display: none; }  /* older Safari */
summary::after { content: "+"; float: right; }
details[open] > summary::after { content: "−"; }
```

`details` is right for FAQs, "show more" sections and optional form help. It's **not** a
navigation menu (use a disclosure button, lesson 04) and not tabs.

## `<dialog>`

```html
<button type="button" id="delete">Delete recipe</button>

<dialog id="confirm" aria-labelledby="confirm-title">
  <form method="dialog">
    <h2 id="confirm-title">Delete this recipe?</h2>
    <p>This can't be undone.</p>
    <button value="cancel">Cancel</button>
    <button value="confirm">Delete</button>
  </form>
</dialog>

<script>
  const dialog = document.getElementById('confirm');
  document.getElementById('delete').addEventListener('click', () => dialog.showModal());
  dialog.addEventListener('close', () => {
    if (dialog.returnValue === 'confirm') { /* delete it */ }
  });
</script>
```

- **`showModal()`** opens it as a **modal**: in the top layer, with a `::backdrop`,
  everything else inert, focus moved inside, Escape to close.
- **`show()`** opens it non-modally — no backdrop, page stays interactive. For
  non-blocking panels.
- **`<form method="dialog">`** closes the dialog on submit instead of navigating, and sets
  `dialog.returnValue` to the clicked button's `value`.

We clicked "Delete recipe", then "Delete" inside the dialog, and read the result:

```text
[dialog.open, dialog.returnValue, document.activeElement.id] = [false, "confirm", "delete"]
```

Closed, the button's value is available, and focus went back to the button that opened
it. (Lesson 03 showed the rest of the modal behaviour: focus moves in, Tab stays inside,
the page behind is inert, and Escape closes it.)

### Styling the dialog and backdrop

```css
dialog {
  max-width: min(32rem, 100% - 2rem);
  border: 0;
  border-radius: 0.75rem;
  padding: 1.5rem;
  box-shadow: 0 10px 40px rgb(0 0 0 / 0.25);
}

dialog::backdrop {
  background: rgb(0 0 0 / 0.5);
  backdrop-filter: blur(2px);
}

body:has(dialog[open]) { overflow: hidden; }   /* stop the page scrolling behind a modal */
```

Newer additions: the `closedby` attribute controls light-dismiss (`closedby="any"` lets
a click on the backdrop close it), and invoker commands let a button open or close a
dialog declaratively — `<button commandfor="confirm" command="show-modal">`. Both were
present in the Chromium and WebKit builds we tested; check support for other browsers
before relying on them, and keep the `showModal()` script as the baseline.

## The `popover` attribute

Popovers are for transient UI that sits above the page without blocking it — menus,
tooltips-with-content, pickers, toast notifications:

```html
<button type="button" popovertarget="help">Help</button>
<div id="help" popover>
  <p>Roast skin-side up so the juices stay in the tray.</p>
</div>
```

No JavaScript at all. We clicked the button and tested three dismissal paths in Chromium:

```text
open after clicking the button:          true
after clicking elsewhere on the page:    false   (light dismiss)
after reopening and pressing Escape:     false
```

What you get for free:

- Rendered in the **top layer**, above every stacking context — no z-index.
- **Light dismiss**: clicking outside or pressing Escape closes it (for the default
  `popover="auto"`). Opening another auto popover closes the first, unless nested.
- The invoking button is associated with the popover for assistive technology, and focus
  returns sensibly when it closes.
- `popover="manual"` turns off light dismiss — for toasts you close yourself.

What you **don't** get is semantics: `popover` isn't a role. A menu still needs menu
semantics if it's an application menu; a popover containing links is usually best left as
a plain list of links. Popovers aren't modal — for anything that must block the page,
use a modal `<dialog>`.

### Positioning popovers

A popover opens centred in the viewport by default (it's `position: fixed` with `inset:
0; margin: auto` in the user-agent styles). To attach it to its button, **CSS anchor
positioning** is the modern answer:

```css
[popovertarget="help"] { anchor-name: --help-btn; }

#help {
  position-anchor: --help-btn;
  inset: auto;                                  /* reset the UA centring */
  position-area: bottom span-right;             /* below the anchor, extending right */
  margin-top: 0.25rem;
  position-try-fallbacks: flip-block;           /* flip above if there's no room below */
}
```

`anchor-name` was supported in both engines we tested; it's newer than the rest of this
lesson, so give popovers a usable default position for browsers without it (the
centred default is fine) and treat anchoring as an enhancement.

## The top layer

`<dialog>` opened with `showModal()` and elements with `popover` live in the **top
layer**: a separate layer the browser paints above the entire document, in the order
elements were added. Consequences:

- No `z-index` needed, and no stacking-context bug (Level 2 · 07) can trap them.
- Ancestors' `overflow: hidden` doesn't clip them, and ancestors' transforms don't
  affect their positioning.
- `::backdrop` exists for top-layer elements only.

This is why native dialogs and popovers solve the "my modal is behind the header" class
of bugs completely.

## A lesson from writing this lesson

Our first dialog test did nothing when the button was clicked. The button had
`id="open"`, and the script said `open.onclick = () => d.showModal()`. Browsers expose
elements with an `id` as global variables — *unless* the name is already taken, and
`open` is `window.open`. The second attempt used `id="opener"`, which collides with
`window.opener` (and threw "Cannot set properties of null"). Always look elements up with
`document.getElementById()` or `querySelector()`.

## How It Actually Works

`<details>` is implemented with a shadow tree: the `<summary>` goes into one slot and
everything else into a second slot, which is hidden when `open` is absent. (Chromium
hides it in a way that keeps it searchable, which is how find-in-page can reveal it.)
Toggling the `open` attribute is the entire state model — which is why you can style
`details[open]` and set it from script.

`showModal()` performs several steps defined in the HTML spec: add the dialog to the top
layer, make it the "blocking" modal so that everything outside it is inert, run the
dialog focusing steps (focus the first focusable descendant, or one with `autofocus`),
and remember the previously focused element so `close()` can restore focus. The Escape
key triggers a `cancel` event, then `close`.

`popover` works through the same top layer, plus a **popover stack** for `auto` popovers:
opening a popover closes any others that aren't its ancestors, and a click outside the
topmost one (or Escape) hides the stack from the top down. `popovertarget` wires a button
to that machinery declaratively, including the accessibility relationship.

## Common mistakes

- **Custom modals built from `div`s** when `<dialog>` exists.
- **`dialog.show()` when you meant `showModal()`** — no backdrop, no inertness.
- **No accessible name** for the dialog — use `aria-labelledby` pointing at its heading.
- **Using `<details>` for site navigation or tabs.**
- **Treating `popover` as a role** — it adds behaviour, not semantics.
- **Forgetting that a closed `<details>` hides its content from assistive tech** — don't
  put essential information in a closed one.
- **Relying on id-named globals** in scripts.

## Exercise

1. Convert the "Care" and "Specifications" sections of your product page into
   `<details>`, styled with a custom marker. Try the find-in-page test on closed content.
2. Build a "Remove from basket?" confirmation with `<dialog>`, `showModal()` and
   `<form method="dialog">`. Verify with the keyboard: focus moves in, Tab stays inside,
   Escape closes, focus returns.
3. Build a "Help" popover with `popovertarget`, then position it under its button with
   anchor positioning. Test in two browsers.
4. Put a positioned element with `z-index: 9999` inside a container with `opacity: 0.99`,
   next to a `showModal()` dialog, and confirm the dialog still wins.
