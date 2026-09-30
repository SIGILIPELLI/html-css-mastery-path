---
description: "Build a design-token system in CSS: primitive, semantic and component tokens, spacing and type scales, themes, and a common resolution bug."
---

# 04 · Design Tokens & Theming Systems

A **design token** is a named design decision: `--color-text`, `--space-3`,
`--radius-md`, `--font-size-lg`, `--duration-fast`. Tokens are how a design system stays
consistent across hundreds of components and several products — change a token once,
and everything that uses it updates. In CSS, custom properties (Level 2 · 09) are the
natural way to implement them, because they cascade, inherit, can be redefined per theme
or per subtree, and are readable from JavaScript.

## Three tiers of tokens

Well-organised systems separate tokens into tiers, each referring only to the tier below:

```css title="css/tokens.css"
@layer tokens {
  :root {
    /* 1. Primitive (reference) tokens: the raw palette and scales. No meaning. */
    --orange-300: oklch(78% 0.10 45);
    --orange-500: oklch(60% 0.16 40);
    --orange-700: oklch(45% 0.15 38);
    --gray-50:  oklch(98.5% 0.004 80);
    --gray-200: oklch(90% 0.008 70);
    --gray-600: oklch(45% 0.01 60);
    --gray-900: oklch(20% 0.01 60);
    --red-600:  oklch(50% 0.19 25);

    /* 2. Semantic (system) tokens: roles. Components use THESE. */
    --color-bg: var(--gray-50);
    --color-surface: white;
    --color-text: var(--gray-900);
    --color-text-muted: var(--gray-600);
    --color-border: var(--gray-200);
    --color-brand: var(--orange-700);
    --color-on-brand: white;
    --color-danger: var(--red-600);
    --color-focus: oklch(45% 0.15 255);
  }
}
```

```css title="css/components/button.css"
@layer components {
  .button {
    /* 3. Component tokens: a component's own knobs, defaulting to semantic tokens */
    --button-bg: var(--color-brand);
    --button-text: var(--color-on-brand);
    --button-radius: var(--radius-md);

    background: var(--button-bg);
    color: var(--button-text);
    border-radius: var(--button-radius);
  }
  .button--danger { --button-bg: var(--color-danger); }
}
```

Why three tiers:

- **Primitives** name *what* a value is (`--orange-700`). They change only when the brand
  palette changes.
- **Semantic tokens** name *what it's for* (`--color-brand`, `--color-danger`). Themes
  remap these; components never touch primitives directly.
- **Component tokens** expose a component's customisation points without leaking its
  internals, and variants become one-line overrides.

A rule of thumb: if a component refers to `--orange-700`, a dark theme can't fix it
without editing the component.

## Scales

Tokens for spacing, type and radius work best as a small **scale** rather than
arbitrary values:

```css
:root {
  /* spacing: a modular scale based on 0.25rem steps */
  --space-1: 0.25rem;
  --space-2: 0.5rem;
  --space-3: 1rem;
  --space-4: 1.5rem;
  --space-5: 2rem;
  --space-6: 3rem;
  --space-7: 4rem;

  /* fluid type scale: each step grows between 360px and 1280px viewports */
  --text-sm:   clamp(0.875rem, 0.84rem + 0.15vw, 0.95rem);
  --text-base: clamp(1rem, 0.95rem + 0.22vw, 1.125rem);
  --text-lg:   clamp(1.2rem, 1.1rem + 0.4vw, 1.4rem);
  --text-xl:   clamp(1.5rem, 1.3rem + 0.9vw, 2rem);
  --text-2xl:  clamp(1.9rem, 1.5rem + 1.8vw, 2.9rem);

  --radius-sm: 0.25rem;
  --radius-md: 0.5rem;
  --radius-lg: 1rem;

  --duration-fast: 120ms;
  --duration-base: 200ms;
  --ease-out: cubic-bezier(0.2, 0.8, 0.2, 1);

  --shadow-1: 0 1px 2px rgb(0 0 0 / 0.08);
  --shadow-2: 0 4px 12px rgb(0 0 0 / 0.12);
}
```

A limited scale is a feature: it stops the "13px here, 14px there" drift, and makes
spacing look deliberate because every gap is one of seven values.

## Themes

With semantic tokens in place, a theme is just a remapping. Support three sources of
theme choice — the OS preference, an explicit user choice, and a scoped section of the
page:

```css
@layer tokens {
  /* OS preference, unless the user explicitly chose light */
  @media (prefers-color-scheme: dark) {
    :root:not([data-theme="light"]) {
      color-scheme: dark;
      --color-bg: var(--gray-900);
      --color-surface: oklch(25% 0.008 60);
      --color-text: var(--gray-50);
      --color-text-muted: oklch(76% 0.01 80);
      --color-border: oklch(38% 0.01 60);
      --color-brand: var(--orange-300);
      --color-on-brand: var(--gray-900);
    }
  }

  /* explicit choice */
  :root[data-theme="dark"] {
    color-scheme: dark;
    --color-bg: var(--gray-900);
    /* …same mapping… */
  }

  /* a scoped theme: e.g. an always-dark promo band */
  .theme-inverse {
    color-scheme: dark;
    --color-bg: var(--gray-900);
    --color-text: var(--gray-50);
    background: var(--color-bg);
    color: var(--color-text);
  }
}
```

The scoped theme works because custom properties inherit: every component inside
`.theme-inverse` reads the redefined tokens without knowing anything changed.

The duplication between the media query and the `[data-theme="dark"]` block is the main
annoyance. Two ways to reduce it: generate the CSS from a token file (below), or use
`light-dark()` for each semantic colour and switch only `color-scheme`:

```css
:root {
  color-scheme: light dark;
  --color-bg: light-dark(var(--gray-50), var(--gray-900));
  --color-text: light-dark(var(--gray-900), var(--gray-50));
}
:root[data-theme="light"] { color-scheme: light; }
:root[data-theme="dark"]  { color-scheme: dark; }
```

`light-dark()` picks its first or second argument based on the element's used
`color-scheme`, so a theme toggle only has to set one property. It only works for colours.
We checked all four combinations of OS preference and `data-theme` in Chromium, reading
the body's background:

```text
OS light, no data-theme      -> rgb(250, 250, 250)
OS dark,  no data-theme      -> rgb(20, 20, 20)
OS dark,  data-theme="light" -> rgb(250, 250, 250)
OS light, data-theme="dark"  -> rgb(20, 20, 20)
```

### Persisting the user's choice without a flash

```html
<script>
  // in <head>, before any stylesheet, so the theme applies before first paint
  try {
    const theme = localStorage.getItem('theme');
    if (theme) document.documentElement.dataset.theme = theme;
  } catch {}
</script>
```

Running this before first paint avoids the page flashing light and then switching to
dark. It's one of the few scripts that should be inline and blocking.

## Tokens as data

Large systems keep tokens in a platform-neutral format and generate CSS, iOS, Android and
design-tool values from it. The W3C **Design Tokens Community Group** format is a JSON
structure that tools like Style Dictionary and design-tool plugins can read:

```json title="tokens/color.tokens.json"
{
  "color": {
    "brand": { "$type": "color", "$value": "{orange.700}" },
    "danger": { "$type": "color", "$value": "{red.600}" }
  },
  "orange": {
    "700": { "$type": "color", "$value": "oklch(45% 0.15 38)" }
  }
}
```

The format was still evolving at the time of writing (check the group's current draft
before adopting it, and check which colour formats your tools accept). The principle is
stable: one source of truth, generated outputs, no hand-copied hex codes.

## Worked example: auditing a stylesheet for token adoption

Before migrating an existing codebase, find the raw values. A quick shell check for
hard-coded colours outside the tokens file:

```bash
grep -rnoE '#[0-9a-fA-F]{3,8}\b|rgb\([^)]*\)|oklch\([^)]*\)' css/ --include='*.css' | grep -v 'css/tokens.css'
```

Each hit is a candidate: map it to an existing semantic token, or decide it's a new role
and add one. Stylelint can then enforce the result — the
`declaration-property-value-disallowed-list` rule can ban raw colour values on `color`,
`background-color` and `border-color` outside the tokens file (via an `overrides` entry
in the config).

## How It Actually Works

Token systems rely on two custom-property behaviours from Level 2 · 09: **inheritance**
and **late resolution**. `--color-brand: var(--orange-700)` is stored as tokens and
resolved per element at computed-value time. So when `.theme-inverse` redefines
`--color-text`, every element inside it computes `color: var(--color-text)` against the
new value; nothing is recompiled.

The flip side: references resolve **on the element where they're used**, against that
element's inherited values. If you define `--button-bg: var(--color-brand)` on `:root`,
it's resolved at `:root` — and a scoped theme that later redefines `--color-brand` on a
section will *not* change `--button-bg`, because the inherited value of `--button-bg` is
already the resolved colour from `:root`. That's why the component tokens in the example
are declared **on `.button` itself**: they're resolved on each button, against whatever
theme it's in. It's the most common bug in custom-property theming.

We reproduced it with two buttons — one reading a token declared on `:root`
(`--early-bg: var(--color-brand)`), one declaring its token on itself — placed outside
and then inside a section that redefines `--color-brand`:

```text
outside: btn-early rgb(200, 80, 30)    btn-late rgb(200, 80, 30)
inside:  btn-early rgb(200, 80, 30)    btn-late rgb(250, 180, 120)
```

Inside the themed section, only the button whose token is resolved on itself picked up
the new brand colour.

## Common mistakes

- **Components using primitive tokens** (`--orange-700`) directly.
- **Component tokens declared on `:root`**, which then ignore scoped themes.
- **Too many tokens** — a token for every value defeats the purpose. A scale of 6–10
  spacing values is plenty.
- **Dark themes that only swap background and text**, leaving borders, shadows and focus
  rings designed for light backgrounds.
- **A theme toggle that flashes** the wrong theme on load.
- **Tokens that exist in the design tool and the CSS separately**, drifting apart.

## Exercise

1. Write a `tokens.css` for your projects with primitive, semantic and (for buttons and
   cards) component tokens, plus a spacing scale, type scale and radii.
2. Run the grep audit on your CSS and replace every raw colour and spacing value.
3. Add OS-preference dark mode *and* a toggle button that sets `data-theme`, persisted
   in `localStorage`, with the inline head script to avoid a flash.
4. Add a `.theme-inverse` section and put a button inside it. If the button doesn't pick
   up the inverse theme, find which token is resolved too early.
