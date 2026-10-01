---
description: "Layout with Tailwind: display, flex direction and alignment, flex-1 vs flex-auto measured, the truncate plus min-w-0 fix, and positioning."
---

# 06 · Layout & Flexbox Utilities

Tailwind's layout utilities are thin wrappers over CSS layout properties, so the real
skill is knowing Flexbox and Grid; the class names come quickly after that. This lesson
maps Flexbox to its utilities, measures the handful whose behaviour surprises people, and
covers display, positioning and z-index. (For the underlying algorithm, see
[Flexbox Fundamentals](https://sigilipelli.github.io/html-css-mastery-path/level-2/03-flexbox-fundamentals/)
and [Flexbox in Depth](https://sigilipelli.github.io/html-css-mastery-path/level-2/04-flexbox-in-depth/)
in the HTML & CSS course.)

## Display

`block`, `inline-block`, `inline`, `flex`, `inline-flex`, `grid`, `inline-grid`,
`contents`, `flow-root`, `table`, `hidden` (`display: none`). `sr-only` hides something
visually but keeps it for screen readers (Level 3 · 04).

## Flex containers

| Utility | CSS |
|---------|-----|
| `flex` | `display: flex` |
| `flex-row` / `flex-col` / `flex-row-reverse` | `flex-direction` |
| `flex-wrap` / `flex-nowrap` | `flex-wrap` |
| `justify-start` `center` `end` `between` `around` `evenly` | `justify-content` |
| `items-start` `center` `end` `baseline` `stretch` | `align-items` |
| `content-*` | `align-content` (multi-line) |
| `gap-4`, `gap-x-6`, `gap-y-2` | `gap` |

```html
<header class="flex flex-wrap items-center gap-x-6 gap-y-2 p-4">
  <a class="font-bold" href="/">Weeknight Kitchen</a>
  <nav class="ms-auto" aria-label="Main">
    <ul class="flex gap-4">
      <li><a href="/recipes">Recipes</a></li>
      <li><a href="/about">About</a></li>
    </ul>
  </nav>
</header>
```

`ms-auto` (`margin-inline-start: auto`) pushes the nav to the end — an auto margin
absorbs free space before `justify-content` does.

## Flex items

| Utility | CSS | Use |
|---------|-----|-----|
| `flex-1` | `flex: 1` (= `1 1 0%`) | Equal-width columns, ignoring content |
| `flex-auto` | `flex: auto` (= `1 1 auto`) | Start at content size, share leftover |
| `flex-initial` | `flex: 0 1 auto` | The CSS default |
| `flex-none` | `flex: none` | Rigid: never grow or shrink |
| `grow`, `grow-0` | `flex-grow` | |
| `shrink`, `shrink-0` | `flex-shrink` | `shrink-0` on icons and avatars |
| `basis-64`, `basis-1/3` | `flex-basis` | |
| `order-first`, `order-last`, `order-2` | `order` | Visual only — see below |
| `self-start`, `self-center`… | `align-self` | |

### `flex-1` vs `flex-auto`, measured

Three items with different amounts of text in a 600px row:

```text
flex-1    -> [200, 200, 200]
flex-auto -> [126, 359, 115]
```

`flex-1` sets the basis to 0, so all the space is shared equally regardless of content.
`flex-auto` starts from each item's content width and only shares the leftover, so the
longer item stays wider. Choose deliberately — most "why are my columns unequal?" questions
are a `flex-auto` that should have been `flex-1`, or the reverse.

## The truncate trap, measured

A notification row: a 24px icon, then a title that should truncate. Three versions in a
300px flex row, measuring the text box:

```html
<!-- A: truncate on the flex item itself -->
<div class="flex w-[300px] items-center gap-2">
  <span class="size-6 shrink-0"></span>
  <p class="truncate">A very long notification title that should be truncated nicely</p>
</div>

<!-- B: truncate inside a wrapper div -->
<div class="flex …"><span …></span><div><p class="truncate">…</p></div></div>

<!-- C: same, with min-w-0 on the wrapper -->
<div class="flex …"><span …></span><div class="min-w-0"><p class="truncate">…</p></div></div>
```

```text
A: text width 268, row scrollWidth 300   (truncated correctly)
B: text width 427, row scrollWidth 459   (overflowed the row)
C: text width 268, row scrollWidth 300   (truncated correctly)
```

In A, `truncate` sets `overflow: hidden` on the flex item itself, which also removes its
automatic minimum size. In B, the wrapper `div` is the flex item, and flex items default
to `min-width: auto` — "never narrower than my content" — so it grew to the full text
width and overflowed. **`min-w-0` on the flex item** (C) is the fix. The same applies to
code blocks, long URLs and tables inside flex children.

## Visual order and accessibility

`order-*`, `flex-row-reverse` and `flex-col-reverse` change what you see but not the DOM
order — which is the keyboard and screen-reader order. Use them only when order doesn't
carry meaning or when no focusable elements are affected.

## Positioning and stacking

```html
<div class="relative">
  <img class="rounded-lg" src="soup.jpg" alt="…" width="600" height="400">
  <span class="absolute top-2 end-2 rounded bg-white px-2 py-1 text-xs font-semibold">New</span>
</div>

<header class="sticky top-0 z-10 bg-white/90 backdrop-blur">…</header>
```

`static`, `relative`, `absolute`, `fixed`, `sticky`; offsets `inset-0`, `inset-x-0`,
`top-4`, `start-4`, `end-2` (logical — mirror in RTL); `z-0` to `z-50` and `z-auto`,
arbitrary `z-[60]`. The CSS rules are exactly the plain-CSS ones: an absolute element
positions against its nearest positioned ancestor, and `z-index` only competes within a
stacking context. If a `z-50` element still hides behind something, a parent with
`opacity`, `transform` or `filter` has probably created a stacking context —
[this lesson](https://sigilipelli.github.io/html-css-mastery-path/level-2/07-positioning-stacking/)
explains the rules. `isolate` (`isolation: isolate`) creates one deliberately.

## Worked example: a media object with actions

```html
<article class="flex items-start gap-4 rounded-lg border border-gray-200 p-4">
  <img class="size-12 shrink-0 rounded-full" src="priya.jpg" alt="" width="48" height="48">
  <div class="min-w-0 flex-1">
    <div class="flex items-baseline gap-2">
      <h3 class="truncate font-semibold">Priya Rao</h3>
      <time class="shrink-0 text-xs text-gray-500" datetime="2026-09-30">2 h ago</time>
    </div>
    <p class="mt-1 text-sm text-gray-700">
      Made this twice this week — roasting the garlic whole is the trick.
    </p>
  </div>
  <button type="button" class="shrink-0 rounded-md px-2 py-1 text-sm hover:bg-gray-100">Reply</button>
</article>
```

Each class answers a layout question: the avatar and button must never shrink
(`shrink-0`), the middle column takes the rest (`flex-1`) and is allowed to be narrower than
its content (`min-w-0`) so the name can truncate, and name and time share a text baseline
(`items-baseline`).

## How It Actually Works

Layout utilities are among the simplest in Tailwind: most compile to a single declaration
(`.flex { display: flex; }`, `.flex-1 { flex: 1; }`, `.shrink-0 { flex-shrink: 0; }`),
with no variables involved — there's nothing to theme about `display: flex`. Values that
*are* themeable — `gap`, `basis`, `inset`, `top` — use `var(--spacing)` multiples like the
spacing utilities, so the whole layout scale moves together.

The `order-first` utility shows a small implementation detail: it compiles to
`order: -9999` rather than a small negative number, so it beats any reasonable explicit
order value.

## Common mistakes

- **`truncate` or long content in a flex child without `min-w-0`.**
- **`flex-auto` when you wanted equal columns** (`flex-1`), or the reverse.
- **Squashed icons** — add `shrink-0`.
- **`space-x-*` in rows that wrap** — use `gap-*`.
- **Raising `z-*` values** to fix a stacking-context problem.
- **`order-*` on interactive elements**, scrambling keyboard order.

## Exercise

1. Build the media-object card. Delete `min-w-0` and use a very long name to see the
   overflow; put it back.
2. Make a header with a logo, a nav and a "Sign in" button, where the nav and button sit at
   the end, wrapping neatly on narrow screens.
3. Build three pricing columns with `flex-1`, then switch to `flex-auto` with different
   text lengths and describe the difference.
4. Overlay a "Sold out" badge on the top-end corner of a product image with `relative` and
   `absolute`.
