---
description: "Every line of an HTML document skeleton explained: doctype, html lang, charset, the viewport meta tag, title, stylesheets, scripts and favicons."
---

# 02 · Document Structure & the Head Element

Every HTML page shares the same skeleton. It's short, and it's worth knowing exactly why
each line is there, because each one changes something measurable: how text is decoded,
how wide a phone thinks the page is, what a screen reader pronounces, what appears in a
browser tab and a search result.

## The skeleton, line by line

```html title="index.html"
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Tomato Soup — Weeknight Recipes</title>
    <meta name="description" content="A 30-minute tomato soup with pantry ingredients.">
    <link rel="icon" href="/favicon.svg" type="image/svg+xml">
    <link rel="stylesheet" href="/css/styles.css">
    <script src="/js/app.js" defer></script>
  </head>
  <body>
    <!-- visible content goes here -->
  </body>
</html>
```

### `<!doctype html>`

Not a tag, but a signal: "use standards mode." As lesson 01 showed, without it Chromium
switches to quirks mode (`document.compatMode === "BackCompat"`). It's case-insensitive;
`<!DOCTYPE html>` is identical.

### `<html lang="en">`

The root element. The `lang` attribute declares the page's primary language, and it has
real effects:

- Screen readers choose a pronunciation voice from it. English text read with a French
  voice is barely intelligible.
- Browsers use it for automatic hyphenation (`hyphens: auto` in CSS needs `lang` to know
  the language's hyphenation rules) and when choosing fonts for some scripts.
- Translation tools use it to decide whether to offer translation.

Use a BCP 47 language tag: `en`, `en-GB`, `fr`, `pt-BR`, `zh-Hant`, `te` (Telugu),
`hi` (Hindi). You can override it on any element: `<p lang="fr">Bon appétit</p>`.

### `<head>` and `<body>`

`<head>` holds metadata — information *about* the document, almost none of it visible.
`<body>` holds the content. Both tags are technically optional (the parser inserts them),
but write them: it makes the structure obvious and keeps tools happy.

### `<meta charset="utf-8">`

Tells the browser how to turn the file's bytes into characters. Two rules:

1. It must appear within the **first 1024 bytes** of the file, because the browser only
   looks that far before guessing. Put it first in `<head>`.
2. Your editor must actually save the file as UTF-8 (every modern editor does by default).

In our test page, `document.characterSet` reported `UTF-8`. If the server sends a
`Content-Type: text/html; charset=...` header, that header takes precedence over the
meta tag — worth knowing when a page's accents break only in production.

### `<meta name="viewport" ...>`

This line is the difference between a page that works on phones and one that doesn't.
Early smartphones had to display sites designed for desktops, so mobile browsers pretend
to be a wide screen and then zoom out. The viewport meta tag switches that off.

We loaded the same page in Chromium emulating a 390-pixel-wide phone, with and without
the tag, and read `window.innerWidth`:

```text
without viewport meta innerWidth: [980,980,"16px"]
with viewport meta innerWidth: [390,390,"16px"]
```

Without the tag, the page lays out at a **980-pixel** virtual width and is then scaled
down to fit the 390-pixel screen, so 16px text renders at roughly 40% size. Your media
queries (Level 2 · 08) never fire because the page believes it's 980px wide. With
`width=device-width, initial-scale=1`, CSS pixels match the device's width.

Don't add `maximum-scale=1` or `user-scalable=no`. They block pinch-zoom, which people
with low vision rely on; WCAG treats it as a failure.

### `<title>`

The text in the browser tab, in bookmarks, in history, and usually as the blue link in
search results. It's also the first thing a screen reader announces when a page loads.
Make it specific and put the unique part first: "Tomato Soup — Weeknight Recipes"
beats "Weeknight Recipes — Tomato Soup" when twenty tabs are open and each shows only its
first few words.

### `<meta name="description">`

Not displayed on the page. Search engines may use it as the snippet under your title
(they also often write their own). One or two honest sentences about this specific page.
Level 4 · 06 covers metadata and social previews in depth.

### `<link>`

Links the document to external resources. The `rel` attribute says what the relationship
is:

| `rel` | What it does |
|-------|--------------|
| `stylesheet` | Loads CSS. Render-blocking by default. |
| `icon` | The favicon. SVG favicons work in current Chromium, Firefox and Safari; provide a PNG or `.ico` too if you need older browsers. |
| `preload` | Starts fetching a resource early (Level 4 · 02). |
| `canonical` | The preferred URL for this content (Level 4 · 06). |
| `alternate` | Alternative versions — translations, RSS feeds. |

### `<script>`

A classic `<script src>` in the `<head>` stops the parser while it downloads and runs.
Two attributes change that:

- `defer` — download in parallel, run **after** the document is parsed, in document
  order. The right default for scripts that work with the page.
- `async` — download in parallel, run **as soon as it arrives**, in any order. For
  independent scripts like analytics.
- `type="module"` scripts are deferred automatically.

This course is about HTML and CSS; when you need JavaScript,
[JavaScript Mastery Path](https://sigilipelli.github.io/javascript-mastery-path/) covers it.

## Worked example: a page skeleton you can reuse

Here's the template we'll build every Level 1 exercise on. It adds a skip link and the
main landmark structure, which Level 3 explains in depth — including them from day one
costs nothing.

```html title="template.html"
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Page name — Site name</title>
    <meta name="description" content="One specific sentence about this page.">
    <link rel="stylesheet" href="styles.css">
  </head>
  <body>
    <a class="skip-link" href="#main">Skip to content</a>

    <header>
      <p class="site-name"><a href="/">Site name</a></p>
      <nav aria-label="Main">
        <ul>
          <li><a href="/">Home</a></li>
          <li><a href="/about.html">About</a></li>
        </ul>
      </nav>
    </header>

    <main id="main">
      <h1>Page name</h1>
      <p>Content.</p>
    </main>

    <footer>
      <p><small>© 2026 Your Name</small></p>
    </footer>
  </body>
</html>
```

### Validate it

Browsers never report HTML errors — they silently repair them. So use a validator:

- The **W3C Nu HTML Checker** at `validator.w3.org/nu/` accepts a URL, an uploaded file
  or pasted markup.
- **html-validate** runs locally from npm and is what we used while writing this course.
  Level 4 · 09 sets it up properly; for a quick check:

```bash
npx html-validate template.html
```

Our first run (html-validate 11.16) reported one error — and not the kind we expected:

```text
  1:1  error  DOCTYPE should be uppercase  doctype-style
```

That's a *style* rule in the tool's recommended preset, not an HTML requirement: the
doctype is case-insensitive. We turned it off with a config file next to the page:

```json title=".htmlvalidate.json"
{ "extends": ["html-validate:recommended"], "rules": { "doctype-style": "off" } }
```

The next run printed nothing and exited with code 0, which is what "no problems" looks
like. Then we deleted `lang="en"` and ran it again:

```text
  2:2  error  <html> is missing required "lang" attribute  element-required-attributes
```

That's the kind of problem browsers will never tell you about.

## How It Actually Works

The parser follows **insertion modes** — states that decide where each token goes. At the
start it's in "initial" mode, where the doctype is handled (and the rendering mode set).
It moves to "before html", "before head", "in head", "after head", "in body" and so on.

That's why structure is enforced even when you omit it. If you write text before any
`<head>` or `<body>` tag, the parser implicitly creates `<html>`, `<head>` and `<body>`
and puts the text in the body. And if visible content appears *inside* `<head>`, the
parser ends the head right there. We tested this page:

```html
<head><meta charset="utf-8"><p>oops</p><link rel="stylesheet" href="..."><title>Late title</title></head>
```

Chromium's DOM had only `META` in the head; the body contained `P, LINK, TITLE, H1`.
Everything after the stray paragraph was moved into the body. The stylesheet still
applied and `document.title` was still "Late title" — browsers tolerate those elements
in the body — but the document is now invalid, and a `<meta charset>` in that position
would be too late to matter.

The practical rule: in `<head>`, only put `<meta>`, `<title>`, `<link>`, `<style>`,
`<script>`, `<base>` and `<noscript>`. Anything visible belongs in `<body>`.

The viewport tag is not part of the parser at all — it's read by the browser's layout
engine to set the **initial containing block** width. Everything with a percentage width,
every media query and every `vw` unit is ultimately measured against that value.

## Common mistakes

- **Forgetting the viewport meta tag.** The page "looks tiny on phones" and media queries
  seem broken.
- **`lang` missing or wrong.** Easy to overlook because it has no visual effect for sighted
  users; very noticeable for screen-reader users.
- **Generic titles.** Every page titled "Home" or the same site name makes tabs,
  bookmarks and search results useless.
- **Visible content inside `<head>`.** It gets moved into the body and silently ends the
  head early.
- **Scripts in `<head>` without `defer`.** They block rendering and can't find elements
  that haven't been parsed yet.
- **Disabling zoom** with `user-scalable=no`.

## Exercise

1. Save `template.html` and give it a real title and description for a page about a
   hobby of yours.
2. Open it in Chrome devtools' device toolbar (the phone icon), pick a phone size, and run
   `innerWidth` in the console. Remove the viewport meta tag, reload and run it again.
3. Run the file through the W3C checker or `npx html-validate`. Introduce three errors —
   remove `lang`, put a `<p>` in the `<head>`, nest an `<a>` inside another `<a>` — and read
   each message.
4. Add `<p lang="fr">Bon appétit</p>` to the body. If your OS has a screen reader
   (VoiceOver on macOS: <kbd>Cmd</kbd>+<kbd>F5</kbd>), listen to how the voice changes.
