---
description: "Container queries in Tailwind v4: @container, @sm to @7xl, range and named queries, cq units, and the inline-size containment pitfalls."
---

# 07 · Container Queries in Tailwind

Breakpoints answer "how wide is the **screen**?". Components usually care about something
else: "how much room do **I** have?". A product card in a three-column grid on a wide
monitor might have less space than the same card alone on a tablet. With breakpoints,
you'd have to know where the card will be placed and write different classes for each
place. Container queries let the card adapt to its own container, so the same markup
works anywhere.

Container queries are built into Tailwind v4 (in v3 they needed a plugin). For the
underlying CSS, see
[Container Queries](https://sigilipelli.github.io/html-css-mastery-path/level-3/06-container-queries/)
on the HTML & CSS course.

## The two halves: a container and a query

```html
<div class="@container">
  <article class="flex flex-col gap-4 @md:flex-row">
    <img class="aspect-video w-full rounded-lg object-cover @md:w-40" src="…" alt="">
    <div>…</div>
  </article>
</div>
```

- `@container` marks an element as a query container. It compiles to
  `container-type: inline-size;`.
- `@md:flex-row` applies when the **nearest ancestor container** is at least `md` wide:

```css
@container (width >= 28rem) {
  .\@md\:flex-row { flex-direction: row; }
}
```

Notice that `@md` is **28rem**, not the 48rem of the `md:` breakpoint. Container sizes
come from the `--container-*` theme scale, which is much smaller than the breakpoint
scale, because containers are usually smaller than screens:

| Variant | Size | | Variant | Size |
|---|---|---|---|---|
| `@3xs:` | 16rem | | `@lg:` | 32rem |
| `@2xs:` | 18rem | | `@xl:` | 36rem |
| `@xs:` | 20rem | | `@2xl:` | 42rem |
| `@sm:` | 24rem | | `@3xl:` | 48rem |
| `@md:` | 28rem | | … up to `@7xl:` | 80rem |

## Measured: one component, two widths

We put that exact card twice on a 1280px-wide page: once in an 18rem sidebar (`w-72`)
and once in the flexible main column. Same classes, same viewport:

```text
sidebar container: 288px wide -> flex-direction: column
main container:    920px wide -> flex-direction: row
```

With a breakpoint like `lg:flex-row`, both would have been rows (the screen is wide), and
the sidebar card would have squeezed its image and text into 288px.

## Max and range queries

They work just like the breakpoint versions:

```html
<nav class="@max-md:hidden">…</nav>                  <!-- container < 28rem -->
<div class="@sm:@max-lg:grid">…</div>               <!-- 24rem ≤ width < 32rem -->
<div class="@min-[30rem]:p-4 @[30rem]:p-5">…</div>  <!-- arbitrary sizes -->
```

`@[30rem]:` and `@min-[30rem]:` both compiled to `@container (width >= 30rem)`.

## Named containers

A query matches the **nearest** container ancestor. When containers are nested (a card
inside a panel inside a page), you sometimes want to query an outer one. Name it:

```html
<main class="@container/main">
  <section class="@container">
    <ul class="grid gap-4 @lg/main:grid-cols-3">…</ul>
  </section>
</main>
```

```css
.\@container\/main { container-type: inline-size; container-name: main; }
@container main (width >= 32rem) {
  .\@lg\/main\:grid-cols-3 { grid-template-columns: repeat(3, minmax(0, 1fr)); }
}
```

## Container query units

Inside a container you can size things relative to it with `cqw` (1% of the container's
width), `cqi` (1% of its inline size) and friends, via arbitrary values:

```html
<h3 class="text-[clamp(1rem,5cqi,1.75rem)] font-semibold">Fluid to the card, not the screen</h3>
```

The heading scales with its card, clamped between 1rem and 1.75rem.

## The containment pitfalls

`container-type: inline-size` tells the browser the element's width must **not depend on
its contents** (that's what makes querying it possible without an infinite loop). Two
consequences catch people out:

1. **Shrink-to-fit containers collapse.** An `@container` that's `inline-block`, a
   flex item with no width, a float, or an absolutely positioned box normally takes the
   width of its content. With size containment, its content counts as zero. We tested
   `<div class="@container inline-block">Some content here</div>` and it measured
   **0px** wide. Give such containers an explicit width (`w-full`, `w-72`, `flex-1`) or
   put `@container` on a block-level wrapper instead.

2. **An element can't query itself.** `@md:` looks at the nearest container *ancestor*.
   We put `@container @md:bg-red-500` on one 40rem-wide element and the background was
   not applied. Put `@container` on a wrapper and the `@md:` classes on its children.

Containment doesn't change everything about the element, though. We checked a common
worry: a `fixed` child inside an `@container` (offset 200px from the page edge) was
still positioned against the viewport at `top: 0; left: 0`, exactly like one outside a
container. Inline-size containment is about sizing, not positioning.

## Worked example: a reusable stat tile

```html
<div class="@container">
  <div class="grid gap-3 rounded-xl border border-gray-200 p-4 @xs:grid-cols-[auto_1fr]
              @xs:items-center @md:p-6">
    <span class="grid size-10 place-items-center rounded-lg bg-sky-50 text-sky-700
                 @md:size-12" aria-hidden="true">
      <svg class="size-5" viewBox="0 0 20 20" fill="currentColor"><circle cx="10" cy="10" r="6"/></svg>
    </span>
    <div>
      <p class="text-sm text-gray-500">Active users</p>
      <p class="text-[clamp(1.25rem,8cqi,2.25rem)] font-semibold tabular-nums">1,284</p>
    </div>
    <p class="text-sm text-emerald-700 @md:col-span-2 @max-xs:hidden">+4.2% vs last week</p>
  </div>
</div>
```

(The figures are sample data.) Put it in a narrow dashboard column and it stacks the
icon above the number and hides the trend line. Put it in a wide column and the icon
sits beside the number, the padding grows, and the trend appears. The page layout
decides the width; the tile decides how to use it.

## Breakpoints or container queries?

- **Page-level layout** (how many columns the page has, whether the sidebar is shown):
  breakpoints. The page really does depend on the screen.
- **Component internals** (does this card lay out horizontally, how big is this
  heading): container queries, so the component works wherever it's placed.
- **Typography on long-form pages:** usually breakpoints, or fluid `clamp()` with
  viewport units.

## How It Actually Works

`@container` is a utility that sets `container-type` (and `container-name` for the
`/name` form). The `@{size}` variants are generated from the `--container-*` theme
tokens: each one wraps the rule in `@container (width >= <value>)`, and `@max-{size}`
uses `<`. Named versions put the name in the query: `@container main (…)`. Like
breakpoint variants, they're sorted by size so larger queries override smaller ones.

At runtime the browser does all the work. When it lays out a container, it resolves the
container's width first (that's why the width can't depend on content), then evaluates
the queries for every descendant against that width. Each element finds its nearest
matching container by walking up the tree, filtered by name if the query has one.

## Common mistakes

- **Expecting `@md` to mean 768px.** It's 28rem; container sizes are their own scale.
- **Forgetting the `@container` ancestor.** Without one, `@md:` classes never apply.
- **Putting `@container` and `@md:` on the same element** and expecting it to query
  itself.
- **Making an `inline-block`, floated or unsized flex item a container**, which collapses
  to zero width.
- **Using container queries for page layout** that genuinely depends on the viewport.
- **Nesting unnamed containers** and querying the wrong one. Name the outer container.

## Exercise

1. Build the card from the first example and place it in a 2-column grid, a 4-column grid
   and a narrow sidebar on one page. Confirm with DevTools (containers show a badge in
   the Elements panel) which layout each copy uses.
2. Reproduce the zero-width container: put `@container` on an `inline-block` element,
   then fix it.
3. Build the stat tile and make the number's size fluid with `cqi` units. Resize its
   column and watch it scale.
4. Nest a named page container and an unnamed card container, and use `@lg/main:` on an
   element inside the card.
