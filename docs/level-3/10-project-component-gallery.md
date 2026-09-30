---
description: "Level 3 project: accessible tabs, toggle, disclosure, dialog, popover, form errors and a container-query card, all keyboard-tested."
---

# 10 · Project — An Accessible Component Gallery

Design systems usually start as a gallery page: every reusable component on one page,
where it can be reviewed, tested and documented. In this project you'll build one for the
Weeknight Kitchen site, containing seven components that between them use everything in
Level 3 — ARIA done properly, keyboard support, focus management, native interactive
elements, container queries, cascade layers, nesting, `:has()` and motion that respects
user preferences.

The reference solution was validated, audited with axe-core in both colour schemes, at
three widths and with the dialog and popover open, and then **driven entirely from the
keyboard** by a Playwright script. The test log is below the code.

## Requirements

1. **Tabs** following the WAI-ARIA Authoring Practices pattern: one Tab stop for the tab
   list, arrow keys and Home/End to move between tabs, panels labelled by their tabs.
2. **A toggle button** ("Save recipe") using `aria-pressed`, with a status message
   announced through a live region.
3. **A disclosure** using `<details>`.
4. **A confirmation dialog** using `<dialog>` and `showModal()`, reporting what the user
   chose.
5. **A popover** attached to an icon button, positioned with anchor positioning where
   supported.
6. **A form field** with the accessible inline-error pattern from lesson 05, which clears
   when fixed.
7. **A card** that changes layout with a container query, shown in a wide and a narrow
   slot.
8. **CSS organised in cascade layers**, light/dark theme via `light-dark()`, visible
   `:focus-visible` styles, and animation only under `prefers-reduced-motion:
   no-preference`.
9. Clean html-validate and axe-core results; no horizontal scroll at 360px.

## Reference solution: HTML

```html title="index.html"
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Component Gallery — Weeknight Kitchen</title>
    <meta name="description" content="Accessible, keyboard-tested interface components for the Weeknight Kitchen site: tabs, dialog, popover, disclosure, toggle, form errors and cards.">
    <link rel="stylesheet" href="styles.css">
    <script src="gallery.js" defer></script>
  </head>
  <body>
    <a class="skip-link" href="#main">Skip to components</a>
    <header class="site-header">
      <p class="logo"><a href="index.html">Weeknight Kitchen</a></p>
      <nav aria-label="Main">
        <ul>
          <li><a href="recipes.html">Recipes</a></li>
          <li><a href="index.html" aria-current="page">Components</a></li>
        </ul>
      </nav>
    </header>

    <main id="main" tabindex="-1">
      <h1>Component gallery</h1>
      <p>Every component works with a keyboard, has a name and state in the accessibility
        tree, and respects reduced-motion settings.</p>

      <section aria-labelledby="tabs-heading">
        <h2 id="tabs-heading">Tabs</h2>
        <div class="tabs">
          <div role="tablist" aria-label="Recipe details">
            <button type="button" role="tab" id="tab-ingredients" aria-selected="true" aria-controls="panel-ingredients">Ingredients</button>
            <button type="button" role="tab" id="tab-method" aria-selected="false" aria-controls="panel-method" tabindex="-1">Method</button>
            <button type="button" role="tab" id="tab-notes" aria-selected="false" aria-controls="panel-notes" tabindex="-1">Notes</button>
          </div>
          <div role="tabpanel" id="panel-ingredients" aria-labelledby="tab-ingredients" tabindex="0">
            <ul><li>1 kg tomatoes</li><li>1 onion</li><li>500 ml stock</li></ul>
          </div>
          <div role="tabpanel" id="panel-method" aria-labelledby="tab-method" tabindex="0" hidden>
            <ol><li>Roast for 30 minutes.</li><li>Simmer with stock.</li><li>Blend.</li></ol>
          </div>
          <div role="tabpanel" id="panel-notes" aria-labelledby="tab-notes" tabindex="0" hidden>
            <p>Freezes well for up to three months.</p>
          </div>
        </div>
      </section>

      <section aria-labelledby="toggle-heading">
        <h2 id="toggle-heading">Toggle button and status message</h2>
        <button type="button" class="toggle" aria-pressed="false" id="save">
          <svg aria-hidden="true" width="16" height="16" viewBox="0 0 16 16"><path d="M8 14s-6-3.6-6-8a3.5 3.5 0 0 1 6-2.4A3.5 3.5 0 0 1 14 6c0 4.4-6 8-6 8z" fill="currentColor"/></svg>
          Save recipe
        </button>
        <p role="status" class="status" id="save-status"></p>
      </section>

      <section aria-labelledby="disclosure-heading">
        <h2 id="disclosure-heading">Disclosure</h2>
        <details>
          <summary>Can I make it ahead?</summary>
          <p>Yes. It keeps for three days in the fridge; reheat gently.</p>
        </details>
      </section>

      <section aria-labelledby="dialog-heading">
        <h2 id="dialog-heading">Modal dialog</h2>
        <button type="button" id="open-dialog">Remove from basket</button>
        <p id="dialog-result" role="status"></p>
        <dialog id="confirm" aria-labelledby="confirm-title">
          <form method="dialog">
            <h3 id="confirm-title">Remove tomato soup?</h3>
            <p>You can add it again later.</p>
            <div class="actions">
              <button type="submit" value="cancel">Keep it</button>
              <button type="submit" value="remove" class="danger">Remove</button>
            </div>
          </form>
        </dialog>
      </section>

      <section aria-labelledby="popover-heading">
        <h2 id="popover-heading">Popover</h2>
        <p>Serving size
          <button type="button" class="info" popovertarget="serving-help" aria-label="What counts as a serving?">?</button>
        </p>
        <div id="serving-help" popover>
          <p>One serving is about 300 ml — a large mug.</p>
        </div>
      </section>

      <section aria-labelledby="form-heading">
        <h2 id="form-heading">Form with errors</h2>
        <form id="subscribe" novalidate>
          <div class="field">
            <label for="email">Email address</label>
            <p class="hint" id="email-hint">One recipe a week. No spam.</p>
            <input id="email" name="email" type="email" autocomplete="email" required aria-describedby="email-hint">
          </div>
          <button type="submit">Subscribe</button>
        </form>
      </section>

      <section aria-labelledby="cards-heading">
        <h2 id="cards-heading">Responsive card (container query)</h2>
        <div class="demo-columns">
          <div class="card-slot">
            <article class="card">
              <div class="card-thumb" aria-hidden="true"></div>
              <div>
                <h3><a href="soup.html">Roasted tomato soup</a></h3>
                <p>45 minutes · serves 4</p>
              </div>
            </article>
          </div>
          <div class="card-slot narrow">
            <article class="card">
              <div class="card-thumb" aria-hidden="true"></div>
              <div>
                <h3><a href="naan.html">Quick skillet naan</a></h3>
                <p>25 minutes · makes 6</p>
              </div>
            </article>
          </div>
        </div>
      </section>
    </main>

    <footer class="site-footer">
      <p><small>Course exercise — Level 3, HTML &amp; CSS Mastery Path.</small></p>
    </footer>
  </body>
</html>
```

## Reference solution: JavaScript

About 70 lines, for the three components HTML can't do alone — tabs, the toggle's
status message and the form errors — plus reading the dialog's return value.

```js title="gallery.js"
// Tabs: roving tabindex, arrow keys, Home/End, automatic activation
for (const tablist of document.querySelectorAll('[role="tablist"]')) {
  const tabs = [...tablist.querySelectorAll('[role="tab"]')];
  const select = (tab) => {
    for (const t of tabs) {
      const selected = t === tab;
      t.setAttribute('aria-selected', String(selected));
      t.tabIndex = selected ? 0 : -1;
      document.getElementById(t.getAttribute('aria-controls')).hidden = !selected;
    }
    tab.focus();
  };
  tablist.addEventListener('click', (e) => {
    const tab = e.target.closest('[role="tab"]');
    if (tab) select(tab);
  });
  tablist.addEventListener('keydown', (e) => {
    const i = tabs.indexOf(document.activeElement);
    const next = { ArrowRight: i + 1, ArrowLeft: i - 1, Home: 0, End: tabs.length - 1 }[e.key];
    if (next === undefined) return;
    e.preventDefault();
    select(tabs[(next + tabs.length) % tabs.length]);
  });
}

// Toggle button + live region
const save = document.getElementById('save');
const saveStatus = document.getElementById('save-status');
save.addEventListener('click', () => {
  const pressed = save.getAttribute('aria-pressed') === 'true';
  save.setAttribute('aria-pressed', String(!pressed));
  saveStatus.textContent = pressed ? 'Removed from saved recipes.' : 'Saved to your recipes.';
});

// Dialog
const dialog = document.getElementById('confirm');
const result = document.getElementById('dialog-result');
document.getElementById('open-dialog').addEventListener('click', () => dialog.showModal());
dialog.addEventListener('close', () => {
  result.textContent = dialog.returnValue === 'remove' ? 'Tomato soup removed.' : '';
});

// Form errors
const form = document.getElementById('subscribe');
form.addEventListener('submit', (e) => {
  e.preventDefault();
  const input = form.elements.email;
  let error = document.getElementById('email-error');
  if (input.checkValidity()) {
    error?.remove();
    input.removeAttribute('aria-invalid');
    input.setAttribute('aria-describedby', 'email-hint');
    return;
  }
  if (!error) {
    error = document.createElement('p');
    error.id = 'email-error';
    error.className = 'error';
    input.before(error);
  }
  error.textContent = input.validity.valueMissing
    ? 'Enter your email address'
    : 'Enter an email address in the format name@example.com';
  input.setAttribute('aria-invalid', 'true');
  input.setAttribute('aria-describedby', 'email-error email-hint');
  input.focus();
});
```

## Reference solution: CSS

```css title="styles.css"
@layer reset, base, components;

@layer reset {
  *, *::before, *::after { box-sizing: border-box; }
  body { margin: 0; }
  button, input { font: inherit; }
  [hidden] { display: none !important; }
}

@layer base {
  :root {
    color-scheme: light dark;
    --bg: light-dark(oklch(99% 0.004 80), oklch(18% 0.005 60));
    --surface: light-dark(white, oklch(24% 0.006 60));
    --text: light-dark(oklch(22% 0.01 60), oklch(93% 0.005 80));
    --muted: light-dark(oklch(45% 0.01 60), oklch(76% 0.01 80));
    --border: light-dark(oklch(85% 0.01 60), oklch(40% 0.01 60));
    --brand: light-dark(oklch(48% 0.15 38), oklch(76% 0.12 50));
    --on-brand: light-dark(white, oklch(20% 0.02 50));
    --error: light-dark(oklch(48% 0.19 25), oklch(75% 0.14 25));
    --focus: light-dark(oklch(45% 0.15 255), oklch(78% 0.1 255));
    --radius: 0.5rem;
  }
  body {
    font-family: system-ui, -apple-system, "Segoe UI", Roboto, sans-serif;
    line-height: 1.55;
    background: var(--bg);
    color: var(--text);
  }
  main, .site-header, .site-footer { width: min(100% - 2rem, 50rem); margin-inline: auto; }
  main:focus { outline: none; }
  h1, h2, h3 { line-height: 1.2; }
  section { margin-block: 2.5rem; }
  a { color: var(--brand); }
  :focus-visible { outline: 3px solid var(--focus); outline-offset: 2px; }
  html { scroll-padding-top: 1rem; }
}

@layer components {
  .skip-link { position: absolute; left: 1rem; top: -10rem; }
  .skip-link:focus { top: 1rem; background: var(--surface); padding: 0.5rem 1rem; z-index: 1; }

  .site-header {
    display: flex; flex-wrap: wrap; align-items: center; gap: 1rem;
    padding-block: 1rem; border-bottom: 1px solid var(--border);
    & .logo { margin: 0; font-weight: 700; }
    & nav { margin-inline-start: auto; }
    & ul { display: flex; gap: 1rem; list-style: none; margin: 0; padding: 0; }
    & [aria-current="page"] { font-weight: 700; }
  }

  button {
    padding: 0.55rem 1rem;
    border: 1px solid var(--border);
    border-radius: var(--radius);
    background: var(--surface);
    color: var(--text);
    cursor: pointer;
    min-block-size: 2.5rem;
  }

  /* Tabs */
  [role="tablist"] { display: flex; gap: 0.25rem; border-bottom: 1px solid var(--border); }
  [role="tab"] {
    border-radius: var(--radius) var(--radius) 0 0;
    border-bottom: 0;
    margin-bottom: -1px;
    &[aria-selected="true"] { border-color: var(--brand); box-shadow: inset 0 3px 0 var(--brand); font-weight: 600; }
  }
  [role="tabpanel"] { padding: 1rem; border: 1px solid var(--border); border-top: 0; }

  /* Toggle */
  .toggle {
    display: inline-flex; align-items: center; gap: 0.5rem;
    & svg { color: var(--muted); }
    &[aria-pressed="true"] { border-color: var(--brand); font-weight: 600; }
    &[aria-pressed="true"] svg { color: var(--brand); }
  }
  .status { min-height: 1.5em; color: var(--muted); }

  /* Disclosure */
  details {
    border: 1px solid var(--border); border-radius: var(--radius); padding: 0.5rem 1rem;
    & summary { cursor: pointer; font-weight: 600; }
    & summary::marker { color: var(--brand); }
  }

  /* Dialog */
  dialog {
    max-width: min(28rem, 100% - 2rem);
    border: 1px solid var(--border); border-radius: 0.75rem; padding: 1.5rem;
    background: var(--surface); color: var(--text);
    & h3 { margin-top: 0; }
    &::backdrop { background: rgb(0 0 0 / 0.5); }
  }
  body:has(dialog[open]) { overflow: hidden; }
  .actions { display: flex; flex-wrap: wrap; justify-content: flex-end; gap: 0.5rem; }
  .danger { background: var(--error); color: var(--on-brand); border-color: var(--error); }

  /* Popover */
  .info { min-block-size: 1.75rem; min-inline-size: 1.75rem; padding: 0; border-radius: 50%; font-weight: 700; }
  [popover] {
    max-width: 18rem; padding: 0.75rem 1rem; margin: 0;
    border: 1px solid var(--border); border-radius: var(--radius);
    background: var(--surface); color: var(--text);
    box-shadow: 0 6px 20px rgb(0 0 0 / 0.2);
    & p { margin: 0; }
  }
  @supports (anchor-name: --a) {
    .info { anchor-name: --info; }
    #serving-help { position-anchor: --info; inset: auto; position-area: bottom span-right; margin-top: 0.25rem; position-try-fallbacks: flip-block; }
  }

  /* Form */
  .field { display: grid; gap: 0.25rem; margin-bottom: 1rem; max-width: 24rem; }
  .field label { font-weight: 600; }
  .hint { margin: 0; color: var(--muted); font-size: 0.9rem; }
  .field input { padding: 0.5rem; border: 1px solid var(--border); border-radius: var(--radius); background: var(--surface); color: var(--text); }
  .error { margin: 0; color: var(--error); font-weight: 600; }
  [aria-invalid="true"] { border: 2px solid var(--error); }

  /* Container-query card */
  .demo-columns { display: grid; grid-template-columns: minmax(0, 2fr) minmax(0, 1fr); gap: 1rem; }
  .card-slot { container: card / inline-size; }
  .card {
    display: grid; gap: 0.75rem; padding: 1rem;
    border: 1px solid var(--border); border-radius: var(--radius); background: var(--surface);
    & h3 { margin: 0; font-size: clamp(1rem, 0.85rem + 2cqi, 1.35rem); }
    & p { margin: 0.25rem 0 0; color: var(--muted); }
  }
  .card-thumb { aspect-ratio: 16 / 9; border-radius: calc(var(--radius) / 2); background: linear-gradient(135deg, oklch(70% 0.12 40), oklch(55% 0.15 30)); }
  @container card (width >= 20rem) {
    .card { grid-template-columns: 7rem 1fr; align-items: center; }
    .card-thumb { aspect-ratio: 1; }
  }
  @media (width < 36rem) {
    .demo-columns { grid-template-columns: 1fr; }
  }

  .site-footer { padding-block: 2rem; border-top: 1px solid var(--border); color: var(--muted); }

  @media (prefers-reduced-motion: no-preference) {
    button { transition: background-color 120ms ease-out, border-color 120ms ease-out; }
    [popover], dialog { transition: opacity 150ms ease-out, display 150ms allow-discrete, overlay 150ms allow-discrete; }
    @starting-style { [popover]:popover-open, dialog[open] { opacity: 0; } }
  }
}
```

## Testing it

### Validation

The first html-validate run reported three errors:

```text
  74:16  error  <button> is missing recommended "type" attribute  no-implicit-button-type
  75:16  error  <button> is missing recommended "type" attribute  no-implicit-button-type
  99:12  error  <button> is missing recommended "type" attribute  no-implicit-button-type
```

Those were the dialog's two buttons and the Subscribe button. They *were* meant to submit
(a button in a form defaults to `type="submit"`), but writing `type="submit"` makes the
intent explicit and protects against the classic bug of a non-submit button submitting
a form. After adding the attribute, validation was clean.

### Automated audit

axe-core 4.13 (WCAG 2.0–2.2 A/AA plus best practices) found **0 violations** at 360px,
800px and 1280px in light mode, and at 360px and 1280px in dark mode — and still 0 with
the dialog open and with the popover open. `scrollWidth` matched the viewport width at
every size. The container-query cards' computed columns:

```text
360px  (both slots full width):  wide slot "112px 170px"     narrow slot "112px 170px"
800px  (slots side by side):     wide slot "112px 343.328px" narrow slot "216.672px"
1280px:                          wide slot "112px 364.656px" narrow slot "227.344px"
```

On phones the columns stack, so both slots are wide enough for the horizontal layout. On
wider screens the narrow slot is under 20rem, so its card switches to the stacked layout
— on the same page, at the same viewport width, as the wide one.

### Keyboard test log

A Playwright script pressed keys and recorded focus and state after each step:

```text
Tab x5: ["Skip to components","Weeknight Kitchen","Recipes","Components","tab-ingredients"]
ArrowRight -> tab-method        panels hidden: [ingredients true, method false]
End -> tab-notes
ArrowRight (wraps) -> tab-ingredients
Tab from tablist -> panel-ingredients
Tab -> save
Space on save: aria-pressed= true status= Saved to your recipes.
Tab -> Can I make it ahead?
details open after Enter: true
Tab -> open-dialog
dialog open, focus: true Keep it
after Remove: open= false focus= open-dialog result= Tomato soup removed.
Tab -> ?
popover open: true
popover position: {"buttonBottom":638,"buttonLeft":332,"popTop":642,"popLeft":332}
after Escape: popover open= false focus= ?
after bad submit: focus= email error= Enter an email address in the format name@example.com invalid= true
after good submit: error present= false invalid= null describedby= email-hint
```

Every component was reachable and operable. Notice the tab list costs **one** Tab stop,
and the next Tab goes to the *panel* (it has `tabindex="0"` so keyboard users can reach
and scroll panel content that has no focusable elements). The popover opened directly
below its button — top at 642, 4px under the button's bottom edge at 638, matching the
`margin-top: 0.25rem` — because anchor positioning is supported in the Chromium build we
tested; without it, the `@supports` block is skipped and the popover opens centred.

The tab list's accessibility tree:

```text
- tablist "Recipe details":
  - tab "Ingredients" [selected]
  - tab "Method"
  - tab "Notes"
- tabpanel "Ingredients":
  - list:
    - listitem: 1 kg tomatoes
    …
```

What the script **can't** tell you is how the page sounds. We didn't have a screen
reader in our test environment; before shipping components like these, test them with at
least VoiceOver and NVDA.

## How It Actually Works

Look at where the behaviour comes from in each component:

| Component | Semantics from | Keyboard from | State kept in |
|-----------|---------------|---------------|---------------|
| Tabs | ARIA roles | **our script** (roving tabindex) | `aria-selected`, `hidden` |
| Toggle | `<button>` + `aria-pressed` | native button | `aria-pressed` |
| Status | `role="status"` | — | text content |
| Disclosure | `<details>` | native | `open` attribute |
| Dialog | `<dialog>` | native (focus, Escape, inert) | `open`, `returnValue` |
| Popover | button + `popover` | native (light dismiss, Escape) | `:popover-open` |
| Form error | label + `aria-describedby` | native | `aria-invalid` |

Only the tabs needed custom keyboard code — everything else inherits behaviour from HTML.
In every case, **the state is stored in an attribute** that the accessibility tree reads,
and the CSS styles **that same attribute** (`[aria-selected="true"]`,
`[aria-pressed="true"]`, `[aria-invalid="true"]`, `details[open]`). There's no separate
`.is-active` class that could drift out of sync with what assistive technology is told.
That single-source-of-truth approach is the most useful habit to take from this level.

## Common mistakes

- **Tabs where every tab is a Tab stop**, or arrow keys that don't wrap.
- **Toggle buttons that change their label instead of their state** ("Save" → "Saved"
  *and* `aria-pressed` gives a confusing double signal — pick one).
- **Status messages in regions created on the fly**, which may never be announced.
- **Error messages left in place after the field is fixed.**
- **Components tested only with a mouse and an automated checker.**

## Exercise

1. Build the gallery, then add an eighth component: an accessible "star rating" input
   made from radio buttons in a fieldset, styled as stars.
2. Write your own keyboard test script (Playwright, or by hand with a checklist) that
   exercises every component. Include Shift+Tab.
3. Test the whole page with a screen reader. Note everything announced for each
   component and compare with what you expected.
4. Switch the operating system to dark mode and reduced motion, and check that both are
   respected.
