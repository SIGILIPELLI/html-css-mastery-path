---
description: "Level 3 of HTML & CSS: accessibility in depth, keyboard and focus, ARIA, accessible forms, container queries, :has, layers, motion, dialogs."
---

# Level 3 · Advanced <span class="level-badge">Accessibility & Modern CSS</span>

Goal: build interfaces that work for everyone — keyboard users, screen-reader users,
people who zoom, people who need less motion — and use the modern CSS features that
make component-based styling practical: container queries, `:has()`, nesting and cascade
layers.

The two halves of this level fit together more than they first appear. Accessible
components keep their state in attributes (`aria-expanded`, `aria-pressed`, `open`); modern
selectors style directly from those attributes and from relationships between elements;
container queries let a component adapt to wherever it's placed; layers keep all of it
from turning into a specificity war.

Three ideas carry this level:

1. **The accessibility tree is a second rendering of your page.** Roles, names and states
   come from your markup. Native elements provide them for free; ARIA fills genuine gaps
   and changes only what's announced, never behaviour.
2. **Focus is a user's position on the page.** You're responsible for where it goes
   whenever the page changes without a navigation.
3. **Components should adapt to their context, not to the viewport.** Container queries,
   `:has()` and cascade layers are the tools.

## Modules

1. [Accessibility Foundations & the Accessibility Tree](01-accessibility-foundations.md) — who's affected, WCAG, roles/names/states, hiding techniques
2. [Landmarks, Headings & Navigation](02-landmarks-headings.md) — when elements become landmarks, outlines, skip links, `aria-current`
3. [Keyboard Access & Focus Management](03-keyboard-focus.md) — tab order, `tabindex`, `:focus-visible`, `inert`, roving tabindex
4. [ARIA: When and How to Use It](04-aria.md) — the rules, disclosure widgets, live regions
5. [Accessible Forms & Error Handling](05-accessible-forms.md) — hints, `autocomplete`, inline errors, error summaries
6. [Container Queries](06-container-queries.md) — `container-type`, `@container`, `cqi` units, style queries, the zero-width trap
7. [Modern Selectors, Nesting & Cascade Layers](07-modern-selectors-layers.md) — `:is`/`:where`/`:has`, nesting, `@layer`
8. [Transitions & Animations](08-transitions-animations.md) — cheap vs expensive properties, `height: auto`, `@starting-style`, reduced motion
9. [Native Interactive Elements](09-native-interactive-elements.md) — `details`, `dialog`, `popover`, the top layer, anchor positioning
10. [Project — An Accessible Component Gallery](10-project-component-gallery.md) — seven components, validated, audited and keyboard-tested

## Before you start

Complete [Level 2](../level-2/index.md), or be comfortable with Grid, Flexbox and custom
properties. A few lessons use small amounts of JavaScript (event listeners, setting
attributes); if that's new, [JavaScript Mastery Path](https://sigilipelli.github.io/javascript-mastery-path/)
covers what you need. You'll also want:

- **A screen reader to test with.** VoiceOver is built into macOS and iOS; NVDA is free
  for Windows; TalkBack is on Android. Learn five commands: start/stop, next item, next
  heading, next landmark, activate.
- **An automated checker**: the axe DevTools browser extension, or Lighthouse's
  accessibility audit in Chrome devtools.

## How the examples were checked

Accessibility-tree output in this level comes from Playwright's `ariaSnapshot()` (and,
in one lesson, Chromium's DevTools protocol) run against headless Chromium 153; keyboard
behaviour comes from scripted key presses with focus recorded after each one; feature
support from `CSS.supports()` in Chromium 153 and WebKit 26.6. Automated audits use
axe-core 4.13.

!!! warning "What we couldn't test"
    Screen-reader **speech** — what VoiceOver, NVDA or JAWS actually say — isn't
    observable from an automated browser, so we don't show any. Where a lesson describes
    announcements, it's describing documented behaviour; test with a real screen reader
    before relying on it. We also couldn't run Firefox in our environment.
