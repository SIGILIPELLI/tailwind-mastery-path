---
description: "Level 3 project: a tested, token-driven Tailwind component library with cva recipes, class merging, a gallery and a contrast audit."
---

# 10 · Project — A Component Library

This project turns Level 3 into one working package: a small **component library** that
several apps could share. It isn't tied to React or any framework. Each component is a
JavaScript function that returns a class string, so it works in React, Vue, Svelte, server
templates or plain HTML. You'll build:

- a **token theme** with semantic colours and a dark mode (lesson 07)
- a `cn` helper with **tailwind-merge configured for the custom tokens** (lesson 06)
- **cva recipes** for a button, badge, input and alert (lesson 06)
- a **generated gallery** page that renders every variant
- **unit tests** for the recipes (Node's built-in test runner)
- a **contrast and focus audit** of every variant, in light and dark mode (lesson 04)

Everything below was run while writing the lesson, with Tailwind 4.3.3,
class-variance-authority 0.7.1, tailwind-merge 3.7.0, clsx 2.1.1 and Node.js 26.

## Project layout

```text
ui-kit/
├── package.json
├── src/
│   ├── theme.css
│   ├── cn.js
│   └── components/
│       ├── button.js
│       ├── badge.js
│       ├── input.js
│       └── alert.js
├── scripts/
│   └── build-gallery.mjs
├── test/
│   └── components.test.js
└── gallery/            (generated: index.html, ui.css)
```

```json title="package.json"
{
  "name": "ui-kit",
  "version": "0.1.0",
  "type": "module",
  "scripts": {
    "gallery": "node scripts/build-gallery.mjs",
    "css": "tailwindcss -i src/theme.css -o gallery/ui.css --minify",
    "build": "npm run gallery && npm run css",
    "test": "node --test"
  },
  "devDependencies": {
    "@tailwindcss/cli": "^4.3.3",
    "tailwindcss": "^4.3.3"
  },
  "dependencies": {
    "class-variance-authority": "^0.7.1",
    "clsx": "^2.1.1",
    "tailwind-merge": "^3.7.0"
  }
}
```

```bash
npm install -D tailwindcss @tailwindcss/cli
npm install clsx tailwind-merge class-variance-authority
```

## Step 1: the token theme

```css title="src/theme.css"
@import "tailwindcss";
@source "../gallery";
@source "./components";

@custom-variant dark (&:where(.dark, .dark *));

@theme {
  --color-surface: var(--color-white);
  --color-surface-muted: var(--color-slate-50);
  --color-fg: var(--color-slate-900);
  --color-fg-muted: var(--color-slate-600);
  --color-line: var(--color-slate-300);
  --color-action: var(--color-indigo-700);
  --color-action-hover: var(--color-indigo-800);
  --color-on-action: var(--color-white);
  --color-danger: var(--color-red-700);
  --color-danger-hover: var(--color-red-800);

  --radius-control: 0.5rem;
  --radius-panel: 0.75rem;
}

@layer base {
  .dark {
    --color-surface: var(--color-slate-900);
    --color-surface-muted: var(--color-slate-800);
    --color-fg: var(--color-slate-100);
    --color-fg-muted: var(--color-slate-400);
    --color-line: var(--color-slate-600);
    --color-action: var(--color-indigo-300);
    --color-action-hover: var(--color-indigo-200);
    --color-on-action: var(--color-slate-950);
    --color-danger: var(--color-red-300);
    --color-danger-hover: var(--color-red-200);
  }
}
```

- `@source` lines tell Tailwind exactly where class names live: the component recipes
  and the generated gallery (lesson 01).
- Components will only use the semantic tokens (`bg-action`, `text-fg-muted`,
  `rounded-control`), so dark mode is just the `.dark` rule reassigning them.
- `--color-on-action` is the text colour that sits on an action background. It's white in
  light mode, where the action colour is dark, and near-black in dark mode, where the
  action colour (`indigo-300`) is light. A single "white text on brand colour" rule can't
  pass contrast in both modes.

## Step 2: class merging that knows the tokens

```js title="src/cn.js"
import { clsx } from "clsx";
import { extendTailwindMerge } from "tailwind-merge";

// Teach tailwind-merge the custom tokens from theme.css
const twMerge = extendTailwindMerge({
  extend: {
    theme: {
      color: ["surface", "surface-muted", "fg", "fg-muted", "line", "action", "action-hover",
              "on-action", "danger", "danger-hover"],
      radius: ["control", "panel"],
    },
  },
});

export function cn(...inputs) {
  return twMerge(clsx(inputs));
}
```

Without this configuration, tailwind-merge doesn't know these names, and lesson 06
showed it mis-grouping an unknown custom token and dropping a colour. The tests in Step 5
check the configured behaviour.

## Step 3: the recipes

```js title="src/components/button.js"
import { cva } from "class-variance-authority";
import { cn } from "../cn.js";

export const buttonStyles = cva(
  "inline-flex items-center justify-center gap-2 rounded-control font-medium " +
    "transition-colors focus-visible:outline-2 focus-visible:outline-offset-2 " +
    "disabled:pointer-events-none disabled:opacity-50",
  {
    variants: {
      intent: {
        primary: "bg-action text-on-action hover:bg-action-hover focus-visible:outline-action",
        secondary: "bg-surface text-fg ring-1 ring-line hover:bg-surface-muted focus-visible:outline-action",
        danger: "bg-danger text-on-action hover:bg-danger-hover focus-visible:outline-danger",
        ghost: "text-fg hover:bg-surface-muted focus-visible:outline-action",
      },
      size: {
        sm: "h-8 px-3 text-sm",
        md: "h-10 px-4 text-sm",
        lg: "h-12 px-6 text-base",
      },
    },
    defaultVariants: { intent: "primary", size: "md" },
  }
);

export const button = ({ intent, size, class: extra } = {}) =>
  cn(buttonStyles({ intent, size }), extra);
```

```js title="src/components/badge.js"
import { cva } from "class-variance-authority";
import { cn } from "../cn.js";

export const badgeStyles = cva(
  "inline-flex items-center gap-1 rounded-full px-2 py-0.5 text-xs font-medium ring-1 ring-inset",
  {
    variants: {
      tone: {
        neutral: "bg-slate-100 text-slate-800 ring-slate-300 dark:bg-slate-800 dark:text-slate-200 dark:ring-slate-600",
        success: "bg-emerald-50 text-emerald-800 ring-emerald-300 dark:bg-emerald-950 dark:text-emerald-300 dark:ring-emerald-800",
        warning: "bg-amber-50 text-amber-900 ring-amber-300 dark:bg-amber-950 dark:text-amber-300 dark:ring-amber-800",
        danger: "bg-red-50 text-red-800 ring-red-300 dark:bg-red-950 dark:text-red-300 dark:ring-red-800",
      },
    },
    defaultVariants: { tone: "neutral" },
  }
);

export const badge = ({ tone, class: extra } = {}) => cn(badgeStyles({ tone }), extra);
```

```js title="src/components/input.js"
import { cn } from "../cn.js";

export const input = ({ invalid = false, class: extra } = {}) =>
  cn(
    "block w-full rounded-control border bg-surface px-3 py-2 text-fg placeholder:text-fg-muted " +
      "focus:ring-2 focus:outline-hidden disabled:cursor-not-allowed disabled:opacity-60",
    invalid
      ? "border-danger focus:ring-danger/40"
      : "border-line focus:border-action focus:ring-action/30",
    extra
  );

export const label = "block text-sm font-medium text-fg";
export const hint = "mt-1 text-sm text-fg-muted";
export const errorText = "mt-1 text-sm text-danger";
```

```js title="src/components/alert.js"
import { cva } from "class-variance-authority";
import { cn } from "../cn.js";

export const alertStyles = cva("flex gap-3 rounded-panel border p-4 text-sm", {
  variants: {
    tone: {
      info: "border-sky-300 bg-sky-50 text-sky-900 dark:border-sky-800 dark:bg-sky-950 dark:text-sky-200",
      success: "border-emerald-300 bg-emerald-50 text-emerald-900 dark:border-emerald-800 dark:bg-emerald-950 dark:text-emerald-200",
      danger: "border-red-300 bg-red-50 text-red-900 dark:border-red-800 dark:bg-red-950 dark:text-red-200",
    },
  },
  defaultVariants: { tone: "info" },
});

export const alert = ({ tone, class: extra } = {}) => cn(alertStyles({ tone }), extra);
```

Design notes:

- Every public function takes an optional `class` for **small** caller overrides, merged
  last with `cn`.
- Buttons focus with an **outline**, and inputs use `outline-hidden` plus a ring, so both
  keep a visible indicator in forced-colours mode (lesson 04).
- Badges and alerts use explicit palette shades with `dark:` variants, because they need
  their own tinted colours rather than surface tokens. Their text shades are 800/900 in
  light mode and 200/300 in dark mode, chosen for contrast and then measured (Step 6).
- `disabled:pointer-events-none` stops hover styles on disabled buttons.
- Every class is written in full in the source, so the scanner sees all of them.

## Step 4: a generated gallery

A gallery that renders every variant is both documentation and the test bed for visual
and accessibility checks. This script imports the recipes and writes a static page:

```js title="scripts/build-gallery.mjs"
// Render every component variant into gallery/index.html
import { writeFileSync } from "node:fs";
import { button } from "../src/components/button.js";
import { badge } from "../src/components/badge.js";
import { input, label, hint, errorText } from "../src/components/input.js";
import { alert } from "../src/components/alert.js";

const intents = ["primary", "secondary", "danger", "ghost"];
const sizes = ["sm", "md", "lg"];
const tones = ["neutral", "success", "warning", "danger"];

const section = (title, body) => `
    <section class="space-y-4">
      <h2 class="text-lg font-semibold text-fg">${title}</h2>
      ${body}
    </section>`;

const buttons = intents
  .map((intent) => `<div class="flex flex-wrap items-center gap-3">
        ${sizes.map((size) => `<button type="button" data-test="button-${intent}-${size}" class="${button({ intent, size })}">${intent} ${size}</button>`).join("\n        ")}
        <button type="button" disabled class="${button({ intent })}">disabled</button>
      </div>`)
  .join("\n      ");

const badges = `<div class="flex flex-wrap gap-2">
        ${tones.map((tone) => `<span data-test="badge-${tone}" class="${badge({ tone })}">${tone}</span>`).join("\n        ")}
      </div>`;

const fields = `<div class="grid gap-6 sm:grid-cols-2">
        <div>
          <label for="name" class="${label}">Name</label>
          <input id="name" class="${input({ class: "mt-1" })}" placeholder="Ada Lovelace" aria-describedby="name-hint">
          <p id="name-hint" data-test="hint" class="${hint}">As it appears on your card.</p>
        </div>
        <div>
          <label for="email" class="${label}">Email</label>
          <input id="email" class="${input({ invalid: true, class: "mt-1" })}" value="ada@" aria-invalid="true" aria-describedby="email-error">
          <p id="email-error" data-test="error" class="${errorText}">Enter a complete email address.</p>
        </div>
      </div>`;

const alerts = ["info", "success", "danger"]
  .map((tone) => `<div role="${tone === "danger" ? "alert" : "status"}" data-test="alert-${tone}" class="${alert({ tone })}">
        <p><strong class="font-semibold">${tone[0].toUpperCase() + tone.slice(1)}:</strong> an example ${tone} message.</p>
      </div>`)
  .join("\n      ");

const html = `<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>UI kit gallery</title>
  <link rel="stylesheet" href="ui.css">
</head>
<body class="bg-surface text-fg antialiased">
  <main class="mx-auto max-w-4xl space-y-12 px-4 py-10 sm:px-6">
    <h1 class="text-2xl font-bold tracking-tight">UI kit</h1>
    ${section("Buttons", buttons)}
    ${section("Badges", badges)}
    ${section("Fields", fields)}
    ${section("Alerts", alerts)}
  </main>
</body>
</html>
`;

writeFileSync(new URL("../gallery/index.html", import.meta.url), html);
console.log("wrote gallery/index.html");
```

The `data-test` attributes give every rendered variant a stable name for the audit in
Step 6.

```bash
npm run build
```

```text
wrote gallery/index.html
≈ tailwindcss v4.3.3
Done in 54ms
```

The minified `gallery/ui.css` was 17,086 bytes.

## Step 5: unit tests

```js title="test/components.test.js"
import { test } from "node:test";
import assert from "node:assert/strict";
import { button } from "../src/components/button.js";
import { input } from "../src/components/input.js";
import { cn } from "../src/cn.js";

test("default button is primary and medium", () => {
  const c = button();
  assert.match(c, /\bbg-action\b/);
  assert.match(c, /\bh-10\b/);
});

test("caller overrides replace conflicting classes", () => {
  const c = button({ size: "sm", class: "px-8 w-full" });
  assert.match(c, /\bpx-8\b/);
  assert.doesNotMatch(c, /\bpx-3\b/);
  assert.match(c, /\bw-full\b/);
});

test("custom colour tokens merge as colours", () => {
  assert.equal(cn("text-fg", "text-fg-muted"), "text-fg-muted");
  assert.equal(cn("text-sm text-fg", "text-danger"), "text-sm text-danger");
  assert.equal(cn("rounded-control", "rounded-panel"), "rounded-panel");
});

test("invalid input swaps the border colour, not the radius", () => {
  const c = input({ invalid: true });
  assert.match(c, /\bborder-danger\b/);
  assert.doesNotMatch(c, /\bborder-line\b/);
  assert.match(c, /\brounded-control\b/);
});
```

```bash
npm test
```

```text
✔ default button is primary and medium
✔ caller overrides replace conflicting classes
✔ custom colour tokens merge as colours
✔ invalid input swaps the border colour, not the radius
ℹ tests 4
ℹ pass 4
ℹ fail 0
```

(Timing figures trimmed.) These tests check the **class strings**, which are the API of
this library. They're fast and catch the most common regression: a refactor that breaks
merging or default variants. They can't tell you whether the result *looks* right;
that's Step 6 and Level 4 · 09.

## Step 6: audit the rendered gallery

We loaded the gallery in Chromium (with Playwright), and for every `[data-test]` element
computed the contrast between its text colour and its effective background, painting
each ancestor's background onto a canvas so translucent layers blend correctly. Then we
added the `dark` class and repeated it.

```text
light: 21 elements, minimum 6.42:1, none below 4.5
dark:  21 elements, minimum 6.78:1, none below 4.5
```

Selected results:

```text
                     light    dark
button primary       8.09     10.03
button secondary     17.83    16.28
button danger        6.42     10.50
badge warning        8.73     10.37
input hint           7.58     6.78
input error          6.42     9.29
alert info           8.89     10.44
```

With forced colours emulated, keyboard focus on the primary button and on both inputs
computed to a `solid 2px` outline, so focus stays visible in that mode. The gallery had
no horizontal overflow at 375px or 1024px.

### A bug the audit found: transitions during theme switches

Our first dark-mode run reported the ghost buttons at **1:1** contrast. Nothing was wrong
with the colours. The buttons have `transition-colors`, and the audit measured them a
moment after adding the `dark` class, while they were still animating from their light
colours. Users see the same thing: every element with a colour transition visibly fades
when the theme changes, which looks sluggish.

The usual fix is to switch off transitions for the moment of the switch:

```css title="add to src/theme.css"
@layer base {
  html[data-switching] *,
  html[data-switching] *::before,
  html[data-switching] *::after {
    transition: none !important;
  }
}
```

```js
function setTheme(dark) {
  const html = document.documentElement;
  html.setAttribute("data-switching", "");
  html.classList.toggle("dark", dark);
  getComputedStyle(html).color; // apply the new styles while transitions are off
  requestAnimationFrame(() => html.removeAttribute("data-switching"));
}
```

We measured a secondary button 50ms after switching. Without the fix its background was
still mid-fade, with 2 animations running. With the fix it was already at the final dark
colour, with no animations running. This is one of the rare places `!important` is the
right tool: it's a short-lived override that must beat every utility.

## How It Actually Works

The library's real output is **strings**. Each recipe is a lookup that concatenates
classes, and `cn` resolves conflicts between the recipe and the caller. Because every
class appears in full in `src/components`, and the theme's `@source` points there, the
Tailwind compiler generates CSS for every possible variant, whatever an app actually
renders. An app that consumes the library from `node_modules` needs an `@source` for
the package for the same reason.

Theming works through the cascade: utilities read `var(--color-action)` and friends, and
the `.dark` rule (in the `base` layer) redefines them for everything inside. The contrast
audit is the WCAG formula applied to computed colours, and it only works on a *rendered*
page, which is why the gallery exists.

## Common mistakes

- **Testing class strings only.** They can be correct while the rendered result fails
  contrast. Audit the rendered page.
- **One "on brand" text colour for both modes.** Light and dark brand colours usually
  need different text colours on top.
- **Forgetting `@source`** for the component folder (in the library) or the package (in
  apps).
- **Measuring right after a theme switch** without waiting for transitions, or shipping
  a theme switch that animates every element.
- **Letting apps pass large overrides** instead of adding variants.

## Exercise

1. Build the library and run the tests. Add a `link` intent to the button and a test for
   it.
2. Add a `Card` recipe (`padding` variants `sm`/`md`, an `interactive` boolean that adds
   hover styles) and render it in the gallery.
3. Write your own contrast audit (Playwright, or by hand with DevTools) and check every
   variant in both modes. Fix anything under 4.5:1.
4. Add the `data-switching` fix and a theme toggle to the gallery, and confirm colours
   switch instantly.
5. Consume the library from a second small app: import its `theme.css`, add an `@source`
   for the components, and render two buttons.
