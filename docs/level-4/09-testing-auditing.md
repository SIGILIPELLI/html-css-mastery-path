---
description: "Automate HTML and CSS checks: html-validate, Stylelint, axe-core, Playwright keyboard and visual regression tests, and a CI workflow."
---

# 09 · Testing & Auditing HTML and CSS

HTML and CSS fail silently. Browsers repair invalid markup, ignore invalid declarations,
and render inaccessible pages without complaint. A typo in a property name, a missing
label, a heading level skipped in a refactor, a stylesheet change that shifts a layout on
a page nobody looked at — none of these produce an error. The only way to catch them
reliably is to **check automatically, every time the code changes**. This lesson builds
that toolchain from tools used throughout the course, with their real output.

## The layers of checking

| Layer | Tool | Catches | Speed |
|-------|------|---------|-------|
| Markup validity and conventions | html-validate (or the W3C Nu checker) | Invalid nesting, missing attributes, duplicate ids, implicit button types | Instant |
| CSS correctness and conventions | Stylelint | Unknown properties, duplicates, specificity rules, old syntax | Instant |
| Automated accessibility | axe-core (in the browser or in tests) | Missing names, contrast, ARIA misuse, landmark and heading problems | Seconds |
| Behaviour | Playwright (or similar) tests | Keyboard access, focus management, layout at sizes, forms | Seconds |
| Visual regression | Screenshot comparison | Unintended visual changes anywhere on a page | Seconds |
| Manual review | You, a screen reader, a keyboard | Everything automation can't judge: meaningful alt text, sensible reading order, clarity | Minutes |

Run the fast ones in the editor and on every commit, the slower ones in continuous
integration (CI), and the manual review before releases and for new components.

## html-validate

```bash
npm install --save-dev html-validate
npx html-validate "src/**/*.html"
```

With a config file:

```json title=".htmlvalidate.json"
{
  "extends": ["html-validate:recommended"],
  "rules": { "doctype-style": "off" }
}
```

Throughout this course it found real problems in our reference projects:

```text
  1:1  error  DOCTYPE should be uppercase  doctype-style                  (Level 1 · 02 — a style rule we then disabled)
  2:2  error  <html> is missing required "lang" attribute  element-required-attributes
 83:37 error  Prefer to use the native <section> element  prefer-native-element   (Level 1 · 10)
 74:16 error  <button> is missing recommended "type" attribute  no-implicit-button-type (Level 3 · 10)
```

Validators check the **source file**. For pages built by JavaScript, validate the
rendered DOM instead (`page.content()` in Playwright, passed to html-validate's API).

## Stylelint

Lesson 03 set this up. With `stylelint-config-standard` plus specificity rules, one short
sample file produced six errors, including a misspelled property that every browser
would have silently ignored:

```text
   4:3   ✖  Unknown property "colr"                                          property-no-unknown
  13:1   ✖  Too many ID selectors in "#header .nav a:hover", maximum 0       selector-max-id
```

Stylelint exits with a non-zero code when it finds errors (it exited with 2 in our run),
which is what makes it fail a CI job.

## Automated accessibility tests with axe-core

axe-core is the engine inside the axe DevTools extension, Lighthouse's accessibility
audit and many other tools. In a test suite it runs against the **rendered** page — so it
sees computed colours for contrast, and DOM created by JavaScript.

We wrote a Playwright test suite for the Level 1 recipe project using
`@playwright/test` 1.63 and `@axe-core/playwright` 4.13:

```js title="playwright.config.js"
import { defineConfig } from '@playwright/test';

export default defineConfig({
  testDir: 'tests',
  use: { baseURL: 'http://localhost:4173' },
  webServer: {
    command: 'python3 -m http.server 4173 --directory recipe',
    port: 4173,
    reuseExistingServer: true,
  },
  projects: [
    { name: 'desktop', use: { viewport: { width: 1280, height: 800 } } },
    { name: 'phone', use: { viewport: { width: 360, height: 780 } } },
  ],
});
```

```js title="tests/recipe.spec.js"
import { test, expect } from '@playwright/test';
import AxeBuilder from '@axe-core/playwright';

test('has no detectable WCAG A/AA violations', async ({ page }) => {
  await page.goto('/');
  const results = await new AxeBuilder({ page })
    .withTags(['wcag2a', 'wcag2aa', 'wcag21aa', 'wcag22aa'])
    .analyze();
  expect(results.violations).toEqual([]);
});

test('skip link is first and moves focus into main', async ({ page }) => {
  await page.goto('/');
  await page.keyboard.press('Tab');
  await expect(page.getByRole('link', { name: 'Skip to recipe' })).toBeFocused();
  await page.keyboard.press('Enter');
  await page.keyboard.press('Tab');
  const inMain = await page.evaluate(() => document.querySelector('main').contains(document.activeElement));
  expect(inMain).toBe(true);
});

test('page structure is exposed to assistive technology', async ({ page }) => {
  await page.goto('/');
  await expect(page.getByRole('heading', { level: 1 })).toHaveText('Roasted Tomato Soup');
  await expect(page.getByRole('navigation', { name: 'Main' })).toBeVisible();
  await expect(page.getByRole('table', { name: 'Approximate nutrition per serving' })).toBeVisible();
});

test('no horizontal scrolling', async ({ page }) => {
  await page.goto('/');
  const [scrollWidth, innerWidth] = await page.evaluate(
    () => [document.documentElement.scrollWidth, window.innerWidth]);
  expect(scrollWidth).toBeLessThanOrEqual(innerWidth);
});

test('visual snapshot', async ({ page }) => {
  await page.goto('/');
  await expect(page).toHaveScreenshot('recipe.png', { fullPage: true });
});
```

Notice the locators: `getByRole('heading', { level: 1 })`,
`getByRole('navigation', { name: 'Main' })`. Querying by role and accessible name tests the
page the way assistive technology sees it — if someone replaces the `<nav>` with a `<div>`
or removes the table caption, the test fails. That's a free accessibility regression test
on top of whatever the test was really about.

Each test runs in both projects (desktop and phone), so five tests make ten runs.

## Visual regression testing

`toHaveScreenshot()` compares a screenshot with a stored **baseline** image. The first
run has nothing to compare against, and says so:

```text
Error: A snapshot doesn't exist at …/tests/recipe.spec.js-snapshots/recipe-desktop-darwin.png, writing actual.
```

It wrote the baseline and failed deliberately, so a missing baseline can't pass
unnoticed. Once baselines existed for both projects, the full suite passed:

```text
10 passed (1.7s)
```

Then we made a change that's easy to miss in review — `h2` top margin from `2rem` to
`2.25rem`, a 4px difference — and ran the visual test again:

```text
Error: expect(page).toHaveScreenshot(expected) failed
  Expected an image 1280px by 2109px, received 1280px by 2121px. 30847 pixels (ratio 0.02 of all image pixels) are different.
```

Three `h2`s × 4px = 12px taller page, and every pixel below the first heading moved.
Playwright saves expected, actual and a highlighted diff image in `test-results/`.
If the change is intended, you update the baselines with
`npx playwright test --update-snapshots` and commit the new images, so reviewers see the
visual change in the pull request.

Things to know about visual tests:

- **Baselines are platform-specific** — note `darwin` in the file name. Fonts render
  differently on macOS, Linux and Windows, so generate baselines in the same environment
  CI uses (commonly Playwright's Docker image).
- **Dynamic content breaks them** — dates, ads, animations, random images. Mask regions
  (`mask: [locator]`), freeze time, and disable animations (`animations: 'disabled'` is
  the default for `toHaveScreenshot`).
- **Thresholds** (`maxDiffPixels`, `maxDiffPixelRatio`) tolerate tiny anti-aliasing noise.

## Lighthouse

Lighthouse (Chrome devtools → Lighthouse, or the `lighthouse` CLI) produces scored audits
for performance, accessibility, best practices and SEO in one report. It's a good
periodic health check and a useful CI gate via Lighthouse CI, with two caveats: its
performance numbers come from a single simulated load and vary between runs, and a 100
accessibility score means only that its automated rules passed.

## What automation can't check

Automated accessibility rules catch problems that have a mechanical definition: a missing
name, insufficient contrast, an invalid ARIA attribute. They can't judge:

- whether alt text is **accurate** and useful,
- whether headings **describe** their sections,
- whether the focus order **makes sense**,
- whether error messages are **helpful**,
- whether a custom widget is **usable** with a screen reader,
- whether content **reflows** sensibly at 400% zoom.

Keep a short manual checklist for every new page or component:

1. Keyboard only: reach and operate everything; focus always visible; no traps.
2. Screen reader: headings list, landmarks list, form labels, dynamic announcements.
3. Zoom to 200% and 400% (or a 320px-wide window): no loss of content or function.
4. Dark mode, forced colours, reduced motion.
5. Read the alt text and link text out of context.

## Worked example: a CI workflow

A GitHub Actions workflow that runs everything above on each pull request:

```yaml title=".github/workflows/quality.yml"
name: quality
on: [pull_request]

jobs:
  checks:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 22
      - run: npm ci
      - run: npx html-validate "recipe/**/*.html"
      - run: npx stylelint "recipe/**/*.css"
      - run: npx playwright install --with-deps chromium
      - run: npx playwright test
      - uses: actions/upload-artifact@v4
        if: failure()
        with:
          name: playwright-report
          path: |
            playwright-report/
            test-results/
```

(Use the current major versions of the actions when you set this up. The baselines for
this job must be generated on Linux, as noted above.) The
[GitHub Mastery Path](https://sigilipelli.github.io/github-mastery-path/) covers Actions in
depth, and [Playwright Mastery Path](https://sigilipelli.github.io/playwright-mastery-path/)
covers the test runner.

We ran the validation and test commands locally on macOS; we did not run this workflow on
GitHub's runners while writing the lesson.

## How It Actually Works

Each tool inspects a different representation of the page:

- **html-validate** parses the HTML **source** with its own spec-based parser and applies
  rules to that tree — fast, but blind to scripts and CSS.
- **Stylelint** parses CSS into an abstract syntax tree (with PostCSS) and applies rules to
  selectors, properties and values — it knows nothing about which elements exist.
- **axe-core** runs **inside the browser** against the live DOM, computed styles and the
  browser's layout, which is why it can evaluate contrast against the actual background,
  check that an `aria-labelledby` target exists, and see content added by JavaScript. Its
  rules are designed to avoid false positives, so it reports "incomplete" items it can't
  decide rather than guessing.
- **Playwright** drives a real browser through the Chrome DevTools Protocol (or its
  equivalents for Firefox and WebKit). Role-based locators query the browser's
  accessibility tree, and `toHaveScreenshot` captures rendered pixels and compares them
  with a pixel-matching algorithm that tolerates anti-aliasing differences.

Because they look at different layers, they overlap very little — which is exactly why a
quality pipeline uses all of them.

## Common mistakes

- **Validating once** instead of on every change.
- **Treating a clean axe run as "accessible."**
- **Selecting elements in tests by CSS class** instead of role and name, missing
  accessibility regressions.
- **Visual baselines generated on a developer's Mac** and compared on Linux CI.
- **Snapshotting pages with dynamic content** without masking.
- **Ignoring failures by updating baselines** without looking at the diff.

## Exercise

1. Add html-validate and Stylelint to one of your projects with `npm` scripts
   (`"lint:html"`, `"lint:css"`), and fix everything they report.
2. Write a Playwright suite for your Level 3 component gallery: an axe test, a keyboard
   test per component, and a visual snapshot at two viewport sizes.
3. Make a deliberate one-line CSS change and read the diff image Playwright produces.
4. Put the commands in a CI workflow and open a pull request that breaks one of them.
5. Do the five-point manual checklist on one page and write down what automation missed.
