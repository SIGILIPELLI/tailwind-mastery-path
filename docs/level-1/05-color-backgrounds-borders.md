---
description: "Tailwind colour utilities: the oklch palette, opacity modifiers, backgrounds and gradients, borders and rings in v4, and shadows."
---

# 05 · Colors, Opacity, Backgrounds & Borders

Colour utilities follow one pattern — `{property}-{color}-{shade}` — across text,
backgrounds, borders, outlines, rings, shadows, SVG fills and more. This lesson covers the
palette, the opacity modifier, gradients, and two v4 defaults that trip up people coming
from v3: borders and rings now default to `currentColor`, and rings default to 1px.

## The palette

The default theme in 4.3.3 has 26 colour families (we compiled a `bg-{family}-500` for
each name to check) — the neutrals `slate`, `gray`, `zinc`, `neutral`, `stone`, `mauve`,
`olive`, `mist` and `taupe` (the last four were added during the v4 series), and the
hues `red`, `orange`, `amber`, `yellow`, `lime`, `green`, `emerald`, `teal`, `cyan`, `sky`,
`blue`, `indigo`, `violet`, `purple`, `fuchsia`, `pink`, `rose` — each with shades `50`,
`100`, `200` … `900`, `950`, plus `black`, `white` and `transparent`. Values are defined in
`oklch()`:

```css
--color-sky-500: oklch(68.5% 0.169 237.323);
--color-gray-600: oklch(44.6% 0.03 256.802);
```

(Read directly from the compiled theme.) OKLCH makes shades within a family perceptually
even and gives access to more vivid colours on wide-gamut screens; the browser maps them
to what the display can show.

The same colour name works with every colour utility:

| Prefix | Property |
|--------|----------|
| `text-` | `color` |
| `bg-` | `background-color` |
| `border-` (also `border-t-`, `border-x-`…) | `border-color` |
| `outline-` | `outline-color` |
| `ring-` | ring colour (a `box-shadow`) |
| `shadow-` | shadow colour |
| `decoration-` | `text-decoration-color` |
| `accent-` | `accent-color` (checkboxes, radios, range) |
| `caret-` | `caret-color` |
| `fill-`, `stroke-` | SVG `fill`, `stroke` |
| `from-`, `via-`, `to-` | gradient stops |

## Opacity modifiers

Add `/{amount}` to any colour utility:

```css
.bg-sky-500\/50 {
  background-color: color-mix(in srgb, oklch(68.5% 0.169 237.323) 50%, transparent);
  @supports (color: color-mix(in lab, red, red)) {
    background-color: color-mix(in oklab, var(--color-sky-500) 50%, transparent);
  }
}
```

Opacity is implemented with `color-mix()` — mixing the colour with `transparent`. The
first declaration is a fallback with the colour value inlined; browsers that support
`color-mix()` in Lab-family spaces use the second, which refers to the theme variable (so
it follows theme changes at runtime). Any percentage works: `bg-black/35`,
`text-white/[.87]`.

This only fades the colour. `opacity-50` fades the entire element *including its
children*, which is rarely what you want for a background.

## Backgrounds and gradients

```html
<div class="bg-white"></div>
<div class="bg-linear-to-r from-sky-500 to-indigo-500"></div>
<div class="bg-linear-to-b from-white via-sky-50 to-sky-100"></div>
<div class="bg-radial from-amber-200 to-transparent"></div>
<div class="bg-conic from-rose-500 via-amber-400 to-rose-500"></div>
<div class="bg-[url(/img/texture.png)] bg-cover bg-center bg-no-repeat"></div>
```

v4 renamed v3's `bg-gradient-to-r` to `bg-linear-to-r` and added radial and conic
gradients. The compiled linear gradient shows another progressive-enhancement trick:

```css
.bg-linear-to-r {
  --tw-gradient-position: to right;
  @supports (background-image: linear-gradient(in lab, red, red)) {
    --tw-gradient-position: to right in oklab;
  }
  background-image: linear-gradient(var(--tw-gradient-stops));
}
```

In browsers that support colour-space interpolation, gradients are interpolated in OKLab,
which avoids the muddy grey middle that sRGB gradients between saturated colours often
have. You can choose the space yourself: `bg-linear-to-r/oklch`, `bg-linear-to-r/srgb`.

## Borders

```html
<div class="border"></div>                <!-- 1px on all sides -->
<div class="border-2 border-t-0"></div>    <!-- 2px, none on top -->
<div class="border-b border-gray-200"></div>
<div class="divide-y divide-gray-200"></div>   <!-- borders between children -->
<div class="rounded-lg"></div>             <!-- --radius-lg: 0.5rem -->
<div class="rounded-full"></div>
```

```css
.border {
  border-style: var(--tw-border-style);
  border-width: 1px;
}
```

### v4 change: border colour defaults to `currentColor`

In v3, a bare `border` was light grey. In v4 it uses `currentColor` — the element's text
colour. We rendered `<p class="border text-red-600">` and read the computed border:

```text
border-top-color: oklch(0.577 0.245 27.325)   (the same red as the text)
border-top-style: solid
```

So **always pair `border` with a colour** (`border-gray-200`) unless you want it to match
the text. With an explicit colour, `border-2 border-gray-200` computed to
`oklch(0.928 0.006 264.531)`, gray-200 as expected.

## Rings and outlines

A **ring** is a solid box-shadow outline that doesn't affect layout — often used for focus
styles and selected states:

```css
.ring {
  --tw-ring-shadow: var(--tw-ring-inset,) 0 0 0 calc(1px + var(--tw-ring-offset-width)) var(--tw-ring-color, currentcolor);
  box-shadow: var(--tw-inset-shadow), var(--tw-inset-ring-shadow), var(--tw-ring-offset-shadow), var(--tw-ring-shadow), var(--tw-shadow);
}
```

Two more v4 changes are visible there: a bare `ring` is **1px** (it was 3px in v3), and
its colour defaults to **`currentColor`** (it was blue). Write `ring-2 ring-sky-500` or
`ring-3` if you relied on the old default.

Notice how `box-shadow` is assembled from five variables. `shadow-md`, `ring-2` and
`inset-ring` each set only their own variable, so they combine on one element instead of
overwriting each other — the same composition idea as `--tw-leading` in lesson 04.

For **focus** styles, prefer `outline` utilities: `focus-visible:outline-2
focus-visible:outline-offset-2 focus-visible:outline-sky-600`. Outlines survive Windows
forced-colours mode, while box-shadows (rings) are removed in it (Level 3 · 04).
`outline-none` compiles to `outline-style: none` — use it only when you provide a visible
replacement.

## Shadows

`shadow-2xs`, `shadow-xs`, `shadow-sm`, `shadow-md`, `shadow-lg`, `shadow-xl`,
`shadow-2xl`, and `shadow-none`; colour them with `shadow-sky-500/30`. `inset-shadow-*`
and `text-shadow-*` exist for inner and text shadows. (v4 shifted the scale down by one
step compared to v3: v3's `shadow-sm` is v4's `shadow-xs`.)

## Worked example: an alert

```html
<div class="flex gap-3 rounded-lg border border-amber-300 bg-amber-50 p-4 text-amber-900" role="status">
  <svg aria-hidden="true" class="size-5 shrink-0 fill-amber-500" viewBox="0 0 20 20">
    <path d="M10 2 1 18h18L10 2Zm0 5v5m0 3v.01" />
  </svg>
  <p class="text-sm">
    <strong class="font-semibold">Heads up:</strong>
    blend in batches — hot soup expands.
  </p>
</div>
```

One colour family, three shades: a light background (50), a mid border (300), dark text
(900) and a saturated icon (500). Picking shades this way usually gives enough contrast
for text — but check it: amber-900 on amber-50 is fine, amber-600 on amber-50 might not
be. The [HTML & CSS course's colour lesson](https://sigilipelli.github.io/html-css-mastery-path/level-2/02-color-backgrounds/)
explains contrast ratios.

## How It Actually Works

Colour utilities resolve the colour name against theme variables in the `--color-*`
namespace: `bg-sky-500` looks up `--color-sky-500`. That's why custom colours you add to
the theme (Level 2 · 01) immediately work with every colour utility and every opacity
modifier — there's no per-utility configuration.

Only the variables you use are emitted into `@layer theme`, so the full palette costs
nothing unless you use it. And because utilities reference the variable
(`var(--color-sky-500)`) rather than a literal, overriding the variable — for a theme, a
brand, or dark mode — changes every utility that uses it without regenerating CSS.

The opacity modifier's `@supports` block is also generated per utility: Tailwind targets
modern browsers (Tailwind's documentation lists Safari 16.4+, Chrome 111+ and Firefox 128+ as v4's
supported browsers), and
uses feature queries to give slightly older engines within that range a correct result.

## Common mistakes

- **Bare `border` or `ring`** expecting v3's grey/blue defaults.
- **`opacity-50` to fade a background**, fading the content too.
- **Text colours with poor contrast** — mid-range shades (400–500) on white often fail
  for body text.
- **`outline-none` without a replacement focus style.**
- **Colour as the only signal** (a red border alone for errors).

## Exercise

1. Build the alert in four variants — info (sky), success (emerald), warning (amber) and
   error (red) — changing only the colour family.
2. Put `border` on a paragraph with coloured text and observe the v4 default. Then set it
   explicitly.
3. Make a hero with a gradient from `sky-500` to `indigo-600`, then compare
   `bg-linear-to-r/srgb` with the default OKLab interpolation.
4. Style a card that gets `ring-2 ring-sky-500` when selected, and a `shadow-md` on hover.
   Confirm both apply together.
