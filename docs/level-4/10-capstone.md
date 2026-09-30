---
description: "Capstone: a tested conference website with tokens, layers, container queries, accessible forms, theming, RTL, print and a test suite."
---

# 10 · Capstone — A Conference Website

The capstone brings the whole course together in one realistic page: a website for a
one-day conference, **Frontend Commons 2026**. (The event, speakers and venue are
fictional — it's an exercise, and the page says so.) It has a sticky header, a hero,
a schedule table, speaker cards, a registration form, an FAQ, a theme toggle, print
styles, metadata and structured data — and, just as importantly, a **quality pipeline**
that proves it works: HTML validation, CSS linting, automated accessibility audits in both
colour schemes and three viewport widths, and a Playwright suite that runs in Chromium and
WebKit.

## Brief

Build a single-page conference site that:

1. Uses semantic HTML throughout, with one `h1`, a logical heading outline, labelled
   landmarks, and a skip link.
2. Has a complete `<head>`: specific title and description, canonical URL, Open Graph
   tags and an `Event` JSON-LD block that matches the visible content.
3. Is organised as **cascade layers** (`reset, tokens, base, layout, components,
   utilities, print`) with **three tiers of tokens**, fluid type and spacing scales.
4. Uses **logical properties** throughout, so it works right-to-left.
5. Presents the schedule as an accessible table that scrolls on narrow screens.
6. Shows speakers in an `auto-fit` grid of **container-query** cards.
7. Has a registration form with **inline errors and an error summary**, and a status
   message on success.
8. Supports the OS colour scheme **and** a user toggle that persists, with no flash.
9. Supports forced colours, reduced motion and print.
10. Passes html-validate, Stylelint (with a specificity budget), axe-core in light and dark
    at 360/800/1280px, and a Playwright suite in two engines.

## The HTML

```html title="index.html"
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Frontend Commons 2026 — HTML, CSS &amp; Accessibility Conference</title>
    <meta name="description" content="Frontend Commons is a one-day, single-track conference about HTML, CSS and accessibility: schedule, speakers, tickets and venue details.">
    <link rel="canonical" href="https://example.com/frontend-commons-2026/">
    <meta property="og:type" content="website">
    <meta property="og:title" content="Frontend Commons 2026">
    <meta property="og:description" content="One day, one track: HTML, CSS and accessibility.">
    <meta property="og:url" content="https://example.com/frontend-commons-2026/">
    <meta property="og:image" content="https://example.com/frontend-commons-2026/og.png">
    <meta name="twitter:card" content="summary_large_image">
    <script>
      try { const t = localStorage.getItem('theme'); if (t) document.documentElement.dataset.theme = t; } catch {}
    </script>
    <link rel="stylesheet" href="styles.css">
    <script src="site.js" defer></script>
    <script type="application/ld+json">
      {
        "@context": "https://schema.org",
        "@type": "Event",
        "name": "Frontend Commons 2026",
        "description": "A one-day, single-track conference about HTML, CSS and accessibility.",
        "startDate": "2026-11-21T09:00+05:30",
        "endDate": "2026-11-21T17:30+05:30",
        "eventAttendanceMode": "https://schema.org/OfflineEventAttendanceMode",
        "eventStatus": "https://schema.org/EventScheduled",
        "location": {
          "@type": "Place",
          "name": "Example Convention Hall",
          "address": { "@type": "PostalAddress", "addressLocality": "Hyderabad", "addressCountry": "IN" }
        },
        "organizer": { "@type": "Organization", "name": "Frontend Commons (course exercise)", "url": "https://example.com/" }
      }
    </script>
  </head>
  <body>
    <a class="skip-link" href="#main">Skip to content</a>

    <header class="site-header">
      <div class="container site-header__inner">
        <a class="brand" href="#top">Frontend <span>Commons</span></a>
        <nav aria-label="Main" class="site-nav">
          <ul>
            <li><a href="#schedule">Schedule</a></li>
            <li><a href="#speakers">Speakers</a></li>
            <li><a href="#tickets">Tickets</a></li>
            <li><a href="#faq">FAQ</a></li>
          </ul>
        </nav>
        <button type="button" class="theme-toggle" aria-pressed="false">Dark theme</button>
      </div>
    </header>

    <main id="main" tabindex="-1">
      <section class="hero" id="top" aria-labelledby="hero-title">
        <div class="container hero__inner">
          <h1 id="hero-title">Frontend Commons 2026</h1>
          <p class="hero__lede">One day. One track. Talks about the parts of the web platform that
            every site depends on — HTML, CSS and accessibility.</p>
          <dl class="hero__facts">
            <div><dt>Date</dt><dd><time datetime="2026-11-21">Saturday 21 November 2026</time></dd></div>
            <div><dt>Where</dt><dd>Hyderabad, India</dd></div>
            <div><dt>Format</dt><dd>In person, talks captioned live</dd></div>
          </dl>
          <p><a class="button" href="#tickets">Get a ticket</a></p>
        </div>
      </section>

      <section id="schedule" class="container section" aria-labelledby="schedule-title">
        <h2 id="schedule-title">Schedule</h2>
        <p>All times are India Standard Time (UTC+05:30).</p>
        <section class="table-scroll" aria-labelledby="schedule-caption" tabindex="0">
          <table class="schedule">
            <caption id="schedule-caption">Talks on 21 November</caption>
            <thead>
              <tr><th scope="col">Time</th><th scope="col">Session</th><th scope="col">Speaker</th></tr>
            </thead>
            <tbody>
              <tr><th scope="row"><time datetime="09:00">09:00</time></th><td>Registration and breakfast</td><td>—</td></tr>
              <tr><th scope="row"><time datetime="10:00">10:00</time></th><td>The cascade is a feature</td><td>Anika Sharma</td></tr>
              <tr><th scope="row"><time datetime="11:00">11:00</time></th><td>Forms nobody abandons</td><td>Rahul Menon</td></tr>
              <tr><th scope="row"><time datetime="12:00">12:00</time></th><td>Container queries in production</td><td>Leela Varma</td></tr>
              <tr><th scope="row"><time datetime="13:00">13:00</time></th><td>Lunch</td><td>—</td></tr>
              <tr><th scope="row"><time datetime="14:00">14:00</time></th><td>Screen readers, live on stage</td><td>Joseph Dsouza</td></tr>
              <tr><th scope="row"><time datetime="15:30">15:30</time></th><td>Right-to-left without a rewrite</td><td>Sara Qureshi</td></tr>
              <tr><th scope="row"><time datetime="16:30">16:30</time></th><td>Panel: what we'd delete from CSS</td><td>All speakers</td></tr>
            </tbody>
          </table>
        </section>
      </section>

      <section id="speakers" class="container section" aria-labelledby="speakers-title">
        <h2 id="speakers-title">Speakers</h2>
        <p class="note">Speakers and talks are fictional — this page is a course exercise.</p>
        <!-- role="list" restores list semantics in Safari/VoiceOver when list-style is removed -->
        <!-- [html-validate-disable-next no-redundant-role] -->
        <ul class="speakers" role="list">
          <li class="speaker-slot">
            <article class="speaker">
              <div class="speaker__avatar" aria-hidden="true">AS</div>
              <div>
                <h3 class="speaker__name">Anika Sharma</h3>
                <p class="speaker__talk">The cascade is a feature</p>
                <p class="speaker__bio">Maintains the CSS for a large public-sector design system.</p>
              </div>
            </article>
          </li>
          <li class="speaker-slot">
            <article class="speaker">
              <div class="speaker__avatar" aria-hidden="true">RM</div>
              <div>
                <h3 class="speaker__name">Rahul Menon</h3>
                <p class="speaker__talk">Forms nobody abandons</p>
                <p class="speaker__bio">Researches checkout usability and error recovery.</p>
              </div>
            </article>
          </li>
          <li class="speaker-slot">
            <article class="speaker">
              <div class="speaker__avatar" aria-hidden="true">LV</div>
              <div>
                <h3 class="speaker__name">Leela Varma</h3>
                <p class="speaker__talk">Container queries in production</p>
                <p class="speaker__bio">Builds component libraries used across dozens of products.</p>
              </div>
            </article>
          </li>
          <li class="speaker-slot">
            <article class="speaker">
              <div class="speaker__avatar" aria-hidden="true">JD</div>
              <div>
                <h3 class="speaker__name">Joseph Dsouza</h3>
                <p class="speaker__talk">Screen readers, live on stage</p>
                <p class="speaker__bio">Accessibility tester and daily screen-reader user.</p>
              </div>
            </article>
          </li>
          <li class="speaker-slot">
            <article class="speaker">
              <div class="speaker__avatar" aria-hidden="true">SQ</div>
              <div>
                <h3 class="speaker__name">Sara Qureshi</h3>
                <p class="speaker__talk">Right-to-left without a rewrite</p>
                <p class="speaker__bio">Localises interfaces into Urdu, Arabic and Persian.</p>
              </div>
            </article>
          </li>
        </ul>
      </section>

      <section id="tickets" class="container section tickets" aria-labelledby="tickets-title">
        <h2 id="tickets-title">Tickets</h2>
        <form class="register" id="register" novalidate>
          <div class="error-summary" id="error-summary" tabindex="-1" hidden aria-labelledby="error-summary-title">
            <h3 id="error-summary-title">There is a problem</h3>
            <ul></ul>
          </div>

          <fieldset>
            <legend>Ticket type</legend>
            <label class="option"><input type="radio" name="ticket" value="standard" checked> Standard — ₹2,000</label>
            <label class="option"><input type="radio" name="ticket" value="student"> Student — ₹500</label>
          </fieldset>

          <div class="field">
            <label for="name">Full name</label>
            <input id="name" name="name" type="text" autocomplete="name" required>
          </div>

          <div class="field">
            <label for="email">Email address</label>
            <p class="hint" id="email-hint">We'll send your ticket here.</p>
            <input id="email" name="email" type="email" autocomplete="email" required aria-describedby="email-hint">
          </div>

          <div class="field">
            <label for="access">Access requirements <span class="optional">(optional)</span></label>
            <textarea id="access" name="access" rows="3"></textarea>
          </div>

          <button type="submit" class="button">Reserve ticket</button>
          <p role="status" class="register__status"></p>
        </form>
      </section>

      <section id="faq" class="container section" aria-labelledby="faq-title">
        <h2 id="faq-title">FAQ</h2>
        <details>
          <summary>Is the venue step-free?</summary>
          <p>Yes: step-free entrance, lifts to every floor and an accessible toilet on each level.</p>
        </details>
        <details>
          <summary>Will talks be recorded?</summary>
          <p>Yes, with captions and transcripts, published within two weeks.</p>
        </details>
        <details>
          <summary>Is there a code of conduct?</summary>
          <p>Yes. Every attendee, speaker and volunteer agrees to it when registering.</p>
        </details>
      </section>
    </main>

    <footer class="site-footer">
      <div class="container">
        <p><small>Frontend Commons is a fictional event created as the capstone of HTML &amp; CSS Mastery Path.</small></p>
      </div>
    </footer>
  </body>
</html>
```

## The CSS

```css title="styles.css"
@layer reset, tokens, base, layout, components, utilities, print;

@layer reset {
  *, *::before, *::after { box-sizing: border-box; }
  body, h1, h2, h3, p, dl, dd, figure { margin: 0; }
  img { max-inline-size: 100%; block-size: auto; display: block; }
  button, input, textarea, select { font: inherit; }
  /* stylelint-disable-next-line declaration-no-important -- [hidden] must always win */
  [hidden] { display: none !important; }
}

@layer tokens {
  :root {
    color-scheme: light dark;

    /* primitives */
    --teal-300: oklch(80% 0.09 190);
    --teal-700: oklch(45% 0.1 195);
    --plum-700: oklch(38% 0.12 330);
    --sand-50: oklch(98.5% 0.008 85);
    --ink-900: oklch(19% 0.015 250);
    --red-600: oklch(50% 0.19 25);
    --red-300: oklch(76% 0.13 25);

    /* semantic */
    --color-bg: light-dark(var(--sand-50), var(--ink-900));
    --color-surface: light-dark(white, oklch(24% 0.015 250));
    --color-text: light-dark(var(--ink-900), oklch(94% 0.005 85));
    --color-muted: light-dark(oklch(44% 0.02 250), oklch(77% 0.02 250));
    --color-border: light-dark(oklch(86% 0.015 250), oklch(38% 0.02 250));
    --color-brand: light-dark(var(--teal-700), var(--teal-300));
    --color-on-brand: light-dark(white, var(--ink-900));
    --color-accent: light-dark(var(--plum-700), oklch(80% 0.09 330));
    --color-error: light-dark(var(--red-600), var(--red-300));
    --color-focus: light-dark(oklch(45% 0.16 260), oklch(80% 0.1 260));

    /* scales */
    --space-1: 0.25rem; --space-2: 0.5rem; --space-3: 1rem; --space-4: 1.5rem;
    --space-5: 2rem; --space-6: 3rem; --space-7: 4.5rem;
    --text-sm: clamp(0.875rem, 0.85rem + 0.1vw, 0.925rem);
    --text-base: clamp(1rem, 0.96rem + 0.2vw, 1.1rem);
    --text-lg: clamp(1.15rem, 1.05rem + 0.4vw, 1.35rem);
    --text-xl: clamp(1.5rem, 1.25rem + 1vw, 2rem);
    --text-hero: clamp(2.25rem, 1.4rem + 4vw, 4.5rem);
    --radius: 0.5rem;
    --z-header: 10;
  }
  :root[data-theme="light"] { color-scheme: light; }
  :root[data-theme="dark"] { color-scheme: dark; }
}

@layer base {
  html { scroll-padding-block-start: 5rem; }

  @media (prefers-reduced-motion: no-preference) { html { scroll-behavior: smooth; } }

  body {
    font-family: system-ui, -apple-system, "Segoe UI", Roboto, sans-serif;
    font-size: var(--text-base);
    line-height: 1.6;
    color: var(--color-text);
    background: var(--color-bg);
  }
  h1, h2, h3 { line-height: 1.15; text-wrap: balance; }
  h2 { font-size: var(--text-xl); margin-block-end: var(--space-3); }
  h3 { font-size: var(--text-lg); }
  p + p { margin-block-start: var(--space-3); }
  a { color: var(--color-brand); text-underline-offset: 0.15em; }
  :focus-visible { outline: 3px solid var(--color-focus); outline-offset: 2px; }
  main:focus { outline: none; }
}

@layer layout {
  .container { inline-size: min(100% - 2rem, 68rem); margin-inline: auto; }
  .section { padding-block: var(--space-6); }
  .section + .section { border-block-start: 1px solid var(--color-border); }
}

@layer components {
  .skip-link { position: absolute; inset-inline-start: var(--space-3); inset-block-start: -10rem; }
  .skip-link:focus { inset-block-start: var(--space-3); z-index: calc(var(--z-header) + 1); padding: var(--space-2) var(--space-3); background: var(--color-surface); }

  /* header */
  .site-header {
    position: sticky;
    inset-block-start: 0;
    z-index: var(--z-header);
    background: color-mix(in oklch, var(--color-bg) 92%, transparent);
    backdrop-filter: blur(6px);
    border-block-end: 1px solid var(--color-border);
  }
  .site-header__inner { display: flex; flex-wrap: wrap; align-items: center; gap: var(--space-2) var(--space-4); padding-block: var(--space-3); }
  .brand { font-weight: 800; text-decoration: none; color: var(--color-text); }
  .brand span { color: var(--color-brand); }
  .site-nav { margin-inline-start: auto; }
  .site-nav ul { display: flex; flex-wrap: wrap; gap: var(--space-1) var(--space-4); list-style: none; padding: 0; margin: 0; }

  .button, .theme-toggle {
    display: inline-block;
    padding: var(--space-2) var(--space-4);
    border: 2px solid var(--color-brand);
    border-radius: var(--radius);
    font-weight: 700;
    text-decoration: none;
    cursor: pointer;
  }
  .button { background: var(--color-brand); color: var(--color-on-brand); }
  .button:hover { background: color-mix(in oklch, var(--color-brand), black 12%); }
  .theme-toggle { background: transparent; color: var(--color-text); padding: var(--space-1) var(--space-3); }
  .theme-toggle[aria-pressed="true"] { background: var(--color-brand); color: var(--color-on-brand); }

  /* hero */
  .hero {
    padding-block: var(--space-7);
    background:
      radial-gradient(circle at 85% 20%, color-mix(in oklch, var(--color-accent) 25%, transparent), transparent 45%),
      linear-gradient(in oklch 160deg, color-mix(in oklch, var(--color-brand) 14%, var(--color-bg)), var(--color-bg));
  }
  .hero h1 { font-size: var(--text-hero); letter-spacing: -0.02em; max-inline-size: 14ch; }
  .hero__lede { font-size: var(--text-lg); max-inline-size: 45ch; margin-block: var(--space-4); }
  .hero__facts { display: flex; flex-wrap: wrap; gap: var(--space-3) var(--space-5); margin-block-end: var(--space-5); }
  .hero__facts dt { font-size: var(--text-sm); text-transform: uppercase; letter-spacing: 0.06em; color: var(--color-muted); }
  .hero__facts dd { font-weight: 700; }

  /* schedule */
  .table-scroll { overflow-x: auto; margin-block-start: var(--space-4); }
  .schedule { inline-size: 100%; min-inline-size: 32rem; border-collapse: collapse; font-variant-numeric: tabular-nums; }
  .schedule caption { text-align: start; font-weight: 700; padding-block-end: var(--space-2); }
  .schedule th, .schedule td { text-align: start; padding: var(--space-2) var(--space-3); border-block-end: 1px solid var(--color-border); }
  .schedule tbody th { white-space: nowrap; color: var(--color-brand); }

  /* speakers: grid of container-query cards */
  .note { color: var(--color-muted); font-size: var(--text-sm); }

  .speakers {
    list-style: none; padding: 0; margin-block-start: var(--space-4);
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(min(16rem, 100%), 1fr));
    gap: var(--space-4);
  }
  .speaker-slot { container: speaker / inline-size; }

  .speaker {
    display: grid; gap: var(--space-3); padding: var(--space-4);
    background: var(--color-surface); border: 1px solid var(--color-border); border-radius: var(--radius);
    block-size: 100%;
  }

  .speaker__avatar {
    display: grid; place-items: center;
    inline-size: 4rem; aspect-ratio: 1; border-radius: 50%;
    background: var(--color-accent); color: var(--color-bg); font-weight: 800;
  }
  .speaker__talk { font-weight: 600; color: var(--color-brand); }
  .speaker__bio { color: var(--color-muted); font-size: var(--text-sm); margin-block-start: var(--space-1); }

  @container speaker (width >= 22rem) {
    .speaker { grid-template-columns: auto 1fr; align-items: start; }
  }

  /* registration form */
  .register { display: grid; gap: var(--space-4); max-inline-size: 34rem; }
  .register fieldset { border: 0; padding: 0; display: flex; flex-wrap: wrap; gap: var(--space-2) var(--space-3); }
  .register legend { font-weight: 700; margin-block-end: var(--space-2); }

  .option {
    display: flex; align-items: center; gap: var(--space-2);
    padding: var(--space-2) var(--space-3);
    border: 1px solid var(--color-border); border-radius: var(--radius);
    &:has(:checked) { border-color: var(--color-brand); box-shadow: inset 0 0 0 1px var(--color-brand); }
    & input { accent-color: var(--color-brand); margin: 0; }
  }
  .field { display: grid; gap: var(--space-1); }
  .field label { font-weight: 700; }
  .optional, .hint { font-weight: 400; color: var(--color-muted); font-size: var(--text-sm); }

  .field input, .field textarea {
    padding: var(--space-2); border: 1px solid var(--color-muted); border-radius: var(--radius);
    background: var(--color-surface); color: var(--color-text);
  }
  .error { color: var(--color-error); font-weight: 700; }
  [aria-invalid="true"] { border: 2px solid var(--color-error); }

  .error-summary {
    padding: var(--space-4); border: 3px solid var(--color-error); border-radius: var(--radius);
    & ul { margin: var(--space-2) 0 0; padding-inline-start: var(--space-4); }
    & a { color: var(--color-error); font-weight: 700; }
  }
  .register__status { font-weight: 700; color: var(--color-brand); min-block-size: 1.6em; }
  .register .button { justify-self: start; }

  /* FAQ */
  details { border: 1px solid var(--color-border); border-radius: var(--radius); padding: var(--space-3) var(--space-4); background: var(--color-surface); }
  details + details { margin-block-start: var(--space-2); }
  summary { cursor: pointer; font-weight: 700; }
  summary::marker { color: var(--color-brand); }
  details[open] > summary { margin-block-end: var(--space-2); }

  .site-footer { padding-block: var(--space-5); border-block-start: 1px solid var(--color-border); color: var(--color-muted); }

  @media (forced-colors: active) {
    .button, .theme-toggle, .option, details, .speaker { border: 1px solid ButtonText; }
  }
}

@layer print {
  @media print {
    /* stylelint-disable-next-line declaration-no-important -- print hiding must beat components */
    .site-header, .skip-link, .theme-toggle, .hero .button, .tickets, .site-footer { display: none !important; }
    body { font: 11pt/1.45 Georgia, serif; color: black; background: white; }
    .hero { padding-block: 0 1cm; background: none; }
    .hero h1 { font-size: 24pt; }
    .section { padding-block: 0.5cm; break-inside: auto; }
    h2 { break-after: avoid; }
    .schedule { min-inline-size: 0; }
    .speaker, tr { break-inside: avoid; }
    .speaker { border: 1pt solid #999; }
    details::details-content { content-visibility: visible; display: block; }

    @page { size: A4; margin: 1.5cm; }
  }
}
```

## The JavaScript

Two small, independent enhancements. Without JavaScript, the page still works: the theme
follows the OS preference, and the form falls back to native validation... except that
`novalidate` is set on it. That's a deliberate trade-off worth noticing — in production,
you'd add `novalidate` from the script (`form.noValidate = true`) so no-JS users keep the
browser's built-in validation.

```js title="site.js"
// Theme toggle: explicit choice overrides the OS preference and is remembered
const toggle = document.querySelector('.theme-toggle');
const root = document.documentElement;
const isDark = () => root.dataset.theme === 'dark'
  || (!root.dataset.theme && matchMedia('(prefers-color-scheme: dark)').matches);
const sync = () => toggle.setAttribute('aria-pressed', String(isDark()));
sync();
toggle.addEventListener('click', () => {
  root.dataset.theme = isDark() ? 'light' : 'dark';
  try { localStorage.setItem('theme', root.dataset.theme); } catch {}
  sync();
});

// Registration form: inline errors + error summary
const form = document.getElementById('register');
const summary = document.getElementById('error-summary');
const status = form.querySelector('.register__status');
const messages = {
  name: (i) => i.validity.valueMissing && 'Enter your full name',
  email: (i) => (i.validity.valueMissing && 'Enter your email address')
    || (i.validity.typeMismatch && 'Enter an email address in the format name@example.com'),
};

form.addEventListener('submit', (event) => {
  event.preventDefault();
  const errors = [];
  for (const [name, message] of Object.entries(messages)) {
    const input = form.elements[name];
    const hint = input.getAttribute('aria-describedby')?.split(' ').filter((id) => id !== `${name}-error`) ?? [];
    let error = document.getElementById(`${name}-error`);
    const text = message(input);
    if (text) {
      if (!error) {
        error = document.createElement('p');
        error.id = `${name}-error`;
        error.className = 'error';
        input.before(error);
      }
      error.textContent = text;
      input.setAttribute('aria-invalid', 'true');
      input.setAttribute('aria-describedby', [`${name}-error`, ...hint].join(' '));
      errors.push({ id: input.id, text });
    } else {
      error?.remove();
      input.removeAttribute('aria-invalid');
      if (hint.length) input.setAttribute('aria-describedby', hint.join(' '));
      else input.removeAttribute('aria-describedby');
    }
  }

  const list = summary.querySelector('ul');
  list.replaceChildren(...errors.map(({ id, text }) => {
    const li = document.createElement('li');
    const a = document.createElement('a');
    a.href = `#${id}`;
    a.textContent = text;
    li.append(a);
    return li;
  }));
  summary.hidden = errors.length === 0;
  if (errors.length) {
    summary.querySelector('h3').textContent = errors.length === 1 ? 'There is a problem' : `There are ${errors.length} problems`;
    summary.focus();
    status.textContent = '';
  } else {
    status.textContent = 'Ticket reserved. Check your email for confirmation.';
  }
});
```

## The quality pipeline

### Validation and linting

html-validate's first run reported three errors:

```text
    6:94  error  title text cannot be longer than 70 characters   long-title
   98:36  error  Redundant role "list" on <ul>                    no-redundant-role
  158:14  error  <input> is missing recommended "type" attribute  no-implicit-input-type
```

We shortened the title and added `type="text"`. The redundant-role error is a real
disagreement between a validator rule and a known screen-reader behaviour: Safari with
VoiceOver stops announcing a `<ul>` as a list when `list-style: none` is applied (Level 3
· 01), and `role="list"` restores it. We kept the role and disabled that rule for that one
element with a comment explaining why — the right way to handle a lint rule you disagree
with, rather than turning it off globally.

Stylelint with the lesson 03 config reported **85 errors** on the first run. Most were
formatting preferences from `stylelint-config-standard` that conflicted with this
project's conventions: single-line rules with several declarations, BEM class names
(`.speaker__avatar` isn't kebab-case), and hue values written as plain numbers in
`oklch()`. A handful were real: an ID selector (`#tickets`) in the print styles that
broke the specificity budget, and two `!important`s. We fixed the real ones (the print rule
now targets a `.tickets` class, and each deliberate `!important` has a disable comment with
a reason), ran `stylelint --fix` for blank-line formatting, and configured the rest
explicitly:

```json title=".stylelintrc.json"
{
  "extends": ["stylelint-config-standard"],
  "rules": {
    "selector-max-id": 0,
    "selector-max-specificity": "0,2,1",
    "selector-max-compound-selectors": 3,
    "declaration-no-important": true,
    "selector-class-pattern": ["^[a-z][a-z0-9]*(-[a-z0-9]+)*(__[a-z0-9]+(-[a-z0-9]+)*)?(--[a-z0-9]+(-[a-z0-9]+)*)?$", { "message": "Use BEM: block__element--modifier" }],
    "declaration-block-single-line-max-declarations": null,
    "hue-degree-notation": "number",
    "value-keyword-case": ["lower", { "ignoreProperties": ["size"] }]
  }
}
```

After that: both tools exited with code 0, with the specificity budget at `0,2,1`.

### Accessibility audits

axe-core 4.13 (WCAG 2.0–2.2 A and AA plus best practices) reported **0 violations** at
360px, 800px and 1280px in light mode and at 360px and 1280px in dark mode, and
`scrollWidth` equalled the viewport width in every case.

### Behaviour tests

```js title="playwright.config.js"
import { defineConfig } from '@playwright/test';
export default defineConfig({
  testDir: 'tests',
  use: { baseURL: 'http://localhost:4174' },
  webServer: { command: 'python3 -m http.server 4174 --directory ../capstone', port: 4174, reuseExistingServer: true },
  projects: [
    { name: 'chromium-desktop', use: { browserName: 'chromium', viewport: { width: 1280, height: 800 } } },
    { name: 'chromium-phone', use: { browserName: 'chromium', viewport: { width: 360, height: 780 } } },
    { name: 'webkit-desktop', use: { browserName: 'webkit', viewport: { width: 1280, height: 800 } } },
  ],
});
```

```js title="tests/capstone.spec.js"
import { test, expect } from '@playwright/test';
import AxeBuilder from '@axe-core/playwright';

const axe = (page) => new AxeBuilder({ page }).withTags(['wcag2a', 'wcag2aa', 'wcag21aa', 'wcag22aa']).analyze();

test('no axe violations in light and dark', async ({ page }) => {
  for (const scheme of ['light', 'dark']) {
    await page.emulateMedia({ colorScheme: scheme });
    await page.goto('/');
    expect((await axe(page)).violations, scheme).toEqual([]);
  }
});

test('empty submit shows error summary and inline errors', async ({ page }) => {
  await page.goto('/');
  await page.getByRole('button', { name: 'Reserve ticket' }).click();
  const summary = page.locator('#error-summary');
  await expect(summary).toBeFocused();
  await expect(summary.getByRole('heading')).toHaveText('There are 2 problems');
  await expect(page.getByRole('textbox', { name: 'Email address' })).toHaveAttribute('aria-invalid', 'true');
  await summary.getByRole('link', { name: 'Enter your email address' }).click();
  await expect(page.getByRole('textbox', { name: 'Email address' })).toBeFocused();
  expect((await axe(page)).violations).toEqual([]);
});

test('fixing errors clears them and confirms', async ({ page }) => {
  await page.goto('/');
  await page.getByRole('button', { name: 'Reserve ticket' }).click();
  await page.getByRole('textbox', { name: 'Full name' }).fill('Ana Silva');
  await page.getByRole('textbox', { name: 'Email address' }).fill('ana@example.com');
  await page.getByRole('button', { name: 'Reserve ticket' }).click();
  await expect(page.locator('#error-summary')).toBeHidden();
  await expect(page.locator('.error')).toHaveCount(0);
  await expect(page.getByRole('status')).toHaveText('Ticket reserved. Check your email for confirmation.');
});

test('theme toggle overrides OS preference and persists', async ({ page }) => {
  await page.emulateMedia({ colorScheme: 'light' });
  await page.goto('/');
  const toggle = page.getByRole('button', { name: 'Dark theme' });
  await expect(toggle).toHaveAttribute('aria-pressed', 'false');
  const bgBefore = await page.evaluate(() => getComputedStyle(document.body).backgroundColor);
  await toggle.click();
  await expect(toggle).toHaveAttribute('aria-pressed', 'true');
  await page.reload();
  const bgAfter = await page.evaluate(() => getComputedStyle(document.body).backgroundColor);
  expect(bgAfter).not.toBe(bgBefore);
  await expect(page.getByRole('button', { name: 'Dark theme' })).toHaveAttribute('aria-pressed', 'true');
});

test('landmarks and headings', async ({ page }) => {
  await page.goto('/');
  await expect(page.getByRole('banner')).toHaveCount(1);
  await expect(page.getByRole('main')).toHaveCount(1);
  await expect(page.getByRole('contentinfo')).toHaveCount(1);
  await expect(page.getByRole('heading', { level: 1 })).toHaveText('Frontend Commons 2026');
  const levels = await page.locator('h1, h2, h3').evaluateAll((hs) => hs.map((h) => +h.tagName[1]));
  for (let i = 1; i < levels.length; i++) expect(levels[i] - levels[i - 1]).toBeLessThanOrEqual(1);
});

test('no horizontal page scroll', async ({ page }) => {
  await page.goto('/');
  const [sw, iw] = await page.evaluate(() => [document.documentElement.scrollWidth, innerWidth]);
  expect(sw).toBeLessThanOrEqual(iw);
});

test('mirrors in right-to-left', async ({ page }) => {
  await page.goto('/');
  await page.evaluate(() => { document.documentElement.dir = 'rtl'; });
  const brand = await page.locator('.brand').boundingBox();
  const vw = page.viewportSize().width;
  expect(brand.x + brand.width).toBeGreaterThan(vw / 2);
  const [sw, iw] = await page.evaluate(() => [document.documentElement.scrollWidth, innerWidth]);
  expect(sw).toBeLessThanOrEqual(iw);
});
```

Seven tests in three projects:

```text
  21 passed (5.8s)
```

The suite checks things no static tool can: that a failed submit focuses the error summary
and says "There are 2 problems", that each summary link focuses its field, that axe still
passes *with* errors showing, that fixing the fields clears every error and announces
success, that the theme toggle beats the OS preference and survives a reload, that
heading levels never skip, and that setting `dir="rtl"` mirrors the header without causing
horizontal scroll — in Chromium at two sizes and in WebKit.

### Print

With print media emulated in Chromium, the header, ticket form and footer were hidden,
the body switched to Georgia, and the FAQ answers inside closed `<details>` were visible
(`checkVisibility()` returned `true`, thanks to `::details-content`). Print-to-PDF produced
two A4 pages.

## Things the container queries decided

The speaker cards switch to a side-by-side layout at 22rem (352px) of **card** width. We
logged how many grid columns each card used:

```text
 360px viewport: grid "328px"                         cards: 1 column each (stacked)
 800px viewport: grid "372px 372px"                   cards: 2 columns each (side by side)
1280px viewport: grid "346.656px 346.672px 346.656px" cards: 1 column each (stacked)
```

The widest viewport produced the *narrowest* cards, because `auto-fit` fitted three tracks
of just under 352px. A media-query design would have got this backwards; the container
query simply responded to the space each card actually had.

## How It Actually Works

Trace one user action through every layer of the course. A keyboard user tabs to "Reserve
ticket" and presses Enter with an empty form:

1. **HTML** (Level 1): the button's default activation submits the form; `novalidate`
   stops native bubbles.
2. **Script** checks each field with the browser's constraint validation API
   (`validity.valueMissing`), writes messages into new elements, sets `aria-invalid` and
   rewires `aria-describedby` (Level 3 · 05), fills the summary list and focuses it.
3. **The accessibility tree** updates: the summary region gains a heading and links; the
   inputs gain an invalid state and descriptions. A screen reader, following the focus
   move, reads the summary.
4. **The cascade** (Level 1 · 08, Level 3 · 07): `[aria-invalid="true"]` in the
   `components` layer draws the red border; the colour comes from `--color-error`, a
   semantic token that `light-dark()` resolves per theme (Level 4 · 04).
5. **Layout** (Level 2): the summary and error lines are inserted into the form's grid;
   because the form is a grid with `gap`, spacing adjusts with no extra rules.
6. **Rendering** (Level 4 · 01): one style recalculation and one layout for the whole
   batch of DOM changes, because the script only writes — it never reads layout in between.

Every piece does one job, and each is checked by at least one automated test.

## Common mistakes

- **Content that doesn't match the structured data** (dates, prices).
- **Disabling lint rules globally** instead of per-case with a reason.
- **Testing only in one engine** — WebKit and Chromium differ in details that matter.
- **Theme toggles that ignore the OS preference** until clicked, or flash on load.
- **A capstone that's only ever been looked at with a mouse on one screen size.**

## Exercise

1. Build your own event, portfolio or product site to this brief. Keep the ten
   requirements and the pipeline; change everything else.
2. Add a second page (a speaker detail page) that shares the stylesheet, with correct
   `aria-current`, canonical and Open Graph tags.
3. Translate the page's headings into another language and switch `lang` and `dir` —
   right-to-left if you can — and fix anything the RTL test doesn't catch.
4. Add visual snapshots to the suite and generate the baselines in a Linux container.
5. Publish it (GitHub Pages, Netlify or similar), run Lighthouse against the live URL, and
   do the manual checklist from lesson 09 with a real screen reader.
