---
description: "Build HTML forms that work: labels, input types, radio groups, select and textarea, what actually gets submitted, and native constraint validation."
---

# 06 · Forms & Native Validation

Forms are where a page stops being a document and starts being an application — sign-in,
search, checkout, contact, settings. HTML's form elements are remarkably capable on their
own: they handle keyboard input, mobile keyboards, autofill, validation and submission
without a line of JavaScript. Most broken forms on the web are broken because someone
replaced those built-ins with something custom and didn't rebuild all of it.

## Anatomy of a form

```html title="signup.html"
<form action="/signup" method="post">
  <p>
    <label for="email">Email</label>
    <input id="email" name="email" type="email" autocomplete="email" required>
  </p>

  <p>
    <label for="password">Password</label>
    <input id="password" name="password" type="password"
           autocomplete="new-password" minlength="12" required
           aria-describedby="password-hint">
    <span id="password-hint">At least 12 characters.</span>
  </p>

  <fieldset>
    <legend>Plan</legend>
    <label><input type="radio" name="plan" value="free" checked> Free</label>
    <label><input type="radio" name="plan" value="pro"> Pro</label>
  </fieldset>

  <p>
    <label>
      <input type="checkbox" name="newsletter" value="yes">
      Send me the monthly newsletter
    </label>
  </p>

  <button type="submit">Create account</button>
</form>
```

- **`action`** — where to send the data. Omit it to submit to the current URL.
- **`method`** — `get` (the default) puts the data in the URL query string; use it for
  searches and filters, which should be bookmarkable. `post` sends it in the request body;
  use it for anything that changes data or includes secrets.
- **`name`** — the key each value is submitted under. **A field without a `name` is not
  submitted.**
- **`<label>`** — covered next; the single most important form element.
- **`<button type="submit">`** — submits. Pressing Enter in a text field does too.

## Labels

Every form control needs a label. Two ways to associate one:

```html
<!-- explicit: for= matches the input's id -->
<label for="city">City</label>
<input id="city" name="city">

<!-- implicit: the input is inside the label -->
<label>City <input name="city"></label>
```

A label does three jobs: it's the control's **accessible name** (what a screen reader
announces), it's a bigger **click/tap target** (clicking the label focuses the input or
toggles the checkbox), and it's visible instructions for everyone.

`placeholder` is **not** a label. It disappears as soon as you type, is usually low
contrast, and isn't reliably announced. Use it, if at all, for an example format
("e.g. 560001").

## Input types

The `type` attribute changes validation, the on-screen keyboard on phones and the
built-in UI:

| Type | Use for | What you get |
|------|---------|--------------|
| `text` | Anything free-form | Default |
| `email` | Email addresses | Format validation, `@` keyboard |
| `tel` | Phone numbers | Numeric phone keypad; no format validation (formats vary worldwide) |
| `url` | Web addresses | URL validation |
| `number` | Quantities you'd increment | Spin buttons, `min`/`max`/`step` validation |
| `password` | Secrets | Masked input |
| `search` | Search boxes | Clear button in some browsers |
| `date`, `time`, `datetime-local`, `month`, `week` | Dates and times | Native pickers |
| `checkbox` | Independent yes/no | |
| `radio` | One choice from a group (same `name`) | Arrow keys move within the group |
| `range` | Approximate value on a slider | |
| `color` | Colour picker | |
| `file` | Uploads (`accept=".pdf,image/*"`, `multiple`) | Needs `method="post"` and `enctype="multipart/form-data"` |
| `hidden` | Values the user doesn't edit | |

`type="number"` is for quantities, not for anything that merely *contains digits* —
postcodes, card numbers and OTP codes should be `type="text"` with
`inputmode="numeric"` (which shows a number keypad without spin buttons or number
parsing that strips leading zeros).

### Other controls

```html
<label for="size">Size</label>
<select id="size" name="size">
  <option value="">Choose a size</option>
  <optgroup label="Soups">
    <option value="s">Small (250 ml)</option>
    <option value="m" selected>Medium (400 ml)</option>
  </optgroup>
</select>

<label for="notes">Delivery notes</label>
<textarea id="notes" name="notes" rows="4"></textarea>
```

## Grouping with `fieldset` and `legend`

Radio buttons and related checkboxes need a group label: "Free" and "Pro" mean nothing
without "Plan". `<fieldset>` groups them and `<legend>` names the group; screen readers
announce "Plan, group" on entering it. `<fieldset disabled>` disables every control
inside at once.

## Worked example: what actually gets submitted

Submitted data is a list of **name/value pairs**, built by a precise algorithm. We built
this form in Chromium and read it with `new FormData(form)`:

```html
<form id="f">
  <input name="email" type="email" value="ana@example.com">
  <input value="no-name">
  <input name="plan" type="radio" value="free">
  <input name="plan" type="radio" value="pro" checked>
  <input name="news" type="checkbox">
  <input name="terms" type="checkbox" checked>
  <input name="tags" type="checkbox" value="css" checked>
  <input name="tags" type="checkbox" value="html" checked>
  <input name="token" value="abc" disabled>
  <input name="uid" value="42" readonly>
  <select name="size"><option>S</option><option selected>M</option></select>
  <textarea name="notes">hi</textarea>
  <button name="action" value="save">Save</button>
</form>
```

```text
FormData: [["email","ana@example.com"],["plan","pro"],["terms","on"],["tags","css"],
           ["tags","html"],["uid","42"],["size","M"],["notes","hi"]]
URL-encoded: email=ana%40example.com&plan=pro&terms=on&tags=css&tags=html&uid=42&size=M&notes=hi
```

Every rule in that output is one you'll trip over eventually:

- `<input value="no-name">` — **no `name`, not submitted.**
- `news` — an **unchecked checkbox sends nothing at all**, not `news=off`. The server has
  to treat "missing" as false.
- `terms` — a checked checkbox without a `value` sends **`on`**.
- `tags` appears **twice**. Multiple values under one name is normal; your server code
  must expect a list.
- `token` — **disabled fields are not submitted**. `uid` is `readonly` and *is* submitted.
  Use `readonly` for "show but don't edit".
- `size=M` — an `<option>` without `value` submits its text.
- `action=save` is absent because we constructed `FormData` directly; the button's
  name/value is only included when that button is what submitted the form. That's how
  "Save" and "Publish" buttons in one form are told apart.

With `method="get"`, the URL-encoded string becomes the query string:
`/signup?email=ana%40example.com&plan=pro...`.

## Native constraint validation

Attributes declare the rules; the browser enforces them on submit, shows a message, and
focuses the first invalid field:

| Attribute | Rule |
|-----------|------|
| `required` | Must have a value (for radios: one in the group checked) |
| `type="email"` / `"url"` | Must match the format |
| `minlength`, `maxlength` | Text length |
| `min`, `max`, `step` | Numbers and dates |
| `pattern` | A regular expression the *whole* value must match |

We checked four fields in Chromium via each element's `validity` object and
`validationMessage`:

```text
type=email required value="ana@"       -> typeMismatch   "Please enter a part following '@'. 'ana@' is incomplete."
type=number min=1 max=10 value="12"    -> rangeOverflow  "Value must be less than or equal to 10."
pattern="[A-Z]{3}-[0-9]{3}" "ab-123"   -> patternMismatch "Please match the requested format."
required, empty                        -> valueMissing   "Please fill out this field."
```

The messages are the browser's own and vary by browser and language. The generic
"Please match the requested format" is why you should always describe the expected format
in visible text next to a `pattern` field.

### Styling valid and invalid states

```css
input:user-invalid {
  border-color: #b00020;
  outline-color: #b00020;
}
```

Use `:user-invalid` rather than `:invalid`. `:invalid` matches immediately — every empty
`required` field is red before the user has typed anything. `:user-invalid` only matches
after the user has interacted with the field or tried to submit.

### Client-side validation is a convenience, not security

Anyone can bypass it — by editing the HTML in devtools, adding `novalidate`, or sending
a request without a browser at all. **Always validate again on the server.** Client-side
validation exists to give fast feedback, not to protect your data.

## How It Actually Works

On submission the browser runs the **form submission algorithm**:

1. Fire the `submit` event (scripts can cancel it).
2. Unless the form has `novalidate` or the submitter has `formnovalidate`, run **static
   constraint validation**: check each control's `validity`. If any fail, fire `invalid`
   events, report the first problem to the user, and stop.
3. **Construct the entry list**: walk the form's controls in tree order, skip those that
   are disabled, have no name, are unchecked checkboxes/radios, or are buttons other than
   the submitter; add name/value pairs for the rest.
4. **Encode** the list using the form's `enctype` — `application/x-www-form-urlencoded`
   (the default: `a=1&b=2` with percent-encoding), `multipart/form-data` (needed for
   files), or `text/plain` (don't use it).
5. **Navigate** to the `action` URL with that data, using `method`.

Controls belong to a form by being inside it — or, for controls elsewhere on the page, by
a `form="form-id"` attribute. That's occasionally useful for layouts where a submit
button must live outside the `<form>` element.

## Common mistakes

- **Missing `name`**, so the field silently never submits.
- **Placeholder instead of label.**
- **Radios without `fieldset`/`legend`**, or radios with different `name`s (then they
  aren't a group and all can be checked).
- **`type="number"` for codes and IDs** — leading zeros vanish, and scroll wheels change
  the value.
- **Buttons without `type`**. Inside a form, a `<button>` defaults to `type="submit"`, so
  a "Show password" button submits the form. Write `type="button"` for non-submit buttons.
- **Relying on client-side validation for security.**
- **Turning off autofill** with `autocomplete="off"` on login fields. Use correct
  `autocomplete` tokens instead (`email`, `current-password`, `new-password`,
  `postal-code`, `one-time-code`) so password managers and autofill work.

## Exercise

1. Build a "Book a cooking class" form with: name, email, phone, a date, a class choice
   (radios in a fieldset), dietary requirements (checkboxes), a notes textarea and a
   submit button. Give every field a label, a `name` and a sensible `autocomplete` token.
2. Set `method="get"` and no `action`. Submit it and read the URL — confirm every rule
   from the worked example (unchecked boxes missing, repeated names, etc.).
3. Add `required`, `minlength` and a `min` date. Try submitting empty, then partly filled.
4. Add `input:user-invalid` styling and compare it with `input:invalid`.
5. In devtools, delete the `required` attribute from a field and submit. What does this
   tell you about server-side validation?
