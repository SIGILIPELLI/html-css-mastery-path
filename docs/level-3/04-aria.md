---
description: "Use ARIA correctly: the five rules, roles, states and properties, labelling, a disclosure widget, live regions, and why role=button isn't enough."
---

# 04 · ARIA: When and How to Use It

**ARIA** (Accessible Rich Internet Applications) is a set of HTML attributes — `role` and
the `aria-*` family — that change what an element exposes in the accessibility tree. It
exists for the gaps: interactive patterns HTML has no element for (tabs, tree views,
comboboxes), dynamic updates a screen reader wouldn't otherwise notice, and extra
labelling. It's powerful precisely because it changes what assistive technology is told,
and dangerous for the same reason: ARIA changes **only** what's announced, never how
anything behaves.

## The rules of ARIA use

The W3C's guidance boils down to five rules:

1. **If a native HTML element does the job, use it.** A `<button>` beats
   `<div role="button">` every time.
2. **Don't change native semantics** unless you really have to. `<h2 role="tab">`
   destroys a heading; put the heading and the tab in separate elements.
3. **All interactive ARIA controls must be keyboard operable.** If you give something a
   widget role, you must implement the keyboard behaviour that role promises.
4. **Don't use `role="presentation"` or `aria-hidden="true"` on a focusable element.**
5. **All interactive elements must have an accessible name.**

A widely cited finding from WebAIM's annual survey of the top million home pages is that
pages *with* ARIA present average noticeably *more* detected accessibility errors than
pages without it. ARIA isn't the cause of the errors, but it's a strong signal of custom
widgets built without the behaviour to back them up.

## Why `role="button"` isn't enough — measured

```html
<div role="button" id="fake" onclick="…">Fake</div>
<button id="real" onclick="…">Real</button>
```

We pressed Tab twice, then focused each element programmatically and pressed Enter and
Space on it, counting clicks:

```text
tab order:                         ["real", "body"]
activations (Enter + Space) fake:  0
activations (Enter + Space) real:  2
```

The fake button is announced as a button — and then can't be reached with Tab, and
ignores both keys even when focused. To make it equivalent you'd need `tabindex="0"`, a
`keydown` handler for Enter (on keydown) and Space (on keyup, as native buttons do),
`aria-disabled` handling, and form submission if it's meant to submit. Or: use
`<button>`, and reset its styles with `all: unset` or a few properties if the default look
is the issue.

## Roles, states and properties

- **Roles** say what something is: `role="tablist"`, `role="tab"`, `role="tabpanel"`,
  `role="dialog"`, `role="alert"`, `role="status"`, `role="switch"`...
- **States** change with interaction: `aria-expanded`, `aria-pressed`, `aria-checked`,
  `aria-selected`, `aria-invalid`, `aria-busy`, `aria-disabled`, `aria-current`.
- **Properties** are more static: `aria-label`, `aria-labelledby`, `aria-describedby`,
  `aria-controls`, `aria-haspopup`, `aria-live`, `aria-required`.

Any state you set, **you must keep in sync** with the visual state. An `aria-expanded`
that says `false` while the panel is visible is worse than no attribute at all.

## Labelling and describing

```html
<!-- name from visible text elsewhere: preferred, stays in sync with what's shown -->
<h2 id="cart-heading">Your basket</h2>
<ul aria-labelledby="cart-heading">…</ul>

<!-- name with no visible text: icon buttons -->
<button type="button" aria-label="Remove tomato soup from basket">
  <svg aria-hidden="true" …></svg>
</button>

<!-- extra description, read after the name and role -->
<label for="pw">Password</label>
<input id="pw" type="password" aria-describedby="pw-hint">
<p id="pw-hint">At least 12 characters.</p>
```

`aria-labelledby` and `aria-describedby` take **one or more ids** and concatenate the
referenced text. They can reference hidden elements, which is occasionally useful.

Prefer visible labels. Voice-control users say what they *see*: if the visible text is
"Remove" but the `aria-label` is "Delete item", saying "click Remove" fails. WCAG 2.5.3
(*Label in Name*) requires the accessible name to contain the visible text.

## Worked example: a disclosure (show/hide) button

The most common custom widget, and the simplest correct one:

```html
<button type="button" id="t" aria-expanded="false" aria-controls="panel">
  Nutrition facts
</button>
<div id="panel" hidden>
  <p>180 kcal</p>
</div>

<script>
  const toggle = document.getElementById('t');
  const panel = document.getElementById('panel');
  toggle.addEventListener('click', () => {
    const isOpen = toggle.getAttribute('aria-expanded') === 'true';
    toggle.setAttribute('aria-expanded', String(!isOpen));
    panel.hidden = isOpen;
  });
</script>
```

Chromium's accessibility tree before and after one click:

```text
before:
- button "Nutrition facts"

after click:
- button "Nutrition facts" [expanded]
- paragraph: 180 kcal
```

Pressing Enter toggled it back (`aria-expanded="false"`, panel hidden) — the keyboard
works because it's a real `<button>`. The screen reader announces "Nutrition facts,
button, collapsed" and then "expanded" after activation. Note the choices: the state
lives on the **button** (what the user interacts with), `hidden` removes the panel from
both sight and the accessibility tree, and the visual style can key off the attribute:

```css
[aria-expanded="true"] .chevron { rotate: 180deg; }
```

For this particular pattern, `<details>`/`<summary>` (lesson 09) gives you the same
result with no script at all. Build the ARIA version when you need behaviour
`<details>` can't provide.

## Live regions: announcing changes

When content changes somewhere other than where focus is — "Added to basket", "3 results
found", "Saved" — screen-reader users won't know unless you tell them. A **live region**
is an element whose changes are announced automatically:

```html
<p id="status" role="status"></p>        <!-- polite: waits for the user to pause -->
<div role="alert"></div>                  <!-- assertive: interrupts; for urgent errors only -->
```

```js
document.getElementById('status').textContent = 'Tomato soup added to basket.';
```

Rules that make live regions work reliably:

- **The region must exist in the DOM before you change it.** Adding an element that
  already contains text often isn't announced; update the text of a region that was
  there on page load.
- **Keep messages short** and put them in text, not in `aria-label`.
- **Use `role="status"`** (`aria-live="polite"`) for almost everything; `role="alert"`
  (`aria-live="assertive"`) only for things that genuinely need to interrupt.
- **Don't announce everything.** A live region on a constantly updating timer makes the
  page unusable.

We can't show you screen-reader speech output from our test environment — live-region
announcements are produced by the screen reader, not the browser — so test these with a
real screen reader (VoiceOver or NVDA).

## Common roles you'll actually write

| Pattern | Markup |
|---------|--------|
| Icon-only button | `<button aria-label="…">` with `aria-hidden` SVG |
| Toggle button | `<button aria-pressed="false">`, flipped to `"true"` when on |
| Disclosure | `<button aria-expanded aria-controls>` + `hidden` panel |
| Status message | `role="status"` region |
| Tabs | `role="tablist"` > `role="tab"` (with `aria-selected`, roving tabindex) + `role="tabpanel"` |
| Switch | `<button role="switch" aria-checked>` or `<input type="checkbox" role="switch">` |
| List styled without bullets (Safari) | `<ul role="list">` |

For anything more complex — combobox, tree, grid, menu — follow the **WAI-ARIA Authoring
Practices Guide** (APG) patterns, including their keyboard tables, and test with at least
two screen readers. These patterns are genuinely hard; many design systems get them wrong.

## How It Actually Works

When the browser computes a node's role (lesson 01), an explicit `role` attribute
overrides the implicit one — *for the accessibility tree only*. The DOM element is still a
`div` with a `div`'s behaviour: no focusability, no activation behaviour, no form
participation. ARIA attributes are likewise read during accessibility-tree construction
and turned into platform API properties (for example `aria-expanded="true"` becomes the
`expanded` state on Windows and the `AXExpanded` attribute on macOS).

When an ARIA attribute changes, the browser fires an accessibility event (such as "state
changed") that screen readers listen to. Live regions work the same way: the browser
watches the subtree of any element with `aria-live` (or an implicitly live role like
`status` and `alert`) and, when text is added or changed, fires an event carrying the new
text and the politeness level. The screen reader decides when to speak it — `polite`
queues it after current speech, `assertive` may interrupt. That's also why the region has
to exist first: the browser only watches regions it already knows about.

## Common mistakes

- **ARIA instead of HTML**: `role="button"`, `role="link"`, `role="heading"`,
  `role="list"` on elements that have native equivalents.
- **Roles without behaviour** — `role="tab"` with no arrow-key support.
- **Stale states** — `aria-expanded` never updated.
- **`aria-label` on elements that can't be named** — plain `div`s and `span`s with no role
  ignore it in many screen readers.
- **`aria-label` that contradicts the visible text.**
- **`aria-hidden="true"` on a container with focusable children.**
- **Live regions added dynamically** and never announced, or `role="alert"` for routine
  messages.

## Exercise

1. Find every icon-only control on your product page and give it an accessible name that
   includes context ("Remove tomato soup from basket", not just "Remove").
2. Build the disclosure widget and verify the `[expanded]` state in devtools'
   Accessibility pane after each click and each Enter/Space.
3. Add a `role="status"` region to the product page and announce "Added to basket"
   when the form is submitted (prevent the real submission in your test). Listen with a
   screen reader.
4. Read the APG "Tabs" pattern and implement it — `tablist`, `tab`, `tabpanel`, roving
   tabindex and arrow keys. Then ask whether your content really needed tabs, or would
   have worked as headings on one page.
