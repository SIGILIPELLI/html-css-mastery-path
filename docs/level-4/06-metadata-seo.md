---
description: "The HTML search engines and social apps read: titles, descriptions, canonical URLs, robots rules, Open Graph, JSON-LD and sitemaps."
---

# 06 · Metadata, SEO & Structured Data

Search engines, social networks, messaging apps and browsers all read your pages — and
most of what they read is in the `<head>`, not the visible content. Good metadata doesn't
trick anyone into ranking you higher; it makes sure the systems that summarise your page
summarise it **correctly**: the right title in search results, the right image when
someone shares a link, the right URL when the same content is reachable at several
addresses.

This lesson sticks to what's defined in public standards and search engines' published
documentation. Ranking algorithms are proprietary and change constantly; anyone who
claims to know exact ranking weights is guessing.

## The foundation is the content

Search engines are built to find pages that answer a query well. The HTML-level things
that genuinely help are the same things that help people and assistive technology:

- A **descriptive `<title>`** and a single, matching **`<h1>`**.
- **Headings** that outline the content (Level 3 · 02).
- **Real text** rather than text in images; **`alt`** for meaningful images.
- **Descriptive link text** — search engines use it to understand the target page.
- **Fast, stable pages** (lessons 01–02) — page experience signals are part of how
  Google evaluates pages, per its own documentation.
- **Semantic HTML** so the main content is identifiable (`<main>`, `<article>`).

## Title and description

```html
<title>Roasted Tomato Soup (45 Minutes) — Weeknight Kitchen</title>
<meta name="description" content="A roasted tomato soup for four with pantry ingredients: 10 minutes' prep, 35 minutes in the oven, freezes well.">
```

- Titles: unique per page, specific, most important words first. Search results truncate
  long titles by pixel width, so front-load.
- Descriptions: a unique summary of *this* page. Search engines may show it as the snippet
  or write their own from the page text; either way, it isn't a ranking lever — it's your
  pitch in the results.

## Canonical URLs

The same content is often reachable at several URLs — with and without `www`, with
tracking parameters, with a trailing slash or not, through pagination or filters. Tell
search engines which one is the real one:

```html
<link rel="canonical" href="https://example.com/recipes/roasted-tomato-soup/">
```

Use an absolute URL, point every variant at the one preferred version, and make sure the
canonical URL itself returns a 200 and isn't blocked. A page can declare itself as
canonical (a "self-referencing canonical"), which is a sensible default.

## Robots rules

Two different mechanisms, often confused:

```text title="/robots.txt"
User-agent: *
Disallow: /admin/
Sitemap: https://example.com/sitemap.xml
```

```html
<meta name="robots" content="noindex, follow">
```

- **`robots.txt`** controls **crawling** — whether a crawler may fetch URLs. It does not
  reliably keep a page out of search results: a disallowed URL can still be indexed from
  links pointing to it (without its content).
- **`<meta name="robots" content="noindex">`** (or the `X-Robots-Tag` HTTP header) controls
  **indexing**. For it to work, the crawler must be *allowed* to fetch the page and see the
  tag — so don't also block that page in robots.txt.

Neither is access control. Anything private must be behind authentication.

## Social previews: Open Graph and Twitter cards

When a link is shared in a chat app or social network, the platform fetches the page and
reads **Open Graph** tags (originally Facebook's protocol, now read widely):

```html
<meta property="og:type" content="article">
<meta property="og:title" content="Roasted Tomato Soup">
<meta property="og:description" content="Roasting concentrates the sweetness — 45 minutes, pantry ingredients.">
<meta property="og:url" content="https://example.com/recipes/roasted-tomato-soup/">
<meta property="og:image" content="https://example.com/img/og/roasted-tomato-soup.jpg">
<meta property="og:image:width" content="1200">
<meta property="og:image:height" content="630">
<meta property="og:image:alt" content="A bowl of tomato soup with basil and cream">
<meta property="og:site_name" content="Weeknight Kitchen">

<meta name="twitter:card" content="summary_large_image">
```

- Use **absolute URLs** for `og:image` and `og:url`.
- **1200×630** is the commonly recommended image size for large previews; keep important
  content away from the edges, because platforms crop differently.
- Platforms cache previews aggressively; most offer a debugging tool that re-fetches.
- Note `property=` for Open Graph and `name=` for Twitter tags — a common copy-paste error.

## Structured data with JSON-LD

Structured data describes the page's content in a machine-readable vocabulary —
**schema.org** — so search engines can understand entities (a recipe, a product, an
article, an event, an organisation) and potentially show rich results such as recipe
cards with ratings and cooking time.

JSON-LD is the format search engines recommend: a `<script>` block that doesn't affect
the page at all.

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Recipe",
  "name": "Roasted Tomato Soup",
  "image": ["https://example.com/img/soup-16x9.jpg"],
  "author": { "@type": "Person", "name": "Priya Rao" },
  "datePublished": "2026-09-30",
  "description": "A roasted tomato soup for four with pantry ingredients.",
  "prepTime": "PT10M",
  "cookTime": "PT35M",
  "totalTime": "PT45M",
  "recipeYield": "4 servings",
  "recipeIngredient": ["1 kg ripe tomatoes", "1 onion", "4 garlic cloves", "500 ml vegetable stock"],
  "recipeInstructions": [
    { "@type": "HowToStep", "text": "Roast the tomatoes, onion and garlic at 200 °C for 30 minutes." },
    { "@type": "HowToStep", "text": "Simmer with the stock for 5 minutes, then blend." }
  ]
}
</script>
```

Rules that matter:

- **Describe only what's visible on the page.** Marking up ratings, prices or reviews
  that users can't see violates search engines' structured-data policies and can lead to
  manual penalties.
- The durations use **ISO 8601** (`PT35M`), the same format as the `<time>` element's
  `datetime` (Level 1 · 03).
- Check it with the **Schema Markup Validator** (validator.schema.org) for vocabulary
  correctness and Google's **Rich Results Test** for eligibility for Google's rich results.
- Eligibility isn't a guarantee: search engines decide whether to show rich results.

A malformed JSON-LD block is silently ignored by browsers, so at minimum make sure it
parses. Running `JSON.parse` on the text content of each block — one line in the console —
catches trailing commas and unescaped quotes before any validator does:

```js
[...document.querySelectorAll('script[type="application/ld+json"]')].map(s => JSON.parse(s.textContent)['@type'])
```

## Sitemaps

A sitemap lists the URLs you want crawled, optionally with last-modified dates:

```xml title="/sitemap.xml"
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <url>
    <loc>https://example.com/recipes/roasted-tomato-soup/</loc>
    <lastmod>2026-09-30</lastmod>
  </url>
</urlset>
```

Reference it from `robots.txt` and submit it in the search engines' webmaster tools
(Google Search Console, Bing Webmaster Tools). Only list canonical, indexable URLs, and
keep `lastmod` honest — a date that changes on every build teaches crawlers to ignore it.
Static site generators, including the one that builds this course, create sitemaps
automatically.

## Other useful head elements

```html
<link rel="alternate" hreflang="en" href="https://example.com/en/soup/">
<link rel="alternate" hreflang="te" href="https://example.com/te/soup/">
<link rel="alternate" hreflang="x-default" href="https://example.com/soup/">
<link rel="alternate" type="application/rss+xml" title="New recipes" href="/feed.xml">
<meta name="theme-color" content="#8a2f12">
<link rel="manifest" href="/site.webmanifest">
```

`hreflang` links connect translations of the same page (each version should list all
versions, including itself). `theme-color` tints some browsers' UI. A web app manifest
describes the site's name and icons for installation.

## Worked example: a complete head for an article page

```html
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Roasted Tomato Soup (45 Minutes) — Weeknight Kitchen</title>
  <meta name="description" content="A roasted tomato soup for four with pantry ingredients: 10 minutes' prep, 35 in the oven, freezes well.">
  <link rel="canonical" href="https://example.com/recipes/roasted-tomato-soup/">

  <meta property="og:type" content="article">
  <meta property="og:title" content="Roasted Tomato Soup">
  <meta property="og:description" content="Roasting concentrates the sweetness — 45 minutes, pantry ingredients.">
  <meta property="og:url" content="https://example.com/recipes/roasted-tomato-soup/">
  <meta property="og:image" content="https://example.com/img/og/roasted-tomato-soup.jpg">
  <meta property="og:image:alt" content="A bowl of tomato soup with basil and cream">
  <meta name="twitter:card" content="summary_large_image">

  <link rel="icon" href="/favicon.svg" type="image/svg+xml">
  <link rel="stylesheet" href="/css/site.css">
  <script type="application/ld+json">{ "@context": "https://schema.org", "@type": "Recipe", "name": "Roasted Tomato Soup" }</script>
</head>
```

(The JSON-LD here is abbreviated; use the full recipe block above in practice.)

## How It Actually Works

Crawlers fetch URLs (respecting robots.txt), then **render** pages — major search
engines run a real browser engine, so content produced by JavaScript can be indexed, but
rendering may happen later than the initial fetch and costs crawl resources, which is one
reason server-rendered HTML remains the most reliable. The rendered DOM is parsed for
text, links, headings, metadata and structured data; canonical tags and redirects are
used to cluster duplicate URLs and choose one to index.

Social platforms are simpler: their fetchers generally **don't run JavaScript**. They
download the HTML and read `<meta>` tags from it directly — which is why Open Graph tags
added by client-side JavaScript usually don't work.

JSON-LD is parsed as JSON-LD (JSON with linked-data context): `@context` maps the short
property names to full schema.org vocabulary URLs, and `@type` says which schema.org type
the object is, which determines the expected properties.

## Common mistakes

- **Duplicate titles and descriptions** across pages.
- **Blocking a page in robots.txt *and* adding `noindex`** — the crawler never sees the
  `noindex`.
- **Relative URLs in canonical and Open Graph tags.**
- **Structured data for content that isn't on the page.**
- **Open Graph tags injected by JavaScript.**
- **Keyword stuffing** in titles, alt text or hidden text — ignored at best, penalised at
  worst.
- **Invalid JSON** in JSON-LD, silently ignored.

## Exercise

1. Write the complete `<head>` for your recipe page: title, description, canonical, Open
   Graph, Twitter card and a full `Recipe` JSON-LD block.
2. Validate the JSON-LD with the Schema Markup Validator and the Rich Results Test.
3. Create a 1200×630 preview image and test the link in a platform's sharing debugger.
4. Write a `robots.txt` and `sitemap.xml` for a three-page site, and explain the
   difference between disallowing a page and `noindex`ing it.
5. View the source of a large recipe or news site and find its canonical link, Open Graph
   tags and JSON-LD. Is anything missing or wrong?
