---
description: "Level 4 of HTML & CSS: performance, CSS architecture, design tokens, i18n, SEO metadata, print, browser support and automated testing."
---

# Level 4 · Master <span class="level-badge">Production</span>

Goal: ship and maintain HTML and CSS at a professional level — pages that load fast and
stay smooth, stylesheets that many people can change for years without fear, sites that
work in any language and on paper, metadata that search and social platforms read
correctly, and an automated pipeline that catches problems before users do.

Levels 1–3 were about *writing* correct, accessible HTML and CSS. Level 4 is about
everything around it:

1. **Measure before optimising.** Rendering and loading costs are observable — layout
   counts, first paint, layout shift. Every performance claim in this level comes with a
   measurement.
2. **Structure beats cleverness.** Layers, tokens, naming conventions and specificity
   budgets keep a stylesheet predictable as it grows.
3. **Automate what can be automated, and know what can't.** Validators, linters, axe and
   browser tests catch regressions cheaply; judgement about meaning, clarity and usability
   stays with people.

## Modules

1. [The Rendering Pipeline & CSS Performance](01-rendering-performance.md) — style/layout/paint/composite, layout thrashing measured, containment, `content-visibility`
2. [Loading Performance](02-loading-performance.md) — render-blocking CSS measured, critical CSS, preload, fonts, layout shift
3. [CSS Architecture at Scale](03-css-architecture.md) — BEM, utilities, layers as architecture, specificity budgets, Stylelint, `@scope`
4. [Design Tokens & Theming Systems](04-design-tokens.md) — three token tiers, scales, themes, the early-resolution bug
5. [Logical Properties, Writing Modes & Internationalization](05-logical-properties-i18n.md) — LTR/RTL measured, vertical text, bidi, other scripts
6. [Metadata, SEO & Structured Data](06-metadata-seo.md) — titles, canonical, robots, Open Graph, JSON-LD, sitemaps
7. [Print Styles & Other Media](07-print-styles.md) — `@media print`, `@page`, breaks, forced colours
8. [Progressive Enhancement & Browser Support](08-progressive-enhancement.md) — fallbacks, `@supports`, Baseline, prefixes
9. [Testing & Auditing HTML and CSS](09-testing-auditing.md) — html-validate, Stylelint, axe, Playwright, visual regression, CI
10. [Capstone — A Conference Website](10-capstone.md) — everything in one page, with a full quality pipeline

## Before you start

Complete [Level 3](../level-3/index.md). Several lessons use Node.js tools
(html-validate, Stylelint, Playwright); install a current LTS version of Node.js. The
performance lessons use Chrome or Edge devtools, whose Performance panel exposes the
rendering details discussed.

## How the examples were checked

Performance numbers were measured in headless Chromium 153 on the laptop this course was
written on, using the DevTools protocol's performance metrics and the browser's own
`performance` entries; absolute times will differ on your machine, so the lessons focus
on ratios and counts. Tool output comes from html-validate 11.16, Stylelint 17.15,
axe-core 4.13 and Playwright 1.63 (Chromium 153 and WebKit 26.6).

!!! note "What we couldn't run"
    Search-engine indexing, rich-result eligibility and social-platform previews depend on
    external services, so lesson 06 describes documented behaviour rather than showing
    output. The CI workflow in lesson 09 was run locally step by step, not on GitHub's
    runners. As elsewhere in the course, Firefox couldn't be launched in our test
    environment.
