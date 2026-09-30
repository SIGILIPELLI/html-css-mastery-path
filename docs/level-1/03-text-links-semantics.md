---
description: "Mark up text by meaning: headings, paragraphs, lists, emphasis, quotes, code and time, plus how relative, absolute and fragment links resolve."
---

# 03 · Text, Links & Semantic Meaning

Most of the web is text, and most of HTML is a vocabulary for saying what each piece of
text *is*. A heading, a step in a procedure, a term being defined, a warning, a date, a
line of code. Getting this vocabulary right is what people mean by **semantic HTML**,
and it pays off in three places at once: assistive technology can navigate the page,
search engines can understand it, and your CSS gets simpler because the structure is
already in the markup.

## Why semantics matter — measured

We rendered two versions of the same content in Chromium and asked Playwright for the
**accessibility tree** — the structure that screen readers and other assistive tools
actually consume. First, the version built from generic `<div>`s and `<span>`s:

```html
<div class="h">Ingredients</div>
<div>• 2 tomatoes</div>
<div>• 1 onion</div>
<span onclick="...">Read more</span>
```

```text
- text: Ingredients • 2 tomatoes • 1 onion Read more
```

One undifferentiated run of text. There is no heading to jump to, no list ("list, 2
items"), and "Read more" is not a link — it can't be reached with the Tab key at all.
Now the semantic version:

```html
<h2>Ingredients</h2>
<ul>
  <li>2 tomatoes</li>
  <li>1 onion</li>
</ul>
<a href="/soup#method">Read the method</a>
```

```text
- heading "Ingredients" [level=2]
- list:
  - listitem: 2 tomatoes
  - listitem: 1 onion
- link "Read the method":
  - /url: /soup#method
```

Same pixels (after CSS), completely different document. Screen-reader users commonly
navigate by pulling up a list of headings or links; the first version gives them nothing
to pull up.

## Headings

`<h1>` to `<h6>` form an outline of the page:

```html
<h1>Tomato Soup</h1>
  <h2>Ingredients</h2>
  <h2>Method</h2>
    <h3>Roasting the tomatoes</h3>
    <h3>Blending</h3>
  <h2>Storage</h2>
```

(Indentation here is only for readability — HTML ignores it.)

Rules that hold up in practice:

- **One `<h1>` per page**, describing the page's main content.
- **Don't skip levels going down** (`h2` → `h4`). Going back up any number of levels is
  fine.
- **Choose the level by structure, never by size.** If an `<h2>` should look small, style
  it small.
- **Don't use headings for things that aren't section titles** — a large promo sentence is
  a paragraph with a class.

## Paragraphs, line breaks and thematic breaks

`<p>` is a paragraph. Whitespace in HTML collapses: any run of spaces, tabs and newlines
becomes one space, so blank lines in your source don't create gaps. Spacing is CSS's job.

`<br>` is a line break *within* content where the break is part of the meaning —
addresses and poems:

```html
<p>
  Mastery Kitchen<br>
  12 Market Street<br>
  Hyderabad
</p>
```

Don't use `<br><br>` to separate paragraphs. `<hr>` is a *thematic* break — a change of
topic within a section — and is drawn as a line by default.

## Lists

```html
<ul>                         <!-- order doesn't matter -->
  <li>Olive oil</li>
  <li>Garlic</li>
</ul>

<ol>                         <!-- order matters -->
  <li>Roast the tomatoes.</li>
  <li>Blend until smooth.</li>
</ol>

<dl>                         <!-- name–value groups -->
  <dt>Prep time</dt>
  <dd>10 minutes</dd>
  <dt>Cook time</dt>
  <dd>20 minutes</dd>
</dl>
```

The only valid children of `<ul>` and `<ol>` are `<li>` (plus `<script>` and
`<template>`). Lists nest by putting a new list *inside* an `<li>`:

```html
<ul>
  <li>Vegetables
    <ul>
      <li>Tomatoes</li>
      <li>Onion</li>
    </ul>
  </li>
</ul>
```

`<ol>` has useful attributes: `start="5"`, `reversed`, and `type="a"` / `"i"` when the
numbering style is part of the meaning (for example, when legal text refers to "clause
(b)"). Navigation menus are lists of links — `<ul>` inside `<nav>` is the standard
pattern.

## Text-level semantics

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

The distinction between `<em>` and `<i>`, or `<strong>` and `<b>`, is about meaning:

```html
<p>You <em>must</em> let the soup cool before blending.</p>
<p><strong>Warning:</strong> hot liquid expands in a blender.</p>
<p>The Italians call this <i lang="it">passata</i>.</p>
<p>Press <kbd>Ctrl</kbd>+<kbd>S</kbd> to save.</p>
<p>Keeps for <time datetime="P3D">three days</time>, published
   <time datetime="2026-09-30">30 September 2026</time>.</p>
```

The `datetime` value uses standard formats: dates (`2026-09-30`), times (`18:30`),
date-times (`2026-09-30T18:30+05:30`) and durations (`P3D`, `PT20M`).

### Quotations and preformatted text

```html
<figure>
  <blockquote cite="https://example.com/interview">
    <p>Cook the tomatoes until they give up.</p>
  </blockquote>
  <figcaption>— A. Chef, <cite>Kitchen Interviews</cite></figcaption>
</figure>

<pre><code>for step in steps:
    print(step)</code></pre>
```

`<pre>` preserves whitespace and line breaks exactly — the one place where your source
spacing matters. Put `<code>` inside it to say the content is code.

## Links

```html
<a href="https://developer.mozilla.org/">MDN Web Docs</a>
```

Link text should make sense **on its own**, because people scanning links (by eye or by
screen reader) see it out of context. "Read the method" is good; "click here" and
"more" are not.

### How URLs resolve

A link's `href` is resolved against the document's base URL. We set a base of
`https://example.com/recipes/soups/tomato.html` and read each link's resolved `.href`
in Chromium:

```text
basil.html               -> https://example.com/recipes/soups/basil.html
../breads/naan.html      -> https://example.com/recipes/breads/naan.html
/about                   -> https://example.com/about
#method                  -> https://example.com/recipes/soups/tomato.html#method
?print=1                 -> https://example.com/recipes/soups/tomato.html?print=1
//cdn.example.org/x.css  -> https://cdn.example.org/x.css
https://other.org        -> https://other.org/
```

- **Relative** (`basil.html`, `../breads/naan.html`) — relative to the current *folder*.
  Great for sites you move around, but `/about` vs `about` is a common source of broken
  links when pages live at different depths.
- **Root-relative** (`/about`) — from the domain root. Careful on GitHub Pages project
  sites, where your site lives under `/repo-name/`, not `/`.
- **Fragment** (`#method`) — jumps to the element with `id="method"` on the same page.
- **Protocol-relative** (`//cdn...`) — inherits `http:`/`https:`. Just write `https://`.

### Other link types

```html
<a href="#method">Skip to the method</a>                      <!-- same page -->
<a href="mailto:hello@example.com">Email us</a>
<a href="tel:+911234567890">Call us</a>
<a href="/menu.pdf" download>Download the menu (PDF, 240 KB)</a>
<a href="https://other.org/" target="_blank" rel="noopener">Other site (opens in a new tab)</a>
```

About `target="_blank"`: opening new tabs takes control away from the user, so do it
sparingly and say so in the link text. Modern browsers treat `target="_blank"` as
`rel="noopener"` by default (the new page can't reach back through `window.opener`), but
writing `rel="noopener"` explicitly costs nothing.

### Links vs buttons

A **link goes somewhere** (it has an `href`); a **button does something** (submits,
opens, toggles). Using `<a href="#" onclick="...">` for actions, or a `<div>` with a
click handler for either, breaks keyboard access, the right-click menu, and what assistive
tech announces. Level 3 · 03 returns to this.

## How It Actually Works

Every element has an implicit **ARIA role** defined by the HTML-AAM specification: `h2`
maps to role `heading` with level 2, `ul` to `list`, `li` to `listitem`, `a[href]` to
`link`, `nav` to `navigation`. The browser builds the accessibility tree from the DOM
using these mappings, then computes an **accessible name** for each node (for a link, its
text content; for an image, its `alt`). Operating-system accessibility APIs expose that
tree to screen readers, voice control and switch devices.

Note the `a[href]` detail: an `<a>` *without* `href` has no link role and isn't
focusable — it's a placeholder. That's why `<span onclick>` in the first example
vanished from the tree: nothing about it said "interactive."

Default styles — italic `em`, bold `strong`, bullets on `ul`, blue underlined links —
come from the browser's **user-agent stylesheet**, a real CSS file you can see in
devtools labelled "user agent stylesheet." They're just defaults. Semantics live in the
element choice; the look is always overridable.

## Common mistakes

- **Headings chosen for size**, or bold paragraphs pretending to be headings.
- **Fake lists** made with line breaks and bullet characters.
- **Vague link text** — "click here", "read more", bare URLs.
- **Using `<a>` without `href`** or `href="#"` for actions. Use `<button>`.
- **`<br>` for spacing.** Use margins.
- **Root-relative links on a site served from a subfolder** (common on GitHub Pages).
- **`<b>`/`<i>` when you mean importance or stress** — or `<strong>` for everything bold.

## Exercise

1. Mark up a recipe (real or invented) using: one `h1`, `h2`s for Ingredients, Method and
   Notes, a `ul` of ingredients, an `ol` of steps, a `dl` for prep/cook times, one `<time>`,
   one `<abbr>`, and one `<strong>` warning.
2. Add a "Jump to method" link at the top that targets the Method heading by `id`.
3. Open devtools → **Accessibility** pane (Chrome: Elements → Accessibility tab; enable
   "full-page accessibility tree"). Find your list and confirm it's announced with its
   item count.
4. Create two files in different folders (`index.html` and `recipes/soup.html`) and link
   them to each other with relative URLs. Then break it on purpose with a root-relative
   link and explain why it fails when opened via `file://`.
