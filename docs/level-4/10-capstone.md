---
description: "Capstone: a SaaS marketing site and app shell in Tailwind v4 with tokens, dark mode, RTL, print styles and a Playwright test suite."
---

# 10 · Capstone — A SaaS Marketing Site & App Shell

The capstone brings the whole course together in one small product: **Relay**, a
fictional shared-inbox tool. You'll build its marketing page and its application
screen on one token system, then prove they work with an automated test suite rather
than a quick look. It uses, among other things:

- semantic **tokens** with a **dark mode** that switches without colour fades
  (Level 2 · 01–03, Level 3 · 10)
- **logical utilities** so the whole app mirrors for right-to-left languages
  (Level 4 · 03)
- **container queries** inside cards and list items (Level 2 · 07)
- **ARIA and data attributes** as the source of styling state (Level 2 · 06)
- an **accessible off-canvas sidebar** with `inert` (Level 2 · 10)
- **print styles**, a **skip link**, and focus that survives **forced colours**
  (Levels 3 · 04, 4 · 04)
- a **Playwright suite**: contrast in both themes, overflow on desktop and phone,
  visual baselines, keyboard, print and RTL checks (Level 4 · 09)

Everything here was built and tested while writing the lesson, with Tailwind 4.3.3 and
`@playwright/test` 1.61.0.

## Project layout

```text
relay/
├── package.json
├── playwright.config.js
├── src/app.css           Tailwind entry: tokens, dark mode, base rules
├── site/
│   ├── index.html        marketing page
│   ├── app.html          application shell
│   ├── theme.js          theme toggle
│   └── app.css           (generated)
└── e2e/
    ├── contrast.js
    └── site.spec.js
```

```json title="package.json"
{
  "name": "relay-capstone",
  "private": true,
  "type": "module",
  "scripts": {
    "dev": "tailwindcss -i src/app.css -o site/app.css --watch",
    "build": "tailwindcss -i src/app.css -o site/app.css --minify",
    "test": "playwright test"
  },
  "devDependencies": {
    "@playwright/test": "^1.61.0",
    "@tailwindcss/cli": "^4.3.3",
    "tailwindcss": "^4.3.3"
  }
}
```

```bash
npm install -D tailwindcss @tailwindcss/cli @playwright/test
npx playwright install chromium
```

## Step 1: tokens

```css title="src/app.css"
@import "tailwindcss" source(none);
@source "../site";

@custom-variant dark (&:where(.dark, .dark *));

@theme {
  --color-bg: var(--color-white);
  --color-bg-subtle: var(--color-zinc-50);
  --color-fg: var(--color-zinc-900);
  --color-fg-muted: var(--color-zinc-600);
  --color-line: var(--color-zinc-200);
  --color-action: var(--color-violet-700);
  --color-action-hover: var(--color-violet-800);
  --color-on-action: var(--color-white);
  --color-accent-soft: var(--color-violet-50);
  --color-accent-fg: var(--color-violet-800);

  --radius-control: 0.5rem;
  --radius-panel: 1rem;
  --spacing-sidebar: 15rem;
  --container-page: 72rem;
}

@layer base {
  .dark {
    --color-bg: var(--color-zinc-950);
    --color-bg-subtle: var(--color-zinc-900);
    --color-fg: var(--color-zinc-100);
    --color-fg-muted: var(--color-zinc-400);
    --color-line: var(--color-zinc-800);
    --color-action: var(--color-violet-300);
    --color-action-hover: var(--color-violet-200);
    --color-on-action: var(--color-zinc-950);
    --color-accent-soft: var(--color-violet-950);
    --color-accent-fg: var(--color-violet-200);
  }

  /* No colour fades while the theme switches (Level 3 · 10) */
  html[data-switching] *,
  html[data-switching] *::before,
  html[data-switching] *::after {
    transition: none !important;
  }

  button:not(:disabled) { cursor: pointer; }
}
```

Choices worth noting:

- **`source(none)` plus one `@source`.** Only `site/` contains classes.
- **`--color-on-action` changes with the theme.** The dark theme's action colour is a
  light violet, so text on it must be dark (Level 3 · 10).
- **`--spacing-sidebar` and `--container-page`** make the layout constants named
  utilities (`w-sidebar`, `max-w-page`) and let the app grid template read the same
  value.
- **`button:not(:disabled) { cursor: pointer }`** puts back the v3-style pointer cursor
  that Preflight no longer sets (Level 3 · 08), as a base style any utility can override.

## Step 2: the theme toggle

Both pages put this in `<head>`, before the stylesheet, so the saved theme applies before
the first paint:

```html
<script>
  try {
    const t = localStorage.getItem("theme");
    if (t === "dark" || (!t && matchMedia("(prefers-color-scheme: dark)").matches)) {
      document.documentElement.classList.add("dark");
    }
  } catch {}
</script>
```

and load this module at the end of `<body>`:

```js title="site/theme.js"
// Shared by both pages. The inline <head> snippet applies the saved theme before paint;
// this file wires up the toggle buttons.
const html = document.documentElement;

function applyTheme(choice) {
  const dark = choice === "dark" ||
    (choice === "system" && matchMedia("(prefers-color-scheme: dark)").matches);
  html.setAttribute("data-switching", "");
  html.classList.toggle("dark", dark);
  getComputedStyle(html).color;
  requestAnimationFrame(() => html.removeAttribute("data-switching"));
}

for (const button of document.querySelectorAll("[data-theme-toggle]")) {
  button.addEventListener("click", () => {
    const next = html.classList.contains("dark") ? "light" : "dark";
    try { localStorage.setItem("theme", next); } catch {}
    applyTheme(next);
    button.setAttribute("aria-pressed", String(next === "dark"));
  });
  button.setAttribute("aria-pressed", String(html.classList.contains("dark")));
}
```

The toggle sets `aria-pressed`, so assistive technology announces it as "Dark theme,
toggle button, pressed/not pressed", and `data-switching` prevents every colour on the
page from fading during the switch.

## Step 3: the marketing page

```html title="site/index.html"
<!doctype html>
<html lang="en" dir="ltr" class="scheme-light dark:scheme-dark">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Relay — shared inboxes for small support teams</title>
  <script>
    try {
      const t = localStorage.getItem("theme");
      if (t === "dark" || (!t && matchMedia("(prefers-color-scheme: dark)").matches)) {
        document.documentElement.classList.add("dark");
      }
    } catch {}
  </script>
  <link rel="stylesheet" href="app.css">
</head>
<body class="bg-bg text-fg antialiased">
  <a href="#main" class="sr-only focus:not-sr-only focus:fixed focus:start-4 focus:top-4 focus:z-50
                         focus:rounded-control focus:bg-bg focus:px-4 focus:py-2 focus:outline-2
                         focus:outline-action">Skip to content</a>

  <header class="border-b border-line print:hidden">
    <div class="mx-auto flex max-w-page items-center gap-6 px-4 py-4 sm:px-6">
      <a href="index.html" class="flex items-center gap-2 font-semibold">
        <span class="grid size-8 place-items-center rounded-control bg-action text-on-action" aria-hidden="true">R</span>
        Relay
      </a>
      <nav aria-label="Main" class="max-md:hidden">
        <ul class="flex gap-6 text-sm font-medium text-fg-muted">
          <li><a href="#features" class="hover:text-fg">Features</a></li>
          <li><a href="#pricing" class="hover:text-fg">Pricing</a></li>
          <li><a href="#faq" class="hover:text-fg">FAQ</a></li>
        </ul>
      </nav>
      <div class="ms-auto flex items-center gap-2">
        <button type="button" data-theme-toggle
                class="grid size-10 place-items-center rounded-control text-fg-muted hover:bg-bg-subtle
                       focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-action">
          <span class="sr-only">Dark theme</span>
          <svg class="size-5" viewBox="0 0 20 20" fill="currentColor" aria-hidden="true">
            <path d="M10 2a8 8 0 1 0 8 8 6 6 0 0 1-8-8z"/>
          </svg>
        </button>
        <a href="app.html" class="rounded-control bg-action px-4 py-2 text-sm font-medium text-on-action
                                  hover:bg-action-hover focus-visible:outline-2 focus-visible:outline-offset-2
                                  focus-visible:outline-action">Open app</a>
      </div>
    </div>
  </header>

  <main id="main">
    <section class="mx-auto max-w-page px-4 py-16 sm:px-6 lg:py-24">
      <p class="text-sm font-semibold text-accent-fg">For teams of 2 to 20</p>
      <h1 class="mt-3 max-w-3xl text-4xl font-bold tracking-tight text-balance sm:text-6xl">
        One shared inbox for every customer conversation
      </h1>
      <p class="mt-6 max-w-2xl text-lg/8 text-pretty text-fg-muted">
        Relay brings email, chat and form messages into one queue, assigns them fairly, and
        shows who is replying so nobody answers twice.
      </p>
      <div class="mt-8 flex flex-wrap gap-3 print:hidden">
        <a href="app.html" class="rounded-control bg-action px-5 py-3 font-medium text-on-action hover:bg-action-hover
                                  focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-action">
          Try the demo
        </a>
        <a href="#pricing" class="rounded-control px-5 py-3 font-medium ring-1 ring-line hover:bg-bg-subtle
                                  focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-action">
          See pricing
        </a>
      </div>
    </section>

    <section id="features" aria-labelledby="features-title" class="border-y border-line bg-bg-subtle">
      <div class="mx-auto max-w-page px-4 py-16 sm:px-6">
        <h2 id="features-title" class="text-3xl font-bold tracking-tight">What Relay does</h2>
        <ul class="mt-10 grid gap-6 sm:grid-cols-2 lg:grid-cols-3">
          <li class="@container rounded-panel border border-line bg-bg p-6 print:break-inside-avoid">
            <div class="flex flex-col gap-3 @xs:flex-row @xs:items-start">
              <span class="grid size-10 shrink-0 place-items-center rounded-control bg-accent-soft text-accent-fg" aria-hidden="true">1</span>
              <div>
                <h3 class="font-semibold">Fair assignment</h3>
                <p class="mt-1 text-sm/6 text-fg-muted">New conversations go to whoever has the fewest open.</p>
              </div>
            </div>
          </li>
          <li class="@container rounded-panel border border-line bg-bg p-6 print:break-inside-avoid">
            <div class="flex flex-col gap-3 @xs:flex-row @xs:items-start">
              <span class="grid size-10 shrink-0 place-items-center rounded-control bg-accent-soft text-accent-fg" aria-hidden="true">2</span>
              <div>
                <h3 class="font-semibold">Collision warnings</h3>
                <p class="mt-1 text-sm/6 text-fg-muted">See when a teammate is already typing a reply.</p>
              </div>
            </div>
          </li>
          <li class="@container rounded-panel border border-line bg-bg p-6 print:break-inside-avoid">
            <div class="flex flex-col gap-3 @xs:flex-row @xs:items-start">
              <span class="grid size-10 shrink-0 place-items-center rounded-control bg-accent-soft text-accent-fg" aria-hidden="true">3</span>
              <div>
                <h3 class="font-semibold">Saved replies</h3>
                <p class="mt-1 text-sm/6 text-fg-muted">Answer common questions in two keystrokes.</p>
              </div>
            </div>
          </li>
        </ul>
      </div>
    </section>

    <section id="pricing" aria-labelledby="pricing-title" class="mx-auto max-w-page px-4 py-16 sm:px-6">
      <h2 id="pricing-title" class="text-3xl font-bold tracking-tight">Pricing</h2>
      <p class="mt-2 text-fg-muted">Example prices for this demo site.</p>
      <div class="mt-10 grid gap-6 md:grid-cols-3">
        <article class="rounded-panel border border-line p-6">
          <h3 class="font-semibold">Starter</h3>
          <p class="mt-2 text-3xl font-bold">$0</p>
          <p class="mt-1 text-sm text-fg-muted">Up to 2 teammates</p>
        </article>
        <article class="rounded-panel border-2 border-action p-6">
          <h3 class="flex items-center gap-2 font-semibold">
            Team <span class="rounded-full bg-accent-soft px-2 py-0.5 text-xs font-medium text-accent-fg">Popular</span>
          </h3>
          <p class="mt-2 text-3xl font-bold">$9 <span class="text-base font-normal text-fg-muted">per seat / month</span></p>
          <p class="mt-1 text-sm text-fg-muted">Up to 20 teammates</p>
        </article>
        <article class="rounded-panel border border-line p-6">
          <h3 class="font-semibold">Custom</h3>
          <p class="mt-2 text-3xl font-bold">Let's talk</p>
          <p class="mt-1 text-sm text-fg-muted">SSO and audit logs</p>
        </article>
      </div>
    </section>

    <section id="faq" aria-labelledby="faq-title" class="mx-auto max-w-3xl px-4 pb-20 sm:px-6">
      <h2 id="faq-title" class="text-3xl font-bold tracking-tight">Questions</h2>
      <div class="mt-6 divide-y divide-line border-y border-line">
        <details class="group py-4">
          <summary class="flex cursor-pointer list-none items-center justify-between gap-4 font-medium
                          focus-visible:outline-2 focus-visible:outline-offset-4 focus-visible:outline-action
                          [&::-webkit-details-marker]:hidden">
            Can I import old conversations?
            <svg class="size-5 shrink-0 text-fg-muted transition-transform group-open:rotate-180 motion-reduce:transition-none"
                 viewBox="0 0 20 20" fill="currentColor" aria-hidden="true">
              <path d="M5.3 7.3a1 1 0 0 1 1.4 0L10 10.6l3.3-3.3a1 1 0 1 1 1.4 1.4l-4 4a1 1 0 0 1-1.4 0l-4-4a1 1 0 0 1 0-1.4z"/>
            </svg>
          </summary>
          <p class="mt-3 text-fg-muted">Yes. In this demo the answer is illustrative only.</p>
        </details>
        <details class="group py-4">
          <summary class="flex cursor-pointer list-none items-center justify-between gap-4 font-medium
                          focus-visible:outline-2 focus-visible:outline-offset-4 focus-visible:outline-action
                          [&::-webkit-details-marker]:hidden">
            Does it work in right-to-left languages?
            <svg class="size-5 shrink-0 text-fg-muted transition-transform group-open:rotate-180 motion-reduce:transition-none"
                 viewBox="0 0 20 20" fill="currentColor" aria-hidden="true">
              <path d="M5.3 7.3a1 1 0 0 1 1.4 0L10 10.6l3.3-3.3a1 1 0 1 1 1.4 1.4l-4 4a1 1 0 0 1-1.4 0l-4-4a1 1 0 0 1 0-1.4z"/>
            </svg>
          </summary>
          <p class="mt-3 text-fg-muted">The layout uses logical utilities, so setting dir="rtl" mirrors it.</p>
        </details>
      </div>
    </section>
  </main>

  <footer class="border-t border-line">
    <div class="mx-auto flex max-w-page flex-wrap justify-between gap-2 px-4 py-8 text-sm text-fg-muted sm:px-6">
      <p>Relay is a fictional product used for this course.</p>
      <p class="hidden print:block">Printed from relay.example</p>
    </div>
  </footer>
  <script src="theme.js" type="module"></script>
</body>
</html>
```

What each section demonstrates:

- **Skip link**, visible on the first Tab press (`sr-only focus:not-sr-only`, with
  `focus:start-4` so it sits on the correct side in RTL).
- **Header**: `max-md:hidden` nav, `ms-auto` to push the actions to the end, an
  icon-only toggle with an `sr-only` label and a 40px target.
- **Hero**: `text-balance` on the heading and `text-pretty` on the paragraph.
- **Features**: each card is an `@container`; its icon sits beside the text once the
  *card* is at least `@xs` (20rem) wide, regardless of the screen.
- **Pricing**: the featured plan has a `border-2 border-action` and a "Popular" badge
  that's text, not colour alone. Prices are example values.
- **FAQ**: native `<details>` elements, keyboard-accessible with no JavaScript. The
  chevron rotates with `group-open:rotate-180`, `motion-reduce:transition-none` removes
  the animation for reduced-motion users, and `[&::-webkit-details-marker]:hidden` plus
  `list-none` remove the default marker.
- **Print**: header and buttons are `print:hidden`, feature cards don't split across
  pages, and a footer line appears only on paper.

## Step 4: the application shell

```html title="site/app.html"
<!doctype html>
<html lang="en" dir="ltr" class="scheme-light dark:scheme-dark">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Inbox — Relay</title>
  <script>
    try {
      const t = localStorage.getItem("theme");
      if (t === "dark" || (!t && matchMedia("(prefers-color-scheme: dark)").matches)) {
        document.documentElement.classList.add("dark");
      }
    } catch {}
  </script>
  <link rel="stylesheet" href="app.css">
</head>
<body class="bg-bg-subtle text-fg antialiased">
  <div class="min-h-dvh lg:grid lg:grid-cols-[var(--spacing-sidebar)_1fr]">

    <aside id="sidebar" data-open="false" aria-label="Workspace"
           class="fixed inset-y-0 start-0 z-40 w-sidebar -translate-x-full rtl:translate-x-full border-e border-line bg-bg
                  transition-transform duration-200 ease-out motion-reduce:transition-none
                  data-[open=true]:translate-x-0 lg:sticky lg:top-0 lg:h-dvh lg:translate-x-0 rtl:lg:translate-x-0
                  print:hidden">
      <div class="flex h-14 items-center gap-2 px-4 font-semibold">
        <span class="grid size-7 place-items-center rounded-control bg-action text-sm text-on-action" aria-hidden="true">R</span>
        Relay
      </div>
      <nav aria-label="Folders" class="px-2 py-2">
        <ul class="space-y-1 text-sm font-medium">
          <li><a href="#" aria-current="page"
                 class="flex items-center justify-between rounded-control px-3 py-2 text-fg-muted hover:bg-bg-subtle
                        aria-[current=page]:bg-accent-soft aria-[current=page]:text-accent-fg">
                Open <span class="tabular-nums">12</span></a></li>
          <li><a href="#" class="flex items-center justify-between rounded-control px-3 py-2 text-fg-muted hover:bg-bg-subtle">
                Waiting <span class="tabular-nums">4</span></a></li>
          <li><a href="#" class="flex items-center justify-between rounded-control px-3 py-2 text-fg-muted hover:bg-bg-subtle">
                Closed <span class="tabular-nums">87</span></a></li>
        </ul>
      </nav>
    </aside>
    <div id="backdrop" hidden class="fixed inset-0 z-30 bg-black/40 lg:hidden"></div>

    <div class="min-w-0">
      <header class="sticky top-0 z-20 flex h-14 items-center gap-3 border-b border-line bg-bg px-4 sm:px-6 print:static">
        <button id="menu" type="button" aria-controls="sidebar" aria-expanded="false"
                class="-ms-2 grid size-10 place-items-center rounded-control text-fg-muted hover:bg-bg-subtle lg:hidden
                       focus-visible:outline-2 focus-visible:outline-action print:hidden">
          <span class="sr-only">Open folders</span>
          <svg class="size-5" viewBox="0 0 20 20" fill="currentColor" aria-hidden="true"><path d="M3 5h14v2H3zm0 4h14v2H3zm0 4h14v2H3z"/></svg>
        </button>
        <h1 class="font-semibold">Open conversations</h1>
        <button type="button" data-theme-toggle
                class="ms-auto grid size-10 place-items-center rounded-control text-fg-muted hover:bg-bg-subtle
                       focus-visible:outline-2 focus-visible:outline-action print:hidden">
          <span class="sr-only">Dark theme</span>
          <svg class="size-5" viewBox="0 0 20 20" fill="currentColor" aria-hidden="true"><path d="M10 2a8 8 0 1 0 8 8 6 6 0 0 1-8-8z"/></svg>
        </button>
      </header>

      <main id="main" class="grid gap-6 p-4 sm:p-6 xl:grid-cols-[22rem_1fr]">
        <section aria-label="Conversation list" class="rounded-panel border border-line bg-bg">
          <ul class="divide-y divide-line">
            <li class="@container">
              <a href="#" aria-current="true" class="block p-4 hover:bg-bg-subtle aria-[current=true]:bg-accent-soft">
                <div class="flex flex-col gap-1 @xs:flex-row @xs:items-baseline @xs:justify-between">
                  <p class="font-medium">Ana Souza</p>
                  <span data-state="waiting" class="w-fit rounded-full px-2 py-0.5 text-xs font-medium
                         data-[state=waiting]:bg-amber-100 data-[state=waiting]:text-amber-900
                         dark:data-[state=waiting]:bg-amber-950 dark:data-[state=waiting]:text-amber-200">Waiting on you</span>
                </div>
                <p class="mt-1 line-clamp-2 text-sm text-fg-muted">Hi! I was charged twice for the March invoice and need a refund for one of them.</p>
              </a>
            </li>
            <li class="@container">
              <a href="#" class="block p-4 hover:bg-bg-subtle">
                <div class="flex flex-col gap-1 @xs:flex-row @xs:items-baseline @xs:justify-between">
                  <p class="font-medium">Kenji Watanabe</p>
                  <span data-state="replied" class="w-fit rounded-full px-2 py-0.5 text-xs font-medium
                         data-[state=replied]:bg-emerald-100 data-[state=replied]:text-emerald-900
                         dark:data-[state=replied]:bg-emerald-950 dark:data-[state=replied]:text-emerald-200">Replied</span>
                </div>
                <p class="mt-1 line-clamp-2 text-sm text-fg-muted">Thanks, the export works now.</p>
              </a>
            </li>
          </ul>
        </section>

        <article aria-labelledby="conv-title" class="rounded-panel border border-line bg-bg p-6">
          <h2 id="conv-title" class="text-lg font-semibold text-balance">Charged twice for the March invoice</h2>
          <p class="mt-1 text-sm text-fg-muted">Ana Souza · via email</p>
          <div class="mt-6 max-w-prose space-y-4 text-sm/6">
            <p>Hi! I was charged twice for the March invoice and need a refund for one of them.</p>
          </div>
          <form class="mt-8 print:hidden" action="#" method="post">
            <label for="reply" class="block text-sm font-medium">Reply</label>
            <textarea id="reply" rows="4"
                      class="mt-1 block w-full rounded-control border border-line bg-bg px-3 py-2 placeholder:text-fg-muted
                             focus:border-action focus:ring-2 focus:ring-action/30 focus:outline-hidden"
                      placeholder="Write a reply…"></textarea>
            <div class="mt-3 flex justify-end">
              <button class="rounded-control bg-action px-4 py-2 text-sm font-medium text-on-action hover:bg-action-hover
                             focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-action">Send</button>
            </div>
          </form>
        </article>
      </main>
    </div>
  </div>

  <script src="theme.js" type="module"></script>
  <script>
    const sidebar = document.getElementById("sidebar");
    const backdrop = document.getElementById("backdrop");
    const menu = document.getElementById("menu");
    const desktop = matchMedia("(width >= 64rem)");
    function setOpen(open) {
      sidebar.dataset.open = String(open);
      menu.setAttribute("aria-expanded", String(open));
      backdrop.hidden = !open;
      sidebar.inert = !open && !desktop.matches;
    }
    setOpen(false);
    desktop.addEventListener("change", () => setOpen(false));
    menu.addEventListener("click", () => setOpen(sidebar.dataset.open !== "true"));
    backdrop.addEventListener("click", () => setOpen(false));
    document.addEventListener("keydown", (e) => { if (e.key === "Escape") setOpen(false); });
  </script>
</body>
</html>
```

The parts to study:

- **Sidebar in both directions.** Below `lg` it's `fixed` at `start-0` and hidden with a
  transform. In LTR, `-translate-x-full` moves it off the left edge. In RTL, `start-0`
  puts it on the *right*, so it must move the other way: `rtl:translate-x-full`. A
  transform is a physical direction; there's no logical translate, so this is one of the
  places the `rtl:` variant is genuinely needed. Step 6 shows what happens without it.
- **`inert`** keeps the hidden sidebar's links out of the tab order on small screens.
- **State from attributes**: `aria-current="page"` styles the active folder,
  `aria-current="true"` the selected conversation, and `data-state` the status badges.
- **Container queries in list items**: each conversation's name and badge sit on one
  line when the list item is at least `@xs` wide and stack otherwise. The list column
  is 22rem wide on very large screens and full width on small ones, and the same markup
  adapts in both.
- **`line-clamp-2`** keeps message previews to two lines.
- **The reply form** uses `outline-hidden` with a ring (forced-colours safe), and the
  whole form is `print:hidden`.

## Step 5: the tests

```js title="playwright.config.js"
import { defineConfig, devices } from "@playwright/test";

export default defineConfig({
  testDir: "e2e",
  use: { baseURL: `file://${process.cwd()}/site/` },
  projects: [
    { name: "desktop", use: { ...devices["Desktop Chrome"] } },
    { name: "phone", use: { ...devices["Pixel 7"] } },
  ],
});
```

```js title="e2e/contrast.js"
// WCAG contrast for every visible element that has its own text.
// Ancestor backgrounds are painted onto a canvas so translucent layers blend correctly.
export function textContrast() {
  const c = document.createElement("canvas");
  c.width = c.height = 1;
  const x = c.getContext("2d", { willReadFrequently: true });
  const px = () => [...x.getImageData(0, 0, 1, 1).data].slice(0, 3);
  const lum = ([r, g, b]) => {
    const f = (v) => ((v /= 255) <= 0.04045 ? v / 12.92 : ((v + 0.055) / 1.055) ** 2.4);
    return 0.2126 * f(r) + 0.7152 * f(g) + 0.0722 * f(b);
  };
  const ownText = (el) =>
    [...el.childNodes].some((n) => n.nodeType === Node.TEXT_NODE && n.textContent.trim());
  const visible = (el) => el.getClientRects().length > 0 && el.offsetWidth > 1;

  return [...document.body.querySelectorAll("*")]
    .filter((el) => ownText(el) && visible(el) && el.tagName !== "SCRIPT")
    .map((el) => {
      const layers = [];
      for (let e = el; e; e = e.parentElement) {
        const bg = getComputedStyle(e).backgroundColor;
        layers.push(bg);
        x.clearRect(0, 0, 1, 1); x.fillStyle = bg; x.fillRect(0, 0, 1, 1);
        if (x.getImageData(0, 0, 1, 1).data[3] === 255) break;
      }
      x.fillStyle = "#fff"; x.fillRect(0, 0, 1, 1);
      for (const l of layers.reverse()) { x.fillStyle = l; x.fillRect(0, 0, 1, 1); }
      const bg = px();
      x.fillStyle = getComputedStyle(el).color; x.fillRect(0, 0, 1, 1);
      const [hi, lo] = [lum(px()), lum(bg)].sort((a, b) => b - a);
      return { text: el.textContent.trim().slice(0, 30), ratio: +((hi + 0.05) / (lo + 0.05)).toFixed(2) };
    });
}
```

This version measures **every visible element with its own text**, not a hand-picked
list, so new text is covered automatically.

```js title="e2e/site.spec.js"
import { test, expect } from "@playwright/test";
import { textContrast } from "./contrast.js";

const pages = ["index.html", "app.html"];

async function open(page, path, { theme = "light", dir = "ltr" } = {}) {
  await page.emulateMedia({ reducedMotion: "reduce", colorScheme: theme });
  await page.goto(path);
  if (dir === "rtl") await page.evaluate(() => (document.documentElement.dir = "rtl"));
  await page.waitForTimeout(300);
}

for (const path of pages) {
  for (const theme of ["light", "dark"]) {
    test(`${path} ${theme}: all text meets 4.5:1`, async ({ page }) => {
      await open(page, path, { theme });
      const results = await page.evaluate(textContrast);
      expect(results.length).toBeGreaterThan(10);
      const failing = results.filter((r) => r.ratio < 4.5);
      expect(failing, JSON.stringify(failing)).toEqual([]);
    });
  }

  test(`${path}: no horizontal overflow`, async ({ page }) => {
    await open(page, path);
    const overflow = await page.evaluate(
      () => document.documentElement.scrollWidth - document.documentElement.clientWidth
    );
    expect(overflow).toBe(0);
  });

  test(`${path}: matches the visual baseline`, async ({ page }) => {
    await open(page, path);
    await expect(page).toHaveScreenshot(`${path}.png`, { fullPage: true });
  });
}

test("skip link appears on first Tab", async ({ page }) => {
  await open(page, "index.html");
  await page.keyboard.press("Tab");
  const link = page.getByRole("link", { name: "Skip to content" });
  await expect(link).toBeFocused();
  expect((await link.boundingBox()).width).toBeGreaterThan(50);
});

test("focus outline survives forced colours", async ({ page }) => {
  await page.emulateMedia({ forcedColors: "active" });
  await page.goto("app.html");
  await page.locator("#reply").focus();
  await expect(page.locator("#reply")).toHaveCSS("outline-style", "solid");
});

test("printing hides the site header", async ({ page }) => {
  await open(page, "index.html");
  await page.emulateMedia({ media: "print" });
  await expect(page.locator("body > header")).toBeHidden();
});

test("sidebar mirrors in RTL", async ({ page, isMobile }) => {
  await open(page, "app.html", { dir: "rtl" });
  const sidebar = page.locator("#sidebar");
  const vw = page.viewportSize().width;
  if (isMobile) {
    // closed: pushed off the right edge; open: inside the viewport
    expect((await sidebar.boundingBox()).x).toBeGreaterThanOrEqual(vw);
    await page.getByRole("button", { name: "Open folders" }).click();
    await page.waitForTimeout(300);
    const box = await sidebar.boundingBox();
    expect(Math.round(box.x + box.width)).toBe(vw);
  } else {
    const box = await sidebar.boundingBox();
    expect(Math.round(box.x + box.width)).toBe(vw);
  }
});
```

```bash
npm run build
npx playwright test --update-snapshots   # first run: create baselines, then review them
npx playwright test
```

```text
24 passed (5.8s)
```

Twelve tests, each run on a desktop and a phone profile. To make sure the contrast test
was really checking something, we printed what it measured on the desktop profile:

```text
index.html light: 40 text elements, lowest 7.3  ("R")
index.html dark:  40 text elements, lowest 7.59 ("Features")
app.html   light: 20 text elements, lowest 7.04 ("Hi! I was charged twice for th…")
app.html   dark:  20 text elements, lowest 5.81 ("Hi! I was charged twice for th…")
```

Every text element on both pages, in both themes, is above 4.5:1, with the lowest at
5.81:1 (the message preview in the dark theme).

## Step 6: proving the tests catch real bugs

A passing suite only means something if it fails when the product is broken. We removed
`rtl:translate-x-full` from the sidebar, rebuilt, and ran the RTL test:

```text
✓ [desktop] › sidebar mirrors in RTL
✘ [phone]   › sidebar mirrors in RTL
    Expected: >= 412
    Received:    -68
```

On the desktop the sidebar is docked, so the bug doesn't show. On the 412px-wide phone
profile, the "hidden" sidebar was at x = −68: it had been moved 240px the wrong way, from
the right edge to partly across the left side of the screen, covering content while the
menu was closed. With the class restored, all 24 tests passed again.

It's worth doing this for each important test once: break the thing it protects and
watch it fail.

## Shipping checklist

- [ ] `npm run build` (minified CSS) runs in the deploy pipeline; output is fingerprinted
      and cached long-term (Level 3 · 09).
- [ ] Playwright runs in CI on the same OS as the baselines.
- [ ] Prettier with the Tailwind plugin and ESLint with the Tailwind correctness rules run
      on every commit (Level 4 · 01).
- [ ] Contrast, overflow, focus and RTL tests pass in both themes.
- [ ] A real screen reader pass on the main flows, and a manual check with "increase
      contrast" and forced colours.
- [ ] Print preview of the marketing page.

## How It Actually Works

The product is three layers that only meet in the browser. **Tokens** in `src/app.css`
become CSS variables, and every colour utility reads a variable, so a class on `<html>`
(`dark`) or an attribute (`dir="rtl"`) changes the whole interface without touching
component markup. **Markup** carries state as attributes (`aria-current`, `data-state`,
`data-open`, `aria-pressed`) that both assistive technology and Tailwind's variants read,
so styling and accessibility can't drift apart. **Media and container queries** adapt the
layout to the screen, the component's own width, print, motion and contrast settings,
all evaluated continuously by the browser.

The test suite works because Playwright drives a real engine and can emulate each of
those conditions. It checks outcomes (computed colours, positions, focus, visibility)
rather than class names, which is why it caught the RTL bug even though every class in
the file was valid.

## Exercise

1. Build Relay from this lesson and get all 24 tests passing on your machine.
2. Add a third page (a pricing page with a monthly/yearly toggle built from radio
   buttons and `has-checked:`), and extend the tests to cover it.
3. Add a second brand with `data-brand` (Level 4 · 02) and run the whole suite under
   it. Fix any contrast failures.
4. Translate the app shell's text into Arabic or Hebrew (or use placeholder RTL text),
   set `lang` and `dir`, and fix anything the RTL test or a visual review finds.
5. Break three things on purpose (a token, a focus style, a translate direction) and
   confirm a test fails for each. Add a test for any that slipped through.
