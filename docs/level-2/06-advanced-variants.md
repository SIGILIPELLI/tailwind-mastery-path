---
description: "Style from existing state with Tailwind's aria-*, data-*, has-*, not-*, in-* and nth-* variants, and define your own with @custom-variant."
---

# 06 · Advanced Variants: aria, data, has, not & Custom Variants

Interactive components carry their state somewhere. A disclosure button has
`aria-expanded="true"`, a tab has `data-state="active"`, a form row contains an input that
has focus. A common anti-pattern is to mirror that state into extra classes from
JavaScript (`is-open`, `is-active`) just so CSS can see it. Tailwind's attribute and
relational variants let you style from the state that's already there. That means less
JavaScript and, with ARIA, styles that can't drift out of sync with what assistive
technology is told.

## `aria-*`: style from accessibility state

Boolean ARIA attributes have built-in variants that match the value `"true"`. Note that
`aria-expanded:` checks the element the class is on. In a disclosure button, the
attribute is on the `<button>` but the thing you want to rotate is the icon inside it,
so make the button a `group` and use `group-aria-expanded:` on the icon:

```html
<button aria-expanded="false" aria-controls="faq-1"
        class="group flex w-full items-center justify-between py-3 font-medium">
  What is your refund policy?
  <svg class="size-5 transition-transform group-aria-expanded:rotate-180"
       aria-hidden="true" viewBox="0 0 20 20" fill="currentColor">
    <path d="M5.3 7.3a1 1 0 0 1 1.4 0L10 10.6l3.3-3.3a1 1 0 1 1 1.4 1.4l-4 4a1 1 0 0 1-1.4 0l-4-4a1 1 0 0 1 0-1.4z"/>
  </svg>
</button>
<div id="faq-1" hidden class="pb-4 text-gray-600">Full refund within 30 days.</div>
```

```css
.group-aria-expanded\:rotate-180:is(:where(.group)[aria-expanded="true"] *) {
  rotate: 180deg;
}
```

The JavaScript only toggles the attribute (which it must do anyway for screen readers),
and the styling follows:

```js
for (const btn of document.querySelectorAll("[aria-controls]")) {
  btn.addEventListener("click", () => {
    const open = btn.getAttribute("aria-expanded") === "true";
    btn.setAttribute("aria-expanded", String(!open));
    document.getElementById(btn.getAttribute("aria-controls")).hidden = open;
  });
}
```

`aria-{name}:` works for any attribute name and always tests for the value `"true"`.
We compiled `aria-expanded:`, `aria-busy:`, `aria-selected:` and even a made-up
`aria-foo:`, and each became `[aria-…="true"]`. That's right for true/false attributes
(`aria-busy`, `aria-checked`, `aria-disabled`, `aria-expanded`, `aria-pressed`,
`aria-selected` and so on), but wrong for attributes with other values. The classic
example is navigation: `aria-current` is normally `"page"`, so `aria-current:font-bold`
never matches. Use brackets for those: `aria-[current=page]:font-bold`, or
`aria-[sort=ascending]:bg-sky-50`, which compiled to `[aria-sort="ascending"]`.

## `data-*`: style from component state

Headless UI libraries (Radix, Headless UI, Reach, Ark and others) expose state as data
attributes, and they're handy in your own components too:

```html
<button data-state="active" class="border-b-2 border-transparent px-3 py-2
         data-[state=active]:border-sky-600 data-[state=active]:text-sky-700">
  Overview
</button>
```

Compiled: `.data-\[state\=active\]\:border-sky-600[data-state="active"]`.

Without brackets, `data-active:` checks that the attribute **exists**, whatever its value.
That has a sharp edge: we rendered `<div data-active="false" class="data-active:font-bold">`
and it computed to `font-weight: 700`. The attribute exists, so the style applies. Either
remove the attribute when it's off, or match a value explicitly with
`data-[active=true]:`. ARIA variants don't have this problem: `aria-expanded:` requires
`"true"`, and our `aria-expanded="false"` button stayed at weight 400.

## `has-*`: style a parent from what's inside it

`:has()` is CSS's parent selector, and Tailwind exposes it as `has-*`:

```html
<label class="flex items-center gap-3 rounded-lg border border-gray-200 p-4
              has-checked:border-sky-500 has-checked:bg-sky-50
              has-focus-visible:ring-2 has-focus-visible:ring-sky-500">
  <input type="radio" name="plan" value="pro" class="size-4 accent-sky-600">
  <span class="font-medium">Pro</span>
</label>
```

The whole card highlights when its radio is checked, and shows a ring when the radio has
keyboard focus. That used to need JavaScript. Any state variant works after `has-`, and
brackets take any selector: `has-[input:focus]:ring-2` compiled to
`:has(:is(input:focus))`, and in our browser test the wrapper got its 2px ring as soon as
the input was focused.

`group-has-*` and `peer-has-*` combine the two ideas: `group-has-[img]:p-0` removes a
card's inner padding only when the card contains an image.

`:has()` is supported in all current major browsers, but it's newer than most CSS here.
If you must support older browsers, make sure the page still works (just less
decorated) without it.

## `not-*`: the opposite of any variant

`not-` negates a pseudo-class, a selector, or even a media query:

```html
<li class="py-3 not-last:border-b not-last:border-gray-200">…</li>
<a class="text-gray-500 not-[.active]:hover:text-gray-900">…</a>
```

```css
.not-last\:border-b:not(:last-child) { … }
.not-\[\.active\]\:opacity-60:not(:is(.active)) { … }
```

`not-last:border-b` is a tidy alternative to `border-b last:border-0`. Negating a media
variant flips the query: `not-hover:opacity-80` compiled to both `:not(:hover)` and an
`@media not (hover: hover)` block, so it also applies on touch devices.

## `in-*`: like `group`, without the marker

`in-*` checks a state on **any** ancestor, without needing a `group` class:

```html
<div data-state="open">
  <span class="invisible in-data-[state=open]:visible">Visible when open</span>
</div>
```

Compiled: `:where([data-state="open"]) .in-data-\[state\=open\]\:visible`. It's
convenient when you don't control the ancestor's classes (for example, a library sets
`data-state` on a wrapper). Because it matches *any* ancestor, it has the same crosstalk
risk as unnamed groups.

## `nth-*` for positional styling

Beyond `first:`, `last:`, `odd:` and `even:`:

| Class | Selector |
|---|---|
| `nth-3:font-bold` | `:nth-child(3)` |
| `nth-[3n+1]:clear-left` | `:nth-child(3n+1)` |
| `nth-last-2:mb-0` | `:nth-last-child(2)` |

## `@custom-variant`: name your own

When you write the same arbitrary variant often, or need a condition Tailwind doesn't
ship, define a variant in CSS. The short form takes a selector or at-rule:

```css title="src/app.css"
@custom-variant theme-midnight (&:where([data-theme=midnight] *));
@custom-variant pointer-coarse (@media (pointer: coarse));
```

The block form uses `@slot` to mark where the utility's declarations go, which lets you
combine an at-rule and a selector:

```css
@custom-variant any-hover {
  @media (any-hover: hover) {
    &:hover {
      @slot;
    }
  }
}
```

Compiled usage: `theme-midnight:bg-indigo-950` became
`.theme-midnight\:bg-indigo-950:where([data-theme=midnight] *)`, and `any-hover:underline`
became `@media (any-hover: hover) { .any-hover\:underline:hover { … } }`. Custom variants
stack with all the others (`md:theme-midnight:…`).

(Tailwind already ships `pointer-coarse:` and `pointer-fine:`. Check the built-in list
before defining a variant; the example above just shows the syntax.)

## Worked example: a filterable list with no state classes

```html
<fieldset class="flex flex-wrap gap-2">
  <legend class="sr-only">Filter</legend>
  <label class="cursor-pointer rounded-full border border-gray-300 px-3 py-1 text-sm
                has-checked:border-sky-700 has-checked:bg-sky-700 has-checked:text-white
                has-focus-visible:outline-2 has-focus-visible:outline-offset-2
                has-focus-visible:outline-sky-600">
    <input type="radio" name="status" value="all" class="sr-only" checked> All
  </label>
  <label class="cursor-pointer rounded-full border border-gray-300 px-3 py-1 text-sm
                has-checked:border-sky-700 has-checked:bg-sky-700 has-checked:text-white
                has-focus-visible:outline-2 has-focus-visible:outline-offset-2
                has-focus-visible:outline-sky-600">
    <input type="radio" name="status" value="open" class="sr-only"> Open
  </label>
</fieldset>

<ul class="mt-4 divide-y divide-gray-200" aria-busy="false"
    class="aria-busy:opacity-50 aria-busy:cursor-wait">
  <li data-status="open" class="flex justify-between py-3 data-[status=closed]:text-gray-400">
    Fix login redirect <span class="text-xs">open</span>
  </li>
  <li data-status="closed" class="flex justify-between py-3 data-[status=closed]:text-gray-400
                                  data-[status=closed]:line-through">
    Update footer links <span class="text-xs">closed</span>
  </li>
</ul>
```

There's a deliberate bug in that markup. Look at the `<ul>`: it has two `class`
attributes. Browsers keep the first and ignore the second, so `aria-busy:` would never
be applied. It's an easy slip when adding
classes in a hurry, and Tailwind can't warn you (it scans both, generates both, and the
browser drops one). The fix is to merge them:

```html
<ul class="mt-4 divide-y divide-gray-200 aria-busy:cursor-wait aria-busy:opacity-50"
    aria-busy="false">
```

With that fixed:

- The filter chips are real radio buttons (keyboard and screen reader support for free),
  visually hidden with `sr-only`, styled through `has-checked:` and
  `has-focus-visible:` on their labels.
- Each row's appearance comes from its `data-status`.
- While your script fetches new results it sets `aria-busy="true"` on the list, which
  both tells assistive technology to wait and dims the list.

## How It Actually Works

These variants are generated from patterns rather than a fixed list. `aria-{name}`
appends `[aria-{name}="true"]`; `aria-[…]` and `data-[…]` take the bracket contents as an
attribute selector, adding quotes around the value; `data-{name}` appends a presence
check `[data-{name}]`. `has-{variant}` takes another variant's selector and wraps it in
`:has(…)`, `not-{variant}` wraps it in `:not(…)` (or negates the media query), and
`in-{variant}` prepends `:where(<selector>) ` as an ancestor. That compositional design
is why `group-has-`, `peer-aria-`, `not-has-` and so on all work without being defined
individually.

`@custom-variant` registers a new entry in the same variant registry. With the short
form, `&` is replaced by the utility's selector; with the block form, the utility's
declarations are inserted at `@slot`.

## Common mistakes

- **`aria-current:` for the current nav link.** Its value is usually `"page"`, not
  `"true"`; use `aria-[current=page]:`.
- **Using `aria-expanded:` on a child** of the element that has the attribute. Use
  `group-aria-expanded:` (or `in-aria-expanded:`).
- **`data-active:` with `data-active="false"`.** Presence matches; use a value
  (`data-[active=true]:`) or remove the attribute.
- **Mirroring state into classes** (`is-open`) when an ARIA or data attribute already
  carries it.
- **Duplicate `class` attributes** on one element. The browser silently ignores the
  second.
- **`in-*` or unnamed `group-*` under nested stateful ancestors**, which react to the
  wrong ancestor.
- **Redefining a variant Tailwind already has.** Check the built-ins first.

## Exercise

1. Build an accordion of three questions with `aria-expanded` and `group-aria-expanded:`.
   Confirm with a screen reader or the Accessibility pane that the state is announced.
2. Recreate the `data-active="false"` pitfall, then fix it two ways.
3. Build radio-button "plan cards" styled only with `has-checked:` and
   `has-focus-visible:`.
4. Define `@custom-variant theme-midnight` and build a small card that changes colours
   when an ancestor has `data-theme="midnight"`.
