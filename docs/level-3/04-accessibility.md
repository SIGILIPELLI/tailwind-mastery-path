---
description: "Accessible Tailwind UIs with measured evidence: palette contrast ratios, focus that survives forced colours, sr-only, skip links, targets."
---

# 04 · Accessibility with Tailwind

Tailwind doesn't make an interface accessible or inaccessible. Semantics, keyboard
support and meaningful text come from your HTML. But a utility-first workflow does make
some accessibility mistakes very easy: picking a pleasant palette shade that's too faint
to read, removing the focus outline, or hiding things in ways that also hide them from
screen readers. This lesson goes through the places where styling decisions decide
accessibility, with measurements taken while writing this course, including two
problems we found in our own earlier examples.

(For the HTML side — landmarks, headings, ARIA, keyboard support — see the
[Accessibility Foundations](https://sigilipelli.github.io/html-css-mastery-path/level-3/01-accessibility-foundations/)
lessons on the HTML & CSS course.)

## Colour contrast: measure, don't guess

WCAG 2 level AA asks for a contrast ratio of at least **4.5:1** for normal text and
**3:1** for large text (about 24px, or about 18.66px bold) and for UI component
boundaries such as input borders and focus indicators.

We rendered palette colours from Tailwind 4.3.3 in Chromium, converted each to sRGB with
a canvas, and computed the WCAG ratio. Text colour on a white background:

| Text colour | Ratio on white | Normal text (4.5) | Large text (3) |
|---|---|---|---|
| `gray-400` | 2.60 | fail | fail |
| `gray-500` | 4.84 | pass | pass |
| `gray-600` | 7.56 | pass | pass |
| `slate-500` | 4.76 | pass | pass |
| `sky-500` | 2.71 | fail | fail |
| `sky-600` | 4.02 | **fail** | pass |
| `sky-700` | 5.86 | pass | pass |
| `red-500` | 3.81 | fail | pass |
| `red-600` | 4.77 | pass | pass |
| `emerald-600` | 3.65 | fail | pass |
| `emerald-700` | 5.36 | pass | pass |
| `amber-500` | 2.13 | fail | fail |
| `amber-600` | 3.20 | fail | pass |

The ratio is symmetric, so the same numbers apply to **white text on that colour**,
which is how most buttons are built. A few patterns stand out:

- The **500** shades of bright hues (sky, amber, emerald) are too light for text and for
  white-on-colour buttons.
- **`bg-sky-600 text-white`** — about the most common button in Tailwind examples
  online — measured 4.02:1. That passes for large text only. At normal button sizes, use
  `sky-700` (5.86).
- **Yellow and amber are the hardest.** Even `amber-600` fails for normal text on white.
  Use dark text on amber backgrounds instead of white.
- `gray-400` is a placeholder colour, not a body-text colour.

On dark backgrounds, `gray-400` on `gray-900` measured 6.82 and `gray-300` on `gray-800`
9.96, while `gray-500` on `gray-950` measured only 4.16.

These figures are what our method produced. The palette is defined in `oklch`, and some
shades are outside the sRGB range, so a different conversion (or a wide-gamut display)
can give slightly different numbers. Treat values close to a threshold as "check with
your own tools", and use your browser's DevTools contrast checker on the real page,
because translucent backgrounds, gradients and images change the result.

### What we fixed in this course

While building the measurements above, we re-checked earlier lessons:

- Several button examples used `bg-sky-600 text-white`. They now use `bg-sky-700`
  (and `hover:bg-sky-800`).
- The Level 1 project's main button used `bg-emerald-600` with white text (3.65:1). It
  now uses `emerald-700`.
- The Level 2 dashboard's "Pending" badge, amber text on a translucent amber background
  over white, measured 4.48:1 — just under. It now uses `amber-800`.

None of these were visible problems when looking at the page; they only showed up when
measured. That's the lesson: check contrast as part of building, not at the end.

## Focus indicators that survive forced colours

Windows **high contrast** (forced colours) mode replaces your colours with a small
user-chosen palette and **removes box-shadows**. Tailwind's `ring-*` utilities are
box-shadows. So a common pattern for inputs, `focus:outline-none focus:ring-2`, leaves
forced-colours users with no visible focus at all.

We tested two inputs in Chromium with forced colours emulated, focused each one, and
read the computed styles:

```html
<input id="a" class="focus:outline-none focus:ring-2 focus:ring-sky-600">
<input id="b" class="focus:outline-hidden focus:ring-2 focus:ring-sky-600">
```

```text
normal mode:   a: outline none,          ring visible
               b: outline none,          ring visible
forced colors: a: outline none,          box-shadow none   <- no focus indicator
               b: outline solid 2px,     box-shadow none   <- visible outline
```

The difference is in the compiled CSS. `outline-hidden` adds a forced-colours fallback:

```css
.outline-hidden {
  outline-style: none;
  @media (forced-colors: active) {
    outline: 2px solid transparent;
    outline-offset: 2px;
  }
}
```

A *transparent* outline sounds useless, but in forced colours mode the browser repaints
outlines in the system's highlight colour, so it becomes visible. The rules:

- **Prefer outlines for focus**: `focus-visible:outline-2 focus-visible:outline-offset-2
  focus-visible:outline-sky-700`. They work everywhere.
- **If you use a ring instead**, remove the default outline with `outline-hidden`, never
  `outline-none`.
- **Use `outline-none` only** when you're deliberately replacing the outline with
  something forced colours mode keeps, such as a border change you've tested.

Earlier lessons in this course were updated to use `outline-hidden` wherever a ring
replaces the outline.

A focus indicator should also contrast with its surroundings (3:1 against adjacent
colours is the usual target). A `ring-sky-600/30` alone is too faint; in the form lessons
it's paired with a solid border colour change.

## Hiding things correctly

There are three different kinds of "hidden", and mixing them up is a common source of
bugs:

| Goal | Use | Visible? | Screen readers? | Focusable? |
|---|---|---|---|---|
| Hidden from everyone | `hidden` (or the `hidden` attribute) | no | no | no |
| Visible, but ignored by screen readers | `aria-hidden="true"` | yes | no | yes (avoid focusable content) |
| Read by screen readers only | `sr-only` | no | yes | yes |

`sr-only` compiles to the standard visually-hidden pattern:

```css
.sr-only {
  position: absolute; width: 1px; height: 1px; padding: 0; margin: -1px;
  overflow: hidden; clip-path: inset(50%); white-space: nowrap; border-width: 0;
}
```

Use it for text labels on icon-only buttons (`<span class="sr-only">Close</span>`), and
for headings that structure the page for screen readers when the design doesn't show
them. `invisible` (`visibility: hidden`) and `opacity-0` are **not** accessible hiding:
`invisible` hides from everyone but keeps the space, and `opacity-0` hides from sighted
users only — focusable content stays focusable.

### A skip link

The first element on the page should let keyboard users jump past the navigation:

```html
<a href="#main"
   class="sr-only focus:not-sr-only focus:fixed focus:start-4 focus:top-4 focus:z-50
          focus:rounded-md focus:bg-white focus:px-4 focus:py-2 focus:font-medium
          focus:text-gray-900 focus:shadow-lg focus:outline-2 focus:outline-sky-700">
  Skip to main content
</a>
…
<main id="main" tabindex="-1">…</main>
```

`focus:not-sr-only` undoes the hiding when the link receives focus, so it appears at the
top-left (top-right in RTL, thanks to `start-4`) on the first Tab press.

## Target size

Small touch targets are hard to hit for people with tremors, and for everyone on a
phone. WCAG 2.2 AA asks for at least **24×24 CSS pixels** (with exceptions, such as
links within a sentence), and many platform guidelines recommend about 44×44.

```html
<!-- An icon button: 20px icon, 40px target -->
<button class="grid size-10 place-items-center rounded-md hover:bg-gray-100">
  <svg class="size-5" aria-hidden="true">…</svg>
  <span class="sr-only">Delete</span>
</button>
```

Make the *target* big and keep the *icon* small. When the visual design needs a tiny
control, enlarge the hit area with a pseudo-element:
`relative after:absolute after:-inset-2` adds 8px of clickable area on each side
without changing the layout.

## State that isn't only colour

Colour alone can't carry meaning: about 1 in 12 men and 1 in 200 women have some form of
colour vision deficiency, according to commonly cited estimates. Pair colour with text,
icons or patterns:

- Error fields: a message, not just a red border (Level 2 · 08).
- Status badges: the word "Paid", not just a green dot.
- Links in body text: an underline, not just a colour (Preflight removes underlines, so
  add `underline` back in content).
- Charts: labels or patterns as well as colours.

## Motion and preferences

Covered in Level 2 · 09: use `motion-safe:` for decorative movement and
`motion-reduce:` to stop it. Level 4 · 04 covers `contrast-more:`, `forced-colors:` and
the other media variants in more depth.

## Worked example: an accessible icon toolbar

```html
<div role="toolbar" aria-label="Text formatting" class="inline-flex gap-1 rounded-lg border
     border-gray-300 p-1">
  <button type="button" aria-pressed="false"
          class="grid size-10 place-items-center rounded-md text-gray-700
                 hover:bg-gray-100 aria-pressed:bg-gray-900 aria-pressed:text-white
                 focus-visible:outline-2 focus-visible:outline-offset-2
                 focus-visible:outline-sky-700 forced-colors:aria-pressed:border-2">
    <svg class="size-5" viewBox="0 0 20 20" fill="currentColor" aria-hidden="true">
      <path d="M6 3h5a3.5 3.5 0 0 1 2.4 6A3.75 3.75 0 0 1 11.5 17H6zm2 2v4h3a2 2 0 0 0 0-4zm0 6v4h3.5a2 2 0 0 0 0-4z"/>
    </svg>
    <span class="sr-only">Bold</span>
  </button>
  <!-- Italic, Underline … -->
</div>
```

- Each button has a text name (`sr-only`) and a 40px target.
- The pressed state is announced (`aria-pressed`) and styled from the same attribute.
- `gray-700` icons on white and white on `gray-900` both have plenty of contrast.
- In forced colours mode, background colours are replaced, so "pressed" would vanish.
  `forced-colors:aria-pressed:border-2` adds a border that the system colour scheme keeps.
- Focus uses an outline, so it works in every mode.

A real toolbar also needs arrow-key navigation between buttons (the ARIA toolbar
pattern), which is JavaScript, not styling.

## How It Actually Works

Contrast ratio is defined from **relative luminance**: each sRGB channel is linearised,
weighted (green counts most, blue least, matching the eye's sensitivity) and summed. The
ratio is `(L1 + 0.05) / (L2 + 0.05)` with the lighter colour on top, giving values from
1:1 to 21:1. Because Tailwind's palette is in `oklch`, where the "L" is *perceptual*
lightness rather than luminance, two shades with the same step number can have quite
different contrast: `gray-500` passes on white while `sky-500` doesn't.

Forced colours mode works at the cascade level: the browser overrides colour properties
with system colours, sets `box-shadow` and `text-shadow` to none, and keeps borders and
outlines (repainted in system colours). `forced-color-adjust: none` opts an element out,
which you should only do with care. Screen readers, meanwhile, read the accessibility
tree, which is built from your HTML and ARIA, minus anything hidden with `display: none`,
`visibility: hidden` or `aria-hidden`. `sr-only` content stays in the tree because it's
only clipped, not hidden.

## Common mistakes

- **`bg-sky-600 text-white` and similar 600-shade buttons** with white text, which fail
  AA for normal-size text in several hues.
- **`focus:outline-none` with a ring.** Use `outline-hidden`.
- **`opacity-0` to hide focusable content.** It's invisible but still reachable.
- **Icon-only buttons with no text** (add an `sr-only` label).
- **Tiny targets** for icon buttons.
- **Colour-only state** in badges, errors and links.
- **Trusting a palette name** instead of measuring.

## Exercise

1. Pick five text/background pairs from your project and measure them in DevTools.
   Replace any that fail.
2. Turn on forced colours emulation (DevTools → Rendering → "Emulate CSS media feature
   forced-colors") and Tab through your Level 2 dashboard. Fix any element that loses its
   focus indicator or state.
3. Add a skip link to the Level 1 landing page and test it with the keyboard.
4. Build the toolbar and check it with a screen reader: does it announce "Bold, toggle
   button, not pressed"?
