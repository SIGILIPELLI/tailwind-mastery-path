---
description: "Modern CSS through Tailwind v4, measured: gradient interpolation, text-wrap balance, text shadows, masks, 3D and anchor positioning."
---

# 05 · Modern CSS Through Tailwind

CSS has gained a remarkable amount in the last few years, and Tailwind v4 exposes much of
it directly: container queries, `:has()`, subgrid and `@starting-style` were covered
earlier. This lesson covers more: gradient interpolation, balanced text, text shadows,
masks, 3D transforms, and anchor positioning (through arbitrary properties). Each is
shown with what Tailwind generated and what Chromium 149 rendered, plus how to use it
safely when not every browser has it yet.

## Gradients and colour interpolation

v4's gradient utilities are `bg-linear-*`, `bg-radial` and `bg-conic`, with colour stops
from `from-*`, `via-*` and `to-*`. An optional modifier picks the **colour space the
gradient is interpolated in**:

```html
<div class="bg-linear-to-r/srgb from-blue-600 to-yellow-400"></div>
<div class="bg-linear-to-r from-blue-600 to-yellow-400"></div>          <!-- default -->
<div class="bg-linear-to-r/oklch from-blue-600 to-yellow-400"></div>
```

The plain class already isn't sRGB. Compiled:

```css
.bg-linear-to-r {
  --tw-gradient-position: to right;
  @supports (background-image: linear-gradient(in lab, red, red)) {
    --tw-gradient-position: to right in oklab;
  }
  background-image: linear-gradient(var(--tw-gradient-stops));
}
```

So by default, browsers that support interpolation spaces blend in **oklab**. Older
ones fall back to the classic behaviour. We screenshotted each 400px bar and read the
pixel at the midpoint:

```text
/srgb    midpoint rgb(137, 146, 91)    muddy olive
default  midpoint rgb(140, 162, 172)   greyish blue (oklab)
/oklch   midpoint rgb(0, 198, 156)     vivid teal
```

Blue to yellow passes through grey in sRGB and oklab, because they're roughly opposite
colours and the straight line between them crosses the neutral axis. `oklch` interpolates
**hue** around the colour wheel instead, keeping colourfulness, so the middle is a
saturated teal. Which one is "right" depends on the design. For gradients between
distant hues, try `/oklch`; for subtle gradients between similar colours, the default
is fine. `/[in_hsl_longer_hue]` and other arbitrary interpolation methods also work.

Angles and positions: `bg-linear-45`, `bg-linear-to-br`, `bg-radial-[at_25%_25%]`,
`bg-conic-180`.

## Balanced and pretty text

`text-balance` (`text-wrap: balance`) evens out line lengths in short text such as
headings. We measured the line widths of a `text-3xl` heading in a 300px column:

```text
"Weekend sailing courses on the south coast"
  default        228px, 292px, 78px     <- "coast" alone on the last line
  text-balance   228px, 206px, 164px
"How we cut our CSS bundle in half"
  default        283px, 188px
  text-balance   216px, 255px
```

It doesn't always change anything: for some headings the default breaks were already
even, and the measured widths were identical. Browsers limit balancing to a few lines,
so use it on headings, captions and short labels, not paragraphs.

`text-pretty` (`text-wrap: pretty`) is for body text: it asks the browser to avoid
leaving a single short word on the last line of a paragraph, with less cost than full
balancing. Both are progressive enhancements; unsupported browsers just wrap normally.

## Text shadows

```html
<h1 class="text-5xl font-bold text-white text-shadow-lg text-shadow-black/30">…</h1>
```

`text-shadow-md` compiled to three layered shadows using
`var(--tw-text-shadow-color, rgb(0 0 0 / 0.1))`, so a colour utility like
`text-shadow-sky-500/50` recolours all layers. The practical use is legibility of text
over images; as decoration, text shadows date quickly.

## Masks

Mask utilities fade or cut out an element without editing the image:

```html
<!-- fade the bottom half of a long preview -->
<div class="max-h-48 overflow-hidden mask-b-from-50%">…long text…</div>

<!-- soft circular vignette on a photo -->
<img class="mask-radial-from-40%" src="…" alt="…">
```

They compile to `mask-image` built from internal variables, combined with
`mask-composite: intersect`, so several mask utilities on one element combine (a bottom
fade *and* a radial vignette). A masked region is still there for screen readers and
text selection; masking is purely visual.

## 3D transforms

```html
<div class="group perspective-distant">
  <div class="relative size-48 transition-transform duration-500 transform-3d
              group-hover:rotate-y-180">
    <div class="absolute inset-0 grid place-items-center rounded-xl bg-sky-700 text-white
                backface-hidden">Front</div>
    <div class="absolute inset-0 grid place-items-center rounded-xl bg-gray-900 text-white
                rotate-y-180 backface-hidden">Back</div>
  </div>
</div>
```

A card that flips on hover: `perspective-*` on the parent sets the viewing distance,
`transform-3d` (`transform-style: preserve-3d`) lets children live in 3D, and
`backface-hidden` hides each face when it's turned away. Note that 3D rotations
(`rotate-y-180`) use the `transform` property, not `rotate` (Level 2 · 09), and they're
covered by `transition-transform`. Add `motion-safe:` to the hover rotation; a flip is a
large movement. And remember that hover doesn't exist on touch screens, so the back face
needs another way to be shown there (a button, or `focus-within`).

## Anchor positioning (with arbitrary properties)

CSS anchor positioning lets an element position itself relative to another element
anywhere on the page, without JavaScript measuring. Tailwind 4.3 has no dedicated
utilities for it, but arbitrary properties and values cover it:

```html
<button popovertarget="menu" class="[anchor-name:--menu]">Options</button>

<div id="menu" popover
     class="[position-anchor:--menu] top-[anchor(bottom)] left-[anchor(left)] mt-1 m-0
            w-48 rounded-lg bg-white p-1 shadow-lg ring-1 ring-black/5">
  …
</div>
```

In Chromium 149 our test (with an absolutely positioned panel) placed the panel's left
edge exactly at the button's left (100px) and its top 4px below the button's bottom
(148px vs 144px), the `mt-1` gap. Anchor positioning support is still uneven across
browsers, so this is the textbook case for a feature check.

## Feature checks with `supports-*`

`supports-[…]:` wraps a utility in `@supports`, and `not-supports-[…]:` in its negation:

```html
<div class="absolute top-full left-0
            supports-[anchor-name:--x]:[position-anchor:--menu]
            supports-[anchor-name:--x]:top-[anchor(bottom)]">…</div>
```

```css
@supports (anchor-name:--x) {
  .supports-\[anchor-name\:--x\]\:block { display: block; }
}
```

The pattern is **fallback first, enhancement in `supports-*`**. Many modern features
need no check at all because unsupported declarations are simply ignored (`text-balance`,
`field-sizing-content`). Use `supports-*` when the fallback needs *different* declarations
(a different position, a different display), as with anchor positioning.

## Choosing what to adopt

A quick rule for each new feature:

- **Ignored safely when unsupported** (text-wrap, most new colour functions inside
  `@supports` like Tailwind's gradients): use freely.
- **Breaks layout when unsupported** (anchor positioning, some container query units
  used for critical sizing): use with a `supports-*` fallback.
- **Breaks functionality when unsupported**: don't rely on it yet, or polyfill.

Check support for your actual audience (your analytics, plus a compatibility table such
as MDN's or caniuse) rather than guessing.

## How It Actually Works

Tailwind is a thin layer here. For gradients it builds `--tw-gradient-stops` from your
`from`/`via`/`to` utilities and puts the interpolation method into
`--tw-gradient-position`, guarded by an `@supports` check so older browsers get a valid
gradient. For text wrapping, shadows, masks and 3D, each utility maps to one or two
properties, plus internal variables when several utilities need to combine (masks, text
shadow colours). The browser does all the work, which is why behaviour such as how
`text-balance` chooses breaks, or which colour the oklch midpoint lands on, depends on
the browser and its version.

For anything without a utility, arbitrary properties (`[anchor-name:--menu]`) and values
(`top-[anchor(bottom)]`) pass the CSS through unchanged. Tailwind doesn't need to know
about a feature for you to use it.

## Common mistakes

- **Assuming `bg-linear-*` interpolates in sRGB.** The default is oklab; add `/srgb` to
  match a design tool that uses sRGB.
- **`text-balance` on paragraphs.** It's for short text.
- **Hover-only 3D flips** with no touch or keyboard alternative.
- **Using anchor positioning without a fallback** when your audience includes browsers
  that lack it.
- **Masks to hide content** from users. Masked content is still read and selectable.
- **Waiting for a utility** before using a new CSS feature. Arbitrary properties work now.

## Exercise

1. Make three versions of a gradient between two distant hues (`/srgb`, default,
   `/oklch`) and decide which suits your design.
2. Add `text-balance` to every heading in the Level 1 landing page. Use DevTools to find
   one that changed and one that didn't.
3. Build the flip card with `motion-safe:` and a keyboard-accessible way to see the back.
4. Build an anchored popover menu with a `supports-*` fallback, and test it with the
   anchor rules removed to simulate an unsupported browser.
