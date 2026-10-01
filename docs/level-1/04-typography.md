---
description: "Tailwind typography utilities: font sizes with paired line heights, the text-lg/7 shorthand, weight, tracking, wrapping and line clamping."
---

# 04 · Typography Utilities

Text is most of what people read on a page, so type utilities are among the ones you'll
use most. Tailwind's type scale is designed so that the common case needs one class —
`text-lg` sets a size *and* a matching line height — while every property stays
individually adjustable. This lesson covers the scale, how the paired line heights work
under the hood, and the wrapping and truncation utilities that fix most text layout
problems.

## Font size: a scale with built-in line heights

| Class | Font size | Default line height |
|-------|-----------|---------------------|
| `text-xs` | 0.75rem | 1rem |
| `text-sm` | 0.875rem | 1.25rem |
| `text-base` | 1rem | 1.5rem |
| `text-lg` | 1.125rem | 1.75rem |
| `text-xl` | 1.25rem | 1.75rem |
| `text-2xl` | 1.5rem | 2rem |
| `text-3xl` | 1.875rem | 2.25rem |
| `text-4xl` | 2.25rem | 2.5rem |
| `text-5xl` … `text-9xl` | 3rem … 8rem | 1 |

(Line heights are written as ratios in the theme — `--text-3xl--line-height: calc(2.25 /
1.875)` — which works out to the rem values above at each size.)

Here is `text-lg` as generated:

```css
.text-lg {
  font-size: var(--text-lg);
  line-height: var(--tw-leading, var(--text-lg--line-height));
}
```

The line height uses `var(--tw-leading, …)`: *if* a `leading-*` class has set
`--tw-leading` on the element, use that; otherwise use the size's default. That's how
`text-lg leading-tight` works regardless of the order the rules appear in the stylesheet.

### Size and line height in one class

A slash sets both at once:

```css
.text-lg\/7 {
  font-size: var(--text-lg);
  line-height: calc(var(--spacing) * 7);   /* 1.75rem */
}
```

`text-sm/6`, `text-base/relaxed` (named leading) and `text-2xl/[1.1]` (arbitrary) all work.

### Leading and tracking

```css
.leading-tight {
  --tw-leading: var(--leading-tight);
  line-height: var(--leading-tight);        /* 1.25 */
}
.tracking-wide {
  --tw-tracking: var(--tracking-wide);
  letter-spacing: var(--tracking-wide);     /* 0.025em */
}
```

`leading-none` (1), `leading-tight`, `leading-snug`, `leading-normal`, `leading-relaxed`,
`leading-loose`, or a spacing number (`leading-6` = 1.5rem). Tracking runs from
`tracking-tighter` to `tracking-widest`. Large headings usually look better with slightly
negative tracking (`tracking-tight`) and tighter leading.

## Family, weight, style

```html
<p class="font-sans">system UI sans-serif stack</p>
<p class="font-serif">serif stack</p>
<code class="font-mono">monospace stack</code>
<p class="font-light">300</p> <p class="font-medium">500</p> <p class="font-semibold">600</p> <p class="font-bold">700</p>
<p class="italic">italic</p> <p class="not-italic">cancel it</p>
```

The default `font-sans` stack is the operating system's UI font with fallbacks. Using your
own fonts is a theme change (Level 2 · 02).

## Decoration, case and alignment

```html
<a class="underline decoration-2 decoration-sky-500 underline-offset-4 hover:decoration-4">Link</a>
<p class="uppercase tracking-wider text-xs">Label</p>
<p class="text-center md:text-start">Centred on phones, start-aligned from md</p>
<span class="line-through">£42</span>
```

Prefer `text-start` / `text-end` over `text-left` / `text-right` — they follow the text
direction, so they mirror in right-to-left languages.

## Wrapping, truncation and clamping

These fix most "text breaks my layout" bugs:

```css
.truncate {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}
.line-clamp-3 {
  overflow: hidden;
  display: -webkit-box;
  -webkit-box-orient: vertical;
  -webkit-line-clamp: 3;
}
.text-balance { text-wrap: balance; }    /* even line lengths for headings */
.text-pretty  { text-wrap: pretty; }     /* avoid a lonely last word in paragraphs */
.break-words  { overflow-wrap: break-word; }
.wrap-anywhere { overflow-wrap: anywhere; }
```

- `truncate` — one line with an ellipsis. Needs a width constraint, and inside a flex item
  the item also needs `min-w-0`.
- `line-clamp-{n}` — up to n lines, then an ellipsis.
- `text-balance` on headings is one of the cheapest visual improvements you can make.
- Long URLs and code in user content: `break-words` or `wrap-anywhere`.

Truncating hides information. If the full text matters, make sure it's reachable — a link
to the full item, or a `title` attribute for mouse users as a supplement (not a
replacement).

## Worked example: an article header

```html
<header class="mx-auto max-w-2xl">
  <p class="text-sm font-semibold uppercase tracking-wider text-sky-700">Techniques</p>
  <h1 class="mt-2 text-4xl font-bold tracking-tight text-balance text-gray-900 md:text-5xl">
    Why roasting tomatoes concentrates their flavour
  </h1>
  <p class="mt-4 text-lg/8 text-pretty text-gray-600">
    Heat drives off water and caramelises sugars. Here's what that means for your soup.
  </p>
  <p class="mt-6 text-sm text-gray-500">
    By Priya Rao · <time datetime="2026-09-30">30 September 2026</time>
  </p>
</header>
```

A small uppercase eyebrow label with wide tracking; a large, tight, balanced headline that
grows at `md`; a lead paragraph with generous `text-lg/8` leading; muted meta text. Every
size comes from the scale, so it harmonises with the rest of the site.

## How It Actually Works

Typography utilities are a good example of how v4 **composes** related properties through
private custom properties rather than overriding each other:

- `text-lg` writes `line-height: var(--tw-leading, <default>)`.
- `leading-tight` writes both `--tw-leading` and `line-height`.

Both rules set `line-height` and live in the same layer, so the one later in the
stylesheet would normally win. In the generated file, `.text-lg` came before
`.leading-tight` — and even if it hadn't, `text-lg`'s `var(--tw-leading, …)` would pick up
the value `leading-tight` set on the same element. We checked both class orders on an
element: `class="leading-tight text-lg"` and `class="text-lg leading-tight"` each computed
to a line height of **22.5px** (1.25 × 18px). The order of class names in the HTML never
matters; only the order of rules in the CSS does, and Tailwind controls that. The same
trick powers `--tw-tracking`, `--tw-font-weight` and the shadow and ring stacks you'll see
in lesson 05.

These `--tw-*` variables are registered with `@property` and `inherits: false`, so a
`leading-tight` on a parent doesn't leak into children through the variable — each element
only sees its own. We put `leading-loose` on a `div` and `text-lg` on a paragraph inside
it: the paragraph's line height was **28px** — `text-lg`'s own default (1.75rem) — not the
parent's loose leading.

## Common mistakes

- **`truncate` on a flex child without `min-w-0`** — the text never truncates.
- **Setting `leading-*` on a parent** and expecting a child's `text-*` to use it — the
  `--tw-leading` variable doesn't inherit; put `leading-*` on the same element as `text-*`,
  or use `text-lg/7`.
- **Uppercase for long passages** — hard to read; keep it for short labels.
- **Hard-coded `text-left`** that doesn't mirror in RTL.
- **Too many sizes on one page.** Pick four or five steps from the scale and reuse them.

## Exercise

1. Build the article header. Resize across `md` and check the headline's computed
   font size and line height in devtools.
2. Make a card title that truncates to one line, inside a flex row next to an icon. Make it
   fail without `min-w-0`, then fix it.
3. Clamp a product description to three lines with `line-clamp-3`.
4. Put `leading-loose` on a parent `div` and `text-lg` on a child paragraph. Check the
   child's line height. Then move `leading-loose` onto the child.
