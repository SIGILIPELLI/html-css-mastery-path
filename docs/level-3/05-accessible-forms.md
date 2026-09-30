---
description: "Forms everyone can complete: visible labels, hints, grouping, autocomplete, required fields, accessible inline errors and error summaries."
---

# 05 · Accessible Forms & Error Handling

Forms are where accessibility failures cost the most: a person who can't get past a
sign-up, checkout or benefits application is simply locked out. Level 1 · 06 covered how
forms work; this lesson covers how to make them work **for everyone** — including when
things go wrong, which is where most forms fail.

## Start with an audit of a bad form

Here's a form built the way many are:

```html
<form>
  <p>Email</p><input type="email" name="email">
  <input type="text" name="name" placeholder="Full name">
  <input type="checkbox" name="terms"> I agree
  <select name="country"><option>India</option></select>
  <div class="btn" onclick="">Send</div>
</form>
```

We ran axe-core's WCAG A/AA rules against it:

```text
label (2 nodes): Form elements must have labels
select-name (1 nodes): Select element must have an accessible name
```

Axe caught the email field (the "Email" paragraph isn't connected to it), the checkbox
("I agree" is loose text) and the select. Just as instructive is what it **didn't**
report:

- The name field passed, because its placeholder counts as a fallback accessible name.
  It's still a poor label (it vanishes on input, and has low contrast by default).
- The "Send" `div` passed, because a `div` with a click handler isn't recognisably a
  control to an automated tool. It's the worst problem on the page — no keyboard user
  can submit the form.

Automated checks are a floor. The rest of this lesson is the part they can't check.

## Labels, hints and required fields

```html
<div class="field">
  <label for="email">Email address <span class="req">(required)</span></label>
  <p class="hint" id="email-hint">We'll send your receipt here.</p>
  <input id="email" name="email" type="email" autocomplete="email"
         required aria-describedby="email-hint">
</div>
```

- **A visible `<label>`, always**, positioned consistently (above the field is easiest to
  scan and works at any width).
- **Hints go outside the label** and are connected with `aria-describedby`, so they're
  read after the name: "Email address (required), edit text, required, We'll send your
  receipt here."
- **Mark required fields in text**, not just with a red asterisk (colour alone, and an
  asterisk that's read as "star"). If most fields are required, say "All fields are
  required unless marked optional" and mark the optional ones instead.
- **`required`** also exposes the required state to assistive technology.

### `autocomplete`

The `autocomplete` attribute tells browsers and password managers what a field is, so
they can fill it. It's also a WCAG requirement (1.3.5 *Identify Input Purpose*) because
it helps people with memory and motor impairments:

```html
<input name="name" autocomplete="name">
<input name="email" type="email" autocomplete="email">
<input name="tel" type="tel" autocomplete="tel">
<input name="street" autocomplete="street-address">
<input name="postcode" autocomplete="postal-code">
<input name="cc" autocomplete="cc-number" inputmode="numeric">
<input name="code" autocomplete="one-time-code" inputmode="numeric">
<input name="new-pw" type="password" autocomplete="new-password">
```

### Grouping

Related controls need a group label (Level 1 · 06): radios, checkbox sets, and composite
inputs like a date split into day/month/year:

```html
<fieldset>
  <legend>Date of birth</legend>
  <label for="dob-d">Day</label>   <input id="dob-d" name="dob-d" inputmode="numeric" autocomplete="bday-day">
  <label for="dob-m">Month</label> <input id="dob-m" name="dob-m" inputmode="numeric" autocomplete="bday-month">
  <label for="dob-y">Year</label>  <input id="dob-y" name="dob-y" inputmode="numeric" autocomplete="bday-year">
</fieldset>
```

## Errors that everyone can find and fix

WCAG requires that errors are **identified in text** (3.3.1) and that you **suggest a
fix** where you can (3.3.3). A robust pattern:

1. Validate on **submit** (and optionally when a field loses focus after the user has
   typed), not on every keystroke.
2. Put a specific message **next to each invalid field**, connected with
   `aria-describedby`, and set `aria-invalid="true"`.
3. For forms with several fields, show an **error summary** at the top listing each
   error as a link to its field, and move focus to it.
4. Keep the user's input. Never clear a form because one field was wrong.

### Worked example: an inline error

```html
<form novalidate id="checkout">
  <div class="field">
    <label for="email">Email</label>
    <p id="email-hint" class="hint">We'll send the receipt here.</p>
    <input id="email" name="email" type="email" autocomplete="email"
           required aria-describedby="email-hint">
  </div>
  <button>Continue</button>
</form>
```

```js
const form = document.getElementById('checkout');
form.addEventListener('submit', (event) => {
  const input = document.getElementById('email');
  if (input.checkValidity()) return;          // let valid forms submit
  event.preventDefault();

  let error = document.getElementById('email-error');
  if (!error) {
    error = document.createElement('p');
    error.id = 'email-error';
    error.className = 'error';
    input.before(error);
  }
  error.textContent = input.validity.valueMissing
    ? 'Enter your email address'
    : 'Enter an email address in the format name@example.com';

  input.setAttribute('aria-invalid', 'true');
  input.setAttribute('aria-describedby', 'email-error email-hint');
  input.focus();
});
```

`novalidate` turns off the browser's own error bubbles so we can show consistent,
styleable messages — but we still use the browser's validation engine
(`checkValidity()` and `validity`) to decide what's wrong.

We typed `ana@` and clicked Continue in Chromium. The field's properties, read from the
browser's accessibility tree through the DevTools protocol:

```text
role: textbox
name: Email
description: Enter an email address in the format name@example.com We'll send the receipt here.
invalid: true
focused element: email
```

A screen-reader user landing on the field hears the name, "invalid entry", and then the
error followed by the hint — because `aria-describedby` lists `email-error` first. The
message says what to do, not just what's wrong ("Invalid input" helps nobody).

Style the error so it isn't colour-only:

```css
.error { color: #b00020; font-weight: 600; }
.error::before { content: "⚠ " / ""; }   /* decorative icon, empty alt text */
[aria-invalid="true"] { border: 2px solid #b00020; }
```

### Error summary

For longer forms, add a summary at the top:

```html
<div class="error-summary" tabindex="-1" id="error-summary" aria-labelledby="error-summary-title">
  <h2 id="error-summary-title">There are 2 problems</h2>
  <ul>
    <li><a href="#email">Enter an email address in the format name@example.com</a></li>
    <li><a href="#postcode">Enter a postcode</a></li>
  </ul>
</div>
```

After a failed submit, render it and call `errorSummary.focus()`. Screen readers read the
heading and the list; each link jumps straight to the field. This pattern, popularised by
the UK government's GOV.UK Design System, is one of the most thoroughly user-tested form
patterns there is.

Also put the error count in the page `<title>` ("Error: Checkout — Weeknight Kitchen"),
because the title is the first thing announced if the form is submitted to the server and
the page reloads.

## Other form requirements worth knowing

- **Target size** (WCAG 2.2, 2.5.8): pointer targets at least 24×24 CSS pixels, or spaced
  so a 24px circle around each doesn't overlap another. Checkboxes and radios are small
  by default — make their labels the click target (they are, if properly associated) and
  consider enlarging them.
- **Don't disable the submit button** until the form is valid. Disabled buttons can't be
  focused, give no reason, and leave people stuck. Let them submit and show errors.
- **Redundant entry** (3.3.7): don't make people re-type information they gave earlier in
  the same process ("Billing address same as delivery").
- **Accessible authentication** (3.3.8): don't block pasting into password fields, and
  don't rely on puzzles without alternatives.
- **Time limits**: warn and allow extension before a session times out.

## How It Actually Works

When a field is focused, a screen reader assembles the announcement from the
accessibility node: **name** (from the label), **role** ("edit text"), **states**
("required", "invalid entry"), **value**, then **description** (from
`aria-describedby`, concatenating the referenced elements' text in the listed order).
That's why the order of ids in `aria-describedby` matters and why the error message comes
first in our example.

`aria-invalid` doesn't do anything else — it doesn't style the field or block
submission. And the native `:invalid` state doesn't set `aria-invalid`; browsers expose
native invalidity to assistive tech in varying ways, so setting `aria-invalid="true"`
explicitly (and removing it when fixed) is the reliable approach.

Moving focus with `input.focus()` fires a focus event that screen readers follow, which
is why they announce the field immediately after submit. Without the focus move, a
screen-reader user who pressed Enter on the button would hear nothing.

## Common mistakes

- **Placeholders as labels.**
- **Error messages that are only colour** or only an icon.
- **Errors not connected to their fields**, so they're never read.
- **Generic messages** ("Invalid", "Error").
- **Validating on every keystroke**, announcing errors while someone is still typing.
- **Disabled submit buttons.**
- **Clearing the form** after an error.
- **Blocking paste** into password or confirmation fields.

## Exercise

1. Take the bad form at the top of this lesson and fix every issue, including the ones
   axe didn't report. Re-run axe (the axe DevTools extension or Lighthouse).
2. Build a three-field checkout (email, postcode, card number) with the inline error
   pattern and an error summary. Verify each field's name, description and invalid state
   in devtools' Accessibility pane.
3. Complete the form with a screen reader, deliberately making mistakes. Is every error
   announced? Can you get to each field from the summary?
4. Add correct `autocomplete` tokens and test browser autofill.
5. Measure your checkbox targets in devtools. Are they at least 24×24 including the label?
