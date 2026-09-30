---
description: "Level 1 project: build a complete, semantic, accessible recipe page with its own stylesheet, then validate it and audit it with axe-core."
---

# 10 · Project — A Recipe Page

Time to put Level 1 together. You'll build a single recipe page — the kind of page
the web is full of, and one that uses almost every element from this level: a document
skeleton, headings and sections, a figure, lists, a definition list, an in-page table of
contents, a data table, text-level semantics, and a stylesheet built from selectors,
the cascade, the box model and relative units.

The finished version below isn't a mock-up. We built it, ran it through a validator and
an automated accessibility checker, tabbed through it with a keyboard, and loaded it at
phone width — the results are further down.

## Requirements

Your page must:

1. Have a complete `<head>`: charset, viewport, a specific `<title>`, a meta description,
   and a linked stylesheet.
2. Use landmarks: a site `<header>` with a labelled `<nav>`, one `<main>`, a `<footer>`,
   and a skip link as the first focusable element.
3. Contain an `<article>` with one `<h1>`, an introduction, a byline with a `<time>`, and
   `<section>`s for ingredients, method and nutrition, each with an `<h2>`.
4. Include an image with real `alt` text, `width` and `height`, in a `<figure>` with a
   caption.
5. Use the right list for each job: `<ul>` for ingredients, `<ol>` for steps, `<dl>` for
   facts like servings and timings.
6. Include an accessible data table with a caption and `th`/`scope`, in a region that
   scrolls horizontally on small screens.
7. Style everything from one external stylesheet — no `style` attributes — using
   `rem` for type and spacing, `border-box` sizing, and visible `:focus-visible` styles.
8. Pass html-validate with no errors and axe-core with no violations.

Try building it yourself from the requirements first. Then compare with the reference.

## Reference solution: the HTML

```html title="index.html"
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Roasted Tomato Soup — Weeknight Kitchen</title>
    <meta name="description" content="A 45-minute roasted tomato soup for four, with pantry ingredients, step-by-step method, timings and storage notes.">
    <link rel="stylesheet" href="styles.css">
  </head>
  <body>
    <a class="skip-link" href="#main">Skip to recipe</a>

    <header class="site-header">
      <p class="site-name"><a href="index.html">Weeknight Kitchen</a></p>
      <nav aria-label="Main">
        <ul>
          <li><a href="index.html" aria-current="page">Recipes</a></li>
          <li><a href="about.html">About</a></li>
        </ul>
      </nav>
    </header>

    <main id="main">
      <article class="recipe">
        <header>
          <h1>Roasted Tomato Soup</h1>
          <p class="lede">Roasting concentrates the tomatoes' sweetness, so this soup tastes
            like it simmered all afternoon.</p>
          <p class="byline">By <span class="author">Priya Rao</span> ·
            <time datetime="2026-09-30">30 September 2026</time></p>
        </header>

        <figure>
          <img src="images/soup.png" width="1200" height="800"
               alt="A bowl of orange-red tomato soup with a swirl of cream and torn basil leaves">
          <figcaption>Finish with cream and basil just before serving.</figcaption>
        </figure>

        <dl class="facts">
          <div><dt>Serves</dt><dd>4</dd></div>
          <div><dt>Prep</dt><dd><time datetime="PT10M">10 min</time></dd></div>
          <div><dt>Cook</dt><dd><time datetime="PT35M">35 min</time></dd></div>
        </dl>

        <nav aria-label="On this page" class="toc">
          <ul>
            <li><a href="#ingredients">Ingredients</a></li>
            <li><a href="#method">Method</a></li>
            <li><a href="#nutrition">Nutrition</a></li>
          </ul>
        </nav>

        <section aria-labelledby="ingredients">
          <h2 id="ingredients">Ingredients</h2>
          <ul class="ingredients">
            <li>1 kg ripe tomatoes, halved</li>
            <li>1 onion, quartered</li>
            <li>4 garlic cloves, unpeeled</li>
            <li>3 tbsp olive oil</li>
            <li>500 ml vegetable stock</li>
            <li>A handful of basil leaves</li>
            <li>Salt and black pepper</li>
          </ul>
        </section>

        <section aria-labelledby="method">
          <h2 id="method">Method</h2>
          <ol class="steps">
            <li>Heat the oven to 200 °C. Spread the tomatoes (cut side up), onion and garlic
              on a tray, drizzle with oil and season.</li>
            <li>Roast for 30 minutes, until the tomato edges are blackened.</li>
            <li>Squeeze the garlic out of its skins. Tip everything, including the tray
              juices, into a pan with the stock and simmer for 5 minutes.</li>
            <li>Let it cool for 5 minutes, then blend until smooth.</li>
            <li>Taste, adjust the seasoning and serve with basil.</li>
          </ol>
          <p class="warning"><strong>Careful:</strong> hot liquid expands when blended. Fill
            the blender no more than half full and hold the lid down with a towel.</p>
        </section>

        <section aria-labelledby="nutrition">
          <h2 id="nutrition">Nutrition</h2>
          <section class="table-scroll" aria-labelledby="nutrition-caption" tabindex="0">
            <table>
              <caption id="nutrition-caption">Approximate nutrition per serving</caption>
              <thead>
                <tr><th scope="col">Nutrient</th><th scope="col" class="num">Amount</th></tr>
              </thead>
              <tbody>
                <tr><th scope="row">Energy</th><td class="num">180 kcal</td></tr>
                <tr><th scope="row">Carbohydrate</th><td class="num">16 g</td></tr>
                <tr><th scope="row">Fat</th><td class="num">11 g</td></tr>
                <tr><th scope="row">Protein</th><td class="num">4 g</td></tr>
              </tbody>
            </table>
          </section>
          <p class="note"><small>Illustrative figures for this exercise, not a nutritional analysis.</small></p>
        </section>
      </article>
    </main>

    <footer class="site-footer">
      <p><small>© 2026 Weeknight Kitchen. Recipe text licensed CC BY 4.0.</small></p>
    </footer>
  </body>
</html>
```

Decisions worth noticing:

- **Two `<nav>`s, each labelled** (`aria-label="Main"` and `"On this page"`). When a page
  has more than one navigation landmark, labels are how screen-reader users tell them
  apart.
- **`aria-current="page"`** on the link to the current page. It's announced as "current
  page", and the stylesheet uses it as a selector — the markup states the fact once and
  both the accessibility tree and the styling read it.
- **The `<dl>` wraps each term/definition pair in a `<div>`.** HTML allows exactly one
  level of `div` wrapping inside `dl`, specifically so pairs can be styled as units.
- **Each `<section>` points to its heading** with `aria-labelledby`, turning it into a
  named region.
- **The nutrition figures are placeholders for the exercise** and the page says so. Don't
  publish numbers you haven't actually worked out.

## Reference solution: the CSS

```css title="styles.css"
/* 1. Reset the few defaults that fight us */
*, *::before, *::after { box-sizing: border-box; }

img { max-width: 100%; height: auto; display: block; }

button, input, select, textarea { font: inherit; }

/* 2. Base typography */
body {
  margin: 0;
  font-family: system-ui, -apple-system, "Segoe UI", Roboto, sans-serif;
  line-height: 1.6;
  color: #222;
  background: #fffdf9;
}

h1, h2 { line-height: 1.2; color: #8a2f12; }
h1 { font-size: 2.25rem; margin-block: 0 0.5rem; }
h2 { font-size: 1.5rem; margin-block: 2rem 0.75rem; }

a { color: #8a2f12; }
a:hover { text-decoration-thickness: 2px; }
a:focus-visible { outline: 3px solid #1a5fb4; outline-offset: 2px; }

/* 3. Page frame */
.site-header,
main,
.site-footer {
  max-width: 44rem;
  margin-inline: auto;
  padding-inline: 1rem;
}

.site-header { padding-block: 1rem; border-bottom: 1px solid #eadfd3; }
.site-name { margin: 0; font-weight: 700; }
.site-name a { text-decoration: none; }

.site-header ul,
.toc ul {
  list-style: none;
  padding: 0;
  margin: 0.5rem 0 0;
}

.site-header li,
.toc li {
  display: inline-block;
  margin-right: 1rem;
}

[aria-current="page"] { font-weight: 700; }

.skip-link {
  position: absolute;
  left: -999px;
}
.skip-link:focus {
  left: 1rem;
  top: 1rem;
  background: #fff;
  padding: 0.5rem 1rem;
  z-index: 1;
}

/* 4. Recipe content */
.lede { font-size: 1.2rem; margin-top: 0; }
.byline { color: #555; font-size: 0.9rem; }

figure { margin: 1.5rem 0; }
figcaption { font-size: 0.9rem; color: #555; margin-top: 0.5rem; }

.facts {
  margin: 1.5rem 0;
  padding: 1rem;
  border: 1px solid #eadfd3;
  border-radius: 0.5rem;
}
.facts div { display: inline-block; margin-right: 2rem; }
.facts dt { font-size: 0.8rem; text-transform: uppercase; letter-spacing: 0.05em; color: #555; }
.facts dd { margin: 0; font-size: 1.25rem; font-weight: 600; }

.ingredients li::marker { color: #8a2f12; }

.steps li { padding: 0.25rem 0.5rem; }
.steps li:nth-child(even) { background: #f7efe6; }
.steps li + li { margin-top: 0.5rem; }

.warning {
  border-left: 4px solid #8a2f12;
  background: #fbeee6;
  padding: 0.75rem 1rem;
}

.table-scroll { overflow-x: auto; }
table { border-collapse: collapse; width: 100%; font-variant-numeric: tabular-nums; }
caption { text-align: start; font-weight: 600; padding-block: 0.5rem; }
th, td { text-align: start; padding: 0.5rem 0.75rem; border-bottom: 1px solid #eadfd3; }
.num { text-align: end; }

.site-footer { margin-top: 3rem; padding-block: 1.5rem; border-top: 1px solid #eadfd3; color: #555; }
```

The stylesheet is organised from general to specific: resets, base typography, the page
frame, then content components. That order matters because of the cascade — broad rules
come first and targeted ones later, so ties in specificity resolve the way you'd expect.
Almost every selector is a single class or element, so specificity stays low and any rule
is easy to override.

The skip link is moved off-screen rather than hidden with `display: none`, because
`display: none` would remove it from the tab order entirely. When it receives focus, it
moves back on screen.

## Checking the result

### Validation

We saved both files with `.htmlvalidate.json` from lesson 02 (the recommended preset with
the doctype-case rule turned off) and ran:

```bash
npx html-validate index.html
```

The first run produced one error:

```text
  83:37  error  Prefer to use the native <section> element  prefer-native-element
```

The scrolling table wrapper was a `<div role="region">`. A `<section>` with an accessible
name has the region role natively, so we switched to that; the next run exited cleanly
with no output.

### Automated accessibility audit

We injected axe-core 4.13.0 into the page in Chromium and ran the WCAG 2.0/2.1/2.2 A and
AA rule sets plus axe's best-practice rules. It reported **0 violations** (42 rules
passed on the final version).

That's a necessary baseline, not proof of accessibility. Automated tools can't tell
whether your `alt` text is accurate, your headings make sense or your link text is
meaningful — only that they exist. Level 3 covers manual testing.

### Keyboard

Pressing Tab from the top of the page, the first three focus stops were:

```text
a "Skip to recipe" at (16,16) outline=solid
a "Weeknight Kitchen" at (304,19) outline=solid
a "Recipes" at (304,53) outline=solid
```

The skip link is first and appears in the top-left corner when focused; every stop has a
visible outline.

### Phone width

At a 360px viewport, `document.documentElement.scrollWidth` was 360 — no horizontal
page scroll. The image scales down (`max-width: 100%`), the text column is fluid, and
the table can scroll inside its own region if it ever needs to.

## How It Actually Works

Look at the page's accessibility tree (Playwright's `ariaSnapshot()` of `body`, abridged)
to see what all the semantic choices produced:

```text
- link "Skip to recipe"
- banner:
  - navigation "Main":
    - list: …
- main:
  - article:
    - heading "Roasted Tomato Soup" [level=1]
    - figure "Finish with cream and basil just before serving.":
      - img "A bowl of orange-red tomato soup with a swirl of cream and torn basil leaves"
    - term: Serves
    - definition: "4"
    - navigation "On this page": …
```

A few mappings to notice. The site `<header>` became a **banner** landmark because it's
a direct child of `body`; the `<header>` inside the article didn't, because a header
scoped to an article isn't the page banner. The `<figure>` took its accessible name from
its `<figcaption>`. The `<dl>` became **term**/**definition** pairs. None of this needed
a single ARIA role attribute — the elements carry the semantics, and the browser
translates them. That's the payoff of Level 1: when you pick the right elements, the
accessible structure comes for free.

## Common mistakes

- **Skipping the skip link**, or hiding it with `display: none` so it can never be
  focused.
- **Unlabelled duplicate `<nav>`s.**
- **An `<h1>` for the site name on every page *and* one for the page content.** Here the
  site name is a paragraph; the recipe title is the `<h1>`.
- **Styling with `style=""`** "just this once" — it immediately wins every cascade fight.
- **Trusting a clean automated audit as "accessible".**

## Exercise

1. Build your own recipe page (or any single-topic page: a book review, a travel guide,
   a product spec) meeting all eight requirements.
2. Validate it and fix every error. Keep a note of each message and what it taught you.
3. Run an automated checker — the axe DevTools browser extension, or Lighthouse's
   Accessibility category in Chrome devtools — and fix every violation.
4. Unplug your mouse (or don't touch it) and use the page with Tab, Shift+Tab and Enter
   only. Can you reach and see every link?
5. Add a second page (`about.html`) that shares the stylesheet and header, and move
   `aria-current="page"` to the right link on each page.
