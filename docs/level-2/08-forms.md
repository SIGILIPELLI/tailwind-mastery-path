---
description: "Accessible forms with Tailwind: what Preflight does to inputs, validation variants, accent-color, field-sizing and the forms plugin."
---

# 08 · Styling Forms

Forms are where Tailwind beginners get the biggest surprise: drop an `<input>` on a page
and it's invisible. No border, no background, just a blinking caret. That's Preflight
(Tailwind's base styles) resetting form controls so they look the same in every browser,
which also means you're responsible for styling them. This lesson covers what the reset
does, a solid pattern for text fields, checkboxes, radios and selects, validation and
error messages, and when the official forms plugin is worth adding.

## What Preflight does to form controls

These rules come from Tailwind 4.3.3's base layer:

```css
*, ::after, ::before, ::backdrop, ::file-selector-button {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
  border: 0 solid;
}
button, input, select, optgroup, textarea, ::file-selector-button {
  font: inherit;
  color: inherit;
  border-radius: 0;
  background-color: transparent;
  /* … */
}
::placeholder { opacity: 1; color: color-mix(in oklab, currentcolor 50%, transparent); }
textarea { resize: vertical; }
```

We rendered a bare `<input>` and measured `border-top-width: 0px` and a transparent
background. That's why it disappears. The upside is that controls now use your page
font at your page size, instead of each browser's small system default.

## A text field done properly

```html
<div>
  <label for="email" class="block text-sm font-medium text-gray-900">Email</label>
  <input id="email" name="email" type="email" autocomplete="email" required
         aria-describedby="email-hint"
         class="mt-1 block w-full rounded-md border border-gray-300 bg-white px-3 py-2
                text-gray-900 shadow-xs placeholder:text-gray-400
                focus:border-sky-600 focus:ring-2 focus:ring-sky-600/30 focus:outline-hidden
                disabled:cursor-not-allowed disabled:bg-gray-50 disabled:text-gray-500
                read-only:bg-gray-50">
  <p id="email-hint" class="mt-1 text-sm text-gray-500">We'll only use this for receipts.</p>
</div>
```

The pieces that matter:

- **A visible `<label>` tied with `for`/`id`.** Clicking it focuses the input, and screen
  readers announce it. A placeholder is not a label.
- **`aria-describedby`** connects the hint, so it's read after the label.
- **Border and background** put back what Preflight removed.
- **Focus**: the default outline is removed only because the border colour and a ring
  replace it, and it's removed with `outline-hidden` rather than `outline-none`.
  Rings are box-shadows, which Windows forced-colours mode strips out;
  `outline-hidden` restores a real outline in that mode (Level 3 · 04). Text inputs always match `:focus-visible` as well, so `focus:` and
  `focus-visible:` behave the same here.
- **`autocomplete` and the right `type`** give users the right mobile keyboard and
  autofill. They're not styling, but they matter more than any class.

## Validation and error messages

Use `user-invalid:` for styling, so fields don't show errors before the user has touched
them (Level 1 · 08):

```html
<div>
  <label for="zip" class="block text-sm font-medium text-gray-900">Postcode</label>
  <input id="zip" name="zip" required pattern="[0-9]{5}" inputmode="numeric"
         aria-describedby="zip-error"
         class="peer mt-1 block w-full rounded-md border border-gray-300 px-3 py-2
                focus:border-sky-600 focus:ring-2 focus:ring-sky-600/30 focus:outline-hidden
                user-invalid:border-red-600 user-invalid:focus:ring-red-600/30">
  <p id="zip-error" class="mt-1 hidden text-sm text-red-700 peer-user-invalid:block">
    Enter a 5-digit postcode.
  </p>
</div>
```

Three accessibility details:

- **Don't rely on colour alone.** The message text is the real signal; the red border is
  a reinforcement.
- **Server-side errors** need more than CSS. After a failed submit, render the field with
  `aria-invalid="true"` and the message visible, then style with
  `aria-invalid:border-red-600`.
- **Wrap each field and its message in its own element** so `peer` can't leak between
  fields (the unnamed-peer problem from Level 1 · 08).

## Checkboxes and radios: `accent-*` first

The cheapest way to brand native checkboxes, radios and range sliders is `accent-color`.
It keeps the native control, so keyboard behaviour, high-contrast mode and screen reader
support stay intact:

```html
<label class="flex items-start gap-3">
  <input type="checkbox" name="terms" required class="mt-1 size-4 accent-sky-600">
  <span class="text-sm text-gray-700">I agree to the <a class="underline" href="/terms">terms</a>.</span>
</label>
```

Wrapping the input in its `<label>` makes the whole text clickable. `mt-1` aligns the box
with the first line of text when the label wraps.

If the design needs a fully custom look, the pattern from Level 1 · 08 applies: keep a
real (visually hidden) input and style a sibling with `peer-checked:` and
`peer-focus-visible:`, or style the parent with `has-checked:`.

## Selects

A native `<select>` with Preflight keeps its arrow in most browsers but looks
inconsistent. A dependable approach is to remove the native appearance and draw your own
arrow:

```html
<div class="relative">
  <label for="country" class="block text-sm font-medium text-gray-900">Country</label>
  <select id="country" name="country"
          class="mt-1 block w-full appearance-none rounded-md border border-gray-300 bg-white
                 py-2 ps-3 pe-10 focus:border-sky-600 focus:ring-2 focus:ring-sky-600/30
                 focus:outline-hidden">
    <option>India</option>
    <option>Canada</option>
  </select>
  <svg class="pointer-events-none absolute end-3 bottom-2.5 size-5 text-gray-500"
       aria-hidden="true" viewBox="0 0 20 20" fill="currentColor">
    <path d="M5.3 7.3a1 1 0 0 1 1.4 0L10 10.6l3.3-3.3a1 1 0 1 1 1.4 1.4l-4 4a1 1 0 0 1-1.4 0l-4-4a1 1 0 0 1 0-1.4z"/>
  </svg>
</div>
```

`pe-10` reserves room so long option text doesn't run under the arrow, and
`pointer-events-none` lets clicks on the arrow reach the select.

## Textareas that grow: `field-sizing-content`

`field-sizing-content` compiles to `field-sizing: content`, which makes a textarea grow
with its text instead of scrolling. In Chromium, a four-line textarea measured 96px with
it and 48px (the default two rows) without it. Browser support is still incomplete, so
treat it as an enhancement: add `min-h-24` so it's usable either way, and don't depend
on it for layout.

## The forms plugin: when to add it

`@tailwindcss/forms` gives every form control a consistent, styled-but-plain default
(border, padding, white background, focus ring), so you don't have to start from
nothing. In v4 you load plugins from CSS:

```bash
npm install -D @tailwindcss/forms
```

```css title="src/app.css"
@import "tailwindcss";
@plugin "@tailwindcss/forms";
```

With version 0.5.11, this added rules in the **base** layer such as:

```css
input:where([type='text']), input:where(:not([type])), input:where([type='email']), …,
textarea, select {
  appearance: none;
  background-color: #fff;
  border-color: oklch(55.1% 0.027 264.364);
  border-width: 1px;
  padding-top: 0.5rem; /* … */
}
```

Because it's in the base layer with `:where()`, any utility you add still overrides it.
If you'd rather opt in per element, use the class strategy:

```css
@plugin "@tailwindcss/forms" { strategy: "class"; }
```

Then nothing is styled globally, and you add `form-input`, `form-select`,
`form-checkbox` where you want them. The class strategy is the safer choice when
adding the plugin to an existing site, since the global strategy restyles every input on
every page, including third-party widgets.

Add the plugin when you have many forms and want consistent defaults. Skip it when your
forms are few, or when you're using a component library that styles its own controls.

## Worked example: a sign-up form

```html
<form class="mx-auto max-w-md space-y-5" action="/signup" method="post" novalidate>
  <div>
    <label for="name" class="block text-sm font-medium">Full name</label>
    <input id="name" name="name" autocomplete="name" required
           class="peer mt-1 block w-full rounded-md border border-gray-300 px-3 py-2
                  focus:border-sky-600 focus:ring-2 focus:ring-sky-600/30 focus:outline-hidden
                  user-invalid:border-red-600">
    <p class="mt-1 hidden text-sm text-red-700 peer-user-invalid:block">Enter your name.</p>
  </div>

  <div>
    <label for="pw" class="block text-sm font-medium">Password</label>
    <input id="pw" name="password" type="password" autocomplete="new-password"
           minlength="12" required aria-describedby="pw-hint"
           class="peer mt-1 block w-full rounded-md border border-gray-300 px-3 py-2
                  focus:border-sky-600 focus:ring-2 focus:ring-sky-600/30 focus:outline-hidden
                  user-invalid:border-red-600">
    <p id="pw-hint" class="mt-1 text-sm text-gray-500 peer-user-invalid:text-red-700">
      At least 12 characters.
    </p>
  </div>

  <fieldset>
    <legend class="text-sm font-medium">Plan</legend>
    <div class="mt-2 grid gap-3 sm:grid-cols-2">
      <label class="flex cursor-pointer gap-3 rounded-lg border border-gray-300 p-3
                    has-checked:border-sky-600 has-checked:bg-sky-50">
        <input type="radio" name="plan" value="free" checked class="mt-0.5 accent-sky-600">
        <span><span class="block font-medium">Free</span>
              <span class="block text-sm text-gray-500">For personal use</span></span>
      </label>
      <label class="flex cursor-pointer gap-3 rounded-lg border border-gray-300 p-3
                    has-checked:border-sky-600 has-checked:bg-sky-50">
        <input type="radio" name="plan" value="team" class="mt-0.5 accent-sky-600">
        <span><span class="block font-medium">Team</span>
              <span class="block text-sm text-gray-500">Shared workspaces</span></span>
      </label>
    </div>
  </fieldset>

  <button class="w-full rounded-md bg-sky-700 px-4 py-2.5 font-medium text-white
                 hover:bg-sky-800 focus-visible:outline-2 focus-visible:outline-offset-2
                 focus-visible:outline-sky-600">Create account</button>
</form>
```

Notes:

- `novalidate` turns off the browser's own error bubbles so your inline messages are the
  only ones. Without it, users get both. You must then check validity in JavaScript on
  submit (`form.checkValidity()`), focus the first invalid field, and still validate on
  the server.
- The password hint turns red instead of a separate error appearing. The rule stays
  visible, so users know what's expected before they make a mistake.
- Radios in a `<fieldset>` with a `<legend>` are announced as a group ("Plan, Free, radio
  button, 1 of 2").

## How It Actually Works

Native form controls are partly drawn by the operating system. CSS can restyle the
"outside" (border, background, padding, font), and `appearance: none` asks the browser to
drop the native drawing so your styles take over completely. Preflight does the minimum
(inherit fonts and colours, remove borders and backgrounds) and doesn't set
`appearance: none` on checkboxes, radios or selects, which is why they still look
native. `accent-color` is a separate hook the
browser gives you to recolour native controls without replacing them.

The forms plugin is a set of base styles written with `:where()` selectors, so their
specificity is that of the element alone. Utility classes, being one class, always beat
them. Your validation variants (`user-invalid:`, `peer-user-invalid:`) are ordinary
pseudo-class selectors, re-evaluated by the browser as the user types and blurs.

## Common mistakes

- **Shipping an unstyled input** after Preflight removed its border.
- **Placeholder instead of a label.**
- **`focus:outline-none` with no replacement**, or with only a ring, which disappears
  in forced-colours mode. Use `focus:outline-hidden` plus a ring.
- **Showing errors on page load** with `invalid:`. Use `user-invalid:` and server-side
  `aria-invalid`.
- **Colour-only error states**, with no message text.
- **Rebuilding checkboxes from scratch** when `accent-*` would do.
- **Adding the forms plugin globally to an existing site** and restyling every input
  everywhere. Consider `strategy: "class"`.
- **`novalidate` without JavaScript validation**, so invalid forms submit.

## Exercise

1. Drop a bare `<input>` and `<select>` on a page and inspect them. Then style a text
   field with label, hint, focus ring and disabled state.
2. Build the sign-up form. Test it with only the keyboard and with a screen reader.
3. Add the forms plugin with the class strategy, convert one field to `form-input`, and
   compare its compiled CSS to your hand-written classes.
4. Add a textarea with `field-sizing-content` and a `min-h-*`, and check it in a browser
   that doesn't support `field-sizing`.
