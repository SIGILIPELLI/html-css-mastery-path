---
description: "Animate responsibly: transitions, keyframes, timing functions, animating to height auto, cheap vs expensive properties, reduced motion."
---

# 08 · Transitions & Animations

Motion in an interface does real work when it's used well: it shows where something came
from, confirms that an action happened, and helps the eye follow a change. Used badly it
distracts, slows people down, and — for people with vestibular disorders — can cause
dizziness and nausea. CSS gives you two tools, **transitions** (animate between two
states when a value changes) and **keyframe animations** (run a defined sequence), plus
a media query to respect people who want less motion.

## Transitions

A transition says: "when this property changes, animate to the new value instead of
jumping."

```css
.button {
  background: var(--brand);
  transition: background-color 150ms ease-out, transform 150ms ease-out;
}
.button:hover { background: var(--brand-hover); }
.button:active { transform: translateY(1px); }
```

The shorthand is `property duration timing-function delay`. List properties explicitly
rather than `transition: all` — `all` animates things you didn't intend (a colour change
from a theme switch, a width change during a resize) and makes debugging harder.

We hovered an element with `transition: transform .3s, opacity .3s` and asked Chromium
which animations were running (`document.getAnimations()`):

```text
transitions running on hover: ["opacity", "transform"]
```

Transitions are real animation objects you can inspect — and pause, slow down and scrub
in devtools' **Animations** panel.

### Timing functions

| Function | Feels like |
|----------|-----------|
| `ease` (default) | Quick start, gentle finish |
| `ease-out` | Fast start, slow end — good for things **entering** |
| `ease-in` | Slow start, fast end — good for things **leaving** |
| `ease-in-out` | Slow at both ends — movement between two positions |
| `linear` | Constant speed — spinners, progress |
| `cubic-bezier(x1, y1, x2, y2)` | Custom curve |
| `steps(4)` | Discrete jumps — sprite animations |
| `linear(0, 0.6 20%, 1.1 50%, 1)` | A piecewise curve, e.g. springs and bounces |

Durations for UI feedback are short: 100–200ms for hovers and presses, 200–400ms for
things moving across the screen. Longer feels sluggish.

## Keyframe animations

```css
@keyframes pulse {
  from { scale: 1; }
  50%  { scale: 1.08; }
  to   { scale: 1; }
}

.badge-new {
  animation: pulse 1.2s ease-in-out 3;   /* name duration timing iteration-count */
}
```

Other useful sub-properties: `animation-delay`, `animation-direction: alternate`,
`animation-fill-mode: both` (hold the first frame during the delay and the last frame
afterwards), `animation-play-state: paused`.

Note `scale` (and `rotate`, `translate`) are individual transform properties — you can
animate one without overwriting the others, which a single `transform` property can't do.

## What's cheap to animate

Every animation frame has to be produced within about 16ms (at 60Hz) to look smooth.
Which pipeline stages a property touches (Level 4 · 01) decides the cost:

| Property | Triggers | Cost |
|----------|---------|------|
| `transform` (`translate`, `scale`, `rotate`), `opacity` | composite only | **Cheap** — can run on the compositor, off the main thread |
| `filter` (in many cases) | composite | Usually cheap |
| `color`, `background-color`, `box-shadow`, `border-radius` | paint | Moderate |
| `width`, `height`, `top`, `left`, `margin`, `padding`, `font-size` | **layout** + paint | **Expensive** — and can move everything around it |

So: animate `translate` instead of `left`, `scale` instead of `width`, `opacity` instead of
`visibility` toggling. A transform animation also keeps running smoothly when the main
thread is busy with JavaScript, because the compositor handles it.

## Animating to `height: auto`

The classic problem: you can't transition to `height: auto`, because `auto` isn't a
number the browser can interpolate. We transitioned three 200px-content panels from
closed to open over 1s (linear) and measured each at roughly the halfway point:

```css
.p { height: 0; transition: height 1s linear; }                     .p.open { height: auto; }
.q { height: 0; transition: height 1s linear; interpolate-size: allow-keywords; }
                                                                     .q.open { height: auto; }
.r { display: grid; grid-template-rows: 0fr; transition: grid-template-rows 1s linear; }
                                                                     .r.open { grid-template-rows: 1fr; }
```

```text
heights at ~0.5s: plain height:auto = 200, interpolate-size = 97, grid 0fr→1fr = 97
```

- **Plain `height: auto`** snapped straight to 200px — no animation.
- **`interpolate-size: allow-keywords`** made it animate. It's a newer property: it was
  supported in the Chromium build we tested but **not** in WebKit 26.6, so treat it as an
  enhancement (browsers without it snap, which is an acceptable fallback). You can set it
  once on `:root`.
- **The grid-rows trick** (`0fr` → `1fr`, with `overflow: hidden` on the child) animates in
  every current browser, because `fr` values *are* interpolable.

Height animations still trigger layout on every frame, so keep them short and don't run
many at once.

## Animating things that appear: `@starting-style`

When an element goes from `display: none` to visible, there's no "before" style to
transition from, so it just appears. `@starting-style` defines one:

```css
.toast {
  opacity: 1;
  translate: 0 0;
  transition: opacity 200ms ease-out, translate 200ms ease-out,
              display 200ms allow-discrete, overlay 200ms allow-discrete;
}

@starting-style {
  .toast { opacity: 0; translate: 0 1rem; }
}

.toast[hidden] { display: none; opacity: 0; }   /* exit state */
```

`transition-behavior: allow-discrete` (the `allow-discrete` keyword above) lets discrete
properties like `display` participate, so the element stays displayed until the exit
animation finishes. The `overlay` entry does the same for elements in the top layer
(dialogs and popovers, lesson 09). Both engines we tested supported `@starting-style`.

## Respecting reduced motion

Operating systems let users request reduced motion (macOS: Accessibility → Display →
Reduce motion; Windows: Settings → Accessibility → Visual effects → Animation effects;
iOS and Android have equivalents). CSS reads it with `prefers-reduced-motion`.

The best pattern is **opt-in**: write motion only for people who haven't asked to avoid it.

```css
.spinner { /* static fallback: e.g. a "Loading…" label */ }

@media (prefers-reduced-motion: no-preference) {
  .spinner { animation: spin 2s linear infinite; }
  .card { transition: translate 150ms ease-out; }
}
```

We ran a spinning-element page twice with Playwright emulating each preference, and
listed running animations:

```text
reducedMotion=no-preference: ["spin running"]
reducedMotion=reduce:        []
```

"Reduced" doesn't have to mean "none". Fades and colour changes are generally fine;
large movements, parallax, zooming, and anything that moves across a lot of the screen
are the problems. WCAG 2.3.3 (AAA) asks that motion triggered by interaction can be
disabled, and 2.2.2 (A) requires a way to pause anything that moves automatically for
more than five seconds — carousels and background videos included. And never flash more
than three times per second (2.3.1), which can trigger seizures.

## Worked example: an accessible "added to basket" confirmation

```html
<button type="button" class="add">Add to basket</button>
<p class="confirm" role="status"></p>
```

```css
.confirm { min-height: 1.5em; color: var(--success); font-weight: 600; }

@media (prefers-reduced-motion: no-preference) {
  .confirm.is-visible { animation: rise 300ms ease-out; }
  @keyframes rise {
    from { opacity: 0; translate: 0 0.5rem; }
  }
}
```

```js
const button = document.querySelector('.add');
const confirm = document.querySelector('.confirm');
button.addEventListener('click', () => {
  confirm.textContent = 'Added to basket';
  confirm.classList.remove('is-visible');
  void confirm.offsetWidth;          // force a style flush so the animation restarts
  confirm.classList.add('is-visible');
});
```

The message is text in a live region (lesson 04), so screen-reader users hear it; sighted
users get a short rise-and-fade; users who prefer reduced motion get the message without
the movement. The `@keyframes` rule has only a `from` block — the `to` state defaults to
the element's normal styles.

## How It Actually Works

When a transitioned property's computed value changes, the style engine creates a
**CSS transition** object with a start value, end value, duration and timing function.
On each animation frame (driven by the display's refresh), the browser samples every
running animation: progress = elapsed / duration, passed through the timing function,
then used to interpolate between start and end. The interpolated value is fed back into
the cascade at the "animations" and "transitions" origins (Level 1 · 08) — which is why
a running transition beats even `!important` author styles for the moment it runs.

For `transform` and `opacity`, the browser can hand the whole animation to the
**compositor thread**: the element is painted once into its own layer, and each frame the
compositor just moves or fades that layer on the GPU. For layout-affecting properties,
every frame requires style → layout → paint on the main thread, and any long-running
JavaScript on that thread makes the animation stutter.

`getAnimations()` returns every running `CSSTransition` and `CSSAnimation` object (plus
any created with the Web Animations API), which is how we observed them — and how you
can pause, reverse or `finish()` them from script.

## Common mistakes

- **`transition: all`.**
- **Animating `width`, `top` and `left`** where transforms would do.
- **Long, slow UI animations** (a 600ms hover).
- **Ignoring reduced motion**, or implementing it as "turn off all feedback".
- **Auto-playing carousels and background videos** with no pause control.
- **Hover-only animations** that carry information.
- **Expecting `height: auto` to animate** without `interpolate-size` or the grid trick.

## Exercise

1. Add hover and active transitions to your buttons using only `background-color` and
   `translate`. Keep durations under 200ms.
2. Build an accordion panel that animates open using the grid `0fr → 1fr` technique. Then
   add `interpolate-size: allow-keywords` on `:root` and animate `height` directly —
   compare in Chrome and Safari.
3. Make a toast notification that animates in with `@starting-style` and out with
   `allow-discrete`.
4. Turn on reduced motion in your operating system and go through your site. Anything
   still moving that shouldn't be?
5. Open devtools' Animations panel, trigger your animations, and slow them to 10% to
   inspect the easing.
