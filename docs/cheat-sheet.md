---
title: "HTML & CSS Cheat Sheet: Quick Reference, Beginner to Advanced"
description: "A free HTML & CSS cheat sheet: 27 quick-reference tables from every level of the HTML & CSS Mastery Path, each linked to its full lesson."
---

# HTML & CSS Cheat Sheet

Quick-reference tables from **27 lessons** of the free HTML & CSS Mastery Path, in course order from beginner to advanced. Each section links to the full lesson with explanations and runnable examples.

## Level 1 · Entry

### [02 · Document Structure &amp; the Head Element](level-1/02-document-structure.md)

*`<link>`*

| `rel` | What it does |
|-------|--------------|
| `stylesheet` | Loads CSS. Render-blocking by default. |
| `icon` | The favicon. SVG favicons work in current Chromium, Firefox and Safari; provide a PNG or `.ico` too if you need older browsers. |
| `preload` | Starts fetching a resource early (Level 4 · 02). |
| `canonical` | The preferred URL for this content (Level 4 · 06). |
| `alternate` | Alternative versions — translations, RSS feeds. |

### [03 · Text, Links &amp; Semantic Meaning](level-1/03-text-links-semantics.md)

*Text-level semantics*

| Element | Meaning | Default look |
|---------|---------|--------------|
| `<em>` | Stress emphasis — changes the meaning of the sentence | italic |
| `<strong>` | Importance, seriousness or urgency | bold |
| `<i>` | Alternate voice: foreign phrases, taxonomic names, thoughts, ship names | italic |
| `<b>` | Draws attention without extra importance: keywords, product names | bold |
| `<mark>` | Highlighted for relevance, e.g. search matches | yellow background |
| `<small>` | Side comments, fine print | smaller |
| `<abbr title="...">` | Abbreviation, with expansion | dotted underline in some browsers |
| `<cite>` | Title of a work | italic |
| `<q>` | Inline quotation — the browser adds quotation marks | quotes |
| `<code>`, `<kbd>`, `<samp>`, `<var>` | Code, keyboard input, program output, variables | monospace / italic |
| `<time datetime="...">` | A date or time with a machine-readable value | none |
| `<data value="...">` | Any value with a machine-readable version | none |
| `<sub>`, `<sup>` | Subscript and superscript where typographically required (H<sub>2</sub>O) | lowered/raised |

### [04 · Images &amp; Media](level-1/04-images-media.md)

*Writing `alt` text*

| Situation | Good `alt` |
|-----------|-----------|
| Photo that illustrates the content | `alt="A bowl of tomato soup topped with basil"` |
| Image that is a link | Describe the destination: `alt="Home"` for a logo linking home |
| Chart | The takeaway: `alt="Sales doubled between March and June"` — and put the data in a table or text nearby |
| Image of text | The text itself |
| Purely decorative (a flourish, a background texture) | `alt=""` — empty, not missing |

### [05 · Tables](level-1/05-tables.md)

*A complete, accessible table*

| Element | Role |
|---------|------|
| `<table>` | The table itself |
| `<caption>` | Its title. Must be the first child. Becomes the table's accessible name. |
| `<thead>`, `<tbody>`, `<tfoot>` | Row groups: header rows, body rows, footer (e.g. totals) rows |
| `<tr>` | A row |
| `<th>` | A header cell |
| `<td>` | A data cell |
| `scope="col"` / `scope="row"` | Which cells this header applies to |

### [06 · Forms &amp; Native Validation](level-1/06-forms.md)

*Input types*

| Type | Use for | What you get |
|------|---------|--------------|
| `text` | Anything free-form | Default |
| `email` | Email addresses | Format validation, `@` keyboard |
| `tel` | Phone numbers | Numeric phone keypad; no format validation (formats vary worldwide) |
| `url` | Web addresses | URL validation |
| `number` | Quantities you'd increment | Spin buttons, `min`/`max`/`step` validation |
| `password` | Secrets | Masked input |
| `search` | Search boxes | Clear button in some browsers |
| `date`, `time`, `datetime-local`, `month`, `week` | Dates and times | Native pickers |
| `checkbox` | Independent yes/no | |
| `radio` | One choice from a group (same `name`) | Arrow keys move within the group |
| `range` | Approximate value on a slider | |
| `color` | Colour picker | |
| `file` | Uploads (`accept=".pdf,image/*"`, `multiple`) | Needs `method="post"` and `enctype="multipart/form-data"` |
| `hidden` | Values the user doesn't edit | |

### [07 · CSS Syntax &amp; Selectors](level-1/07-css-selectors.md)

*Basic selectors*

| Selector | Matches |
|----------|---------|
| `p` | Every `<p>` (type selector) |
| `.note` | Every element with `note` in its `class` list |
| `#main` | The element with `id="main"` |
| `*` | Every element |
| `p.note` | `<p>` elements with class `note` (no space = same element) |
| `.card.featured` | Elements with **both** classes |

### [08 · The Cascade, Specificity &amp; Inheritance](level-1/08-cascade-specificity.md)

*Origins and `!important`*

| Precedence (highest first) |
|---------------------------|
| Transitions in progress |
| User-agent `!important` |
| User `!important` |
| Author `!important` |
| Animations (`@keyframes`) |
| Author normal |
| User normal |
| User-agent normal |

### [09 · The Box Model, Display &amp; Units](level-1/09-box-model-units.md)

*Display types: block, inline and friends*

| `display` | Outer behaviour | Respects `width`/`height`? | Vertical margins? |
|-----------|-----------------|---------------------------|-------------------|
| `block` | Starts on a new line, fills available width | Yes | Yes |
| `inline` | Flows within a line of text | **No** | **No** (padding draws but doesn't push lines apart) |
| `inline-block` | Flows within text, but is a box inside | Yes | Yes |
| `none` | Not rendered at all, removed from the accessibility tree | — | — |
| `flex`, `grid` | Block outside, flex/grid layout for children (Level 2) | Yes | Yes |
| `inline-flex`, `inline-grid` | Inline outside, flex/grid inside | Yes | Yes |

## Level 2 · Intermediate

### [01 · Typography &amp; Web Fonts](level-2/01-typography.md)

*`font-display`: what happens while the font loads*

| Value | Behaviour |
|-------|-----------|
| `auto` | Browser decides (usually like `block`) |
| `block` | Text invisible for up to ~3s, then fallback, swap when loaded |
| `swap` | Fallback immediately, swap whenever the font arrives |
| `fallback` | Very short invisible period (~100ms), short swap window (~3s), then stick with the fallback |
| `optional` | Very short invisible period; use the font only if it's already available (e.g. cached), otherwise keep the fallback for this page view |

### [02 · Color, Backgrounds &amp; Gradients](level-2/02-color-backgrounds.md)

*Contrast*

| Content | Minimum ratio (AA) | AAA |
|---------|--------------------|-----|
| Body text | **4.5:1** | 7:1 |
| Large text (≥ 24px, or ≥ 18.66px bold) | **3:1** | 4.5:1 |
| UI components and meaningful graphics (borders of inputs, icons, focus rings) | **3:1** | — |

### [03 · Flexbox Fundamentals](level-2/03-flexbox-fundamentals.md)

*The two axes*

| `flex-direction` | Main axis | Cross axis |
|------------------|-----------|------------|
| `row` (default) | horizontal, in the text direction | vertical |
| `column` | vertical, top to bottom | horizontal |
| `row-reverse`, `column-reverse` | reversed | — |

### [04 · Flexbox in Depth: How Sizes Are Resolved](level-2/04-flexbox-in-depth.md)

*The three properties*

| Shorthand | Means | Use for |
|-----------|-------|---------|
| `flex: initial` (the default) | `0 1 auto` | Content-sized, can shrink, won't grow |
| `flex: auto` | `1 1 auto` | Start at content size, share extra space |
| `flex: 1` | `1 1 0%` | **Ignore content size**, share all space equally |
| `flex: none` | `0 0 auto` | Rigid: content size, never grows or shrinks |
| `flex: 0 0 250px` | fixed 250px | Sidebars, fixed columns |
| `flex: 2 1 200px` | start at 200px, grow with weight 2 | |

### [05 · CSS Grid Fundamentals](level-2/05-grid-fundamentals.md)

*Alignment*

| Property | Aligns | Within |
|----------|--------|--------|
| `justify-items` / `justify-self` | items horizontally (inline axis) | their cell |
| `align-items` / `align-self` | items vertically (block axis) | their cell |
| `justify-content` / `align-content` | the whole set of tracks | the container, when tracks are smaller than it |
| `place-items` | shorthand for `align-items justify-items` | |

### [07 · Positioning &amp; Stacking Contexts](level-2/07-positioning-stacking.md)

*The five values*

| `position` | In normal flow? | Offsets (`top`/`left`/…) are relative to | Scrolls with page? |
|------------|-----------------|-------------------------------------------|--------------------|
| `static` (default) | yes | offsets ignored | yes |
| `relative` | yes (space kept) | its own normal position | yes |
| `absolute` | **no** (space collapses) | the **containing block**: nearest positioned ancestor | yes |
| `fixed` | **no** | the viewport (usually) | no |
| `sticky` | yes | its scroll container, once a threshold is crossed | until its parent scrolls away |

## Level 3 · Advanced

### [01 · Accessibility Foundations &amp; the Accessibility Tree](level-3/01-accessibility-foundations.md)

*Who uses what*

| People | Often use | What breaks for them |
|--------|-----------|----------------------|
| Blind and low-vision | Screen readers (NVDA, JAWS, VoiceOver, TalkBack, Narrator), magnifiers, zoom | Missing alt text, unlabelled controls, meaningless structure, text that doesn't reflow |
| Motor impairments | Keyboard, switch access, voice control (Voice Control, Dragon), eye tracking | Mouse-only interactions, tiny targets, no visible focus, time limits |
| Deaf and hard of hearing | Captions, transcripts | Uncaptioned video, audio-only alerts |
| Cognitive and learning | Consistent layouts, plain language, reader modes | Clutter, unpredictable changes, jargon, animations |
| Vestibular disorders | Reduced-motion settings | Parallax, large animations |
| Colour vision deficiency | Nothing special — they just see differently | Information conveyed by colour alone |

### [02 · Landmarks, Headings &amp; Navigation](level-3/02-landmarks-headings.md)

*Landmark elements*

| Element | Landmark role | Becomes a landmark when… |
|---------|---------------|--------------------------|
| `<header>` | `banner` | it's **not** inside `article`, `aside`, `main`, `nav` or `section` |
| `<nav>` | `navigation` | always |
| `<main>` | `main` | always (one per page) |
| `<aside>` | `complementary` | always in current browsers (some older mappings excluded nested ones) |
| `<footer>` | `contentinfo` | it's **not** inside `article`, `aside`, `main`, `nav` or `section` |
| `<section>` | `region` | it has an accessible name (`aria-label`/`aria-labelledby`) |
| `<form>` | `form` | it has an accessible name |
| `<search>` | `search` | always |

### [03 · Keyboard Access &amp; Focus Management](level-3/03-keyboard-focus.md)

*The keys*

| Key | Does |
|-----|------|
| <kbd>Tab</kbd> / <kbd>Shift</kbd>+<kbd>Tab</kbd> | Move focus to the next / previous focusable element |
| <kbd>Enter</kbd> | Follow a link; activate a button; submit a form from a text field |
| <kbd>Space</kbd> | Activate a button; toggle a checkbox; scroll the page when nothing is focused |
| Arrow keys | Move within a group: radio buttons, select options, tabs, menus; scroll |
| <kbd>Escape</kbd> | Close dialogs, popovers, menus |
| <kbd>Home</kbd> / <kbd>End</kbd> | Start/end of a text field or list |

### [04 · ARIA: When and How to Use It](level-3/04-aria.md)

*Common roles you'll actually write*

| Pattern | Markup |
|---------|--------|
| Icon-only button | `<button aria-label="…">` with `aria-hidden` SVG |
| Toggle button | `<button aria-pressed="false">`, flipped to `"true"` when on |
| Disclosure | `<button aria-expanded aria-controls>` + `hidden` panel |
| Status message | `role="status"` region |
| Tabs | `role="tablist"` > `role="tab"` (with `aria-selected`, roving tabindex) + `role="tabpanel"` |
| Switch | `<button role="switch" aria-checked>` or `<input type="checkbox" role="switch">` |
| List styled without bullets (Safari) | `<ul role="list">` |

### [06 · Container Queries](level-3/06-container-queries.md)

*`container-type` values*

| Value | Queries you can make | Containment applied |
|-------|---------------------|---------------------|
| `inline-size` | width (inline size) | inline-size: its width can't depend on its children |
| `size` | width and height | size: neither dimension can depend on its children |
| `normal` (default) | only style queries | none |

### [08 · Transitions &amp; Animations](level-3/08-transitions-animations.md)

*Timing functions*

| Function | Feels like |
|----------|-----------|
| `ease` (default) | Quick start, gentle finish |
| `ease-out` | Fast start, slow end — good for things **entering** |
| `ease-in` | Slow start, fast end — good for things **leaving** |
| `ease-in-out` | Slow at both ends — movement between two positions |
| `linear` | Constant speed — spinners, progress |
| `cubic-bezier(x1, y1, x2, y2)` | Custom curve |
| `steps(4)` | Discrete jumps — sprite animations |
| `linear(0, 0.6 20%, 1.1 50%, 1)` | A piecewise curve, e.g. springs and bounces |

### [10 · Project — An Accessible Component Gallery](level-3/10-project-component-gallery.md)

*How It Actually Works*

| Component | Semantics from | Keyboard from | State kept in |
|-----------|---------------|---------------|---------------|
| Tabs | ARIA roles | **our script** (roving tabindex) | `aria-selected`, `hidden` |
| Toggle | `<button>` + `aria-pressed` | native button | `aria-pressed` |
| Status | `role="status"` | — | text content |
| Disclosure | `<details>` | native | `open` attribute |
| Dialog | `<dialog>` | native (focus, Escape, inert) | `open`, `returnValue` |
| Popover | button + `popover` | native (light dismiss, Escape) | `:popover-open` |
| Form error | label + `aria-describedby` | native | `aria-invalid` |

## Level 4 · Master

### [01 · The Rendering Pipeline &amp; CSS Performance](level-4/01-rendering-performance.md)

*The pipeline, revisited*

| You change… | Stages that run |
|-------------|----------------|
| `width`, `height`, `margin`, `padding`, `top`, `font-size`, text content, adding/removing elements | style → **layout** → paint → composite |
| `color`, `background`, `box-shadow`, `border-radius`, `outline` | style → **paint** → composite |
| `transform`, `opacity` (on an element with its own layer) | style → **composite** only |

### [02 · Loading Performance: Critical CSS, Fonts &amp; Layout Shift](level-4/02-loading-performance.md)

*Core Web Vitals in brief*

| Metric | Measures | "Good" threshold (75th percentile of real visits) |
|--------|----------|---------------------------------------------------|
| **LCP** — Largest Contentful Paint | When the largest image or text block in the viewport rendered | ≤ 2.5 s |
| **CLS** — Cumulative Layout Shift | How much visible content moved unexpectedly | ≤ 0.1 |
| **INP** — Interaction to Next Paint | How quickly the page responds visually to clicks, taps and key presses | ≤ 200 ms |

### [03 · CSS Architecture at Scale](level-4/03-css-architecture.md)

*Pattern 3: layers as the architecture*

| Layer | Contains | Typical selectors |
|-------|----------|------------------|
| reset | Normalising browser defaults | elements, `:where()` |
| tokens | Custom properties only | `:root` |
| base | Element defaults: typography, links, forms | elements |
| layout | Page-level structure: containers, grids, stacks | `.l-*` classes |
| components | Self-contained UI pieces | one class per rule, BEM-style |
| utilities | Single-purpose helpers that must win | `.u-*` classes |
| overrides | Third-party fixes, temporary hacks — kept small and reviewed | anything |

### [05 · Logical Properties, Writing Modes &amp; Internationalization](level-4/05-logical-properties-i18n.md)

*Physical vs logical*

| Physical | Logical |
|----------|---------|
| `margin-left` | `margin-inline-start` |
| `margin-right` | `margin-inline-end` |
| `margin-top` / `margin-bottom` | `margin-block-start` / `margin-block-end` |
| `margin-left` + `margin-right` | `margin-inline` |
| `padding-top` + `padding-bottom` | `padding-block` |
| `border-left` | `border-inline-start` |
| `width` / `height` | `inline-size` / `block-size` |
| `max-width` | `max-inline-size` |
| `top` / `left` (positioning) | `inset-block-start` / `inset-inline-start` |
| `text-align: left` | `text-align: start` |
| `float: left` | `float: inline-start` |
| `border-top-left-radius` | `border-start-start-radius` |

### [07 · Print Styles &amp; Other Media](level-4/07-print-styles.md)

*Other media types and features*

| Feature | Use |
|---------|-----|
| `prefers-color-scheme` | Dark mode |
| `prefers-reduced-motion` | Motion sensitivity |
| `prefers-contrast` | Users asking for more (or less) contrast |
| `forced-colors: active` | Windows High Contrast / forced colours mode |
| `hover`, `pointer`, `any-pointer` | Input capabilities |
| `resolution` | High-DPI tweaks (`(resolution >= 2dppx)`) |
| `scripting: none` | JavaScript disabled |
| `display-mode: standalone` | Installed web app |

### [09 · Testing &amp; Auditing HTML and CSS](level-4/09-testing-auditing.md)

*The layers of checking*

| Layer | Tool | Catches | Speed |
|-------|------|---------|-------|
| Markup validity and conventions | html-validate (or the W3C Nu checker) | Invalid nesting, missing attributes, duplicate ids, implicit button types | Instant |
| CSS correctness and conventions | Stylelint | Unknown properties, duplicates, specificity rules, old syntax | Instant |
| Automated accessibility | axe-core (in the browser or in tests) | Missing names, contrast, ARIA misuse, landmark and heading problems | Seconds |
| Behaviour | Playwright (or similar) tests | Keyboard access, focus management, layout at sizes, forms | Seconds |
| Visual regression | Screenshot comparison | Unintended visual changes anywhere on a page | Seconds |
| Manual review | You, a screen reader, a keyboard | Everything automation can't judge: meaningful alt text, sensible reading order, clarity | Minutes |

---

Go deeper with the full [HTML & CSS Mastery Path](index.md) curriculum or test yourself with the [interview questions](interview-questions.md).
