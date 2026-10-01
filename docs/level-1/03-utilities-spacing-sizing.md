---
description: "How Tailwind utility names map to CSS: the spacing scale built on one variable, padding, margin, width, height, size and fractions."
---

# 03 · Utility Basics: Spacing, Sizing & the Scale

Most of the classes you'll write in any Tailwind project are spacing and sizing: padding,
margin, gaps, widths and heights. They follow one naming pattern and one scale, and once
you know both you can guess almost any class name correctly. This lesson decodes the
pattern and shows, from the compiler's actual output, what each class becomes.

## The naming pattern

Most utilities are `{property}-{value}`, with optional side or axis letters:

| Prefix | Property | Sides / axes |
|--------|----------|--------------|
| `p` | padding | `pt` `pr` `pb` `pl` (top/right/bottom/left), `px` `py` (inline/block axes), `ps` `pe` (inline start/end) |
| `m` | margin | same pattern: `mt`, `mx`, `my`, `ms`, `me`… |
| `gap` | gap | `gap-x`, `gap-y` |
| `w`, `h` | width, height | `min-w`, `max-w`, `min-h`, `max-h` |
| `size` | width **and** height | — |
| `inset`, `top`, `left`… | position offsets | `inset-x`, `inset-y` |

So `px-6` is horizontal padding at step 6, `mt-2` is top margin at step 2, `gap-x-4` is
column gap at step 4.

## The spacing scale is one variable

In v4, the whole spacing scale is derived from a single theme variable, `--spacing`, which
defaults to `0.25rem` (4px at the default browser font size). A number after a spacing
utility is a **multiplier**. Here is what the compiler generated for a selection:

```css
.p-4    { padding: calc(var(--spacing) * 4); }      /* 1rem */
.p-4\.5 { padding: calc(var(--spacing) * 4.5); }    /* 1.125rem */
.p-13   { padding: calc(var(--spacing) * 13); }     /* 3.25rem */
.w-128  { width: calc(var(--spacing) * 128); }      /* 32rem */
.-mt-2  { margin-top: calc(var(--spacing) * -2); }  /* -0.5rem */
.gap-4  { gap: calc(var(--spacing) * 4); }
.size-10 {
  width: calc(var(--spacing) * 10);
  height: calc(var(--spacing) * 10);
}
```

Three consequences:

1. **Any multiple works**, including ones that weren't in older fixed scales (`p-13`,
   `w-128`) and quarter steps like `p-4.5`. Steps must be multiples of 0.25: in our test
   `p-4.25` and `p-4.75` were generated, `p-4.3` silently wasn't.
2. **Negative values** are a leading minus: `-mt-2`, `-translate-x-4`.
3. **Retuning the scale is one line.** Set `--spacing: 0.3rem` in your theme (Level 2 · 02)
   and every `p-*`, `m-*`, `gap-*`, `w-*`, `h-*` changes proportionally. We tried it: with
   `--spacing: 0.3rem`, an element with `p-6` computed to `28.8px` of padding (6 × 0.3rem ×
   16px) instead of 24px.

A useful mental conversion: **step × 4 = pixels** at default settings. `p-6` is 24px,
`gap-3` is 12px, `w-64` is 256px.

## Other kinds of values

Not every value is a spacing multiple:

```css
.w-1\/2        { width: calc(1 / 2 * 100%); }   /* fractions: w-1/3, w-2/5, ... */
.w-screen      { width: 100vw; }
.h-dvh         { height: 100dvh; }              /* also h-svh, h-lvh, h-screen (100vh) */
.max-w-prose   { max-width: 65ch; }
.max-w-md      { max-width: var(--container-md); }  /* --container-md: 28rem */
.mx-auto       { margin-inline: auto; }
.p-\[13px\]    { padding: 13px; }               /* arbitrary value, Level 2 · 04 */
```

Keywords you'll use constantly: `w-full` (100%), `w-auto`, `w-fit` (`fit-content`),
`min-w-0` (essential inside flex and grid — see
[the min-width: auto trap](https://sigilipelli.github.io/html-css-mastery-path/level-2/04-flexbox-in-depth/)),
`h-full`, `min-h-dvh`.

The `max-w-*` named sizes (`max-w-sm`, `md`, `lg`, `xl`, `2xl` … `7xl`) come from the
`--container-*` theme variables and are the usual way to limit a column's width.
`max-w-prose` limits text to a comfortable 65 characters.

## `space-*` vs `gap`

To put space between children, there are two tools:

```html
<div class="flex gap-4">…</div>        <!-- gap: only works in flex and grid containers -->
<div class="space-y-4">…</div>          <!-- margins between children, any layout -->
```

`space-x-4` compiles to a rule on the children:

```css
:where(.space-x-4 > :not(:last-child)) {
  --tw-space-x-reverse: 0;
  margin-inline-start: calc(calc(var(--spacing) * 4) * var(--tw-space-x-reverse));
  margin-inline-end: calc(calc(var(--spacing) * 4) * calc(1 - var(--tw-space-x-reverse)));
}
```

It adds an end margin to every child except the last. Prefer `gap` in flex and grid
layouts: it doesn't depend on children being visible, handles wrapping correctly (margins
break when items wrap to a new line) and doesn't interfere with margins you set on the
children yourself. `space-y-*` is still handy for vertical rhythm in plain block content,
like paragraphs in an article.

## Worked example: a profile card

```html
<article class="mx-auto max-w-sm rounded-xl border border-gray-200 p-6">
  <img class="size-16 rounded-full" src="avatar.jpg" alt="" width="64" height="64">
  <h2 class="mt-4 text-lg font-semibold">Priya Rao</h2>
  <p class="mt-1 text-sm text-gray-600">Recipe developer</p>
  <div class="mt-6 flex gap-3">
    <a class="rounded-md px-4 py-2 text-sm font-medium" href="/priya">Profile</a>
    <a class="rounded-md px-4 py-2 text-sm font-medium" href="/priya/recipes">Recipes</a>
  </div>
</article>
```

Reading it as spacing decisions: the card is centred (`mx-auto`) and never wider than
24rem (`max-w-sm`); it has 1.5rem padding (`p-6`); vertical rhythm between blocks is
`mt-1`, `mt-4`, `mt-6` — small gaps within a group, larger ones between groups; buttons
are separated by a 0.75rem `gap-3`. Every value comes from one scale, which is why
Tailwind layouts tend to look evenly spaced without a designer checking each margin.

## How It Actually Works

When the compiler sees `p-4.5`, it:

1. **Splits the candidate** into a utility root and a value: root `p`, value `4.5`. Roots
   are matched longest-first, so `px-4` is root `px`, not `p` with value `x-4`, and
   `space-x-4` is root `space-x`.
2. **Resolves the value.** For spacing roots, a bare number is valid if it's a multiple of
   0.25; it becomes `calc(var(--spacing) * 4.5)`. Named values (`auto`, `full`, `screen`,
   `px`) and fractions are looked up in built-in tables; theme keys like `md` resolve to
   `--container-md`.
3. **Emits the declaration** for that root: `p` → `padding`, `px` → `padding-inline`,
   `mt` → `margin-top`.

Notice `px-6` generates `padding-inline` and `mx-auto` generates `margin-inline` — logical
properties, which mirror automatically in right-to-left layouts (Level 4 · 03). And
because the result is `calc(var(--spacing) * N)` rather than a hard-coded length, the
value stays connected to the theme: devtools shows you the variable, and changing the
variable at runtime changes the layout.

## Common mistakes

- **Guessing pixel values**: `p-16` is 64px, not 16px.
- **Using `space-*` in wrapping flex rows** — use `gap`.
- **Arbitrary values for things the scale already covers** (`p-[16px]` is just `p-4`).
- **`w-screen` for full width**, which includes the scrollbar and causes horizontal scroll;
  use `w-full`.
- **`h-screen` on mobile**, taller than the visible area when browser toolbars show; use
  `h-dvh` or `min-h-dvh`.
- **Forgetting `min-w-0`** on flex children that contain long text or code.

## Exercise

1. Without looking anything up, write the classes for: 24px padding on all sides, 8px top
   margin, a 12px gap between flex items, a 40px square avatar, and a column limited to 42rem
   and centred. Compile and check.
2. Build the profile card. Then set `--spacing: 0.3rem` in a `@theme` block (preview of Level
   2 · 01) and watch every spacing value grow proportionally.
3. Make a flex row of tags with `space-x-2` and `flex-wrap`, narrow the window until it wraps,
   and compare with `gap-2`.
