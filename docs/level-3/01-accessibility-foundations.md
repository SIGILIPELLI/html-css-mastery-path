---
description: "Web accessibility in practice: who is affected, WCAG's POUR principles, roles, names and states, and how browsers build the accessibility tree."
---

# 01 · Accessibility Foundations & the Accessibility Tree

Accessibility means people can use what you build **regardless of how they perceive and
operate it**: with a screen reader, a keyboard only, voice commands, a switch device,
magnification at 400%, high-contrast colours, captions, or just with one hand holding a
baby. The World Health Organization estimates that about one in six people worldwide live
with a significant disability — and nearly everyone has temporary or situational
limitations (a broken wrist, bright sunlight, a noisy train).

For HTML and CSS authors, accessibility is mostly not an extra layer. It's a
consequence of the choices from Levels 1 and 2 — semantic elements, labels, contrast,
focus styles, reflow at zoom. This level makes those choices deliberate, explains the
machinery underneath, and covers the cases where native HTML isn't enough.

## Who uses what

| People | Often use | What breaks for them |
|--------|-----------|----------------------|
| Blind and low-vision | Screen readers (NVDA, JAWS, VoiceOver, TalkBack, Narrator), magnifiers, zoom | Missing alt text, unlabelled controls, meaningless structure, text that doesn't reflow |
| Motor impairments | Keyboard, switch access, voice control (Voice Control, Dragon), eye tracking | Mouse-only interactions, tiny targets, no visible focus, time limits |
| Deaf and hard of hearing | Captions, transcripts | Uncaptioned video, audio-only alerts |
| Cognitive and learning | Consistent layouts, plain language, reader modes | Clutter, unpredictable changes, jargon, animations |
| Vestibular disorders | Reduced-motion settings | Parallax, large animations |
| Colour vision deficiency | Nothing special — they just see differently | Information conveyed by colour alone |

## WCAG in one page

The **Web Content Accessibility Guidelines** (WCAG) are the international standard, and
the basis of most accessibility laws. Version 2.2 is current; each version is backwards
compatible. Success criteria are organised under four principles — **POUR**:

- **Perceivable** — text alternatives, captions, content that adapts (reflow, zoom),
  sufficient contrast.
- **Operable** — everything works from a keyboard, visible focus, enough time, no
  seizure-inducing flashing, targets big enough to hit.
- **Understandable** — readable text, predictable behaviour, help avoiding and fixing
  errors.
- **Robust** — valid, well-structured markup that assistive technologies can interpret.

Each criterion has a level: **A** (the minimum), **AA** (the usual legal and contractual
target), **AAA** (enhanced). "WCAG 2.2 AA" is the target to design for. The criteria
themselves are testable statements, like 1.4.3 *Contrast (Minimum)*: text has a contrast
ratio of at least 4.5:1.

## The accessibility tree

Assistive technologies don't read your HTML or look at your pixels. The browser builds a
second tree alongside the DOM — the **accessibility tree** — and exposes it through the
operating system's accessibility API (UI Automation on Windows, NSAccessibility on macOS,
AT-SPI on Linux, and the mobile equivalents). Every node has:

- a **role** — what it is: `button`, `link`, `heading`, `checkbox`, `navigation`...
- a **name** — what it's called: "Close dialog", "Search recipes".
- **states and properties** — checked, expanded, disabled, invalid, level 2...
- sometimes a **value** and a **description**.

A screen reader announces those: "Search recipes, search text field", "Vegetarian,
checkbox, checked". Voice control uses the names: the user says "click Close dialog".

You can see the tree in devtools (Chrome/Edge: Elements → Accessibility pane, with "full
accessibility tree" enabled; Firefox: the Accessibility panel; Safari: Elements → Node →
Accessibility). Throughout this level we used Playwright's `ariaSnapshot()`, which prints
the role and name of each node as YAML.

## Worked example: where names come from

We rendered a set of common elements in Chromium and printed the accessibility tree:

```html
<button><svg aria-hidden="true">…</svg></button>
<button aria-label="Close dialog"><svg aria-hidden="true">…</svg></button>
<button aria-label="Ignored label" aria-labelledby="t">x</button><span id="t">Delete draft</span>
<img src="a.png" alt="Company logo">
<img src="b.png" alt="">
<img src="c.png">
<label for="q">Search recipes</label><input id="q" type="search">
<input type="text" placeholder="Only a placeholder">
<input type="text" title="Only a title">
<a href="/x"><img src="d.png" alt="Home"></a>
<button>Save <span class="visually-hidden">recipe</span></button>
```

```text
- button
- button "Close dialog"
- button "Delete draft": x
- img "Company logo"
- img
- searchbox "Search recipes"
- textbox "Only a placeholder"
- textbox "Only a title"
- link "Home":
  - img "Home"
- button "Save recipe"
```

Line by line:

1. **An icon-only button with no text has no name.** A screen reader says "button" and
   nothing else. This is one of the most common failures on the web.
2. `aria-label` provides a name where there's no visible text.
3. `aria-labelledby` **beats** `aria-label` — the name is "Delete draft", taken from the
   referenced element. Neither appears visually on the button; the button's own text `x`
   is ignored for naming.
4. `alt` names an image.
5. `alt=""` removes the image from the tree completely — correct for decoration.
6. **No `alt` at all** leaves an unnamed `img` in the tree. Screen readers then often
   announce the file name.
7. A `<label>` names its input.
8. A `placeholder` is used as a **last-resort** name when nothing else exists, and so is
   `title`. They're not reliable substitutes for a label: placeholders vanish when you
   type, and `title` isn't shown on touch devices.
9. A link's name comes from its content — here the image's alt.
10. Visually hidden text (explained below) still contributes to the name: "Save recipe".

This follows the **accessible name computation** algorithm, which checks sources in
priority order: `aria-labelledby`, then `aria-label`, then native labelling (`<label>`,
`alt`, `<caption>`, `<legend>`...), then the element's content (for roles that allow it,
like buttons and links), then `title` or `placeholder`.

## Hiding things: four techniques, four meanings

We rendered four paragraphs and checked which appeared in the accessibility tree:

```html
<p style="display:none">display none</p>
<p style="visibility:hidden">visibility hidden</p>
<p aria-hidden="true">aria hidden</p>
<p style="opacity:0">opacity zero</p>
```

```text
- paragraph: opacity zero
```

| Technique | Visible? | In accessibility tree? | Focusable? |
|-----------|----------|------------------------|------------|
| `display: none`, `hidden` attribute | no | no | no |
| `visibility: hidden` | no (space kept) | no | no |
| `aria-hidden="true"` | **yes** | no | **yes — danger** |
| `opacity: 0` | no | **yes** | yes |
| visually-hidden class | no | **yes** | yes |

The two "danger" cases are mirror images. `aria-hidden` on something focusable creates a
control a keyboard user can reach but a screen reader can't describe. `opacity: 0` leaves
invisible content that screen readers still announce.

The **visually-hidden** pattern hides content from sight but keeps it for assistive tech
— for extra context like the "recipe" in "Save recipe", or a heading that sighted users
don't need:

```css
.visually-hidden:not(:focus):not(:active) {
  position: absolute;
  width: 1px;
  height: 1px;
  overflow: hidden;
  clip-path: inset(50%);
  white-space: nowrap;
}
```

## States

States are exposed too. From our test page:

```text
- checkbox "Vegetarian" [checked]
- button "Bold" [pressed]
- button "Buy" [disabled]
- textbox "Name" [invalid]
- progressbar "Upload"
```

Native elements report state automatically — a checked checkbox is `[checked]` without
you doing anything. The toggle button and the invalid state came from ARIA attributes
(`aria-pressed="true"`, `aria-invalid="true"`), which you have to keep in sync yourself
(lesson 04).

## How It Actually Works

The browser derives the accessibility tree from the DOM *and* the computed styles:

1. Start from the DOM. Remove nodes that aren't rendered (`display: none`,
   `visibility: hidden`) or are hidden with `aria-hidden="true"` or `inert`.
2. Compute each node's **role**: an explicit `role` attribute if present and valid,
   otherwise the element's implicit role from the HTML-AAM mapping (which sometimes
   depends on context — a `<header>` is `banner` only when it isn't inside
   `article`/`section`/`main`...).
3. Prune nodes that don't matter for assistive tech: generic `div`s and `span`s with no
   role, name or interesting content are "ignored" and their children promoted. Images
   with `alt=""` get a `presentation` role and are dropped.
4. Compute **names and descriptions** with the accname algorithm.
5. Compute **states** from native state (checked, disabled, required, the `open` attribute
   on `<details>`...) and ARIA attributes.
6. Expose it through the platform API. Screen readers query the tree and listen for
   **events** (focus moved, value changed, children added) to know when to speak.

Because the tree depends on CSS, styling decisions are accessibility decisions. For
example, older browser versions dropped the semantics of elements given `display:
contents`, and `list-style: none` on a list causes Safari/VoiceOver to stop announcing it as a list
unless it's inside `<nav>` (a deliberate WebKit heuristic — add `role="list"` if the list
semantics matter).

## Common mistakes

- **Icon buttons and icon links with no name.**
- **Missing or file-name `alt` text.**
- **`aria-hidden="true"` on focusable elements**, or on a parent of them.
- **Hiding content with `opacity: 0` or off-screen positioning** when it should be gone
  for everyone.
- **Believing a clean automated scan means accessible.** Automated tools catch only part
  of the problem (published estimates vary widely); names that exist but are wrong, confusing focus order and unusable
  custom widgets need a human.
- **Treating accessibility as a final QA step** rather than a property of the markup you
  write from the start.

## Exercise

1. Open the accessibility tree for your Level 2 product page. Find every interactive
   element and write down its role and name. Are any names missing or confusing?
2. Build the ten-element naming example and inspect each one in devtools' Accessibility
   pane. Change `aria-labelledby` to point at a non-existent id and see what happens.
3. Turn on a screen reader (VoiceOver: <kbd>Cmd</kbd>+<kbd>F5</kbd> on macOS; NVDA is free
   on Windows; TalkBack on Android) and navigate a page by headings, then by links. Five
   minutes of this teaches more than any article.
4. Add a visually-hidden heading to a section that lacks one visually, and confirm it
   appears in the heading list of your screen reader or devtools.
