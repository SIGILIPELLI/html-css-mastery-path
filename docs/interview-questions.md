---
title: "HTML & CSS Interview Questions and Answers (179 Questions)"
description: "179 HTML & CSS interview questions with answers and explanations, from beginner to advanced, organised by level. Free, from the HTML & CSS Mastery Path."
---

# HTML & CSS Interview Questions and Answers

**179 questions** with the correct answer and a short explanation, organised from beginner (Level 1) to advanced (Level 4). Every question comes from a lesson in the free HTML & CSS Mastery Path, so you can open the lesson to go deeper.

!!! tip "Practise, don't just read"
    Each lesson also ends with an interactive quiz, and the [final quiz](quiz.md) tests all four levels. Try to answer each question below before reading the answer.

## Level 1 · Entry

### [01 · How Browsers Turn HTML &amp; CSS into Pixels](level-1/01-how-browsers-render.md)

**Q1.** What do HTML and CSS describe?

**Answer:** HTML describes what content is; CSS describes how matching things should look, and the browser works out layout and paints

They're descriptions, not programs; the browser decides the rest.

**Q2.** Why can the same page work on a phone, a 4K screen and a screen reader?

**Answer:** You give the browser information and it decides layout for the device

You're not controlling the browser, you're giving it information.

**Q3.** Which pair of files makes a working first page with no build step?

**Answer:** index.html and styles.css

Double-click index.html to open it; no server or dependencies needed.

**Q4.** Why learn to serve a folder over HTTP (e.g. python3 -m http.server)?

**Answer:** Some later features such as fetching JSON and certain loading rules behave differently on file:// URLs

Everything in Level 1 works on file://.

### [02 · Document Structure &amp; the Head Element](level-1/02-document-structure.md)

**Q5.** What does &lt;!doctype html&gt; do?

**Answer:** Signals standards mode; without it browsers switch to quirks mode

It's case-insensitive, and not a tag.

**Q6.** Why set &lt;html lang="en"&gt;?

**Answer:** Screen readers pick a pronunciation voice, hyphens: auto and translation tools rely on it

Use BCP 47 tags such as en, en-GB, pt-BR; you can override on any element.

**Q7.** Where must &lt;meta charset="utf-8"&gt; appear?

**Answer:** Within the first 1024 bytes, so put it first in &lt;head&gt;

The browser only looks that far before guessing.

**Q8.** What does &lt;head&gt; hold versus &lt;body&gt;?

**Answer:** &lt;head&gt; holds metadata about the document, mostly invisible; &lt;body&gt; holds the content

The parser would insert them anyway, but write them.

### [03 · Text, Links &amp; Semantic Meaning](level-1/03-text-links-semantics.md)

**Q9.** Why does semantic HTML matter?

**Answer:** Assistive technology can navigate it, search engines understand it and your CSS gets simpler

The accessibility tree for div soup is one undifferentiated run of text.

**Q10.** Which heading rules hold up in practice?

**Answer:** One h1 per page, don't skip levels going down, choose the level by structure not size

If an h2 should look small, style it small.

**Q11.** Why is &lt;span onclick&gt; 'Read more' a problem?

**Answer:** It isn't a link, so it can't be reached with the Tab key

Use &lt;a href&gt; for navigation.

**Q12.** How does HTML treat runs of whitespace?

**Answer:** Collapses any run of spaces, tabs and newlines into one space

Blank lines in source don't create paragraphs.

### [04 · Images &amp; Media](level-1/04-images-media.md)

**Q13.** What is the difference between alt="" and a missing alt?

**Answer:** Empty tells assistive tech to skip the image; missing makes many screen readers read the file name

Use alt="" for purely decorative images.

**Q14.** Why write width and height on every img?

**Answer:** The browser reserves space, preventing layout shift when the image arrives (CLS)

Combine with img { max-width: 100%; height: auto; }.

**Q15.** What should alt text for a chart say?

**Answer:** The takeaway, e.g. 'Sales doubled between March and June', with the data in text or a table nearby

Describe what the image communicates in context.

**Q16.** What should you avoid starting alt text with?

**Answer:** 'Image of…' or 'Picture of…', since the role is already announced

Screen readers already say 'image'.

### [05 · Tables](level-1/05-tables.md)

**Q17.** When is a &lt;table&gt; the right element?

**Answer:** When the content would make sense in a spreadsheet with meaningful row and column headers

CSS Grid and Flexbox do layout.

**Q18.** What does scope="row" / scope="col" on a th do?

**Answer:** Says which cells the header applies to, so a screen reader can announce 'Monday, Close, 17:00'

Without th cells it would announce just '17:00'.

**Q19.** Where must &lt;caption&gt; be and what does it become?

**Answer:** The first child of the table; its accessible name

It's the table's title.

**Q20.** What do colspan and rowspan do?

**Answer:** Merge cells across columns or rows

They produce spanning cells like a class schedule.

### [06 · Forms &amp; Native Validation](level-1/06-forms.md)

**Q21.** Which fields get submitted with a form?

**Answer:** Only those with a name attribute

A field without a name is not submitted.

**Q22.** When should you use method="get" versus "post"?

**Answer:** get for bookmarkable searches/filters (data in the URL); post for anything that changes data or includes secrets

post sends data in the request body.

**Q23.** What are the three jobs of a label?

**Answer:** The control's accessible name, a larger click/tap target and visible instructions

Associate with for= matching the input's id, or wrap the input.

**Q24.** Why are most broken forms broken?

**Answer:** Someone replaced the built-ins with custom controls and didn't rebuild keyboard, autofill, validation and submission

Native form elements handle all of that without JavaScript.

### [07 · CSS Syntax &amp; Selectors](level-1/07-css-selectors.md)

**Q25.** What happens with an invalid declaration versus an invalid selector?

**Answer:** An invalid declaration is ignored on its own; an invalid selector makes the whole rule ignored

Forgiving selectors are an exception.

**Q26.** What does p.note match?

**Answer:** &lt;p&gt; elements with class note (no space means the same element)

A space would make it a descendant selector.

**Q27.** Why do most teams avoid IDs for styling?

**Answer:** IDs have very high specificity and are unique per page

Classes are the main tool.

**Q28.** What does .card.featured match?

**Answer:** Elements with both classes

class holds a space-separated list.

### [08 · The Cascade, Specificity &amp; Inheritance](level-1/08-cascade-specificity.md)

**Q29.** In what order does the cascade compare criteria?

**Answer:** Origin/importance, context, style attribute, layers, specificity, then order of appearance

Day to day it's almost always specificity or order.

**Q30.** What does !important do to precedence?

**Answer:** It reverses origin order for important declarations and jumps above every normal author declaration regardless of specificity

In the test, the weakest selector with !important won.

**Q31.** Why does the origin reversal for !important exist?

**Answer:** So a user needing high contrast can always override the site

User !important beats author normal.

**Q32.** How many values does an element end up with per property?

**Answer:** Exactly one

The cascade picks the winner.

### [09 · The Box Model, Display &amp; Units](level-1/09-box-model-units.md)

**Q33.** What are the four areas of the CSS box, from inside out?

**Answer:** Content, padding, border, margin

Padding takes the background; margin is always transparent.

**Q34.** What does width mean with the default box-sizing: content-box?

**Answer:** The content width only; padding and border are added on top

border-box makes width include padding and border.

**Q35.** What does padding: 1rem 2rem 0.5rem 0 set?

**Answer:** top, right, bottom, left (clockwise)

Two values mean top/bottom and left/right.

**Q36.** What are logical properties like margin-inline and padding-block?

**Answer:** They follow the text direction: inline is the direction text runs, block is the direction paragraphs stack

In English they're left/right and top/bottom.

### [10 · Project — A Recipe Page](level-1/10-project-recipe-page.md)

**Q37.** Which landmarks does the recipe page need?

**Answer:** A site header with a labelled nav, one main, a footer and a skip link as the first focusable element

They allow quick navigation by assistive tech.

**Q38.** Which list element fits each job?

**Answer:** ul for ingredients, ol for steps, dl for facts like servings and timings

Choose the list by meaning.

**Q39.** How should the stylesheet be written?

**Answer:** One external stylesheet, no style attributes, rem units, border-box sizing and visible :focus-visible styles

The page must also pass html-validate and axe-core.

**Q40.** How should the data table behave on small screens?

**Answer:** It lives in a region that scrolls horizontally

It also needs a caption and th with scope.

## Level 2 · Intermediate

### [01 · Typography &amp; Web Fonts](level-2/01-typography.md)

**Q41.** Why end a font stack with a generic family?

**Answer:** So something sensible appears if no named font matches

Examples: serif, sans-serif, monospace, system-ui.

**Q42.** How does font fallback work?

**Answer:** Per character: glyphs missing from your web font come from the next font in the stack that has them

Important for multilingual sites.

**Q43.** What happens if you ask for italic but declare no italic face?

**Answer:** Browsers synthesise a slanted version, which looks worse than a real italic

Declare each face with @font-face.

**Q44.** Which format should you serve for web fonts, and what do variable fonts offer?

**Answer:** WOFF2; a continuous range of weights (and axes) in one file, e.g. font-weight: 650

Declare the range in @font-face.

### [02 · Color, Backgrounds &amp; Gradients](level-2/02-color-backgrounds.md)

**Q45.** Why is OKLCH better than HSL for palettes?

**Answer:** Its lightness matches perceived brightness, so shades look evenly spaced and contrast is predictable

HSL yellow and blue at the same L look very different in brightness.

**Q46.** What does color-mix(in oklch, var(--brand), black 15%) do?

**Answer:** Blends the brand colour with 15% black in OKLCH, handy for a hover shade

Relative colour syntax can also take a colour apart into channels.

**Q47.** What does rgb(179 64 26 / 50%) mean?

**Answer:** The colour at 50% alpha, in modern space-separated syntax

The comma and rgba() forms still work.

**Q48.** What responsibility comes with colour, per the lesson?

**Answer:** Contrast, which is easy to measure and often ignored

WCAG contrast ratios can be computed.

### [03 · Flexbox Fundamentals](level-2/03-flexbox-fundamentals.md)

**Q49.** With flex-direction: column, what does justify-content: center do?

**Answer:** Centres items vertically, because it works along the main axis

Alignment properties follow the axes, not horizontal and vertical.

**Q50.** Three 100px items in a 600px flex container with justify-content: space-between have left offsets of:

**Answer:** 0, 250, 500

First and last touch the edges, with equal gaps between.

**Q51.** Which elements become flex items when a container has display: flex?

**Answer:** Only its direct children

Grandchildren decide their own layout.

**Q52.** When does justify-content have no visible effect?

**Answer:** When items have flex-grow and absorb all the free space

It distributes leftover space only.

### [04 · Flexbox in Depth: How Sizes Are Resolved](level-2/04-flexbox-in-depth.md)

**Q53.** What does flex: 1 expand to?

**Answer:** 1 1 0%, ignoring content size

The basis of 0 is why items end up equal.

**Q54.** A 600px container holds items with flex: 1 1 100px and flex: 2 1 200px. Their widths are:

**Answer:** 200px and 400px

Free space is 300, split 1:2 and added to the bases: 100+100 and 200+200.

**Q55.** What is the default flex value (flex: initial)?

**Answer:** 0 1 auto: content-sized, can shrink, won't grow

flex: none (0 0 auto) is rigid.

**Q56.** How do three items with differing text behave under flex: 1 versus flex: auto in 600px?

**Answer:** flex: 1 gives equal widths of 200 each; flex: auto gives unequal widths based on content

flex: auto starts from content size and shares only the extra space.

### [05 · CSS Grid Fundamentals](level-2/05-grid-fundamentals.md)

**Q57.** In a 1000px grid with columns 200px 1fr 2fr and a 20px gap, how wide is the 2fr column?

**Answer:** 506.67px

1000−200−40 = 760, split 1:2 gives 253.33 and 506.67.

**Q58.** What is the 1fr minimum trap?

**Answer:** 1fr means minmax(auto, 1fr), so a track won't shrink below its content's minimum size

Use minmax(0, 1fr) for truly equal columns.

**Q59.** What does fr measure?

**Answer:** A fraction of the leftover space after fixed tracks and gaps

Like flex-grow, it shares free space.

**Q60.** When should you use Grid rather than Flexbox?

**Answer:** For two-dimensional layouts where rows and columns line up

Flexbox arranges items along one line.

### [06 · Grid in Depth: auto-fit, Implicit Tracks &amp; Subgrid](level-2/06-grid-in-depth.md)

**Q61.** With only two items in a wide container, how do auto-fit and auto-fill differ?

**Answer:** auto-fit collapses empty tracks so the items stretch; auto-fill keeps empty tracks so items stay column-width

At 700px, auto-fit gave 342px items while auto-fill gave about 222px.

**Q62.** What does repeat(auto-fit, minmax(200px, 1fr)) do?

**Answer:** Makes as many columns as fit, each at least 200px, stretched equally, with no media queries

The count changes with container width.

**Q63.** Why write minmax(min(200px, 100%), 1fr)?

**Answer:** It stops overflow when the container is narrower than 200px

min() caps the minimum at the container width.

**Q64.** When should you prefer auto-fill?

**Answer:** When items should keep a consistent size even if the last row is half empty

Choose auto-fit when few items should look deliberate.

### [07 · Positioning &amp; Stacking Contexts](level-2/07-positioning-stacking.md)

**Q65.** Relative to what is an absolutely positioned element's top/right offset measured?

**Answer:** Its containing block: the padding box of the nearest positioned ancestor, else the initial containing block

In the lesson's test the badge landed at the page corner (1260, 0) with no positioned ancestor, and at the card corner (450, 260) once the card had position: relative.

**Q66.** What is the main job of position: relative on its own?

**Answer:** To act as an anchor for absolute descendants and let z-index apply, since its space is kept

It moves the element visually without affecting its neighbours.

**Q67.** What does position: sticky do?

**Answer:** Stays in flow and sticks within its scroll container once a threshold is crossed, until its parent scrolls away

Unlike fixed, it remains in normal flow.

**Q68.** What does inset: 0 mean?

**Answer:** top: 0; right: 0; bottom: 0; left: 0

It's the shorthand for all four offsets.

### [08 · Responsive Design &amp; Media Queries](level-2/08-responsive-design.md)

**Q69.** What does mobile-first mean?

**Answer:** Base styles target the narrowest screen, then (width &gt;= …) queries add complexity as space allows

The lesson's grid goes from one to two to three columns at 40rem and 64rem.

**Q70.** Why does range syntax (width &lt; 48rem and width &gt;= 48rem) avoid an overlap?

**Answer:** max-width: 48rem includes 48rem itself, so pairing it with min-width: 48rem overlaps at exactly 48rem

Range syntax makes the boundary explicit.

**Q71.** When are media queries best used?

**Answer:** For genuine layout changes, such as collapsing navigation or moving a sidebar, after letting the layout flow with min(), clamp() and auto-fit

Fluid techniques adapt continuously without breakpoints.

**Q72.** What does width: min(100% - 2rem, 70rem) with margin-inline: auto give?

**Answer:** Gutters plus a maximum width, centred

One line replaces a breakpoint.

### [09 · Custom Properties &amp; Theming](level-2/09-custom-properties.md)

**Q73.** With :root { --space: 1rem } and .card { --space: 2rem }, what padding does .x { padding: var(--space) } get inside a .card?

**Answer:** 32px, because custom properties inherit and are overridden per subtree

The lesson measured 16px at top level and 32px inside .card.

**Q74.** What does var(--missing, rgb(0, 0, 255)) compute to?

**Answer:** rgb(0, 0, 255), the fallback

The fallback is used when the property is undefined.

**Q75.** Are custom property names case-sensitive?

**Answer:** Yes, --Brand and --brand differ

Names also start with two dashes.

**Q76.** Why are custom properties more than preprocessor variables?

**Answer:** They live in the browser, cascade, inherit, change per component or media query, and JavaScript can read and write them

That makes them the base of theming.

### [10 · Project — A Responsive Product Page](level-2/10-project-product-page.md)

**Q77.** How is the specifications list built?

**Answer:** A dl in an auto-fit grid with no media queries

Requirement 3 of the project.

**Q78.** What does subgrid do for the related-product cards?

**Answer:** Lines up titles, descriptions and prices across each row

Used inside an auto-fill grid.

**Q79.** How should size options be built?

**Answer:** Real radio buttons in a fieldset, visually styled as buttons

This keeps native keyboard and screen-reader behaviour.

**Q80.** How is the dark theme achieved?

**Answer:** One set of semantic custom properties plus color-scheme so form controls follow

Motion is also limited to users who haven't asked for reduced motion.

## Level 3 · Advanced

### [01 · Accessibility Foundations &amp; the Accessibility Tree](level-3/01-accessibility-foundations.md)

**Q81.** What do the four WCAG principles (POUR) stand for?

**Answer:** Perceivable, Operable, Understandable, Robust

WCAG 2.2 is current and each version is backwards compatible.

**Q82.** Roughly what share of people worldwide live with a significant disability, per the WHO figure cited?

**Answer:** About one in six

Nearly everyone also has temporary or situational limits.

**Q83.** For HTML and CSS authors, accessibility is mostly:

**Answer:** A consequence of choices like semantic elements, labels, contrast, focus styles and reflow

Levels 1 and 2 choices already carry most of it.

**Q84.** Which problem especially affects people with colour vision deficiency?

**Answer:** Information conveyed by colour alone

They need no special tools; they just see colours differently.

### [02 · Landmarks, Headings &amp; Navigation](level-3/02-landmarks-headings.md)

**Q85.** When does a &lt;header&gt; become a banner landmark?

**Answer:** When it is not inside article, aside, main, nav or section

In the lesson's tree, the article's header and footer did not become landmarks.

**Q86.** When does a &lt;section&gt; become a region landmark?

**Answer:** When it has an accessible name (aria-label or aria-labelledby)

The unnamed section produced no region; the named one did.

**Q87.** Which landmarks are always exposed regardless of context?

**Answer:** nav, main and search

A form is a landmark only when named.

**Q88.** How do most screen-reader users primarily navigate a page?

**Answer:** By headings, with landmarks to skip whole regions

Both are cheap to get right and have a huge payoff.

### [03 · Keyboard Access &amp; Focus Management](level-3/03-keyboard-focus.md)

**Q89.** Which elements are skipped by Tab in the lesson's test?

**Answer:** div with onclick, a without href, disabled button and tabindex=-1 element

Adding onclick doesn't make anything focusable.

**Q90.** Why did the input with tabindex="3" receive focus first?

**Answer:** Positive tabindex values jump ahead of the natural order

It was fifth in the source; positive values are best avoided.

**Q91.** Which keys activate links and buttons?

**Answer:** Links with Enter only; buttons with Enter and Space

A reason not to fake one with the other.

**Q92.** What is the WCAG 2.1.1 rule?

**Answer:** Everything a mouse user can do, a keyboard user must be able to do

It's also the easiest requirement to test yourself.

### [04 · ARIA: When and How to Use It](level-3/04-aria.md)

**Q93.** What does ARIA change?

**Answer:** Only what's announced in the accessibility tree, never how anything behaves

So a widget role obliges you to implement the keyboard behaviour.

**Q94.** In the lesson's test, what happened with &lt;div role="button"&gt;?

**Answer:** It was not reached by Tab and had 0 activations from Enter and Space, while a real button had 2

Tab order was ["real", "body"].

**Q95.** What is the first rule of ARIA use?

**Answer:** If a native HTML element does the job, use it

A button beats div role=button every time.

**Q96.** Which of the five rules concerns focusable elements?

**Answer:** Don't use role=presentation or aria-hidden=true on a focusable element

Rule 4; rule 5 requires an accessible name on every interactive element.

### [05 · Accessible Forms &amp; Error Handling](level-3/05-accessible-forms.md)

**Q97.** What did axe-core miss in the bad form but is the worst problem?

**Answer:** The div with a click handler as 'Send', which no keyboard user can use to submit

Automated checks are a floor, not a ceiling.

**Q98.** How should a hint be connected to its input?

**Answer:** Outside the label, linked with aria-describedby

It's then read after the name.

**Q99.** Why is placeholder a poor label?

**Answer:** It vanishes on input and has low contrast by default

It passed axe only as a fallback accessible name.

**Q100.** How should required fields be marked?

**Answer:** In text, not just a red asterisk; if most are required say so and mark the optional ones

The required attribute also exposes the state to assistive technology.

### [06 · Container Queries](level-3/06-container-queries.md)

**Q101.** What does an @container rule query?

**Answer:** The nearest ancestor that is a query container, never the element itself

An element's styles can't depend on its own size, or it could loop forever.

**Q102.** In the lesson's test, a card inside a 300px container versus an 800px container produced:

**Answer:** One column at 300px; "160px 624px" (image plus text) at 800px

The same component and CSS gave two layouts, decided by the slot.

**Q103.** What happens when the card has no container ancestor?

**Answer:** The @container rule never matches and the base styles apply as the fallback

The third card measured one 800px column.

**Q104.** Which container-type should you almost always use?

**Answer:** inline-size

container-type: size on a plain div with a paragraph measured 0px tall, because size containment ignores the children's height.

### [07 · Modern Selectors, Nesting &amp; Cascade Layers](level-3/07-modern-selectors-layers.md)

**Q105.** How do :is() and :where() differ?

**Answer:** Specificity: :is() takes its most specific argument's; :where() is always zero

:where() suits resets and library defaults that any single class can override.

**Q106.** What does "forgiving" selector list mean?

**Answer:** Invalid arguments in :is()/:where() are dropped and the rest still work, whereas a plain list is discarded entirely

The lesson's test: p, p:unknown-thing was discarded (black), but :is(h1, h1:unknown-thing) still made h1 blue.

**Q107.** What does .card:has(img) select?

**Answer:** Cards that contain an image

It's more general than a 'parent selector': h2:has(+ p) matches by a following sibling.

**Q108.** Which three needs used to require a preprocessor or JavaScript but are now native?

**Answer:** Selecting by contents (:has), nesting, and controlling which stylesheet wins (@layer)

Covered as :has(), CSS nesting and cascade layers.

### [08 · Transitions &amp; Animations](level-3/08-transitions-animations.md)

**Q109.** Why list transition properties explicitly instead of transition: all?

**Answer:** all animates unintended changes (theme switches, resizes) and makes debugging harder

Shorthand order is property duration timing-function delay.

**Q110.** Which timing function suits things entering the screen, and which things leaving?

**Answer:** ease-out for entering, ease-in for leaving

linear suits spinners and progress; ease-in-out suits movement between positions.

**Q111.** What are typical durations for UI feedback?

**Answer:** 100–200ms for hovers and presses, 200–400ms for moves across the screen

Longer feels sluggish.

**Q112.** Why respect the reduced-motion media query?

**Answer:** Motion can cause dizziness and nausea for people with vestibular disorders

The media query lets people who want less motion get it.

### [09 · Native Interactive Elements: details, dialog, popover](level-3/09-native-interactive-elements.md)

**Q113.** What happens when several &lt;details&gt; elements share the same name attribute?

**Answer:** Opening one closes the others, making an exclusive accordion

The lesson's test gave [false, true] after opening the first, then the second. Use it sparingly, because people often want to compare answers.

**Q114.** What happens to closed &lt;details&gt; content in the accessibility tree?

**Answer:** It is absent, not just visually hidden

In Chromium it is still findable with find-in-page, which opens the details automatically.

**Q115.** Which interactions toggle a &lt;summary&gt;?

**Answer:** Click, Enter or Space

It's focusable, and a toggle event fires if you need script.

**Q116.** Why did JavaScript libraries for disclosures and modals often have bugs?

**Answer:** Focus management, Escape handling, stacking and ARIA state are hard to get right

Native elements are keyboard- and screen-reader-friendly with little or no script.

### [10 · Project — An Accessible Component Gallery](level-3/10-project-component-gallery.md)

**Q117.** How should the tabs component behave for the keyboard?

**Answer:** One Tab stop for the tab list, arrow keys and Home/End move between tabs, panels labelled by their tabs

It follows the WAI-ARIA Authoring Practices pattern.

**Q118.** How does the 'Save recipe' toggle expose its state?

**Answer:** aria-pressed, with a status message announced through a live region

Requirement 2.

**Q119.** How is the confirmation dialog built and opened?

**Answer:** With &lt;dialog&gt; and showModal(), reporting what the user chose

Requirement 4.

**Q120.** How was the reference solution tested?

**Answer:** Validated, audited with axe-core in both colour schemes at three widths, and driven entirely from the keyboard by a Playwright script

The dialog and popover were open during the audit.

## Level 4 · Master

### [01 · The Rendering Pipeline &amp; CSS Performance](level-4/01-rendering-performance.md)

**Q121.** About how much time does the browser have per frame at 60 fps?

**Answer:** About 16.7ms

This budget covers JavaScript, style, layout, paint and composite.

**Q122.** Which stages run when you change transform or opacity on an element with its own layer?

**Answer:** Style then composite only

That's why animating transform and opacity is recommended. Changing width runs layout, paint and composite.

**Q123.** What is layout thrashing?

**Answer:** Alternating style writes and geometry reads, like offsetWidth, which forces a synchronous layout each iteration

Normally layout happens once per frame, lazily.

**Q124.** Which stage runs for changes to color, background or box-shadow, but not layout?

**Answer:** Style, paint, composite

Geometry changes (width, margin, font-size) additionally run layout.

### [02 · Loading Performance: Critical CSS, Fonts &amp; Layout Shift](level-4/02-loading-performance.md)

**Q125.** What are the 'good' thresholds for LCP and CLS?

**Answer:** LCP ≤ 2.5 s and CLS ≤ 0.1

Measured at the 75th percentile of real visits; INP is mostly about JavaScript and rendering work.

**Q126.** In the lesson's test with a 1.5 s delayed stylesheet, what was first-contentful-paint with a blocking &lt;link&gt;?

**Answer:** About 1,530 ms

The text stayed invisible while the CSS was in flight; the media="print" + onload trick painted in about 16–24 ms.

**Q127.** Why is the media="print" onload trick not a fix by itself?

**Answer:** Repainting with different styles 1.5 s later is a worse experience than waiting

The real technique combines it with inlined critical CSS.

**Q128.** Why is CSS render-blocking?

**Answer:** Otherwise the browser would paint unstyled content and then restyle it

It avoids a flash of unstyled content.

### [03 · CSS Architecture at Scale](level-4/03-css-architecture.md)

**Q129.** In BEM, what do the parts of recipe-card--featured and recipe-card__title mean?

**Answer:** Block recipe-card; --featured is a modifier (variant), __title is an element (part)

Selectors are single classes, so specificity stays flat at (0,1,0).

**Q130.** Which are the four problems every CSS methodology addresses?

**Answer:** Global scope, specificity escalation, dead code, coupling to markup structure

For example, .sidebar ul li a breaks when someone adds a div.

**Q131.** Why does BEM allow safe deletion?

**Answer:** When no HTML uses recipe-card, all its CSS can go

The cost is verbose class names.

**Q132.** What does utility-first CSS do?

**Answer:** Provides small single-purpose classes composed in the HTML, so the stylesheet stops growing

Examples: flex, gap-4, p-4, rounded-lg.

### [04 · Design Tokens &amp; Theming Systems](level-4/04-design-tokens.md)

**Q133.** What are the three tiers of tokens?

**Answer:** Primitive, semantic, component

Each tier refers only to the tier below.

**Q134.** Why should components use semantic tokens rather than primitives like --orange-700?

**Answer:** Otherwise a dark theme can't fix it without editing the component

Themes remap semantic tokens.

**Q135.** What do component tokens such as --button-bg provide?

**Answer:** Customisation points without leaking internals; variants become one-line overrides

For example, .button--danger { --button-bg: var(--color-danger); }.

**Q136.** Why are custom properties the natural way to implement tokens?

**Answer:** They cascade, inherit, can be redefined per theme or subtree, and are readable from JavaScript

A token is a named design decision.

### [05 · Logical Properties, Writing Modes &amp; Internationalization](level-4/05-logical-properties-i18n.md)

**Q137.** Why set direction with the dir attribute rather than the CSS direction property?

**Answer:** Direction is content meaning (column order, bidi text, assistive tech), and markup is where meaning lives

dir accepts ltr, rtl or auto.

**Q138.** What does margin-inline-start refer to?

**Answer:** The start of the inline axis: left in English, right in Arabic

One stylesheet then works for every direction and writing mode.

**Q139.** Which pair maps physical to logical correctly?

**Answer:** width → inline-size, text-align: left → text-align: start

height maps to block-size in horizontal writing modes.

**Q140.** What does dir="auto" do?

**Answer:** Decides direction from the first strong character in the content, useful for user-generated text

lang separately sets pronunciation, hyphenation and font selection.

### [06 · Metadata, SEO &amp; Structured Data](level-4/06-metadata-seo.md)

**Q141.** What is the purpose of rel="canonical"?

**Answer:** Tell search engines which of several URLs for the same content is the real one

Use an absolute URL; the canonical must return 200 and not be blocked.

**Q142.** Is the meta description a ranking lever?

**Answer:** No: it's your pitch in the results, and search engines may write their own snippet

The lesson stresses that ranking weights are proprietary and anyone claiming exact weights is guessing.

**Q143.** What HTML-level things genuinely help search and people alike?

**Answer:** Descriptive title and single matching h1, headings, real text and alt, descriptive link text, fast stable pages, semantic HTML

The foundation is the content.

**Q144.** What is good practice for titles?

**Answer:** Unique per page, specific, most important words first, since results truncate by pixel width

Front-load the important words.

### [07 · Print Styles &amp; Other Media](level-4/07-print-styles.md)

**Q145.** Why is &lt;link rel="stylesheet" href="print.css" media="print"&gt; not render-blocking for the screen?

**Answer:** A print stylesheet isn't needed to render the screen

Print rules otherwise live in @media print.

**Q146.** How does a print stylesheet make links useful on paper?

**Answer:** a[href^="http"]::after { content: " (" attr(href) ")"; }

Fragment and javascript: links get empty content.

**Q147.** What do break-after: avoid on headings and break-inside: avoid on figures and tables do?

**Answer:** Keep headings with what follows and keep those blocks from splitting across pages

orphans and widows of 3 keep at least 3 lines at page top or bottom.

**Q148.** How big is 12pt in CSS pixels?

**Answer:** 16px, the same as the screen default

pt, cm, mm and in are the natural units for paper.

### [08 · Progressive Enhancement &amp; Browser Support](level-4/08-progressive-enhancement.md)

**Q149.** What does a browser do with a declaration it doesn't understand?

**Answer:** Skips just that declaration; unknown selectors skip the rule, unknown at-rules skip the block

That makes fallbacks trivial: declare the safe value first, the new one second.

**Q150.** For .b { background: rgb(200,0,0); background: color-mix(in oklch, red, blue); } what does an old browser without color-mix() render?

**Answer:** The red fallback

Each browser keeps the last declaration it understands.

**Q151.** Which case does the fallback-first pattern not protect?

**Answer:** Declarations using var(), since they can't be checked at parse time

A comma-separated selector list is also all-or-nothing unless you use :is() or :where().

**Q152.** What is progressive enhancement?

**Answer:** Build a baseline that works everywhere, then layer improvements for browsers that support them

It lets you use new features without breaking the page for others.

### [09 · Testing &amp; Auditing HTML and CSS](level-4/09-testing-auditing.md)

**Q153.** Why must HTML and CSS be checked automatically on every change?

**Answer:** They fail silently: browsers repair invalid markup and ignore invalid declarations without errors

A typo, missing label or skipped heading level produces no error.

**Q154.** Which tool catches missing names, contrast and ARIA misuse automatically?

**Answer:** axe-core

Stylelint checks CSS correctness; html-validate checks markup.

**Q155.** What can only manual review judge?

**Answer:** Meaningful alt text and sensible reading order, using a screen reader and keyboard

Automation is a floor, not a ceiling.

**Q156.** For pages built by JavaScript, what should you validate?

**Answer:** The rendered DOM, e.g. page.content() in Playwright

Validators check the source file by default.

### [10 · Capstone — A Conference Website](level-4/10-capstone.md)

**Q157.** What are the cascade layers in the capstone, in order?

**Answer:** reset, tokens, base, layout, components, utilities, print

Later layers win over earlier ones.

**Q158.** How is the speakers section built?

**Answer:** An auto-fit grid of container-query cards

Requirement 6.

**Q159.** How must the registration form handle errors?

**Answer:** Inline errors plus an error summary, and a status message on success

Requirement 7.

**Q160.** What does the quality pipeline include?

**Answer:** html-validate, Stylelint with a specificity budget, axe-core in light and dark at 360/800/1280px, and Playwright in two engines

The fictional event page says so openly.

## Mixed questions across all levels

**Q161.** With box-sizing: content-box, `.box { width: 200px; padding: 20px; border: 5px solid; }` renders how wide in total?

**Answer:** 250px

Content 200 + padding 2×20 + border 2×5 = 250px. With border-box it would be 200px.

**Q162.** Two rules match one span: `#a .b { color: red }` and `.b.c.d { color: blue }`. The span has classes b c d inside #a. Which colour wins?

**Answer:** Red: one ID outranks any number of classes

Specificity compares IDs first: (1,1,0) beats (0,3,0). Verified in the browser: the computed colour was the #a rule's.

**Q163.** A 600px flex container holds three items with `flex: 1`, `flex: 2` and `flex: 3`. Their widths are:

**Answer:** 100px, 200px, 300px

flex: n means n 1 0%, so all 600px is free and is split 1:2:3.

**Q164.** In a 300px flex container, item A is `width: 200px` (default flex-shrink: 1) and item B is `width: 200px; flex-shrink: 0`. Final widths?

**Answer:** A 100px, B 200px

B refuses to shrink, so A absorbs all the 100px overflow. Measured: [100, 200].

**Q165.** Two `flex: 1` items in a 300px container with `gap: 20px` are each:

**Answer:** 140px

300 − 20 = 280 shared equally = 140px each.

**Q166.** A 1000px grid uses `repeat(auto-fit, minmax(200px, 1fr))` and has only two items. How wide is each?

**Answer:** 500px; empty tracks collapse to 0

auto-fit collapses the three empty tracks, so the two items stretch: 500px 500px. With auto-fill they would stay 200px.

**Q167.** A grid has `grid-template-columns: 200px 1fr 2fr; gap: 20px` in 1000px. How wide is the 2fr column?

**Answer:** 506.67px

1000 − 200 − 40 gaps = 760, split 1:2 → 253.33 and 506.67.

**Q168.** `.badge { position: absolute; top: 0; right: 0 }` sits inside a card that has no positioned ancestor. Where does it land?

**Answer:** The top-right corner of the page (initial containing block)

Add position: relative to the card to anchor it to the card.

**Q169.** For `<div style="font-size:20px"><p style="font-size:1.5em; padding:2em">`, the p's padding-top is:

**Answer:** 60px

em in font-size refers to the parent (20 → 30px), but em in padding refers to the element's own font-size: 2 × 30 = 60px. Measured.

**Q170.** Why does `z-index: 9999` sometimes not lift an element above another?

**Answer:** Its stacking context is trapped below the other one

z-index only competes within the same stacking context.

**Q171.** With :root{--space:1rem} and .card{--space:2rem}, what padding does .x{padding:var(--space)} get when .x is inside .card?

**Answer:** 2rem, because custom properties inherit and are overridden per subtree

Custom properties are inherited, so the nearest definition wins.

**Q172.** A button is `<div role="button">`. What's missing compared with `<button>`?

**Answer:** Tab focus and Enter/Space activation; ARIA only changes what's announced

Measured in the lesson: 0 activations and not in the tab order.

**Q173.** Which `<section>` becomes a region landmark?

**Answer:** One with an accessible name (aria-labelledby or aria-label)

An unnamed section adds no landmark.

**Q174.** A recipe form field needs a visible hint read after the label. What is the correct way?

**Answer:** A &lt;label for&gt; plus a hint linked with aria-describedby

Placeholders vanish and have low contrast; describedby is read after the name.

**Q175.** A card should switch to a two-column layout when its own slot is wide, regardless of the viewport. Which is right?

**Answer:** container-type: inline-size on the wrapper plus @container (width &gt;= 30rem)

@container queries the nearest container ancestor, never the element itself.

**Q176.** Which change is cheapest to animate (style then composite only)?

**Answer:** transform and opacity

Layout properties force layout and paint every frame.

**Q177.** Why prefer `padding-inline-start` over `padding-left` in a site that has Arabic pages?

**Answer:** It follows the text direction, so one stylesheet works in LTR and RTL

Logical properties flip automatically with dir="rtl".

**Q178.** Which pair of tokens follows the three-tier approach correctly?

**Answer:** --orange-700 (primitive) → --color-brand (semantic) → --button-bg (component)

Themes remap semantic tokens, so components should reference those.

**Q179.** `.a { display:block; display:not-a-real-value }` in a browser that doesn't understand the second value. What happens?

**Answer:** The unknown declaration is dropped and block stays

Unknown CSS is skipped declaration by declaration, which makes fallbacks trivial.

## Practical coding questions

### Proportional flex columns

A .row is a 600px flex container with three &lt;i&gt; items. Write CSS so the items are 100px, 200px and 300px wide, using flex-grow only (no widths). Assume `.row { display:flex; width:600px }` and items `.a`, `.b`, `.c`.

Starter code:

```
/* .a, .b, .c are children of .row (display:flex; width:600px) */
.a { }
.b { }
.c { }
```

Expected output:

```text
Widths: 100px, 200px, 300px
```

??? success "Show a solution"

    ```
    .a { flex: 1 1 0; }
    .b { flex: 2 1 0; }
    .c { flex: 3 1 0; }
    ```

    With a basis of 0 all 600px is free space, shared 1:2:3. Verified by measuring the rendered widths: [100, 200, 300].

### Auto-fitting gallery

Make `.gallery` a grid that fits as many columns of at least 200px as will fit and stretches them. In a 1000px container with two items, the items must each be 500px wide.

Starter code:

```
.gallery {
  display: grid;
  /* your columns here */
}
```

Expected output:

```text
Computed columns: 500px 500px 0px 0px 0px (two items, 500px each)
```

??? success "Show a solution"

    ```
    .gallery {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
    }
    ```

    auto-fit collapses the three empty tracks to 0px, so the two items stretch to 500px. Verified: getComputedStyle gave "500px 500px 0px 0px 0px".

---

More free practice: the [HTML & CSS Mastery Path](index.md) curriculum, the [Interview Prep Pack](https://learnmasterypath.com/interview-pack/) and the [free resume ATS check](https://learnmasterypath.com/resume/).
