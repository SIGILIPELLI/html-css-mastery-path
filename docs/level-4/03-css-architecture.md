---
description: "Organise CSS many people maintain for years: file structure, BEM, utility classes, cascade layers as architecture, specificity budgets, linting."
---

# 03 · CSS Architecture at Scale

A stylesheet for one page can be organised any way you like. A stylesheet that twenty
people change over five years, across hundreds of pages, needs rules — otherwise every
new feature adds more specific selectors to beat the old ones, nobody dares delete
anything, and the file only grows. **CSS architecture** is the set of conventions that
keep styles predictable: where a rule goes, what it's allowed to target, how it's named,
and how conflicts are resolved.

## The underlying problems

Every CSS methodology is an answer to the same four problems:

1. **Global scope.** Any rule can affect any element on any page. A `.title` for one
   component styles every other `.title` too.
2. **Specificity escalation.** Overrides need more specific selectors, which need even
   more specific overrides.
3. **Dead code.** It's hard to know whether a rule is still used, so nothing gets deleted.
4. **Coupling to markup structure.** `.sidebar ul li a` breaks when someone adds a `div`.

## Pattern 1: name components, not structure (BEM)

**BEM** (Block, Element, Modifier) makes every selector a single class with a
predictable name:

```html
<article class="recipe-card recipe-card--featured">
  <img class="recipe-card__image" src="…" alt="…">
  <h3 class="recipe-card__title">Roasted tomato soup</h3>
  <p class="recipe-card__meta">45 minutes</p>
</article>
```

```css
.recipe-card { … }                  /* block */
.recipe-card__title { … }           /* element: a part of the block */
.recipe-card--featured { … }        /* modifier: a variant of the block */
.recipe-card--featured .recipe-card__title { … }  /* modifier affecting an element */
```

What you get: flat specificity (almost everything is one class, (0,1,0)); no
dependency on HTML nesting; names that say where the styles live (search for
`recipe-card` and you find everything); and safe deletion — when no HTML uses
`recipe-card`, all its CSS can go.

The cost is verbose class names, and you still need discipline to avoid styling bare
elements inside blocks.

## Pattern 2: utility classes

Utility-first CSS provides small, single-purpose classes and composes them in the HTML:

```html
<article class="flex gap-4 p-4 rounded-lg border">
  <img class="w-24 h-24 object-cover rounded" src="…" alt="…">
  <div>
    <h3 class="text-lg font-semibold">Roasted tomato soup</h3>
    <p class="text-sm text-muted">45 minutes</p>
  </div>
</article>
```

The stylesheet stops growing (every utility is written once), there's no naming, and
styles are local to the markup you're looking at. The costs: long class lists, repeated
combinations across templates (usually solved by components in a framework), and the
need for a build tool to generate only the utilities you use. Frameworks like Tailwind
CSS are built on this idea.

Most real codebases mix approaches: **components** for recurring patterns, **utilities**
for one-off spacing and layout tweaks, and a small **base** layer for element defaults.

## Pattern 3: layers as the architecture

Cascade layers (Level 3 · 07) turn "which kind of rule wins" into an explicit
declaration, independent of selector specificity:

```css title="css/main.css"
@layer reset, tokens, base, layout, components, utilities, overrides;

@import url("reset.css") layer(reset);
@import url("tokens.css") layer(tokens);
@import url("base.css") layer(base);
@import url("layout.css") layer(layout);
@import url("components/button.css") layer(components);
@import url("components/recipe-card.css") layer(components);
@import url("utilities.css") layer(utilities);
```

(`@import` rules must come before other rules except `@layer` statements and
`@charset`. In production, bundle these into one file — Level 4 · 02 explains why
`@import` chains hurt loading — keeping the `layer()` wrappers.)

| Layer | Contains | Typical selectors |
|-------|----------|------------------|
| reset | Normalising browser defaults | elements, `:where()` |
| tokens | Custom properties only | `:root` |
| base | Element defaults: typography, links, forms | elements |
| layout | Page-level structure: containers, grids, stacks | `.l-*` classes |
| components | Self-contained UI pieces | one class per rule, BEM-style |
| utilities | Single-purpose helpers that must win | `.u-*` classes |
| overrides | Third-party fixes, temporary hacks — kept small and reviewed | anything |

The payoff: a utility class like `.u-hidden` beats *any* component rule regardless of
specificity, and a third-party widget's CSS can be dropped into a low layer where its
selectors can't beat yours.

## Specificity budget

Whatever the methodology, adopt a specificity rule and enforce it:

- **No IDs in selectors.**
- **Max one class per selector in components**, except modifier/state combinations.
- **No `!important`** except in utilities (where it's deliberate) and inside the reset
  for things like `[hidden]`.
- **No descendant selectors more than two levels deep.**
- **Style state through attributes**: `[aria-expanded="true"]`, `[aria-current="page"]`,
  `:disabled` — the markup and styles can't drift apart (Level 3 · 10).

## Linting: enforce the rules automatically

Conventions nobody checks decay. **Stylelint** is the standard CSS linter. With the
`stylelint-config-standard` preset, we linted this file:

```css title="sample.css"
.card {
  color: #FFF;
  padding: 10px 10px 10px 10px;
  colr: red;
}

.card .title {
  font-weight: bold;
  color: #b3401a;
  color: #8a2f12;
}

#header .nav a:hover { text-decoration: underline }

@media (min-width: 768px) {
  .card { padding: 20px }
}
```

```text
   3:12  ✖  Expected "10px 10px 10px 10px" to be "10px"      shorthand-property-no-redundant-values
   4:3   ✖  Unknown property "colr"                          property-no-unknown
   9:3   ✖  Duplicate property "color"                       declaration-block-no-duplicate-properties
  15:8   ✖  Expected "context" media feature range notation  media-feature-range-notation

✖ 4 problems (4 errors, 0 warnings)
```

(Stylelint 17.15, exit code 2.) It caught a typo that browsers would silently ignore
(`colr`), a duplicate that hides a merge mistake, and it enforces the modern range
syntax. It did **not** complain about `#header .nav a:hover` — the standard preset has
no specificity rules. Add them to your config to enforce your budget:

```json title=".stylelintrc.json"
{
  "extends": ["stylelint-config-standard"],
  "rules": {
    "selector-max-id": 0,
    "selector-max-specificity": "0,2,1",
    "selector-max-compound-selectors": 3,
    "declaration-no-important": true
  }
}
```

With that config, the same file produced two more errors, both on line 13:

```text
  13:1   ✖  Too many ID selectors in "#header .nav a:hover", maximum 0       selector-max-id
  13:1   ✖  Too high specificity in "#header .nav a:hover", maximum "0,2,1"  selector-max-specificity
```

Run it in CI and in the editor so problems appear as you type.

## Worked example: one component, done properly

```css title="css/components/recipe-card.css"
@layer components {
  .recipe-card {
    /* public API: custom properties a parent may set */
    --recipe-card-accent: var(--color-brand);
    --recipe-card-radius: var(--radius-md);

    container: recipe-card / inline-size;
    display: grid;
    gap: var(--space-2);
    padding: var(--space-3);
    border: 1px solid var(--color-border);
    border-radius: var(--recipe-card-radius);
    background: var(--color-surface);
  }

  .recipe-card--featured { --recipe-card-accent: var(--color-highlight); border-width: 2px; }

  .recipe-card__title { margin: 0; font-size: var(--text-lg); }
  .recipe-card__meta  { margin: 0; color: var(--color-muted); font-size: var(--text-sm); }
  .recipe-card__image { width: 100%; aspect-ratio: 4 / 3; object-fit: cover; }

  @container recipe-card (width >= 28rem) {
    .recipe-card { grid-template-columns: 10rem 1fr; }
  }
}
```

One file per component, all selectors starting with the component name, all values from
tokens (next lesson), a documented set of custom properties as its styling API, and a
container query so it adapts wherever it's used. Deleting the component means deleting
one file.

## How It Actually Works

Scale problems in CSS come from the cascade's global nature: every rule is compared with
every other rule that matches the same element. Methodologies work by **reducing the
number of rules that can ever compete**:

- BEM makes class names unique to one component, so rules from different components
  never match the same element.
- Utilities make each rule set one property, so conflicts are between two utilities on
  the same element — visible right there in the HTML.
- Layers group rules so that conflicts *between* groups are resolved by declared order,
  leaving specificity to arbitrate only within a group.
- Tooling-based scoping (CSS Modules, framework scoped styles, the `@scope` at-rule) goes
  further, rewriting or limiting selectors so a component's rules physically can't match
  outside it.

`@scope` is the native version of the last idea:

```css
@scope (.recipe-card) to (.recipe-card__body) {
  img { border-radius: var(--radius-sm); }   /* only images in the card, not in its body slot */
}
```

We tested the same scope (with an outline, which is easy to read back) on an image in
the card, an image inside `.body` and an image
outside the card: only the first got the outline (`["solid", "none", "none"]`) in both
Chromium 153 and WebKit 26.6. `@scope` arrived in some other
browsers later than the rest of this lesson's techniques — check current support before
building your architecture on it.

## Common mistakes

- **No agreed convention**, so every developer invents their own.
- **Styling bare elements inside components** (`.card h3`), which breaks when the heading
  level changes.
- **`!important` as a fix** outside utilities.
- **Never deleting CSS** because nobody knows what's used — name things so you can search.
- **Linting without enforcing specificity rules.**
- **Treating a methodology as a religion.** The goal is predictability; mix approaches
  where they help.

## Exercise

1. Reorganise your Level 3 gallery CSS into files per layer and per component, with a
   `main.css` that declares the layer order.
2. Rename one component's classes to BEM. Search the codebase for its name and confirm
   every related rule is found.
3. Install Stylelint with the standard config plus the specificity rules above. Run it on
   your project and fix everything it reports.
4. Put a third-party stylesheet (any small CSS library) in a `vendor` layer and confirm your
   single-class component styles now beat its selectors.
