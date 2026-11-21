---
title: "Final Quiz"
description: "Test what you have learned across all four levels of HTML \u0026 CSS Mastery Path with a 19-question final quiz."
---

# Final Quiz

19 questions drawn from all four levels of this course. You need **70% (14 out of 19)** to pass —
you can retake it as many times as you like. Your best result is saved in this browser.

<div id="mp-quiz-root"></div>

<script type="application/json" id="mp-quiz-data">
[
  {
    "question": "With box-sizing: content-box, `.box { width: 200px; padding: 20px; border: 5px solid; }` renders how wide in total?",
    "options": [
      "200px",
      "225px",
      "210px",
      "250px"
    ],
    "correctIndex": 3,
    "explanation": "Content 200 + padding 2×20 + border 2×5 = 250px. With border-box it would be 200px."
  },
  {
    "question": "Two rules match one span: `#a .b { color: red }` and `.b.c.d { color: blue }`. The span has classes b c d inside #a. Which colour wins?",
    "options": [
      "Blue: it comes later",
      "Neither; they conflict",
      "Blue: three classes beat one ID",
      "Red: one ID outranks any number of classes"
    ],
    "correctIndex": 3,
    "explanation": "Specificity compares IDs first: (1,1,0) beats (0,3,0). Verified in the browser: the computed colour was the #a rule's."
  },
  {
    "question": "A 600px flex container holds three items with `flex: 1`, `flex: 2` and `flex: 3`. Their widths are:",
    "options": [
      "300px, 200px, 100px",
      "100px, 200px, 300px",
      "50px, 100px, 150px",
      "200px, 200px, 200px"
    ],
    "correctIndex": 1,
    "explanation": "flex: n means n 1 0%, so all 600px is free and is split 1:2:3."
  },
  {
    "question": "In a 300px flex container, item A is `width: 200px` (default flex-shrink: 1) and item B is `width: 200px; flex-shrink: 0`. Final widths?",
    "options": [
      "Both 200px",
      "A 200px, B 100px",
      "Both 150px",
      "A 100px, B 200px"
    ],
    "correctIndex": 3,
    "explanation": "B refuses to shrink, so A absorbs all the 100px overflow. Measured: [100, 200]."
  },
  {
    "question": "Two `flex: 1` items in a 300px container with `gap: 20px` are each:",
    "options": [
      "130px",
      "150px",
      "140px",
      "160px"
    ],
    "correctIndex": 2,
    "explanation": "300 − 20 = 280 shared equally = 140px each."
  },
  {
    "question": "A 1000px grid uses `repeat(auto-fit, minmax(200px, 1fr))` and has only two items. How wide is each?",
    "options": [
      "500px; empty tracks collapse to 0",
      "250px",
      "200px; tracks stay",
      "1000px"
    ],
    "correctIndex": 0,
    "explanation": "auto-fit collapses the three empty tracks, so the two items stretch: 500px 500px. With auto-fill they would stay 200px."
  },
  {
    "question": "A grid has `grid-template-columns: 200px 1fr 2fr; gap: 20px` in 1000px. How wide is the 2fr column?",
    "options": [
      "760px",
      "400px",
      "666.67px",
      "506.67px"
    ],
    "correctIndex": 3,
    "explanation": "1000 − 200 − 40 gaps = 760, split 1:2 → 253.33 and 506.67."
  },
  {
    "question": "`.badge { position: absolute; top: 0; right: 0 }` sits inside a card that has no positioned ancestor. Where does it land?",
    "options": [
      "Hidden",
      "Its normal position",
      "The top-right corner of the page (initial containing block)",
      "The card's top-right corner"
    ],
    "correctIndex": 2,
    "explanation": "Add position: relative to the card to anchor it to the card."
  },
  {
    "question": "For `<div style=\"font-size:20px\"><p style=\"font-size:1.5em; padding:2em\">`, the p's padding-top is:",
    "options": [
      "60px",
      "20px",
      "30px",
      "40px"
    ],
    "correctIndex": 0,
    "explanation": "em in font-size refers to the parent (20 → 30px), but em in padding refers to the element's own font-size: 2 × 30 = 60px. Measured."
  },
  {
    "question": "Why does `z-index: 9999` sometimes not lift an element above another?",
    "options": [
      "Its stacking context is trapped below the other one",
      "z-index needs px",
      "Max is 999",
      "It only works on grid"
    ],
    "correctIndex": 0,
    "explanation": "z-index only competes within the same stacking context."
  },
  {
    "question": "With :root{--space:1rem} and .card{--space:2rem}, what padding does .x{padding:var(--space)} get when .x is inside .card?",
    "options": [
      "2rem, because custom properties inherit and are overridden per subtree",
      "Invalid",
      "0",
      "1rem"
    ],
    "correctIndex": 0,
    "explanation": "Custom properties are inherited, so the nearest definition wins."
  },
  {
    "question": "A button is `<div role=\"button\">`. What's missing compared with `<button>`?",
    "options": [
      "Only styling",
      "Nothing",
      "Only the name",
      "Tab focus and Enter/Space activation; ARIA only changes what's announced"
    ],
    "correctIndex": 3,
    "explanation": "Measured in the lesson: 0 activations and not in the tab order."
  },
  {
    "question": "Which `<section>` becomes a region landmark?",
    "options": [
      "One with an accessible name (aria-labelledby or aria-label)",
      "Every section",
      "Only the first",
      "One inside nav"
    ],
    "correctIndex": 0,
    "explanation": "An unnamed section adds no landmark."
  },
  {
    "question": "A recipe form field needs a visible hint read after the label. What is the correct way?",
    "options": [
      "Placeholder only",
      "A <label for> plus a hint linked with aria-describedby",
      "Put the hint inside the label",
      "title attribute only"
    ],
    "correctIndex": 1,
    "explanation": "Placeholders vanish and have low contrast; describedby is read after the name."
  },
  {
    "question": "A card should switch to a two-column layout when its own slot is wide, regardless of the viewport. Which is right?",
    "options": [
      "@media (min-width: 30rem)",
      "@supports",
      "A JavaScript resize handler",
      "container-type: inline-size on the wrapper plus @container (width >= 30rem)"
    ],
    "correctIndex": 3,
    "explanation": "@container queries the nearest container ancestor, never the element itself."
  },
  {
    "question": "Which change is cheapest to animate (style then composite only)?",
    "options": [
      "font-size",
      "margin",
      "width",
      "transform and opacity"
    ],
    "correctIndex": 3,
    "explanation": "Layout properties force layout and paint every frame."
  },
  {
    "question": "Why prefer `padding-inline-start` over `padding-left` in a site that has Arabic pages?",
    "options": [
      "It is faster",
      "It works only in Arabic",
      "It follows the text direction, so one stylesheet works in LTR and RTL",
      "left is deprecated"
    ],
    "correctIndex": 2,
    "explanation": "Logical properties flip automatically with dir=\"rtl\"."
  },
  {
    "question": "Which pair of tokens follows the three-tier approach correctly?",
    "options": [
      "--color-brand holds raw hex only",
      "--button-bg → --orange-700 only",
      "All components use primitives",
      "--orange-700 (primitive) → --color-brand (semantic) → --button-bg (component)"
    ],
    "correctIndex": 3,
    "explanation": "Themes remap semantic tokens, so components should reference those."
  },
  {
    "question": "`.a { display:block; display:not-a-real-value }` in a browser that doesn't understand the second value. What happens?",
    "options": [
      "The unknown declaration is dropped and block stays",
      "An error appears",
      "The whole rule is dropped",
      "display: none"
    ],
    "correctIndex": 0,
    "explanation": "Unknown CSS is skipped declaration by declaration, which makes fallbacks trivial."
  }
]
</script>

## Coding challenges

Try these on your own computer, then compare your output. Nothing you write is sent anywhere.

### Proportional flex columns

A .row is a 600px flex container with three <i> items. Write CSS so the items are 100px, 200px and 300px wide, using flex-grow only (no widths). Assume `.row { display:flex; width:600px }` and items `.a`, `.b`, `.c`.

**Starter code**

```css
/* .a, .b, .c are children of .row (display:flex; width:600px) */
.a { }
.b { }
.c { }
```

**Expected output**

```text
Widths: 100px, 200px, 300px
```

<details><summary>Show solution and explanation</summary>

<pre><code>.a { flex: 1 1 0; }
.b { flex: 2 1 0; }
.c { flex: 3 1 0; }
</code></pre>

<p>With a basis of 0 all 600px is free space, shared 1:2:3. Verified by measuring the rendered widths: [100, 200, 300].</p>
</details>

### Auto-fitting gallery

Make `.gallery` a grid that fits as many columns of at least 200px as will fit and stretches them. In a 1000px container with two items, the items must each be 500px wide.

**Starter code**

```css
.gallery {
  display: grid;
  /* your columns here */
}
```

**Expected output**

```text
Computed columns: 500px 500px 0px 0px 0px (two items, 500px each)
```

<details><summary>Show solution and explanation</summary>

<pre><code>.gallery {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
}
</code></pre>

<p>auto-fit collapses the three empty tracks to 0px, so the two items stretch to 500px. Verified: getComputedStyle gave &quot;500px 500px 0px 0px 0px&quot;.</p>
</details>
