---
description: "Keep a large Tailwind codebase consistent: class sorting with Prettier, ESLint rules for conflicts and unknown classes, and review checks."
---

# 01 · Conventions at Scale: Class Order, Linting & Review

On a solo project, Tailwind conventions live in your head. On a team of twenty, with
thousands of components, they need to live in tools. Otherwise the same card is written
five different ways, class lists grow conflicting utilities, someone builds class names
with string concatenation, and reviewers spend their time on formatting instead of
design. This lesson sets up the two tools that do most of that work automatically and
then covers the conventions tools can't enforce.

## Automatic class ordering

When every class list is in the same order, diffs are smaller and you can scan a list for
"does this have a hover state?" in the same place every time. Don't argue about the
order; let a tool decide. The official Prettier plugin sorts classes in the same order
Tailwind outputs the CSS:

```bash
npm install -D prettier prettier-plugin-tailwindcss
```

```json title=".prettierrc.json"
{
  "plugins": ["prettier-plugin-tailwindcss"],
  "tailwindStylesheet": "./src/app.css",
  "tailwindFunctions": ["cn", "cva"]
}
```

- `tailwindStylesheet` points at your v4 CSS entry, so the plugin knows your custom
  theme, utilities and variants.
- `tailwindFunctions` lists helpers whose string arguments contain classes. Without it,
  only `class` and `className` attributes are sorted.

We ran it (Prettier 3.9.9, plugin 0.8.1) on this deliberately messy markup:

```html title="before"
<div class="hover:shadow-lg text-sm md:p-8 p-4 bg-white flex  rounded-xl   bg-action custom-card shadow dark:bg-gray-900 items-center">
  <p class="text-gray-600 mt-2 font-medium sm:text-base">Hi</p>
</div>
```

```html title="after: npx prettier --write src/card.html"
<div
  class="custom-card flex items-center rounded-xl bg-action bg-white p-4 text-sm shadow hover:shadow-lg md:p-8 dark:bg-gray-900"
>
  <p class="mt-2 font-medium text-gray-600 sm:text-base">Hi</p>
</div>
```

And in JavaScript, inside a `cn()` call:

```js title="after"
export const cls = cn(
  "rounded-md bg-action px-4 py-2 text-white hover:bg-action/90",
  props.class,
);
```

What the order tells you:

- **Non-Tailwind classes first** (`custom-card`), so they stand out.
- **Layout, then box model, then visual** (`flex items-center` → `rounded-xl bg-…` →
  `p-4` → `text-sm` → `shadow`), following the stylesheet order.
- **Variants last**, grouped (`hover:`, then breakpoints, then `dark:`).
- Extra whitespace removed.

Notice what it did **not** do: `bg-action` and `bg-white` are both still there. Sorting
isn't linting. It put the conflict side by side, where a reviewer can see it, but
didn't fix it. That's the next tool's job.

## Linting Tailwind classes

`eslint-plugin-better-tailwindcss` reads your v4 stylesheet and checks class strings in
JavaScript and TypeScript (and, with the right parsers, in HTML, Vue, Svelte and others):

```bash
npm install -D eslint eslint-plugin-better-tailwindcss
```

```js title="eslint.config.js"
import betterTailwindcss from "eslint-plugin-better-tailwindcss";

export default [
  {
    files: ["src/**/*.js"],
    plugins: { "better-tailwindcss": betterTailwindcss },
    settings: {
      "better-tailwindcss": { entryPoint: "src/app.css" },
    },
    rules: {
      ...betterTailwindcss.configs.correctness.rules,
      "better-tailwindcss/no-restricted-classes": ["error", {
        restrict: [{
          pattern: "^(.*:)?(bg|text|border)-\\[#",
          message: "Use a theme colour instead of a hex value.",
        }],
      }],
    },
  },
];
```

We linted this file (ESLint 10.11.0, plugin 4.7.0):

```js title="src/badge.js"
import { cn } from "./cn.js";
const tone = "red";
export const a = cn("px-2 py-1 p-3 text-sm text-hero bg-[#ff0000] bg-action");
export const b = cn("rounded-md rounded-lg font-semibold font-bold");
export const c = cn("bg-sky-650 shadow-xs flex-grow-0");
export const d = cn("text-" + tone + "-700");
```

The report, shortened:

```text
3:44  error  Unknown class detected: text-hero                           no-unknown-classes
3:54  error  Conflicting class detected: "bg-[#ff0000]" and "bg-action"
             apply the same CSS properties: "background-color"           no-conflicting-classes
3:54  error  Use a theme colour instead of a hex value                   no-restricted-classes
4:22  error  Conflicting class detected: "rounded-md" and "rounded-lg" …  no-conflicting-classes
4:44  error  Conflicting class detected: "font-semibold" and "font-bold" … no-conflicting-classes
5:22  error  Unknown class detected: bg-sky-650                          no-unknown-classes
6:22  error  Concatenated classes may be purged by Tailwind CSS.
             Avoid dynamic class construction                            no-concatenated-classes
✖ 11 problems (11 errors, 0 warnings)
```

(Each conflict is reported once per class, so there were 11 in total.) It caught:

- a **typo** (`bg-sky-650` doesn't exist) and a class from a token that isn't in this
  project's theme (`text-hero`),
- **conflicting utilities** in the same string,
- a **hard-coded hex colour**, via our custom restriction,
- **string concatenation**, the scanner problem from Level 3 · 01.

It's also worth knowing what it did **not** flag in our test: `px-2 py-1 p-3` (a
shorthand that overlaps the two longhands) passed, as did the v3-era `flex-grow-0`.
Linting reduces review work; it doesn't remove it.

Run Prettier and ESLint in the editor (format on save), in a pre-commit hook, and in CI,
so nothing unformatted or failing reaches review.

## Conventions tools can't enforce

Write these down in a short `CONTRIBUTING` section. Keep it to a page; nobody reads ten.

**When to extract.** "Repeated three times in different files, and stable" → component
or partial. Repeated within one file → leave it, or use a loop. Never extract to save
typing alone.

**Arbitrary values.** Allowed for genuinely one-off values (a third-party embed's exact
size, a `calc()`). If the same arbitrary value appears twice, it becomes a token. Hex
colours: never (enforced by the lint rule above).

**Tokens over palette.** Components use semantic tokens (`bg-surface`, `text-fg-muted`).
Palette colours (`bg-sky-700`) only in places a token doesn't make sense, such as
illustrations or status colours with their own token set.

**Overrides.** Components accept a `class`/`className` for layout tweaks (margin, width,
grid placement). Colour or typography changes go through variants.

**Long class lists.** If a single element needs more than about 15 utilities, look for
a component, or for utilities that don't do anything (a `flex` on an element with one
child, a `text-base` that just restates the default).

**Responsive order.** Write mobile first: unprefixed, then `sm:`, `md:`, `lg:`. Prettier
sorts within this anyway.

## Review checklist

What a reviewer should still look at after the tools pass:

- [ ] Semantic HTML: headings in order, buttons are `<button>`, links are `<a>`.
- [ ] Focus visible on every interactive element (outline, or `outline-hidden` + ring).
- [ ] Text contrast checked for new colour pairings, light and dark.
- [ ] No hover-only affordances; touch and keyboard paths exist.
- [ ] Responsive behaviour checked at a narrow width (no horizontal scroll).
- [ ] New arbitrary values justified, or turned into tokens.
- [ ] Overrides passed to components are layout-only.

## How It Actually Works

Both tools load your actual Tailwind setup rather than a hard-coded list. The Prettier
plugin asks Tailwind for the **sort order** of each class (the same order the compiler
would emit its CSS in) and rewrites the attribute in that order, putting classes Tailwind
doesn't recognise first. The ESLint plugin loads the design system from the
`entryPoint` stylesheet, so it knows your custom tokens and utilities, then for each
class string checks whether each class generates CSS (unknown classes), whether two
classes in the same string generate the same CSS properties under the same variants
(conflicts), and whether the string was built with `+` or template expressions
(concatenation). Neither tool runs your code: they work on literal strings they can find
in the source, just like the compiler's scanner.

## Common mistakes

- **Sorting classes by hand** or arguing about order in review. Automate it.
- **Forgetting `tailwindFunctions`**, so classes inside `cn()` and `cva()` aren't sorted
  or linted.
- **Pointing the tools at no stylesheet**, so custom tokens are reported as unknown or
  sorted as non-Tailwind classes.
- **Treating "lint passes" as "review done".** Overlapping shorthands, contrast and
  semantics still need a human.
- **A conventions document so long nobody reads it.**
- **Turning on every stylistic rule at once** in a large existing codebase. Start with
  correctness rules, fix the backlog, then add more.

## Exercise

1. Add the Prettier plugin to one of your projects and run it over every template. Look
   at the diff: what conflicts became visible?
2. Add the ESLint plugin with the correctness rules and a hex-colour restriction. Fix
   everything it reports.
3. Write a deliberately bad file like `badge.js` above and check which problems your
   setup catches and which it misses.
4. Write your team's one-page Tailwind conventions, including an extraction rule and an
   arbitrary-value policy.
