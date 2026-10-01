---
description: "Migrate Tailwind v3 to v4 with the official upgrade tool, run on a real project: what it converted, what it missed, and what changed visually."
---

# 08 · Migrating from v3 to v4

Tailwind v4 is a rewrite: configuration moved from JavaScript into CSS, several utilities
were renamed, a few defaults changed, and some deprecated utilities were removed. There's
an official upgrade tool that does most of the work. "Most" is the important word, so
instead of listing every change from memory, this lesson runs the tool on a small but
realistic v3 project and compares the result, rendered, against the original.

## The v3 project

A typical v3 setup: a JavaScript config, the three `@tailwind` directives, a component
class built with `@apply`, and markup using a mix of classes that changed in v4.

```js title="tailwind.config.js"
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: ["./src/**/*.{html,js}"],
  darkMode: "class",
  theme: {
    extend: {
      colors: {
        brand: { 50: "#eef6ff", 600: "#1d5fd1", 700: "#174ca8" },
      },
      fontFamily: {
        display: ["Inter Tight", "ui-sans-serif", "system-ui", "sans-serif"],
      },
      borderRadius: { card: "0.875rem" },
    },
  },
  plugins: [],
};
```

```css title="src/input.css"
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer components {
  .btn {
    @apply rounded px-4 py-2 font-medium shadow-sm focus:outline-none focus:ring;
  }
}
```

```html title="src/index.html (body)"
<body class="bg-gray-50 dark:bg-gray-900">
  <div class="rounded-card bg-white p-6 shadow">
    <h1 class="font-display text-2xl font-bold text-brand-700">Hello</h1>
    <p class="mt-2 text-gray-600 text-opacity-80">Body text</p>
    <div class="mt-4 flex flex-grow-0 flex-shrink space-x-2">
      <button class="btn bg-brand-600 bg-opacity-90 text-white ring-brand-600">Save</button>
      <button class="!rounded-full border px-3 py-2 outline-none ring-1 bg-[--accent]">Cancel</button>
      <input class="rounded-sm border shadow-sm blur-sm decoration-slice overflow-ellipsis"
             placeholder="Name">
    </div>
  </div>
</body>
```

It built with Tailwind 3.4.19. We committed it to git first, which is essential: the
upgrade tool edits files in place, and `git diff` is how you review what it did.

## Running the upgrade tool

```bash
npx @tailwindcss/upgrade
```

The tool reported each step: it linked the config to the stylesheet, migrated the config
file, migrated the stylesheet, updated the `tailwindcss` dependency (to `^4.3.3`),
migrated the template, and skipped PostCSS because there was no PostCSS config. Then
it deleted `tailwind.config.js`.

Run it on a clean working tree, on a branch, with a recent Node.js version.

## What it converted

**The stylesheet** became CSS-first configuration:

```css title="src/input.css (after)"
@import 'tailwindcss';

@custom-variant dark (&:is(.dark *));

@theme {
  --color-brand-50: #eef6ff;
  --color-brand-600: #1d5fd1;
  --color-brand-700: #174ca8;

  --font-display: Inter Tight, ui-sans-serif, system-ui, sans-serif;

  --radius-card: 0.875rem;
}

/*
  The default border color has changed to `currentcolor` in Tailwind CSS v4,
  so we've added these compatibility styles to make sure everything still
  looks the same as it did with Tailwind CSS v3.
  …
*/
@layer base {
  *, ::after, ::before, ::backdrop, ::file-selector-button {
    border-color: var(--color-gray-200, currentcolor);
  }
}

@utility btn {
  @apply rounded-sm px-4 py-2 font-medium shadow-xs focus:outline-hidden focus:ring-3;
}
```

- The JavaScript theme became an `@theme` block (Level 2 · 01).
- `darkMode: "class"` became a `@custom-variant`.
- A **compatibility block** keeps v3's gray-200 default border colour (v4 defaults to
  `currentcolor`).
- Renamed utilities were updated inside `@apply` too.

**The template** diff:

```diff
-  <div class="rounded-card bg-white p-6 shadow">
+  <div class="rounded-card bg-white p-6 shadow-sm">
-    <div class="mt-4 flex flex-grow-0 flex-shrink space-x-2">
+    <div class="mt-4 flex grow-0 shrink space-x-2">
-      <button class="!rounded-full border px-3 py-2 outline-none ring-1 bg-[--accent]">Cancel</button>
-      <input class="rounded-sm border shadow-sm blur-sm decoration-slice overflow-ellipsis" …>
+      <button class="rounded-full! border px-3 py-2 outline-hidden ring-1 bg-(--accent)">Cancel</button>
+      <input class="rounded-xs border shadow-xs blur-xs box-decoration-slice text-ellipsis" …>
```

These are the v4 renames, and they explain each other:

| v3 | v4 | Why |
|---|---|---|
| `shadow-sm` / `shadow` | `shadow-xs` / `shadow-sm` | the scale shifted down one step to make room |
| `rounded-sm` / `rounded` | `rounded-xs` / `rounded-sm` | same shift |
| `blur-sm` / `blur` | `blur-xs` / `blur-sm` | same shift |
| `ring` (3px) | `ring-3` | bare `ring` is now 1px |
| `outline-none` | `outline-hidden` | `outline-none` now really sets `outline-style: none` |
| `!rounded-full` | `rounded-full!` | important modifier moved to the end |
| `bg-[--accent]` | `bg-(--accent)` | new variable shorthand |
| `flex-grow-0` / `flex-shrink` | `grow-0` / `shrink` | deprecated aliases removed |
| `decoration-slice`, `overflow-ellipsis` | `box-decoration-slice`, `text-ellipsis` | deprecated aliases removed |

The renames preserved appearance: the button's radius stayed 4px, the input's stayed 2px,
and the focused `.btn` still had a 3px ring.

## What it didn't convert

Building the migrated project with v4 (we also had to `npm install -D @tailwindcss/cli`,
because the tool only updated `tailwindcss`, and in v4 the CLI is a separate package)
and searching the output revealed three things left behind:

1. **`bg-opacity-90` and `text-opacity-80` were left in the HTML** and generate **nothing**
   in v4. The opacity utilities were removed in favour of modifiers.
2. **`space-x-2` was left as is**, but its selector changed. v4 generated
   `:where(.space-x-2 > :not(:last-child))` with margins on the inline *end*; v3
   targeted `> :not([hidden]) ~ :not([hidden])` with margins on the *start*. Usually
   that looks the same; with hidden children or wrapped rows it can differ. Prefer
   `gap-2` on flex and grid containers.
3. **`.btn` moved from `@layer components` to `@utility btn`.** It's now a utility,
   sorted among utilities, so the "a utility always overrides `.btn`" guarantee from
   Level 2 · 05 no longer holds automatically.

## Measured differences

We rendered the original (v3.4.19) and migrated (v4.3.3) pages in Chromium and compared
computed styles:

```text
                     v3                       v4 (migrated)
.btn background      rgba(29, 95, 209, 0.9)   rgb(29, 95, 209)       <- opacity lost
<p> colour           rgba(75, 85, 99, 0.8)    oklch(0.446 0.03 …)     <- opacity lost, new palette
input border colour  rgb(229, 231, 235)       oklch(0.928 0.006 …)    (gray-200, kept by compat block)
.btn border-radius   4px                      4px
input border-radius  2px                      2px
focused .btn ring    3px                      3px
Cancel button cursor pointer                  default                 <- Preflight change
```

So after the tool, a manual pass is still needed:

```diff
- <p class="mt-2 text-gray-600 text-opacity-80">
+ <p class="mt-2 text-gray-600/80">
- <button class="btn bg-brand-600 bg-opacity-90 …">
+ <button class="btn bg-brand-600/90 …">
- <div class="mt-4 flex grow-0 shrink space-x-2">
+ <div class="mt-4 flex grow-0 shrink gap-2">
```

…and, if you want v3's pointer cursor on buttons, a base style:

```css
@layer base {
  button:not(:disabled), [role="button"]:not(:disabled) { cursor: pointer; }
}
```

The palette moved from RGB to `oklch` in v4, so colours like `gray-600` are very close
but not byte-identical. Compare important brand screens visually.

## Other changes to check by hand

The upgrade tool can only change what it can see. Review these yourself:

- **Dynamic class names** built in JavaScript (lesson 01) are invisible to the tool too.
  Search for renamed utilities (`shadow-sm`, `rounded`, `ring`, `outline-none`, `blur`)
  in strings the tool may have skipped.
- **Third-party plugins.** JavaScript plugins still load with `@plugin`, but check each
  one supports v4.
- **`theme()` calls in CSS.** Prefer the CSS variables (`var(--color-brand-600)`).
- **Browser support.** v4 targets modern browsers (it relies on cascade layers,
  `@property` and `color-mix()`). If you must support old browsers, stay on v3.4.
- **Hover on touch devices.** `hover:` is now wrapped in `@media (hover: hover)` (Level 1
  · 08). Interfaces that relied on tap-to-hover will behave differently.
- **Transforms.** v4 uses the individual `translate`/`scale`/`rotate` properties. Custom
  CSS with `transition-property: transform` stops animating them (Level 2 · 09).
- **Build tooling.** The Vite plugin is `@tailwindcss/vite` and the PostCSS plugin is
  `@tailwindcss/postcss`. The tool migrates PostCSS configs it finds; check Vite
  configs yourself.

## A migration plan for a real codebase

1. Branch, clean working tree, take screenshots of key pages (or run your visual tests).
2. Run `npx @tailwindcss/upgrade`. Review `git diff` file by file.
3. Install the v4 build package you need (`@tailwindcss/cli`, `@tailwindcss/vite` or
   `@tailwindcss/postcss`).
4. Search for removed utilities the tool left behind: `-opacity-`, `space-x-`,
   `space-y-`, `divide-` (check), and dynamic strings.
5. Decide what to do with converted `@utility` components: move them back to
   `@layer components` if you relied on utilities overriding them.
6. Compare screenshots (Level 4 · 09 automates this). Fix differences.
7. Remove the compatibility border block once every bordered element has an explicit
   colour.

## How It Actually Works

The upgrade tool reads your v3 config and writes the equivalent v4 CSS: theme values become `@theme` variables, `darkMode` becomes a custom
variant, and plugins it understands become `@plugin` lines. For templates, it scans
files the same way the compiler does, parses each candidate with v3 semantics, and
rewrites it to the v4 spelling when there's a known mapping (renamed scale steps,
removed aliases, the important modifier, the variable shorthand). Candidates without a
v4 equivalent, like `bg-opacity-90`, have no mapping: the tool can't know which colour
utility on the element to attach `/90` to in every case, so it leaves them, and the v4
compiler silently ignores them because they're no longer valid utilities.

## Common mistakes

- **Running the tool on uncommitted work.** You lose the ability to review its changes.
- **Assuming the tool caught everything.** Check for `*-opacity-*` and dynamic classes.
- **Forgetting to install the CLI or build plugin** for v4.
- **Not noticing `@layer components` classes became `@utility`.**
- **Skipping a visual comparison.** Several differences only show up when rendered.
- **Migrating while also redesigning.** Do the migration first, ship it, then change the
  design.

## Exercise

1. Create a small v3 project with the classes above, commit it, and run the upgrade
   tool. Compare your diff with this lesson's.
2. Search the migrated output for `bg-opacity` and `text-opacity` and fix them with
   opacity modifiers.
3. Render both versions and compare computed styles for five elements of your choice.
4. Write a short migration checklist for your own codebase, including which pages need
   visual checks.
