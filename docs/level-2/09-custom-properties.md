---
description: "CSS custom properties in depth: how var() inherits and cascades, fallbacks, the invalid-at-computed-value-time trap, theming, dark mode and @property."
---

# 09 · Custom Properties & Theming

Custom properties — often called CSS variables — let you name a value once and use it
everywhere: `--brand: #b3401a;` then `color: var(--brand)`. That alone would be useful,
but they're more than preprocessor variables. They live in the browser, take part in the
cascade, **inherit** down the tree, can be changed per component or per media query, and
can be read and written by JavaScript. That's what makes them the foundation of theming
and design systems (Level 4 · 04).

## Declaring and using

```css
:root {
  --brand: #b3401a;
  --radius: 0.5rem;
  --space: 1rem;
  --font-ui: system-ui, sans-serif;
}

.button {
  background: var(--brand);
  border-radius: var(--radius);
  padding: calc(var(--space) * 0.5) var(--space);
  font-family: var(--font-ui);
}
```

- Names start with two dashes and are **case-sensitive** (`--Brand` ≠ `--brand`).
- `:root` is the `<html>` element (with higher specificity than `html`). Declaring there
  makes the values available everywhere through inheritance.
- A custom property can hold almost anything: a colour, a length, a list of fonts, a
  number, even a partial value like `10px 20px`.

## They inherit — and can be overridden per subtree

Custom properties are inherited properties. Redefine one on an element and every
descendant sees the new value:

```css
:root { --space: 1rem; }
.card { --space: 2rem; padding: var(--space); }
.x    { padding: var(--space); }
```

We put an `.x` element at the top level and another inside a `.card`, then read their
computed padding in Chromium:

```text
.x at top level:  16px
.x inside .card:  32px
```

The same rule (`.x { padding: var(--space) }`) produced different results depending on
where the element sits. That's the core of component theming: a component reads
variables, and whatever context it's placed in can set them.

```css
.callout          { --accent: #1a5fb4; border-left: 4px solid var(--accent); }
.callout.warning  { --accent: #b3401a; }
.callout.success  { --accent: #26734d; }
```

One rule for the structure, one line per variant.

## Fallbacks

`var(--name, fallback)` uses the fallback if the property isn't defined (or is the
special "guaranteed-invalid" initial value):

```css
.fb { color: var(--missing, rgb(0, 0, 255)); }   /* computed: rgb(0, 0, 255) */
.card { padding: var(--card-padding, var(--space, 1rem)); }   /* nested fallbacks */
```

Fallbacks let a component work with sensible defaults while exposing variables that a
theme can set.

## The trap: invalid at computed-value time

The browser can't check a `var()` value when it parses your CSS, because it doesn't know
what the variable will contain. It accepts the declaration and only substitutes the value
at **computed-value time**. If the result is invalid for that property, the declaration
doesn't fall back to an earlier declaration in your CSS — that's long been discarded by
the cascade. Instead the property becomes **`unset`**: inherited if it's an inherited
property, initial if not.

We tested this directly:

```css
.bad  { --w: red;  width: var(--w);   color: rgb(0, 128, 0); color: var(--w); }
.bad2 { --c: 20px; color: rgb(0, 128, 0); color: var(--c); }
```

```text
.bad  width: 1264px          <- `width: red` is invalid: width became `auto` (full width)
.bad  color: rgb(255, 0, 0)  <- `color: red` is valid, so red it is
.bad2 color: rgb(0, 0, 0)    <- `color: 20px` is invalid: color became inherited black,
                                NOT the green declared one line earlier
```

`.bad2` is the lesson. With ordinary values, an invalid declaration is dropped at parse
time and the previous one (green) would apply. With `var()`, the green declaration lost
the cascade before the substitution happened, so there's nothing to fall back to. That
also means the old trick of writing a fallback declaration before a `var()` declaration
does **not** protect against a bad variable value.

Two more consequences of substitution:

```css
.calc { --n: 3; margin-top: calc(var(--n) * 10px); }   /* 30px: numbers work in calc() */
.str  { --gap: 10 px; padding-top: var(--gap); }       /* "10 px" is two tokens: invalid, 0px */
```

Substitution is token-based: `var(--n)px` doesn't make `3px`, it makes `3 px` (two
tokens), which is invalid. Multiply by a unit instead: `calc(var(--n) * 1px)`.

## Theming and dark mode

Keep raw values in one place and point everything else at *semantic* variables:

```css title="styles.css"
:root {
  color-scheme: light dark;

  /* palette */
  --orange-700: oklch(45% 0.15 38);
  --orange-300: oklch(78% 0.1 45);
  --neutral-50: oklch(98.5% 0.005 80);
  --neutral-900: oklch(20% 0.01 60);

  /* semantic roles */
  --color-bg: var(--neutral-50);
  --color-text: var(--neutral-900);
  --color-link: var(--orange-700);
}

@media (prefers-color-scheme: dark) {
  :root {
    --color-bg: var(--neutral-900);
    --color-text: var(--neutral-50);
    --color-link: var(--orange-300);
  }
}

body { background: var(--color-bg); color: var(--color-text); }
a { color: var(--color-link); }
```

Components only ever use the semantic names (`--color-link`), so dark mode is a remap of
a few roles, not a rewrite of every component.

To let users override the OS preference with a toggle, add an attribute-based theme with
the same variables:

```css
:root[data-theme="dark"] {
  color-scheme: dark;
  --color-bg: var(--neutral-900);
  --color-text: var(--neutral-50);
  --color-link: var(--orange-300);
}
```

A small script sets `document.documentElement.dataset.theme = 'dark'` and remembers the
choice.

## JavaScript access

```js
const root = document.documentElement;
getComputedStyle(root).getPropertyValue('--brand');   // read
root.style.setProperty('--brand', '#26734d');          // write (inline on <html>)
element.style.setProperty('--progress', '0.72');        // per element
```

We read `--space` from the `.card` above with `getPropertyValue` and got the string
`"2rem"` — unregistered custom properties return exactly the tokens you wrote, not a
computed pixel length.

Setting a variable from JavaScript and letting CSS do the rest keeps logic and
presentation separate:

```css
.progress-bar { width: calc(var(--progress, 0) * 100%); }
```

## `@property`: typed custom properties

Ordinary custom properties are untyped strings of tokens, which means the browser can't
**animate** them — it doesn't know what's between `0deg` and `180deg`. Registering a
property with `@property` gives it a type, an initial value and an inheritance rule:

```css
@property --angle {
  syntax: '<angle>';
  inherits: false;
  initial-value: 0deg;
}

.spinner {
  background: conic-gradient(from var(--angle), #b3401a, #e0b457, #b3401a);
  transition: --angle 1s linear;
}
.spinner:hover { --angle: 180deg; }
```

We transitioned a registered and an unregistered property from `0deg` to `180deg` over
1s and read both values halfway through:

```text
registered   --angle: 87.012deg
unregistered --u:     180deg
```

The registered property was genuinely interpolating; the unregistered one jumped straight
to its end value. Registration also means values are type-checked: an invalid value
leaves the property at its `initial-value` (or the inherited value, if it inherits)
instead of passing arbitrary tokens along. And `inherits: false` stops a value leaking
into every descendant.

## How It Actually Works

Custom properties go through the cascade exactly like other properties: each element gets
one winning declaration per custom property, and because they inherit, elements without
a declaration take the parent's value. What's special is **when** `var()` is resolved:

1. **Parse time**: a declaration containing `var()` is accepted as long as the
   non-variable parts are syntactically plausible. Its value is stored as a token
   sequence.
2. **Cascade**: declarations compete as usual. The `var()` declaration can win over a
   perfectly valid earlier one — it hasn't been checked yet.
3. **Computed-value time**: for each element, the browser substitutes the element's
   computed value of each referenced custom property, then parses the result as the real
   property. If it's invalid, the property is treated as `unset`.

Custom properties referencing each other form a dependency graph; a cycle
(`--a: var(--b); --b: var(--a)`) makes all properties in it invalid. Because unregistered
custom properties inherit and computing them is cheap, you can declare hundreds on
`:root` without trouble — but changing a variable on `:root` from JavaScript invalidates
style for **every element that inherits it**, which is the whole page. For values that
change every frame (a mouse position, a scroll progress), set them on the smallest
element that needs them, or register them with `inherits: false`.

## Common mistakes

- **Forgetting the `--`** or the `var()`: `color: --brand` is invalid.
- **Expecting an earlier declaration to act as a fallback** for a bad variable.
- **Concatenating units**: `var(--n)px`. Use `calc(var(--n) * 1px)`.
- **Using variables in media query conditions**: `@media (width >= var(--bp))` doesn't
  work — media queries aren't evaluated per element.
- **Only palette variables, no semantic ones**, so dark mode means touching every rule.
- **Animating unregistered properties** and wondering why they jump.

## Exercise

1. Move every colour, radius and spacing value in your recipe stylesheet into custom
   properties on `:root`, split into palette and semantic roles.
2. Add dark mode by remapping only the semantic variables.
3. Build a `.callout` component with `--accent` and three variants using one-line
   overrides.
4. Reproduce the `.bad2` experiment in your browser and explain the colour you see.
5. Register an `@property` for a colour or angle and animate it on hover. Remove the
   registration and compare.
