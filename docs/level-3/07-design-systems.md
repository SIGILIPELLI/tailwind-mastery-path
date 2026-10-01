---
description: "Build a design system on Tailwind v4: primitive and semantic token tiers, a restricted palette, a shared token package and guardrails."
---

# 07 · Building a Design System on Tailwind

A design system is the shared set of decisions behind an interface: which colours,
which type sizes, which spacing, which components and how they behave. Tailwind is a
good foundation for one, because its utilities already *are* a vocabulary of design
decisions. But the default theme offers far more choices than any product should use.
Twenty-six colour families with eleven shades each is a palette for everyone, which
makes it a palette for no one.

This lesson turns Tailwind into a system for **one** product (or a family of products):
tiers of tokens, a restricted palette, a token package that several apps can import,
component recipes, and guardrails that keep the system from drifting.

## Tier 1: primitive tokens

Primitives are the raw values: every colour and size you're allowed to use, named by
what they **are**:

```css
@theme {
  --color-*: initial;                       /* drop the default palette */

  --color-white: #fff;
  --color-ink-50:  oklch(98% 0.005 260);
  --color-ink-200: oklch(91% 0.01 260);
  --color-ink-600: oklch(45% 0.02 260);
  --color-ink-900: oklch(22% 0.02 260);
  --color-blue-50:  oklch(97% 0.02 255);
  --color-blue-700: oklch(48% 0.17 255);
  --color-red-700:  oklch(50% 0.19 25);
}
```

Resetting `--color-*` (Level 2 · 01) is the key step. After it, `bg-sky-500` no longer
exists: we confirmed it generated nothing in the test package below. Every colour in the
product must come from your list, and a typo or an off-system colour simply produces no
CSS, which you'll notice immediately.

You can reset other namespaces the same way (`--shadow-*`, `--radius-*`), but leave
spacing and breakpoints alone unless you have a strong reason. Their defaults are
reasonable, and replacing them makes every Tailwind example on the internet wrong for
your project.

## Tier 2: semantic tokens

Semantic tokens are named by what they're **for**, and point at primitives:

```css
@theme {
  --color-surface: var(--color-white);
  --color-surface-muted: var(--color-ink-50);
  --color-fg: var(--color-ink-900);
  --color-fg-muted: var(--color-ink-600);
  --color-line: var(--color-ink-200);
  --color-action: var(--color-blue-700);
  --color-action-subtle: var(--color-blue-50);
  --color-danger: var(--color-red-700);

  --radius-control: 0.5rem;
  --radius-panel: 0.75rem;
}
```

Components use the semantic layer almost exclusively: `bg-action`, `text-fg-muted`,
`border-line`, `rounded-control`. This is what makes dark mode and multi-brand theming
(Level 4 · 02) a matter of reassigning a dozen variables rather than editing every
component.

Naming rules that hold up:

- **Read well after any prefix.** `text-fg-muted`, `bg-surface`, `border-line`; avoid
  `--color-text-*` (`text-text-…`) and `--color-bg-*` used with `border-`.
- **Describe purpose, not appearance.** `action`, not `blue`. When the brand colour
  changes, `action` is still correct.
- **Keep the set small.** If two tokens always have the same value in every theme, they
  should probably be one.

## Tier 3: component recipes

The third tier is components themselves, written with semantic tokens. In a component
framework they're `cva` recipes (lesson 06). Without one, they're partials or
`@layer components` classes:

```js title="packages/ui/src/button.js"
import { cva } from "class-variance-authority";

export const button = cva(
  "inline-flex items-center justify-center gap-2 rounded-control font-medium " +
    "focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-action " +
    "disabled:opacity-50",
  {
    variants: {
      intent: {
        primary: "bg-action text-white hover:bg-action/90",
        secondary: "bg-surface text-fg ring-1 ring-line hover:bg-surface-muted",
        danger: "bg-danger text-white hover:bg-danger/90",
      },
      size: { sm: "h-8 px-3 text-sm", md: "h-10 px-4 text-sm" },
    },
    defaultVariants: { intent: "primary", size: "md" },
  }
);
```

No palette colour names appear in the component. Retheme the tokens and every button
follows.

## Sharing the system: a token package

When several apps share a system, publish the tokens as a CSS file in a package. We
built one locally to check that this works:

```text
node_modules/@acme/tokens/
├── package.json
└── theme.css
```

```json title="node_modules/@acme/tokens/package.json"
{
  "name": "@acme/tokens",
  "version": "1.0.0",
  "style": "theme.css",
  "exports": { ".": { "style": "./theme.css" }, "./theme.css": "./theme.css" }
}
```

`theme.css` holds the `@theme` blocks above and any shared custom utilities:

```css title="theme.css (excerpt)"
@utility focus-ring {
  outline: 2px solid var(--color-action);
  outline-offset: 2px;
}
```

Each app's stylesheet then imports Tailwind and the tokens:

```css title="src/app.css"
@import "tailwindcss";
@import "@acme/tokens";
```

Compiling an app page that used `bg-action text-white rounded-control
focus-visible:focus-ring bg-sky-500 text-fg-muted` generated `bg-action`,
`text-white`, `rounded-control`, `text-fg-muted` and `focus-visible:focus-ring` — and
nothing for `bg-sky-500`, because the package reset the palette. The import resolved
through the package's `style` export condition, which Tailwind's import resolver
supports.

If the package also ships components with Tailwind classes (in `node_modules`), remember
to add an `@source` for it in each app (lesson 01).

## Documenting tokens

A token list that's generated can't go out of date. Because the tokens are plain CSS,
a few lines of Node.js can list them:

```js title="scripts/list-tokens.mjs"
// Print every design token declared in @theme blocks of a CSS file
import { readFileSync } from "node:fs";

const css = readFileSync(process.argv[2], "utf8");
const themeBlocks = [...css.matchAll(/@theme[^{]*\{([\s\S]*?)\n\}/g)].map((m) => m[1]);

for (const block of themeBlocks) {
  for (const [, name, value] of block.matchAll(/(--[\w-]+(?:\*)?)\s*:\s*([^;]+);/g)) {
    if (name.endsWith("*")) continue; // namespace resets
    const [, namespace] = name.match(/^--([a-z]+)/);
    console.log(`${namespace.padEnd(8)} ${name.padEnd(20)} ${value.trim()}`);
  }
}
```

Run against our package's `theme.css`:

```text
color    --color-white        #fff
color    --color-ink-900      oklch(22% 0.02 260)
color    --color-ink-600      oklch(45% 0.02 260)
color    --color-blue-700     oklch(48% 0.17 255)
color    --color-blue-50      oklch(97% 0.02 255)
color    --color-surface      var(--color-white)
color    --color-fg           var(--color-ink-900)
color    --color-fg-muted     var(--color-ink-600)
color    --color-action       var(--color-blue-700)
radius   --radius-control     0.5rem
```

It's a simple regular expression, good enough for a token file you control (it doesn't
handle nested `@keyframes` inside `@theme`, for example). Turn its output into a docs
page with swatches, and you have living documentation.

## Guardrails

A system drifts when it's easier to break it than to use it. Some cheap checks:

- **The reset is a guardrail in itself.** Off-system colours generate no CSS.
- **Search for arbitrary colour values** in review or CI. Something like
  `grep -rnoE "(bg|text|border|ring)-\[#" src/` finds hard-coded hex colours that bypass
  tokens.
- **Search for primitive tokens in components.** Components should use `bg-action`, not
  `bg-blue-700`. A grep over component folders for primitive names makes drift visible.
- **Review contrast once, in the token layer.** If `--color-fg-muted` on
  `--color-surface` passes, every use passes (lesson 04).
- **Version the token package.** Renaming or removing a token is a breaking change for
  every app that uses it.

Level 4 · 01 builds these into linting and review conventions.

## How It Actually Works

The tiers are just CSS custom properties referencing each other. A utility like
`bg-action` compiles to `background-color: var(--color-action)`, and at runtime the
browser resolves `--color-action` → `var(--color-blue-700)` → the oklch value. Nothing
is resolved at build time (unless you use `@theme inline`), which is exactly what lets a
`.dark` or `[data-brand=…]` rule reassign a semantic token and have every utility follow.

Importing a token package works because `@import` in Tailwind's CSS is resolved by
Tailwind itself (not left to the browser): it inlines the file, looking up bare
specifiers like `@acme/tokens` in `node_modules` and honouring the package's `style`
export. Theme blocks from all imported files are merged into one theme in import order,
which is why the package's `--color-*: initial` reset removed the default palette even
though it lived in a different file.

## Common mistakes

- **Keeping the full default palette** "just in case". Every unused option is a future
  inconsistency.
- **Components using primitives** (`bg-blue-700`) instead of semantic tokens.
- **Too many semantic tokens**, so nobody knows which to use.
- **Token names that read badly** after Tailwind prefixes.
- **Hand-maintained token docs** that drift from the CSS.
- **Forgetting `@source`** for a shared component package in `node_modules`.
- **Changing a token's meaning** without versioning the package.

## Exercise

1. Write a `theme.css` with a reset palette of at most 12 primitive colours and about 8
   semantic tokens. Confirm a default class like `bg-sky-500` generates nothing.
2. Move it into a local package (`packages/tokens`) and import it from an app.
3. Build a `button` recipe that uses only semantic tokens. Change `--color-action` and
   check every button updates.
4. Run the token-listing script against your theme and turn the output into a simple
   HTML page of swatches.
5. Add a CI step (or a pre-commit hook) that fails when a component file contains an
   arbitrary hex colour.
