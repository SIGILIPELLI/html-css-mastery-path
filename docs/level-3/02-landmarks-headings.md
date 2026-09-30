---
description: "Give pages a navigable structure: when elements become landmarks, labelling duplicates, heading outlines, skip links and aria-current."
---

# 02 · Landmarks, Headings & Navigation

Sighted users take in a page's layout at a glance — header up top, navigation, main
content, sidebar, footer — and jump straight to what they want. Screen-reader users get
the same ability from two structures you've been writing since Level 1: **landmarks** and
**headings**. Most screen-reader users navigate primarily by headings, and landmarks let
them skip whole regions in one keystroke. Getting both right is cheap and has a huge
payoff.

## Landmark elements

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

We rendered a page using all of them and printed Chromium's accessibility tree:

```html
<header><p>Site</p></header>
<nav aria-label="Main"><a href="/">Home</a></nav>
<main>
  <article>
    <header><h1>Title</h1></header>
    <section><h2>Unnamed section</h2></section>
    <section aria-labelledby="n"><h2 id="n">Named section</h2></section>
    <aside><p>Related</p></aside>
    <footer><p>article footer</p></footer>
  </article>
</main>
<aside aria-label="Sidebar"><p>ads</p></aside>
<footer><p>Site footer</p></footer>
<form><input aria-label="x"></form>
<form aria-label="Newsletter"><input aria-label="Email"></form>
<search><input type="search" aria-label="Search site"></search>
```

```text
- banner:
  - paragraph: Site
- navigation "Main":
  - link "Home"
- main:
  - article:
    - heading "Title" [level=1]
    - heading "Unnamed section" [level=2]
    - region "Named section":
      - heading "Named section" [level=2]
    - complementary:
      - paragraph: Related
    - paragraph: article footer
- complementary "Sidebar":
  - paragraph: ads
- contentinfo:
  - paragraph: Site footer
- textbox "x"
- form "Newsletter":
  - textbox "Email"
- search:
  - searchbox "Search site"
```

The details that surprise people:

- The article's `<header>` and `<footer>` did **not** become banner/contentinfo — they
  belong to the article. Only the page-level ones did.
- The unnamed `<section>` produced no region at all; its heading was promoted. The named
  one became `region "Named section"`. That's deliberate: if every `<section>` were a
  landmark, the landmark list would be noise.
- Same for forms: the unnamed form disappeared; the named one is a `form` landmark.
- `<search>` is a newer element for search forms; before it existed you'd write
  `<form role="search">`.

## Rules for landmarks

1. **All content should be inside a landmark.** Content outside any landmark is easy to
   miss when navigating by landmarks. Page-level header, nav, main, footer usually cover
   everything.
2. **One `<main>` per page** (you can have more only if all but one are `hidden`).
3. **Label duplicates.** Two `<nav>`s announced as "navigation, navigation" are useless;
   `aria-label="Main"` and `aria-label="Breadcrumb"` fix it. Don't include the role in
   the label ("Main navigation navigation").
4. **Don't overdo it.** A landmark per paragraph defeats the purpose.

## Headings

Headings are the primary navigation structure. Screen readers list them, and many users
jump through them with a single key (`H` in NVDA and JAWS browse mode) to skim a page
exactly as a sighted user scans for bold text.

A good heading outline reads like a table of contents:

```text
h1 Roasted Tomato Soup
  h2 Ingredients
  h2 Method
    h3 Roasting
    h3 Blending
  h2 Nutrition
```

- **One `<h1>`** that matches the page's purpose (usually close to the `<title>`).
- **No skipped levels on the way down.** `h2` → `h4` makes users wonder what they missed.
- **Headings for every section** that a sighted user would perceive as separate —
  including sidebars and footers. If the design shows no visible heading, add a
  visually-hidden one (lesson 01).
- **Heading text alone should make sense**, since it's read out of context.

### The "document outline algorithm" never happened

Older HTML specs described an outline algorithm in which every `<section>` would reset
heading levels, so you could use `<h1>` everywhere. **No browser or screen reader ever
implemented it**, and the idea has been removed from the spec. Heading level is exactly
the number in the tag. Use `h1`–`h6` according to real nesting.

## Skip links

Keyboard users otherwise have to tab through every header link on every page to reach
the content. A skip link — the first focusable element — jumps them past it:

```html
<body>
  <a class="skip-link" href="#main">Skip to main content</a>
  <header>…many links…</header>
  <main id="main" tabindex="-1">…</main>
</body>
```

```css
.skip-link {
  position: absolute;
  left: 1rem;
  top: -10rem;          /* off-screen until focused */
}
.skip-link:focus {
  top: 1rem;
  z-index: 100;
  background: white;
  padding: 0.5rem 1rem;
}
```

The `tabindex="-1"` on `<main>` makes it programmatically focusable, which makes some
browsers move keyboard focus (not just the scroll position) to it. Modern browsers
generally move the "sequential focus navigation starting point" to a fragment target
anyway; the attribute is harmless insurance. We checked both in Chromium — press Tab
(focuses the skip link), Enter, then Tab again:

```text
<main> without tabindex:  focus after Enter = BODY,  next Tab = first link in main
<main tabindex="-1">:     focus after Enter = main,  next Tab = first link in main
```

Either way the next Tab lands inside the main content, skipping the header links. With
`tabindex="-1"`, focus is *on* `<main>` itself, so a screen reader starts reading from
there immediately.

(If you test in Safari, note that by default <kbd>Tab</kbd> only moves between form
controls; links are reached with <kbd>Option</kbd>+<kbd>Tab</kbd> unless "Press Tab to
highlight each item on a webpage" is turned on in Safari's settings.)

Don't hide the skip link with `display: none` — it must be focusable.

## Navigation patterns

### Current page

```html
<nav aria-label="Main">
  <ul>
    <li><a href="/" aria-current="page">Home</a></li>
    <li><a href="/recipes">Recipes</a></li>
  </ul>
</nav>
```

`aria-current="page"` is announced ("Home, current page, link") and is a perfect styling
hook: `[aria-current="page"] { font-weight: 700; }`. Other values exist for other kinds
of "current": `step` in a multi-step process, `date` in a calendar, `location` in a
breadcrumb trail or map.

### Breadcrumbs

```html
<nav aria-label="Breadcrumb">
  <ol>
    <li><a href="/">Home</a></li>
    <li><a href="/recipes">Recipes</a></li>
    <li><a href="/recipes/soup" aria-current="page">Tomato soup</a></li>
  </ol>
</nav>
```

An ordered list, because the order is the meaning. Draw separators with CSS
(`li + li::before { content: "/" }`) so they aren't read out as "slash" between every
item — or, if your screen reader does read them, use `content: "/" / ""` — the
alternative-text syntax for generated content, which gives it an empty accessible
text.

### Page title changes

In single-page applications, where "navigation" doesn't load a new document, update
`document.title` and move focus to the new content's heading after each route change.
Otherwise screen-reader users get no signal that anything happened.

## How It Actually Works

Screen readers build **navigation lists** from the accessibility tree: all headings (with
levels), all landmarks (with names), all links, all form fields. Commands like "next
heading" or "next landmark" walk those lists. A landmark's announced label is its role
plus its name — "navigation, Breadcrumb" — which is why duplicates need names and why
names shouldn't repeat the role.

The context rules for `<header>`/`<footer>` come from HTML-AAM: the browser checks the
element's ancestors when computing its role. A header nested in sectioning content or
`main` gets a generic role instead of `banner`. The `section` rule is similar: its role
is `region` only when it has an accessible name, otherwise generic.

Fragment navigation (`href="#main"`) does two things: scrolls the target into view and
sets the **sequential focus navigation starting point** to it, so the *next* Tab goes to
the first focusable element after the target. That's why skip links work even though
`<main>` isn't normally focusable.

## Common mistakes

- **Multiple unlabelled `<nav>`s**, or labels like "navigation menu nav".
- **Page content outside all landmarks** — a promo banner between header and main.
- **Heading levels chosen for size**, or skipped levels.
- **Multiple `<h1>`s from components** that each think they're the page title.
- **Skip links hidden with `display: none`**, or pointing at an id that doesn't exist.
- **`<section>` everywhere** expecting it to create landmarks.
- **SPAs that change the view without updating the title or moving focus.**

## Exercise

1. List the landmarks on your product page using devtools or a screen reader's landmark
   menu (VoiceOver rotor: <kbd>Ctrl</kbd>+<kbd>Option</kbd>+<kbd>U</kbd>). Add labels where
   two share a role.
2. Extract your page's heading outline (the "HeadingsMap" browser extension, or
   `[...document.querySelectorAll('h1,h2,h3,h4,h5,h6')].map(h => h.tagName + ' ' + h.textContent)`
   in the console). Fix skipped levels.
3. Add a skip link that is invisible until focused. Test: load the page, press Tab once,
   press Enter, press Tab again — focus should be inside the main content.
4. Build a breadcrumb with CSS separators and `aria-current`, and listen to it with a
   screen reader.
