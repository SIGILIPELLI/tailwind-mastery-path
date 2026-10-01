---
description: "Level 2 project: a responsive admin dashboard with theme tokens, dark mode, an accessible off-canvas sidebar and container-query tiles."
---

# 10 · Project — A Dashboard Layout

The Level 1 project was a marketing page: mostly static, styled with default theme
values. This one is an application screen, the kind of page you'd build every day at
work: an orders dashboard for a made-up shop called **Northwind**. It uses nearly every
Level 2 tool:

- **Theme tokens** for a brand colour and semantic surface/text colours (lessons 01–02)
- **Dark mode** with a flash-free three-way toggle (lesson 03)
- An **arbitrary grid template** driven by a token (lesson 04)
- **`aria-*` and `data-*` variants** for the current page and order status (lesson 06)
- **Container queries** in the stat tiles (lesson 07)
- **Form styling** for the search field (lesson 08)
- A **transitioned off-canvas sidebar** that respects reduced motion (lesson 09)

At the end, you'll verify it the way you'd verify real work: widths, both colour
schemes, keyboard access.

## Setup

Same CLI setup as before, with the stylesheet at `src/app.css` and output at
`dist/app.css`:

```bash
mkdir northwind && cd northwind
npm init -y
npm install tailwindcss @tailwindcss/cli
npx @tailwindcss/cli -i src/app.css -o dist/app.css --watch
```

## Step 1: tokens and dark mode

```css title="src/app.css"
@import "tailwindcss";

@custom-variant dark (&:where(.dark, .dark *));

@theme {
  --color-brand-50: oklch(97% 0.02 265);
  --color-brand-500: oklch(62% 0.19 265);
  --color-brand-600: oklch(54% 0.2 265);
  --color-brand-700: oklch(46% 0.18 265);

  --color-bg: var(--color-white);
  --color-bg-subtle: var(--color-gray-50);
  --color-fg: var(--color-gray-900);
  --color-fg-muted: var(--color-gray-600);
  --color-line: var(--color-gray-200);

  --spacing-sidebar: 16rem;
}

@layer base {
  .dark {
    --color-bg: var(--color-gray-950);
    --color-bg-subtle: var(--color-gray-900);
    --color-fg: var(--color-gray-100);
    --color-fg-muted: var(--color-gray-400);
    --color-line: var(--color-gray-800);
  }
}
```

Every surface and text colour in the markup will use `bg`, `bg-subtle`, `fg`,
`fg-muted` and `line`. The `.dark` rule swaps their values, so the page needs almost
no `dark:` classes. Only the status badges and the active nav item (which use brand
and status colours rather than semantic tokens) get explicit `dark:` tweaks.

`--spacing-sidebar` is a named spacing token, so `w-sidebar` exists, and the same value
feeds the grid template in the next step. Change it once and both follow.

## Step 2: the document and the page shell

```html title="index.html"
<!doctype html>
<html lang="en" class="scheme-light dark:scheme-dark">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Orders — Northwind Admin</title>
  <script>
    (function () {
      var t = null;
      try { t = localStorage.getItem("theme"); } catch (e) {}
      var dark = t === "dark" || (t !== "light" && matchMedia("(prefers-color-scheme: dark)").matches);
      document.documentElement.classList.toggle("dark", dark);
    })();
  </script>
  <link rel="stylesheet" href="dist/app.css">
</head>
<body class="bg-bg-subtle text-fg antialiased">
  <div class="min-h-dvh lg:grid lg:grid-cols-[var(--spacing-sidebar)_1fr]">
    <!-- sidebar, backdrop and content go here -->
  </div>
</body>
</html>
```

The blocking script in `<head>` sets the `dark` class before anything paints (lesson 03).
`scheme-light dark:scheme-dark` makes native scrollbars and controls match.

The shell is a block on small screens and, from `lg`, a two-column grid:
`lg:grid-cols-[var(--spacing-sidebar)_1fr]`. That arbitrary template reads the theme
token directly, so the column is exactly as wide as the sidebar.

## Step 3: the sidebar

```html
<aside id="sidebar" data-open="false"
       class="fixed inset-y-0 start-0 z-40 w-sidebar -translate-x-full border-e border-line bg-bg
              transition-transform duration-200 ease-out data-[open=true]:translate-x-0
              motion-reduce:transition-none lg:sticky lg:top-0 lg:h-dvh lg:translate-x-0">
  <div class="flex h-14 items-center gap-2 px-4 font-semibold">
    <span class="grid size-7 place-items-center rounded-md bg-brand-600 text-sm text-white" aria-hidden="true">N</span>
    Northwind
  </div>
  <nav aria-label="Main" class="px-2 py-4">
    <ul class="space-y-1 text-sm font-medium">
      <li><a href="#" class="block rounded-md px-3 py-2 text-fg-muted hover:bg-bg-subtle hover:text-fg">Overview</a></li>
      <li><a href="#" aria-current="page" class="block rounded-md px-3 py-2 text-fg-muted hover:bg-bg-subtle hover:text-fg aria-[current=page]:bg-brand-50 aria-[current=page]:text-brand-700 dark:aria-[current=page]:bg-brand-500/15 dark:aria-[current=page]:text-brand-500">Orders</a></li>
      <li><a href="#" class="block rounded-md px-3 py-2 text-fg-muted hover:bg-bg-subtle hover:text-fg">Customers</a></li>
      <li><a href="#" class="block rounded-md px-3 py-2 text-fg-muted hover:bg-bg-subtle hover:text-fg">Settings</a></li>
    </ul>
  </nav>
</aside>
<div id="backdrop" hidden class="fixed inset-0 z-30 bg-black/40 lg:hidden"></div>
```

How it behaves at each size:

- **Below `lg`** it's `fixed` to the start edge and moved fully off-screen with
  `-translate-x-full`. When the script sets `data-open="true"`,
  `data-[open=true]:translate-x-0` slides it in. `transition-transform` animates the
  slide (it covers the `translate` property v4 uses, lesson 09), and
  `motion-reduce:transition-none` makes it appear instantly for users who asked for less
  motion.
- **From `lg`** it becomes `sticky` at the top with the full viewport height and is
  always visible (`lg:translate-x-0`).
- `start-0` and `border-e` are logical utilities: they'd flip correctly in a
  right-to-left language (Level 4 · 03).

The current page is marked with `aria-current="page"`, which tells screen reader users
where they are, and the styling comes from that same attribute with
`aria-[current=page]:`. Remember from lesson 06 that the bare `aria-current:` variant
would test for `"true"` and never match.

## Step 4: the header

```html
<header class="sticky top-0 z-20 flex h-14 items-center gap-3 border-b border-line bg-bg/90 px-4 backdrop-blur sm:px-6">
  <button id="menu" type="button" aria-controls="sidebar" aria-expanded="false"
          class="-ms-2 rounded-md p-2 text-fg-muted hover:bg-bg-subtle lg:hidden">
    <span class="sr-only">Open navigation</span>
    <svg class="size-5" viewBox="0 0 20 20" fill="currentColor" aria-hidden="true"><path d="M3 5h14v2H3zm0 4h14v2H3zm0 4h14v2H3z"/></svg>
  </button>
  <h1 class="text-base font-semibold">Orders</h1>
  <div class="ms-auto flex items-center gap-2">
    <label for="theme" class="sr-only">Theme</label>
    <select id="theme" class="rounded-md border border-line bg-bg px-2 py-1 text-sm">
      <option value="system">System</option>
      <option value="light">Light</option>
      <option value="dark">Dark</option>
    </select>
  </div>
</header>
```

- The header is `sticky` with a slightly transparent background
  (`bg-bg/90 backdrop-blur`), so content scrolling underneath is softly visible. The
  opacity modifier works on a token colour just like a palette colour.
- The menu button is only shown below `lg`. It has a visually hidden text label and
  `aria-expanded`, which the script keeps in sync.
- `ms-auto` (margin-inline-start: auto) pushes the theme select to the end of the flex
  row.

## Step 5: stat tiles with container queries

```html
<section aria-label="Summary" class="grid gap-4 sm:grid-cols-2 xl:grid-cols-4">
  <div class="@container rounded-xl border border-line bg-bg p-4">
    <p class="text-sm text-fg-muted">Orders today</p>
    <p class="mt-1 text-2xl font-semibold tabular-nums @[14rem]:text-3xl">128</p>
  </div>
  <div class="@container rounded-xl border border-line bg-bg p-4">
    <p class="text-sm text-fg-muted">Revenue today</p>
    <p class="mt-1 text-2xl font-semibold tabular-nums @[14rem]:text-3xl">$9,412</p>
  </div>
  <div class="@container rounded-xl border border-line bg-bg p-4">
    <p class="text-sm text-fg-muted">Awaiting shipment</p>
    <p class="mt-1 text-2xl font-semibold tabular-nums @[14rem]:text-3xl">23</p>
  </div>
  <div class="@container rounded-xl border border-line bg-bg p-4">
    <p class="text-sm text-fg-muted">Refund requests</p>
    <p class="mt-1 text-2xl font-semibold tabular-nums @[14rem]:text-3xl">4</p>
  </div>
</section>
```

The tile grid goes 1 → 2 → 4 columns with breakpoints, since that's page layout. The
**number size** is a component decision, so it uses a container query:
`@[14rem]:text-3xl` makes the number bigger whenever the tile itself is at least 14rem
wide. (The figures are sample data.)

## Step 6: the orders table

```html
<section aria-labelledby="recent" class="rounded-xl border border-line bg-bg">
  <div class="flex flex-wrap items-center justify-between gap-3 border-b border-line p-4">
    <h2 id="recent" class="font-semibold">Recent orders</h2>
    <label class="relative">
      <span class="sr-only">Search orders</span>
      <input type="search" placeholder="Search…"
             class="w-56 rounded-md border border-line bg-bg px-3 py-1.5 text-sm placeholder:text-fg-muted
                    focus:border-brand-500 focus:ring-2 focus:ring-brand-500/30 focus:outline-hidden">
    </label>
  </div>
  <div class="overflow-x-auto">
    <table class="w-full min-w-[36rem] text-left text-sm">
      <thead class="text-xs text-fg-muted uppercase">
        <tr>
          <th scope="col" class="px-4 py-3 font-medium">Order</th>
          <th scope="col" class="px-4 py-3 font-medium">Customer</th>
          <th scope="col" class="px-4 py-3 font-medium">Status</th>
          <th scope="col" class="px-4 py-3 text-end font-medium">Total</th>
        </tr>
      </thead>
      <tbody class="divide-y divide-line">
        <tr class="hover:bg-bg-subtle">
          <td class="px-4 py-3 font-medium">#1042</td>
          <td class="px-4 py-3">Ana Souza</td>
          <td class="px-4 py-3"><span data-status="paid" class="rounded-full px-2 py-0.5 text-xs font-medium data-[status=paid]:bg-emerald-500/15 data-[status=paid]:text-emerald-700 dark:data-[status=paid]:text-emerald-400">Paid</span></td>
          <td class="px-4 py-3 text-end tabular-nums">$84.00</td>
        </tr>
        <tr class="hover:bg-bg-subtle">
          <td class="px-4 py-3 font-medium">#1041</td>
          <td class="px-4 py-3">Kenji Watanabe</td>
          <td class="px-4 py-3"><span data-status="pending" class="rounded-full px-2 py-0.5 text-xs font-medium data-[status=pending]:bg-amber-500/15 data-[status=pending]:text-amber-800 dark:data-[status=pending]:text-amber-400">Pending</span></td>
          <td class="px-4 py-3 text-end tabular-nums">$212.50</td>
        </tr>
        <tr class="hover:bg-bg-subtle">
          <td class="px-4 py-3 font-medium">#1040</td>
          <td class="px-4 py-3">Fatima Khan</td>
          <td class="px-4 py-3"><span data-status="refunded" class="rounded-full px-2 py-0.5 text-xs font-medium data-[status=refunded]:bg-gray-500/15 data-[status=refunded]:text-fg-muted">Refunded</span></td>
          <td class="px-4 py-3 text-end tabular-nums">$39.99</td>
        </tr>
      </tbody>
    </table>
  </div>
</section>
```

- The table has `min-w-[36rem]` inside an `overflow-x-auto` wrapper. On a phone the
  table scrolls sideways inside its card, and the *page* never scrolls horizontally. The
  wrapper's parent chain includes the `min-w-0` content column, which is what allows it
  to shrink below the table's width (the grid/flex minimum-size issue from Level 1).
- Status badges are styled from `data-status`, so the server only has to output the
  right attribute value. Translucent backgrounds (`bg-emerald-500/15`) work on both light
  and dark surfaces, so only the text colour needs a `dark:` override.
- `tabular-nums` makes digits equal width so amounts line up in the column, and
  `scope="col"` on the headers helps screen readers announce each cell's column.

## Step 7: the script

```html
<script>
  const sidebar = document.getElementById("sidebar");
  const backdrop = document.getElementById("backdrop");
  const menu = document.getElementById("menu");
  const desktop = matchMedia("(width >= 64rem)");
  function setOpen(open) {
    sidebar.dataset.open = String(open);
    menu.setAttribute("aria-expanded", String(open));
    backdrop.hidden = !open;
    // Off-screen links must not be reachable with Tab
    sidebar.inert = !open && !desktop.matches;
  }
  setOpen(false);
  desktop.addEventListener("change", () => setOpen(false));
  menu.addEventListener("click", () => setOpen(sidebar.dataset.open !== "true"));
  backdrop.addEventListener("click", () => setOpen(false));
  document.addEventListener("keydown", (e) => { if (e.key === "Escape") setOpen(false); });

  const select = document.getElementById("theme");
  const media = matchMedia("(prefers-color-scheme: dark)");
  let choice = "system";
  try { choice = localStorage.getItem("theme") || "system"; } catch (e) {}
  select.value = choice;
  function apply() {
    const dark = choice === "dark" || (choice === "system" && media.matches);
    document.documentElement.classList.toggle("dark", dark);
  }
  select.addEventListener("change", () => {
    choice = select.value;
    try { choice === "system" ? localStorage.removeItem("theme") : localStorage.setItem("theme", choice); } catch (e) {}
    apply();
  });
  media.addEventListener("change", () => { if (choice === "system") apply(); });
</script>
```

The JavaScript only changes **state**: a data attribute, `aria-expanded`, the
backdrop's `hidden`, and the `dark` class. Every visual consequence is already in the
classes.

One line deserves attention: `sidebar.inert = !open && !desktop.matches`. A sidebar
moved off-screen with a transform is invisible, but its links are still in the page, so
keyboard users would Tab into links they can't see. `inert` removes them from the tab
order and from the accessibility tree while the drawer is closed on small screens.

## Step 8: verify it

We loaded the finished page in Chromium and checked layout, behaviour and keyboard
access. Layout at four settings:

```text
width  scheme  page scrollWidth  sidebar x  menu button  stat number size
390    light   390               -256       shown        30px
800    light   800               -256       shown        30px
1280   light   1280              0          hidden       24px
1280   dark    1280              0          hidden       24px   (body bg: gray-900)
```

- No horizontal page scrolling at any width. The sidebar is off-screen below `lg` and
  docked from 1280.
- The stat number is **larger on the phone** than on the desktop. On the phone each
  tile spans the full width; on the desktop four tiles share the row, and each tile's
  content box is under 14rem, so the container query doesn't apply. That's the point of
  container queries: the tile responds to its own space, not the screen.
- With dark mode emulated, the `dark` class was applied before paint and the tokens
  switched.
- **Contrast**, computed from the rendered colours (translucent badge backgrounds
  blended over the card): the active nav item measured 6.78:1 in light mode and 4.66:1
  in dark, and the three status badges measured 4.65, 6.32 and 6.28:1 in light and
  8.63, 9.55 and 6.94:1 in dark, all above the 4.5:1 AA threshold. Our first version
  used `text-amber-700` for "Pending", which came out at 4.48:1, just under, so it's
  `amber-800` now. Level 3 · 04 explains the measurement.

Behaviour at 390px: clicking the menu button moved the sidebar to x = 0, set
`aria-expanded="true"` and showed the backdrop. Pressing Escape moved it back to
x = −256 and reset `aria-expanded`.

Keyboard: pressing Tab four times from the top of the page gave this focus order:

```text
390px:  menu button, theme select, search input, table scroll area
1280px: Overview, Orders, Customers, Settings
```

On the phone, focus skipped the four hidden sidebar links because of `inert`. On the
desktop the sidebar is visible and comes first, as expected. The table's scroll area is
focusable because Chromium makes scrollable regions keyboard-reachable, so keyboard
users can scroll it with the arrow keys.

The minified CSS for the whole page was 18,933 bytes with Tailwind 4.3.3.

## How It Actually Works

Three mechanisms carry this page. **Custom properties** do the theming: utilities like
`bg-bg` compile to `background-color: var(--color-bg)`, and the `.dark` rule redefines
that variable for the whole tree, so one class change on `<html>` re-themes everything.
**Attribute selectors** do the state: `data-[open=true]:translate-x-0` and
`aria-[current=page]:text-brand-700` are ordinary CSS attribute selectors, so the script
just sets attributes and the browser re-matches rules. **Media and container queries** do
the responsiveness. Breakpoint rules switch the shell between drawer and docked sidebar,
and container rules size the stat numbers per tile.

None of it involves Tailwind at runtime. The compiled stylesheet is static CSS, and the
only JavaScript is about 30 lines of state handling.

## Common mistakes

- **Hiding an off-canvas menu with a transform only.** Its links stay focusable. Use
  `inert` (or `hidden`/`visibility: hidden` when closed).
- **Forgetting `min-w-0`** on the content column, so a wide table stretches the page.
- **Mixing token colours and palette colours** for the same purpose. Surfaces and text
  should come from tokens, or dark mode needs `dark:` classes everywhere.
- **Using breakpoints for component sizing** that should depend on the component's
  width.
- **`aria-current:` instead of `aria-[current=page]:`.**
- **Status shown by colour only.** The badge text ("Paid") carries the meaning; colour
  just reinforces it.

## Exercise

1. Build the dashboard. Then change `--spacing-sidebar` to `18rem` and confirm the
   sidebar and the grid column both change.
2. Add a fifth stat tile and decide what the grid should do at `xl`.
3. Add a "Cancelled" status with its own colours, light and dark.
4. Make the table rows clickable (link to `/orders/1042`) without breaking keyboard
   access. One approach: a link in the first cell with an `after:absolute after:inset-0`
   overlay and `relative` on the row.
5. Run the checks in Step 8 yourself in DevTools: device toolbar, emulated
   colour scheme, Tab order, and the reduced-motion emulation.
