---
description: "Mobile-first responsive design in Tailwind v4: the default rem breakpoints, max-* and range variants, arbitrary breakpoints, and the viewport meta tag."
---

# 09 · Responsive Design with Breakpoint Variants

Responsive design in Tailwind uses the same mechanism as hover and focus: a variant in
front of a utility. `md:grid-cols-3` means "three columns from the `md` breakpoint up".
There are no separate responsive stylesheets and no media queries to write by hand. The
mental model you need is **mobile first**, and the single most common bug comes from
forgetting it.

(For the underlying CSS, media queries and fluid layouts, see
[Responsive Design](https://sigilipelli.github.io/html-css-mastery-path/level-2/08-responsive-design/)
on the HTML & CSS course.)

## The default breakpoints

Tailwind v4 defines its breakpoints as theme variables in **rem**, not pixels. These
values come straight from the `theme.css` that ships with the package:

| Prefix | Theme variable | Value | Pixels at a 16px root font size |
|---|---|---|---|
| `sm:` | `--breakpoint-sm` | `40rem` | 640px |
| `md:` | `--breakpoint-md` | `48rem` | 768px |
| `lg:` | `--breakpoint-lg` | `64rem` | 1024px |
| `xl:` | `--breakpoint-xl` | `80rem` | 1280px |
| `2xl:` | `--breakpoint-2xl` | `96rem` | 1536px |

Each one compiles to a `min-width` query in the modern range syntax:

```css
@media (width >= 48rem) {
  .md\:p-3 { padding: calc(var(--spacing) * 3); }
}
```

Because the queries are in rem, a user who raises their browser's default font size gets
the narrower layout sooner. That's usually what you want: bigger text needs more room
per column.

## Mobile first: unprefixed means "every size"

An unprefixed utility applies at **all** widths. A prefixed one applies **from that
breakpoint up**. So you write the small-screen style first, then override it as space
grows:

```html
<div class="grid grid-cols-1 gap-4 sm:grid-cols-2 lg:grid-cols-4">
  …
</div>
```

One column on phones, two from 640px, four from 1024px. You never write "phone" styles
with a prefix.

The trap is reading `sm:` as "on small screens". `sm:hidden` does **not** hide something
on phones. It hides it from 640px *up*, which is the opposite. To hide something on
phones only, write `hidden sm:block`. To show something on phones only, write
`sm:hidden`. When a responsive class "does the reverse of what I meant", this is nearly
always why.

## Targeting a range: `max-*` and stacked breakpoints

Sometimes you want a style *only* below a size or *only* between two sizes. Every
breakpoint has a `max-` form, and you can stack two to make a range:

```html
<nav class="max-md:hidden">…</nav>               <!-- only below 48rem: hidden -->
<aside class="md:max-lg:flex hidden">…</aside>   <!-- flex only between md and lg -->
```

Compiled:

```css
@media (width < 48rem) {
  .max-md\:hidden { display: none; }
}
@media (width >= 48rem) {
  @media (width < 64rem) {
    .md\:max-lg\:flex { display: flex; }
  }
}
```

`max-md:` is a strict `<`, so at exactly 48rem the `md:` styles apply and the `max-md:`
ones don't. The two never overlap.

Use `max-*` sparingly. It's handy for "undo this one thing on phones", but if you find
yourself writing styles for every range, mobile-first overrides are almost always
shorter and easier to follow.

## One-off breakpoints

For a single component that needs a break at an unusual width, use an arbitrary value:

```html
<div class="flex flex-col min-[900px]:flex-row">…</div>
<p class="max-[600px]:text-sm">…</p>
```

These compile to `@media (width >= 900px)` and `@media (width < 600px)`. If the same
value turns up in several places, it should be a named breakpoint instead.

## Custom breakpoints

Breakpoints are theme variables, so you add or change them in your CSS with `@theme`
(covered fully in Level 2 · 01):

```css title="src/app.css"
@import "tailwindcss";

@theme {
  --breakpoint-xs: 30rem;
  --breakpoint-3xl: 120rem;
}
```

That immediately creates `xs:` and `3xl:` variants, and the compiler puts them in the
right order with the others. With this theme, the output went `xs` (30rem), then `sm`,
`md`, `lg`, `xl`, `2xl`, then `3xl` (120rem). Ordering matters: for media queries of
equal specificity the later rule wins, so a larger breakpoint must come later in the
file. Tailwind sorts them by value for you.

## The viewport meta tag is not optional

Breakpoints test the width of the **layout viewport**. Phone browsers assume a page
without a viewport meta tag was designed for desktop and lay it out about 980px wide,
then scale it down. We checked this in Chromium with mobile emulation at a 390px-wide
device:

```text
with <meta name="viewport" content="width=device-width, initial-scale=1">
  innerWidth = 390   md:hidden element -> display: block
without the meta tag
  innerWidth = 980   md:hidden element -> display: none
```

Without the tag, the phone got the `md` and `sm` layouts, shrunk to fit. Every page
needs:

```html
<meta name="viewport" content="width=device-width, initial-scale=1">
```

## Worked example: a responsive pricing section

```html
<section class="mx-auto max-w-6xl px-4 py-12 sm:px-6 lg:py-20">
  <h2 class="text-center text-2xl font-bold tracking-tight sm:text-3xl lg:text-4xl">
    Simple pricing
  </h2>
  <p class="mx-auto mt-3 max-w-prose text-center text-gray-600">
    Start free. Upgrade when your team grows.
  </p>

  <div class="mt-10 grid gap-6 md:grid-cols-3 md:items-start">
    <article class="rounded-2xl border border-gray-200 p-6">
      <h3 class="font-semibold">Starter</h3>
      <p class="mt-2 text-3xl font-bold">Free</p>
      <a href="/signup" class="mt-6 block rounded-md border border-gray-300 px-4 py-2
                               text-center font-medium hover:bg-gray-50">Get started</a>
    </article>

    <article class="order-first rounded-2xl border-2 border-sky-600 p-6 shadow-lg
                    md:order-none md:-mt-4 md:pb-10">
      <p class="text-xs font-semibold tracking-wide text-sky-700 uppercase">Most popular</p>
      <h3 class="mt-1 font-semibold">Team</h3>
      <p class="mt-2 text-3xl font-bold">$12 <span class="text-base font-normal
                                                    text-gray-500">/ user / month</span></p>
      <a href="/signup?plan=team" class="mt-6 block rounded-md bg-sky-700 px-4 py-2
                               text-center font-medium text-white hover:bg-sky-800">Start trial</a>
    </article>

    <article class="rounded-2xl border border-gray-200 p-6">
      <h3 class="font-semibold">Enterprise</h3>
      <p class="mt-2 text-3xl font-bold">Custom</p>
      <a href="/contact" class="mt-6 block rounded-md border border-gray-300 px-4 py-2
                                text-center font-medium hover:bg-gray-50">Talk to sales</a>
    </article>
  </div>
</section>
```

What each responsive decision does:

- **Padding and heading size grow** with the screen: `px-4 sm:px-6`, `text-2xl sm:text-3xl
  lg:text-4xl`. Small screens get tighter spacing and smaller headings.
- **The grid** is one column until `md`, then three.
- **The featured plan moves first on phones** (`order-first`), so the plan you most want
  people to see isn't the third card down a long scroll. From `md` it returns to source
  order (`md:order-none`) and is raised a little with a negative margin.
- `md:items-start` stops the three cards stretching to equal height, so the featured
  card's extra padding visibly sticks out.

(The prices are example values, not real pricing.)

A caution on `order-*`: it changes visual order, not the order screen readers and the
Tab key follow. Here that's fine, because reading the plans in source order still makes
sense. Don't use it where the visual and DOM orders would contradict each other.

## How It Actually Works

A breakpoint variant is generated from the theme. For every `--breakpoint-*` variable,
Tailwind registers a variant that wraps the rule in `@media (width >= <value>)`, plus a
`max-*` variant that wraps it in `@media (width < <value>)`. `min-[…]` and `max-[…]` do
the same with the bracket value. Stacking (`md:max-lg:`) nests the media queries, and
nested `@media` rules behave like an AND.

There's no runtime involved. The browser re-evaluates media queries whenever the
viewport changes, and rules start or stop applying. Since the media query adds no
specificity, the deciding factor between `p-2` and `md:p-3` is source order: Tailwind
emits all unprefixed utilities first and then each breakpoint's rules in ascending order,
so a bigger breakpoint always overrides a smaller one. That's why you never need
`!important` for responsive overrides, and why a custom breakpoint must be a theme
variable (so it gets sorted) rather than a hand-written media query in your CSS.

The query uses media query level 4 range syntax (`width >= 48rem`). It's supported in all
current major browsers. If you need very old browsers, check your support matrix.

## Common mistakes

- **Reading `sm:` as "on phones".** Prefixes mean "from this width up". Put phone styles
  unprefixed.
- **Missing viewport meta tag.** Phones render the desktop breakpoints, shrunk.
- **Writing a separate set of classes for every breakpoint.** Only override what changes.
  `p-4 sm:p-4 md:p-4` is three ways of saying `p-4`.
- **Assuming 768px is fixed.** Breakpoints are in rem, so they move with the user's font
  size setting. Test with a larger default font size.
- **Hand-written `@media` rules for custom breakpoints.** They won't be ordered with
  Tailwind's variants. Add a `--breakpoint-*` theme variable instead.
- **Using `order-*` so that the visual order contradicts the reading order.** Keyboard
  and screen reader users follow the DOM.
- **Breakpoints for component-level layout.** A card in a narrow sidebar on a wide screen
  gets the wide styles. That's what container queries solve (Level 2 · 07).

## Exercise

1. Build a three-column feature grid that's one column on phones, two from `sm` and three
   from `lg`. Use the browser's device toolbar to check each range.
2. Add a nav that shows a hamburger button on phones and a horizontal link list from
   `md`, using only `hidden`/`flex` with breakpoint prefixes.
3. Remove the viewport meta tag, view it with mobile emulation, and note what changes.
   Put it back.
4. Add `--breakpoint-xs: 30rem` to your theme and use `xs:` for the two-column change
   instead of `sm:`.
5. Set your browser's default font size to 24px and see which breakpoint each range now
   starts at. Explain why using rem.
