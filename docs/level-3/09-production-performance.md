---
description: "Ship small, fast Tailwind CSS: real minified and compressed sizes, why CSS grows with unique classes not pages, caching and rendering costs."
---

# 09 · Production Builds & Performance

A common worry about Tailwind is "won't all those classes make my CSS huge?". The short
answer is no, and the reason is worth understanding because it tells you what *does*
make the CSS grow. This lesson measures real output sizes, explains the one factor that
drives them, sets up a sensible production build, and covers the runtime performance
issues that have nothing to do with file size.

## Production builds

In development you run the watcher. For production, build once with `--minify`:

```bash
npx @tailwindcss/cli -i src/app.css -o dist/app.css --minify
```

With the Vite plugin (`@tailwindcss/vite`, Level 1 · 02), `vite build` minifies
automatically and writes a hashed file name like `assets/index-<hash>.css`. With
`@tailwindcss/postcss`, minification depends on your PostCSS/bundler setup.

Minifying runs the output through Lightning CSS. Besides removing whitespace, it
**merges rules** and **lowers modern syntax**. In the Level 2 dashboard's minified output,
every media query appeared exactly once (all `lg:` rules under a single
`@media (min-width:64rem)`), and the range syntax `width >= 64rem` had been rewritten to
`min-width:64rem` for wider browser support.

## Real sizes

The two projects from this course, built with Tailwind 4.3.3 and compressed with Node's
`zlib` at default settings:

| Page | Minified | gzip | Brotli |
|---|---|---|---|
| Level 1 landing page | 15,211 B | 3,764 B | 3,213 B |
| Level 2 dashboard | 18,933 B | 4,563 B | 3,977 B |

Servers and CDNs compress CSS on the fly, so the gzip or Brotli figure is what users
actually download: about 4 KB for a complete page design. For comparison, an empty
stylesheet (just `@import "tailwindcss"` with no classes used) minified to 4,165 bytes,
1,416 gzipped: that's Preflight and the handful of theme variables it needs.

## What makes CSS grow: unique classes, not pages

The compiler generates one rule per **unique candidate**, no matter how many times or
in how many files it appears. To show the scaling, we used Tailwind's internal class
list (it reported 21,278 named utilities without negatives in 4.3.3), picked utilities
evenly across it, and built minified CSS:

```text
unique classes   minified    gzip      brotli
0                4,165       1,416     1,173
100              44,774      5,784     4,802
500              165,114     12,762    9,911
1000             310,158     20,154    14,866
2000             605,164     32,099    22,867
500 × 200 pages  165,114     12,762    9,911    <- identical to 1 page
```

Two observations:

- **Two hundred pages using the same 500 classes produced exactly the same file** as one
  page. Adding pages that reuse your design vocabulary costs nothing.
- **Size grows with vocabulary.** Picking evenly from the whole class list is a worst
  case: it samples every exotic utility (3D transforms, conic gradients, every shade of
  every colour), which is why 100 random classes cost more than our entire landing page.
  Real projects reuse a small vocabulary, which is why they stay small. The compressed
  sizes also grow much more slowly than the raw sizes, because utility CSS is extremely
  repetitive and compresses well.

Variants multiply the vocabulary. The same 500 classes plus an `md:` and a `hover:` copy
of each (1,500 candidates) produced 458,397 bytes minified, 28,210 gzipped. A design
system that restricts tokens (lesson 07) is also a size optimisation.

## What to avoid

- **Large safelists.** `@source inline()` patterns generate rules whether or not
  anything uses them (lesson 01).
- **Arbitrary values everywhere.** Each unique `w-[37rem]` is a new rule that can't be
  shared. Repeated values belong in the theme.
- **Scanning too much.** Pointing `@source` at a huge folder of unrelated files (old
  templates, generated docs) can pick up thousands of class-like strings.
- **Shipping development output.** Unminified CSS is several times larger before
  compression.

## Caching

The CSS changes only when your set of classes changes, so it caches well:

- Serve it with a **content hash in the file name** (Vite does this automatically) and a
  long `Cache-Control` max-age with `immutable`. A new deployment produces a new name.
- With the CLI, add hashing in your deployment step, or use your framework's asset
  pipeline (Rails Propshaft, Django's `ManifestStaticFilesStorage`, Laravel Vite).
- **One stylesheet for the whole site** is usually best. Splitting by page saves little,
  because pages share most utilities, and costs extra requests and cache entries.

### Inline the CSS?

For a single landing page around 4 KB compressed, inlining the CSS in a `<style>` tag
removes a render-blocking request. For a multi-page site, a cached external file is
usually better, because inlined CSS is re-downloaded with every HTML page. Measure with
your real pages (Lighthouse or WebPageTest) before deciding.

## Rendering performance

File size is rarely the bottleneck. These matter more:

- **Animate cheap properties.** `opacity`, `translate`, `scale` and `rotate` can be
  handled by the compositor. Animating `width`, `height`, `top`, `margin` or `box-shadow`
  forces layout or paint on every frame (Level 2 · 09).
- **`will-change-*` sparingly.** `will-change-transform` promotes an element to its own
  layer, which uses memory. Add it to an element that is about to animate, not to dozens
  of elements "just in case".
- **Large blurs are expensive.** `backdrop-blur-xl` on a big fixed header re-renders the
  blur whenever content scrolls beneath it. Test on a mid-range phone.
- **`content-visibility: auto`** (as a custom utility, Level 2 · 05) lets the browser
  skip rendering off-screen sections of very long pages. Pair it with
  `contain-intrinsic-size` so the scrollbar doesn't jump.
- **Images.** `aspect-*` plus `w-full` reserves space before an image loads, avoiding
  layout shift; set `width` and `height` attributes as well.

## Worked example: a production setup with the CLI

```json title="package.json (scripts)"
{
  "scripts": {
    "dev": "tailwindcss -i src/app.css -o public/app.css --watch",
    "build:css": "tailwindcss -i src/app.css -o public/app.css --minify",
    "build": "npm run build:css && node scripts/hash-assets.mjs"
  }
}
```

```js title="scripts/hash-assets.mjs"
// Copy public/app.css to public/app.<hash>.css and record the name for templates
import { createHash } from "node:crypto";
import { readFileSync, writeFileSync, copyFileSync } from "node:fs";

const css = readFileSync("public/app.css");
const hash = createHash("sha256").update(css).digest("hex").slice(0, 10);
const name = `app.${hash}.css`;
copyFileSync("public/app.css", `public/${name}`);
writeFileSync("public/asset-manifest.json", JSON.stringify({ "app.css": name }, null, 2));
console.log(`wrote public/${name}`);
```

Your templates read `asset-manifest.json` to link the hashed file, and the server sends
hashed files with `Cache-Control: public, max-age=31536000, immutable`. (Inside
`package.json` scripts, `tailwindcss` refers to the CLI binary installed by
`@tailwindcss/cli`.)

## How It Actually Works

Output size is a function of the set of candidates the scanner found and the CSS each
generates. The compiler keeps candidates in a set, so duplicates across files collapse
to one. Each candidate's rule is generated once and placed in sorted order. Theme
variables are emitted only if something references them, and `@property` registrations
only for the internal variables used (the dashboard needed 37). Minification then removes
whitespace, merges identical at-rules, and shortens values.

Compression works so well because utility CSS repeats the same short sequences
constantly (`{padding:calc(var(--spacing)*`, `@media (min-width:`), and gzip and Brotli
replace repeats with back-references. That's why the 2,000-class build was 605 KB raw
but 23 KB with Brotli.

## Common mistakes

- **Measuring raw size** instead of what's actually transferred (compressed).
- **Splitting CSS per page** for a few hundred bytes, losing caching.
- **Safelisting broadly** "to be safe".
- **Forgetting `--minify`** (or deploying the watcher's output).
- **Optimising bytes while animating layout properties.** Rendering cost usually matters
  more.
- **Treating the scaling numbers above as typical.** They're a deliberately worst-case
  sample; measure your own build.

## Exercise

1. Build your Level 2 project with and without `--minify` and record raw, gzip and
   Brotli sizes (`gzip -c file | wc -c`, `brotli -c file | wc -c` if installed).
2. Duplicate a page 50 times and confirm the CSS size doesn't change.
3. Add 50 arbitrary values (`w-[13px]`, `w-[14px]`, …) and measure the growth. Replace
   them with a theme token and measure again.
4. Set up the hashing script and serve the hashed file with a long cache lifetime.
5. Record a Performance profile while scrolling a page with a large `backdrop-blur`
   header, then without it, and compare.
