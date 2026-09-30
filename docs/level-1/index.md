---
description: "Level 1 of HTML & CSS: how browsers render, document structure, semantic text, images, tables, forms, selectors, the cascade and the box model."
---

# Level 1 · Entry <span class="level-badge">Foundations</span>

Goal: write a complete, valid, accessible web page from an empty file, and be able to
explain what the browser does with every line — why broken markup still renders, why a
style isn't applying, why a box is wider than its `width`, why a phone shows your page
zoomed out.

Three ideas carry this level:

1. **HTML describes meaning; CSS describes presentation.** Choose elements for what
   content *is*. The browser turns those choices into a DOM, an accessibility tree and
   default styles — you get structure for search engines and assistive technology for
   free.
2. **The DOM is the truth, not your source file.** Parsers repair, insert and move
   elements. Devtools shows what actually exists.
3. **CSS is resolved, not executed.** For every element and every property, the cascade
   picks exactly one winning value by origin, specificity and order. Layout then turns
   those values into boxes.

## Modules

1. [How Browsers Turn HTML & CSS into Pixels](01-how-browsers-render.md) — your first page, devtools, parser error recovery, the rendering pipeline
2. [Document Structure & the Head Element](02-document-structure.md) — doctype, `lang`, charset, the viewport tag, title, links and scripts, validation
3. [Text, Links & Semantic Meaning](03-text-links-semantics.md) — headings, lists, text-level elements, URL resolution, links vs buttons
4. [Images & Media](04-images-media.md) — alt text, layout shift, `srcset`/`sizes`, `<picture>`, lazy loading, video with captions
5. [Tables](05-tables.md) — accessible data tables, spanning cells, grouped headers, scrolling wide tables
6. [Forms & Native Validation](06-forms.md) — labels, input types, what gets submitted, constraint validation
7. [CSS Syntax & Selectors](07-css-selectors.md) — rules, combinators, pseudo-classes, pseudo-elements, how matching works
8. [The Cascade, Specificity & Inheritance](08-cascade-specificity.md) — who wins and why, and the global keywords
9. [The Box Model, Display & Units](09-box-model-units.md) — `box-sizing`, block vs inline, margin collapsing, `rem`/`em`/`ch`/`dvh`
10. [Project — A Recipe Page](10-project-recipe-page.md) — a validated, audited, keyboard-tested page

## What you need before starting

- **A modern browser** — Chrome, Edge, Firefox or Safari, current version.
- **A text editor.** VS Code is free and has good HTML/CSS support; any editor that saves
  plain UTF-8 text works.
- **Optionally, Node.js**, to run the command-line validator (`npx html-validate`). You
  can use the W3C checker in the browser instead.

No prior coding experience is assumed. If you already know some HTML, skim lessons 01–03
but read the **How It Actually Works** sections — they cover the parts most tutorials
skip.

## How the examples were checked

Code blocks are titled with a file name when they belong to a file you're building.
Blocks labelled `text` are **output we actually observed** while writing this course,
from headless **Chromium 153** and **WebKit 26.6** (Safari's engine) driven by
Playwright 1.63, html-validate 11.16 and axe-core 4.13. We queried the live page —
computed styles, box sizes, the accessibility tree, `FormData` — rather than describing
what "should" happen. Where our first attempt failed a check, the lesson shows that too.

!!! note "About browser differences"
    Everything in Level 1 behaves the same in all current major browsers. Default
    styles (form control fonts, validation message wording) differ slightly between
    browsers, and the lessons point out where. We weren't able to run Firefox in our
    test environment, so Firefox-specific output isn't shown anywhere in this course.
