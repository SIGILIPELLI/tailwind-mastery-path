---
description: "Level 1 project: build a complete responsive product landing page with Tailwind v4 using only Level 1 utilities, then verify it at three widths."
---

# 10 · Project — A Landing Page

This project brings Level 1 together: one complete, responsive landing page for a
made-up note-taking app called **Fieldnote**. It uses only what you've covered so far:
spacing, sizing, typography, colour, borders, Flexbox, Grid, state variants and
breakpoints. There's no custom CSS and no configuration beyond the one `@import`.

At the end you'll check the page the way you'd check real work: at several widths, for
horizontal overflow, and for keyboard focus.

## What you'll build

- A header with a logo, a nav that hides on phones, and a call-to-action button
- A hero: two columns on large screens, stacked on small ones
- A three-card feature grid (1 → 2 → 3 columns)
- A numbered "How it works" row
- A FAQ list with dividers
- A dark sign-up band with an email form
- A footer

## Setup

Use the CLI setup from lesson 02:

```bash
mkdir fieldnote && cd fieldnote
npm init -y
npm install tailwindcss @tailwindcss/cli
mkdir src
echo '@import "tailwindcss";' > src/input.css
npx @tailwindcss/cli -i src/input.css -o dist/output.css --watch
```

Leave the watcher running and create `index.html` next to `src/`.

## Step 1: the document and header

```html title="index.html"
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Fieldnote — notes that file themselves</title>
  <link rel="stylesheet" href="dist/output.css">
</head>
<body class="bg-white text-slate-800 antialiased">

  <header class="border-b border-slate-200">
    <div class="mx-auto flex max-w-6xl items-center justify-between gap-6 px-4 py-4 sm:px-6">
      <a href="/" class="flex items-center gap-2 font-semibold text-slate-900">
        <span class="grid size-8 place-items-center rounded-lg bg-emerald-600 text-white"
              aria-hidden="true">F</span>
        Fieldnote
      </a>
      <nav aria-label="Main" class="max-md:hidden">
        <ul class="flex items-center gap-8 text-sm font-medium text-slate-600">
          <li><a href="#features" class="hover:text-slate-900">Features</a></li>
          <li><a href="#how" class="hover:text-slate-900">How it works</a></li>
          <li><a href="#faq" class="hover:text-slate-900">FAQ</a></li>
        </ul>
      </nav>
      <a href="#signup" class="rounded-md bg-slate-900 px-4 py-2 text-sm font-medium text-white
                               hover:bg-slate-700 focus-visible:outline-2 focus-visible:outline-offset-2
                               focus-visible:outline-slate-900">Get early access</a>
    </div>
  </header>
```

The pattern `mx-auto max-w-6xl px-4 sm:px-6` will repeat for every section. It's the
page's **container**: centred, capped at 72rem, with side padding that grows a little
from `sm`. The background colour or border goes on the outer element so it spans the
full width, and the container goes inside.

Other decisions:

- The logo mark is a one-cell grid with `place-items-center`, the shortest way to
  centre a letter in a square (lesson 07). `size-8` sets width and height together.
- The nav is hidden **below** `md` with `max-md:hidden`. On a phone the header then has
  just the logo and the button, which `justify-between` pushes to opposite edges. A real
  site would add a menu button for phones. That needs a little JavaScript or a
  `<details>` element, and Level 2 returns to it.
- `focus-visible:` gives keyboard users a clear ring that mouse users won't see on click
  (lesson 08).

## Step 2: the hero

```html
  <main>
    <section class="mx-auto grid max-w-6xl items-center gap-12 px-4 py-16 sm:px-6
                    lg:grid-cols-2 lg:py-24">
      <div>
        <p class="text-sm font-semibold tracking-wide text-emerald-700 uppercase">Private beta</p>
        <h1 class="mt-3 text-4xl font-bold tracking-tight text-balance text-slate-900 sm:text-5xl">
          Notes that file themselves
        </h1>
        <p class="mt-5 max-w-prose text-lg/8 text-slate-600">
          Write anything. Fieldnote suggests where it belongs, links it to related notes,
          and keeps your projects tidy without folders.
        </p>
        <div class="mt-8 flex flex-wrap gap-3">
          <a href="#signup" class="rounded-md bg-emerald-700 px-5 py-3 font-medium text-white
                                   hover:bg-emerald-800 focus-visible:outline-2
                                   focus-visible:outline-offset-2 focus-visible:outline-emerald-600">
            Join the beta</a>
          <a href="#how" class="rounded-md px-5 py-3 font-medium text-slate-700 ring-1
                                ring-slate-300 hover:bg-slate-50">See how it works</a>
        </div>
      </div>

      <div class="rounded-2xl bg-slate-100 p-3 ring-1 ring-slate-200">
        <div class="rounded-xl bg-white p-5 shadow-sm">
          <p class="text-xs font-medium text-slate-500">Today · 09:14</p>
          <p class="mt-2 font-medium text-slate-900">Call with supplier about Q3 packaging</p>
          <p class="mt-1 text-sm text-slate-600">Agreed to switch to recycled board from August…</p>
          <div class="mt-4 flex flex-wrap gap-2 text-xs">
            <span class="rounded-full bg-emerald-50 px-2.5 py-1 font-medium text-emerald-700">
              Suggested: Packaging</span>
            <span class="rounded-full bg-slate-100 px-2.5 py-1 text-slate-600">Linked: Q3 plan</span>
          </div>
        </div>
      </div>
    </section>
```

- The section is the grid itself. Below `lg` it's a single column (a grid with no
  template has one implicit column), so the text and the preview card stack. From `lg`
  it becomes two equal columns, and `items-center` lines them up vertically.
- `text-balance` evens out the line lengths of the headline so you don't get one word
  stranded on the last line.
- `text-lg/8` sets font size and line height in one class (lesson 04), and `max-w-prose`
  keeps the paragraph to a comfortable reading width.
- The main button is `bg-emerald-700`, not the brighter `emerald-600`. White text on
  emerald-600 measured about 3.65:1 contrast, below the 4.5:1 that WCAG AA asks for normal
  text; on emerald-700 it's about 5.36:1. Level 3 · 04 shows how those figures were
  measured.
- The buttons sit in a `flex flex-wrap gap-3` row, so if the screen is too narrow for
  both, the second one wraps instead of overflowing.
- The secondary button uses `ring-1` instead of `border`. A ring is drawn with
  `box-shadow`, so it doesn't add to the box size, and the two buttons stay exactly the
  same height.

## Step 3: features and "How it works"

```html
    <section id="features" class="bg-slate-50 py-16 lg:py-24">
      <div class="mx-auto max-w-6xl px-4 sm:px-6">
        <h2 class="text-3xl font-bold tracking-tight text-slate-900">Less filing, more thinking</h2>
        <ul class="mt-10 grid gap-6 sm:grid-cols-2 lg:grid-cols-3">
          <li class="rounded-xl border border-slate-200 bg-white p-6">
            <h3 class="font-semibold text-slate-900">Smart placement</h3>
            <p class="mt-2 text-sm/6 text-slate-600">Every new note gets a suggested project.
              Accept it with one key.</p>
          </li>
          <!-- two more <li> cards: "Automatic links", "Plain-text export" -->
        </ul>
      </div>
    </section>

    <section id="how" class="mx-auto max-w-6xl px-4 py-16 sm:px-6 lg:py-24">
      <h2 class="text-3xl font-bold tracking-tight text-slate-900">How it works</h2>
      <ol class="mt-10 grid gap-8 md:grid-cols-3">
        <li class="border-t-2 border-emerald-600 pt-4">
          <p class="text-sm font-semibold text-emerald-700">Step 1</p>
          <h3 class="mt-1 font-semibold text-slate-900">Capture</h3>
          <p class="mt-2 text-sm/6 text-slate-600">Type, paste or forward an email to your
            inbox address.</p>
        </li>
        <!-- Step 2 "Review", Step 3 "Find" -->
      </ol>
    </section>
```

The feature cards go 1 → 2 → 3 columns. With three cards, two columns leaves one card
alone on the second row. That's acceptable here, but with your own content decide
whether `sm:grid-cols-2` is worth it or whether you'd rather jump straight from one
column to three at `lg`.

The steps are an `<ol>` because the order matters. The visible "Step 1" text repeats
what the list already says for sighted users, and Preflight removes list markers (Level 3
· 02 explains why and what that means for screen readers).

## Step 4: FAQ, sign-up and footer

```html
    <section id="faq" class="border-t border-slate-200">
      <div class="mx-auto max-w-3xl px-4 py-16 sm:px-6">
        <h2 class="text-3xl font-bold tracking-tight text-slate-900">Questions</h2>
        <dl class="mt-8 divide-y divide-slate-200">
          <div class="py-5 first:pt-0">
            <dt class="font-medium text-slate-900">Is there a free plan?</dt>
            <dd class="mt-2 text-slate-600">The beta is free. Pricing will be announced
              before it ends.</dd>
          </div>
          <div class="py-5">
            <dt class="font-medium text-slate-900">Can I import from other apps?</dt>
            <dd class="mt-2 text-slate-600">Yes, any folder of Markdown or plain-text files.</dd>
          </div>
        </dl>
      </div>
    </section>

    <section id="signup" class="bg-slate-900">
      <div class="mx-auto max-w-6xl px-4 py-16 sm:px-6">
        <h2 class="text-3xl font-bold tracking-tight text-white">Get early access</h2>
        <form class="mt-6 flex max-w-md flex-col gap-3 sm:flex-row" action="/beta" method="post">
          <label for="email" class="sr-only">Email address</label>
          <input id="email" name="email" type="email" required autocomplete="email"
                 placeholder="you@example.com"
                 class="min-w-0 flex-1 rounded-md bg-white/10 px-4 py-3 text-white ring-1
                        ring-white/20 placeholder:text-slate-400 focus:ring-2
                        focus:ring-emerald-400 focus:outline-hidden">
          <button class="rounded-md bg-emerald-500 px-5 py-3 font-medium text-slate-900
                         hover:bg-emerald-400 focus-visible:outline-2 focus-visible:outline-offset-2
                         focus-visible:outline-emerald-400">Join</button>
        </form>
      </div>
    </section>
  </main>

  <footer class="border-t border-slate-200">
    <div class="mx-auto flex max-w-6xl flex-col gap-2 px-4 py-8 text-sm text-slate-500
                sm:flex-row sm:justify-between sm:px-6">
      <p>© Fieldnote</p>
      <p><a href="/privacy" class="hover:text-slate-700">Privacy</a></p>
    </div>
  </footer>
</body>
</html>
```

- `divide-y divide-slate-200` puts a border *between* FAQ items without a stray line at
  the top or bottom. `first:pt-0` removes the top padding of the first item so it sits
  right under the heading.
- The FAQ uses a narrower container (`max-w-3xl`), because long lines of answer text are
  hard to read.
- The form stacks on phones (`flex-col`) and becomes a row from `sm`. `min-w-0 flex-1`
  lets the input take the remaining space **and** shrink below its default width. Without
  `min-w-0`, an input's built-in minimum width can push the button out of a narrow row
  (lesson 06).
- The input's label is `sr-only`. It isn't visible, but screen readers still announce
  "Email address". The placeholder is *not* a label: it disappears as soon as you type.
- `bg-white/10` and `ring-white/20` are white at 10% and 20% opacity, which look right on
  any dark background.
- `focus:outline-hidden` removes the browser's default outline because `focus:ring-2`
  replaces it with a visible ring. It's `outline-hidden`, not `outline-none`, for a
  reason: in Windows high-contrast (forced colours) mode, box-shadow rings are removed,
  and `outline-hidden` brings back a visible outline there while `outline-none` leaves
  nothing. Level 3 · 04 shows the test.

## Step 5: check it

Don't stop at "looks fine on my screen". We loaded the finished page in Chromium at three
viewport widths and measured three things: the document's scroll width (anything wider
than the viewport means horizontal scrolling), whether the nav is displayed, and the
hero's computed grid columns:

```text
width 390   scrollWidth 390    nav: none    hero columns: 358px
width 800   scrollWidth 800    nav: block   hero columns: 752px
width 1280  scrollWidth 1280   nav: block   hero columns: 528px 528px
```

No horizontal overflow at any width. The nav appears from `md`, and the hero becomes two
columns only at `lg`. You can do the same check yourself without scripts: open DevTools'
device toolbar, drag the width from 320px upward, and watch for a horizontal scrollbar.
Then:

1. **Keyboard:** reload and press Tab repeatedly. Every link, the input and the button
   should get a visible focus indicator, in a sensible order.
2. **Zoom:** zoom the browser to 200%. The layout should switch to the narrower
   arrangement rather than overflow: zooming shrinks the viewport width in CSS pixels, so
   the breakpoints respond to it.
3. **Content:** make the headline twice as long and the email placeholder much longer. The
   layout should still hold.

Size of the generated CSS for this page, built with Tailwind v4.3.3: 18,499 bytes as
written by the watcher, 15,211 bytes with `--minify`, and 3,772 bytes after gzip. About
half of the unminified file (9,830 bytes) is the utilities layer. The rest is Preflight,
the theme variables the page actually uses, and a few `@property` registrations. An
empty page still produced about 4.6 KB. Your numbers will vary slightly with Tailwind
version and the exact classes you use.

## How It Actually Works

The finished stylesheet contains only the classes found in `index.html`. The compiler
scanned the file, collected every token that looked like a class name, generated CSS for
the ones it recognised, and threw away everything else. That's why the output is small
even though Tailwind can generate tens of thousands of utilities.

Every responsive change on the page is a utility with a breakpoint variant, and none of
them needed `!important`: unprefixed utilities are emitted first and breakpoint rules
later, in ascending order, so `lg:grid-cols-2` beats the implicit single column, and
`sm:flex-row` beats `flex-col`. The section backgrounds span the full width because the
background belongs to the outer `<section>`, while the inner container is limited by
`max-w-6xl` and centred by `mx-auto` (auto margins split the remaining space equally).

## Common mistakes

- **Putting the container classes and the background on the same element.** The
  background then stops at 72rem on wide screens.
- **Forgetting `min-w-0` on a flex input**, so the button gets pushed out on narrow
  screens.
- **Using the placeholder as the label.** Add a real `<label>`, visually hidden with
  `sr-only` if the design needs it.
- **`outline-none` without a replacement**, which removes keyboard focus entirely. When
  a ring replaces the outline, use `outline-hidden` so forced-colours users still see
  focus.
- **Only checking at one desktop width.** Breakage almost always shows up between
  breakpoints, so drag the width through the whole range.
- **Copying a class list without knowing why each class is there.** If you can't explain
  a class, remove it and see what changes.

## Exercise

1. Build the page, then change the accent colour from emerald to another palette
   everywhere. Notice how many places you had to edit. (Level 2 · 01 fixes this with a
   theme colour.)
2. Add a fourth feature card and decide what the grid should do at `sm` and `lg`.
3. Add a testimonials section: a quote, a name and a role, in a two-column grid from `md`.
4. Add a phone menu with a `<details>` element: a "Menu" `<summary>` visible below `md`
   and a vertical link list inside it.
5. Run the three checks in Step 5 and fix anything you find.
