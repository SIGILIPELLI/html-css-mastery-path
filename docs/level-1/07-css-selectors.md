---
description: "CSS rule syntax and everyday selectors: type, class, id, attribute, combinators, pseudo-classes and pseudo-elements, with measured match counts."
---

# 07 · CSS Syntax & Selectors

CSS has one core idea: **select some elements, then declare values for their
properties.** Everything else — the cascade, layout, animation — builds on that. This
lesson covers the "select" half thoroughly, because precise selectors are what keep a
stylesheet small and predictable.

## Anatomy of a rule

```css
/* selector      declaration block */
.recipe h2 {
  color: #b3401a;         /* declaration: property: value; */
  margin-block: 1.5rem 0.5rem;
}
```

- The **selector** (`.recipe h2`) chooses elements.
- Each **declaration** sets one property. The semicolon *separates* declarations; the
  last one can omit it, but always writing it avoids bugs when you add a line.
- An invalid declaration is **ignored on its own** — the rest of the rule still applies.
  An invalid *selector* makes the whole rule be ignored (with an exception for forgiving
  selectors, lesson Level 3 · 07).
- Comments are `/* ... */`. There are no `//` comments in CSS.

Several selectors can share one block, separated by commas:

```css
h1, h2, h3 { line-height: 1.2; }
```

## Three places CSS can live

```html
<!-- 1. External stylesheet: the normal choice -->
<link rel="stylesheet" href="styles.css">

<!-- 2. A <style> element in the document -->
<style>
  .notice { background: #fff3cd; }
</style>

<!-- 3. The style attribute on one element -->
<p style="color: red">Avoid this except for quick experiments.</p>
```

External files are cached across pages and keep content and presentation apart. Inline
`style` attributes override almost everything (lesson 08) and can't use media queries or
pseudo-classes, which makes them hard to manage.

## Basic selectors

| Selector | Matches |
|----------|---------|
| `p` | Every `<p>` (type selector) |
| `.note` | Every element with `note` in its `class` list |
| `#main` | The element with `id="main"` |
| `*` | Every element |
| `p.note` | `<p>` elements with class `note` (no space = same element) |
| `.card.featured` | Elements with **both** classes |

The `class` attribute holds a space-separated list: `<article class="card featured">`
matches `.card`, `.featured` and `.card.featured`. Classes are your main tool; IDs are
unique per page and have very high specificity (lesson 08), so most teams avoid them for
styling.

## Attribute selectors

| Selector | Matches when the attribute… |
|----------|------------------------------|
| `[disabled]` | exists |
| `[type="email"]` | equals exactly |
| `[href^="https"]` | starts with |
| `[href$=".pdf"]` | ends with |
| `[href*="example"]` | contains |
| `[class~="note"]` | contains the whole word (how `.note` works internally) |
| `[type="email" i]` | equals, case-insensitively |

(There's also a rarer "dash-match" form, `[lang|="en"]`, which matches `en` or anything
starting with `en-`.)

Attribute selectors are great for styling based on state or data that's already in the markup:

```css
a[href$=".pdf"]::after { content: " (PDF)"; }
a[target="_blank"]::after { content: " ↗"; }
input[type="checkbox"], input[type="radio"] { accent-color: #b3401a; }
```

## Combinators

Combinators select elements by their relationship to others:

| Combinator | Example | Matches |
|------------|---------|---------|
| descendant (space) | `article p` | any `p` inside an `article`, at any depth |
| child `>` | `article > p` | `p` whose **parent** is an `article` |
| next sibling `+` | `h2 + p` | the `p` immediately after an `h2` |
| subsequent sibling `~` | `h2 ~ p` | every `p` after an `h2` with the same parent |

## Pseudo-classes: state and position

Pseudo-classes start with one colon and select by state or position in the tree.

**User-interaction state:**

```css
a:hover { text-decoration-thickness: 2px; }
a:focus-visible { outline: 3px solid #1a73e8; outline-offset: 2px; }
button:active { transform: translateY(1px); }
a:visited { color: #6a3d9a; }
```

`:focus-visible` matches when the browser decides a focus indicator should be shown —
keyboard focus yes, mouse click on a button usually not. Use it for focus styles instead
of `:focus`, and never remove the outline without replacing it.

**Form state:** `:checked`, `:disabled`, `:required`, `:invalid`, `:user-invalid`,
`:placeholder-shown`.

**Tree position:**

| Pseudo-class | Matches |
|--------------|---------|
| `:first-child`, `:last-child`, `:only-child` | position among all siblings |
| `:nth-child(2)`, `:nth-child(odd)`, `:nth-child(3n + 1)` | by formula |
| `:first-of-type`, `:nth-of-type(2)` | position among siblings **of the same element type** |
| `:not(.done)` | elements not matching the argument |
| `:empty` | no children, not even text |

## Pseudo-elements: parts of an element

Pseudo-elements use two colons and style something that isn't a separate element in the
DOM:

```css
p::first-line { font-variant: small-caps; }
p::first-letter { font-size: 2em; float: left; }
::selection { background: #ffd8a8; }
input::placeholder { color: #666; }
li::marker { color: #b3401a; }

.required::after {
  content: " *";
  color: #b00020;
}
```

`::before` and `::after` create a generated box as the first/last child of the element;
they only render if `content` is set (even to `""`). Generated text isn't reliably
announced by all assistive tech and can't be selected, so don't put essential information
there.

## Worked example: counting matches

We loaded this page in Chromium and counted `document.querySelectorAll(selector).length`
for each selector — the same matching engine CSS uses:

```html
<nav>
  <ul>
    <li><a href="/">Home</a></li>
    <li><a href="/docs/intro">Intro</a></li>
    <li><a href="https://github.com/x" target="_blank">GitHub</a></li>
    <li><a href="/guide.pdf">PDF</a></li>
  </ul>
</nav>
<article>
  <h2>T</h2>
  <p>first</p>
  <p>second</p>
  <div><p>nested</p></div>
  <p>third</p>
</article>
```

```text
nav a                     4
a[href^="http"]           1
a[href$=".pdf"]           1
a[target="_blank"]        1
article p                 4
article > p               3
h2 + p                    1
h2 ~ p                    3
article p:first-of-type   2
li:nth-child(odd)         2
p:not(:first-of-type)     2
li:last-child a           1
```

The interesting rows:

- `article p` is 4 but `article > p` is 3 — the "nested" paragraph is a grandchild.
- `h2 ~ p` is 3, not 4: sibling combinators only look at the **same parent**, and "nested"
  lives inside the `div`.
- `article p:first-of-type` is **2**, which surprises most people. It matches every `p`
  inside the article that is the first `p` *among its own siblings*: "first" (first `p`
  child of `article`) **and** "nested" (first `p` child of the `div`). The pseudo-class
  describes the element's position in its parent, not in the selector.
- `p:not(:first-of-type)` is 2: "second" and "third".

When a selector matches more or fewer elements than expected, pasting it into
`document.querySelectorAll()` in the console is the fastest way to debug.

## How It Actually Works

Browsers match selectors **right to left**. For `nav ul li a`, the engine doesn't find
every `nav` and search downward. It takes each element that could be styled, checks the
rightmost part (the "key selector", here `a`) first, and only then walks *up* the
ancestors looking for `li`, `ul` and `nav`. Most elements fail on the key selector
immediately, which makes this fast.

Engines add more shortcuts on top: rules are bucketed by their key selector's id, class
or tag, so an `<a>` element only gets checked against rules whose key is `a`, a class it
has, or `*`. And a **Bloom filter** of ancestor ids/classes/tags lets the engine reject
`nav ul li a` for an element with no `nav` ancestor without walking the tree.

This is why old advice about "expensive selectors" is mostly obsolete for normal
stylesheets — matching is rarely the bottleneck. What *can* cost more is selectors that
force re-matching many elements when something changes, like `:has()` on a large subtree
(Level 3 · 07) — and even that is heavily optimised in current engines.

Pseudo-elements don't exist in the DOM; they exist in the **box tree** that layout builds
from the DOM. That's why you can't select them with `querySelector` or attach event
listeners to them.

## Common mistakes

- **Space vs no space**: `p .note` (a `.note` inside a `p`) vs `p.note` (a `p` with class
  `note`).
- **Expecting `:first-of-type` to mean "the first one this selector finds."**
- **Forgetting `content` on `::before`/`::after`**, so nothing appears.
- **Using ids for styling** — then struggling to override them (lesson 08).
- **Removing focus outlines** (`outline: none`) with no replacement.
- **Deep chains** like `.header .nav ul li a span`, which break when markup changes. Target
  a class on the element you mean.

## Exercise

1. Style your recipe page: body font and max-width; headings in one colour; `h2 + p` with
   a slightly larger font; ingredient list markers coloured with `li::marker`.
2. Make PDF links show "(PDF)" and external links show "↗" using attribute selectors and
   `::after`.
3. Style method steps so every other step has a light background with `:nth-child`.
4. Add visible `:focus-visible` styles for links and buttons. Tab through the page to check
   them.
5. Predict, then check in the console, how many elements `li:first-of-type` matches on
   your page. Explain the number.
