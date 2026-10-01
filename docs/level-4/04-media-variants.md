---
description: "Tailwind's media variants tested in Chromium: print, contrast-more, forced-colors, motion, pointer-coarse, noscript and orientation."
---

# 04 · Print, Contrast, Motion & Other Media Variants

Breakpoints and dark mode are the media features everyone knows. Browsers expose many
more facts about the user's environment and preferences: printing, a request for more
contrast, Windows high contrast mode, reduced motion, a touch screen, scripts turned off.
Tailwind has a variant for each one. This lesson goes through them with a test of each
in Chromium (using Playwright's media emulation), and shows how to design for the
environments they describe rather than just knowing they exist.

## The variants, tested

One test page, rendered under different emulated conditions. Each row shows the class
under test and what it computed to:

```text
condition         class                                    default      under condition
print             print:hidden                             block        none
print             hidden print:block                       none         block
contrast: more    border-gray-200 contrast-more:border-gray-700   gray-200     gray-700
forced colors     forced-colors:border-2                   0px          2px
reduced motion    animate-spin motion-reduce:animate-none  spin         none
touch phone       p-2 pointer-coarse:p-4                   8px          16px
JavaScript off    hidden noscript:block                    none         block
```

Every variant applied exactly under its condition and not otherwise. What they compile
to:

| Variant | Media query |
|---|---|
| `print:` | `@media print` |
| `contrast-more:` / `contrast-less:` | `@media (prefers-contrast: more / less)` |
| `forced-colors:` | `@media (forced-colors: active)` |
| `motion-safe:` / `motion-reduce:` | `@media (prefers-reduced-motion: no-preference / reduce)` |
| `pointer-fine:` / `pointer-coarse:` / `pointer-none:` | `@media (pointer: …)` |
| `any-pointer-coarse:` and friends | `@media (any-pointer: …)` |
| `portrait:` / `landscape:` | `@media (orientation: …)` |
| `noscript:` | `@media (scripting: none)` |
| `inverted-colors:` | `@media (inverted-colors: inverted)` |

All of them stack with other variants: `md:print:hidden`, `dark:contrast-more:text-white`,
`motion-safe:hover:-translate-y-1`.

## Print

People still print invoices, recipes, boarding passes and documentation, or "print to
PDF" to save them. A page that prints its navigation, cookie banner and dark background
is a poor experience. A minimal print pass:

```html
<header class="print:hidden">…</header>
<nav class="print:hidden">…</nav>

<main class="mx-auto max-w-3xl print:max-w-none print:text-black">
  <article>
    <h2 class="print:break-after-avoid">Ingredients</h2>
    <figure class="print:break-inside-avoid">…</figure>
    <p>See <a href="https://example.com/guide"
              class="text-sky-700 underline print:after:content-['_('_attr(href)_')']">the guide</a>.</p>
  </article>
</main>

<footer class="hidden print:block print:mt-8 print:text-xs">
  Printed from example.com
</footer>
```

With print emulated, the link's `::after` computed to `" (https://example.com/guide)"`,
so the printed page shows the URL that a reader can't click. `break-after-avoid` on
headings keeps them from being stranded at the bottom of a page, and `break-inside-avoid`
stops a figure or table row from splitting across pages; both computed to `avoid`.

Other print habits: remove dark backgrounds (`print:bg-white`), make text black for
toner and contrast, and remove shadows. Use the browser's print preview, or Playwright's
`page.pdf()`, to check. (For the CSS behind this, see
[Print Styles](https://sigilipelli.github.io/html-css-mastery-path/level-4/07-print-styles/)
on the HTML & CSS course.)

## More contrast

`prefers-contrast: more` is set by users who find low-contrast interfaces hard to read
(on macOS, "Increase contrast"; similar settings exist elsewhere). The design fix is
usually small: darken borders that separate controls, darken muted text, and add
borders where you relied on a subtle background difference.

```html
<input class="border border-gray-200 contrast-more:border-gray-700 …">
<p class="text-gray-500 contrast-more:text-gray-700">Helper text</p>
<div class="bg-gray-50 contrast-more:border contrast-more:border-gray-700">…</div>
```

## Forced colours

Covered in depth in Level 3 · 04: forced-colours mode replaces your colours with a
system palette and removes shadows. In our test above the `border-gray-200` input's
border computed to `rgb(0, 0, 0)` in forced-colours mode: the system colour, not yours.
Use `forced-colors:` to **add** structure that colour used to provide: a border on a
selected tab, an outline on a pressed toggle, a visible edge on a card that was only
distinguished by its shadow.

Avoid `forced-color-adjust-none` except for things where the exact colours carry
meaning that must survive (a colour picker swatch, a brand logo).

## Motion

From Level 2 · 09: `motion-safe:` for decorative motion (opt in), `motion-reduce:` to stop
something that animates by default. In our test, `motion-reduce:animate-none` stopped the
spin under reduced motion. Replace essential motion with a non-moving equivalent, rather
than just removing it.

## Pointer and hover capabilities

- `pointer-coarse:` means the primary pointer is imprecise (a finger). Use it to enlarge
  touch targets: our `p-2 pointer-coarse:p-4` element got 16px padding in a mobile
  emulation and 8px on desktop.
- `pointer-fine:` means a mouse or trackpad.
- `any-pointer-coarse:` matches if **any** input is coarse, like a laptop with a touch
  screen.
- Remember that `hover:` is already wrapped in `@media (hover: hover)` in v4 (Level 1 · 08).

Pointer type is not screen size. A large touch-screen monitor is coarse; a small laptop
window is fine. Use pointer variants for interaction details (target size, hover
affordances) and breakpoints or container queries for layout.

## No JavaScript

`noscript:` applies when scripting is unavailable. In our test with JavaScript disabled,
`hidden noscript:block` computed to `block`; with it enabled, `none`. Use it to show a
fallback message or a plain form when an enhanced widget won't work:

```html
<div class="noscript:hidden" id="date-picker-app"></div>
<p class="hidden noscript:block text-sm text-gray-600">
  Enter the date as YYYY-MM-DD.
  <input name="date" class="mt-1 block rounded-md border border-gray-300 px-3 py-2">
</p>
```

This is a CSS-only alternative to the `<noscript>` element, useful when the markup is
rendered by a server and you want both versions in the page.

## Orientation

`portrait:` and `landscape:` match the viewport's aspect. They're occasionally useful
for full-screen experiences (a photo viewer, a game, a video player) on phones. For most
layouts, breakpoints and container queries are a better fit.

## Custom media variants

For a media feature Tailwind doesn't have a variant for, use `@custom-variant`
(Level 2 · 06):

```css
@custom-variant short (@media (height < 32rem));
@custom-variant display-standalone (@media (display-mode: standalone));
```

`short:` lets you collapse a tall header on landscape phones; `display-standalone:`
targets a page installed as a progressive web app.

## How It Actually Works

Each of these variants is a function that wraps a utility rule in a media query. The
browser evaluates media queries continuously and applies or ignores the wrapped rules as
conditions change: printing, toggling an accessibility setting, rotating a device.
Nothing is detected with JavaScript, and no class is added to the page; it's the same
mechanism as breakpoints. That's also why `noscript:` works: `scripting: none` is a media
feature the browser reports itself, so it applies even though no script ran.

Playwright (like DevTools' Rendering panel) emulates these features by telling the
browser engine to report different values, which is why the variants can be tested
automatically (Level 4 · 09).

## Common mistakes

- **Printing the whole UI**, including navigation, banners and dark backgrounds.
- **Treating `pointer-coarse` as "mobile"** or `md:` as "desktop". They answer different
  questions.
- **Removing essential motion entirely** under `motion-reduce:` instead of replacing it.
- **Ignoring `contrast-more`** when a small border change would help.
- **`forced-color-adjust-none` on whole components**, which disables the user's chosen
  colours.

## Exercise

1. Add a print pass to your Level 1 landing page: hide navigation and the signup band,
   print link URLs, and check it with print preview.
2. Turn on "increase contrast" (or emulate `prefers-contrast: more` in DevTools) and
   improve two places in the Level 2 dashboard.
3. Enlarge icon buttons with `pointer-coarse:` and confirm the change with mobile
   emulation.
4. Add a `noscript:` fallback for one JavaScript-enhanced widget and test with JavaScript
   disabled.
