---
description: "Inside Tailwind v4: automatic source detection, candidate extraction, why dynamic class names fail, @source and inline() safelisting."
---

# 01 · How the Engine Works: Scanning & Generation

Most confusing Tailwind bugs come down to one question: **did the compiler see the class
name?** A class built with string concatenation, a component library inside
`node_modules`, a template in a git-ignored folder: in each case the HTML has the class
at runtime, but the CSS doesn't. Level 3 starts by opening up the engine, so that you can
predict exactly what will be generated and fix it when it isn't.

## The pipeline

When you run the CLI, the Vite plugin or the PostCSS plugin, the same steps happen:

1. **Parse your CSS.** `@import "tailwindcss"`, your `@theme` blocks, `@utility`,
   `@custom-variant`, `@plugin` and `@source` directives are read. Together they define
   the **design system**: which utilities and variants exist and which values they
   accept.
2. **Scan source files.** A fast native scanner walks your project and extracts every
   string that *could* be a class name. These are called **candidates**.
3. **Parse each candidate** into parts: variants (`md:`, `hover:`), a root (`bg`), a
   value (`sky-600`, `[#1da1f2]`), a modifier (`/50`) and an important flag (`!`).
4. **Generate CSS** for each candidate the design system recognises. Anything it doesn't
   recognise is dropped silently.
5. **Sort and print.** Rules are ordered (by variant, then by property) and written into
   the `utilities` layer. Theme variables that are referenced get written to the `theme`
   layer.
6. **Optimise** (when minifying): the output goes through Lightning CSS, which minifies
   it and lowers modern syntax (nesting, range media queries) for the configured targets.

You can see the pieces in `node_modules`: `@tailwindcss/oxide` (with a platform-specific
package such as `@tailwindcss/oxide-darwin-arm64`) is the native scanner, and
`lightningcss` is the optimiser.

## What gets scanned automatically

With a plain `@import "tailwindcss";`, Tailwind scans from the current working
directory and skips:

- files and folders listed in your **`.gitignore`**,
- **`node_modules`**,
- binary files (images, videos, archives), CSS files, and common lock files.

We set up a small project to confirm this: one JSX file in `src/`, a file in
`node_modules/ui-kit/` using `ring-4`, and a file in a git-ignored folder using
`skew-y-6`. Neither `ring-4` nor `skew-y-6` was generated.

## Candidates are found by shape, not by meaning

The scanner doesn't understand JavaScript, Python or templates. It reads text and pulls
out anything shaped like a class: runs of letters, digits, dashes, colons, slashes,
brackets and so on. That has two consequences.

**Anything that looks like a class counts, wherever it appears.** Our JSX file had a
comment, `// A comment that mentions underline and grid-cols-7`, and both `underline` and
`grid-cols-7` were generated. A string variable `const s = "mt-[13px]"` produced
`mt-[13px]`. This is harmless (a few unused rules) and it's why you can keep class names
in plain JavaScript objects.

**Class names that only exist at runtime can't be found.** Here's the component we
scanned:

```jsx title="src/App.jsx"
const color = "red";
export function Alert({ tone }) {
  const tones = { info: "bg-sky-100 text-sky-900", danger: "bg-red-100 text-red-900" };
  return <div className={`p-4 bg-${color}-500 text-${tone}-700 ${tones[tone]}`}>Hi</div>;
}
```

The utilities generated from this file were:

```text
.bg-red-100  .bg-sky-100  .p-4  .text-red-900  .text-sky-900
```

`bg-${color}-500` and `text-${tone}-700` were **not** generated. The scanner saw the
fragments `bg-`, `-500`, `text-` and `-700`, none of which is a class. The `tones` map
worked perfectly, because every full class name is written out in the source.

That's the rule: **always write complete class names in your source.** Map props to
full strings instead of building them:

```jsx
// Broken: the compiler never sees "bg-red-500"
<button className={`bg-${color}-500`} />

// Works: every class appears in full
const bg = { red: "bg-red-500", amber: "bg-amber-500", sky: "bg-sky-500" };
<button className={bg[color]} />
```

When a value is truly dynamic (a user's chosen colour, a percentage), use a CSS variable
and a class that reads it: `style={{ "--accent": user.color }}` with `bg-(--accent)`
(Level 2 · 04).

## `@source`: scanning more

To include files that aren't detected automatically, such as a UI library in
`node_modules` that ships Tailwind classes, register them with `@source`. Paths are
relative to the CSS file:

```css title="src/app.css"
@import "tailwindcss";
@source "../node_modules/ui-kit";
```

With that line, `ring-4` from the library was generated. You can also point at a sibling
package in a monorepo (`@source "../../packages/ui/src";`).

## `@source not`: scanning less

Exclude paths that contain class-like strings you don't want, such as a large legacy
folder or generated files:

```css
@source not "./legacy";
@source not "./App.jsx";
```

In our test, `@source not "./App.jsx"` removed every utility that only that file used.

## `@source inline()`: safelisting

Some class names never appear in any file: they come from a CMS, a database, or user
content. List them explicitly with `@source inline()`, which supports brace expansion:

```css
@source inline("{hover:,}bg-{red,amber}-{100,500}");
```

That one line generated eight rules: `bg-red-100`, `bg-red-500`, `bg-amber-100`,
`bg-amber-500`, and the `hover:` version of each. The empty alternative in `{hover:,}`
means "with or without the prefix". There's also `@source not inline("…")` to stop
specific candidates from being generated even if they're found, which is useful when a
false positive in a comment or string produces an unwanted rule.

Safelist sparingly. Every pattern adds rules to every page, used or not, and wide
expansions (all colours × all shades × several variants) add up quickly.

## Taking full control: `source(none)` and `source(…)`

You can change where automatic detection starts, or turn it off:

```css
@import "tailwindcss" source("../src");   /* scan only ../src */
@import "tailwindcss" source(none);       /* scan nothing automatically */
@source "../templates";                   /* …then list exactly what to scan */
```

`source(none)` is useful in monorepos and in projects with several CSS entry points
(say, an admin app and a public site) where each stylesheet should only contain its own
classes.

## Debugging "my class doesn't work"

Work through these in order:

1. **Is the class in the compiled CSS?** Search the output file (or DevTools → Sources)
   for the escaped class name. If it's missing, it's a scanning or naming problem.
2. **If missing:** is the full class name written in a scanned file? Is the file
   git-ignored, in `node_modules`, or excluded by `@source not`? Is the name valid (a
   typo like `bg-sky-650` generates nothing)?
3. **If present but not applied:** it's a cascade problem. Check DevTools for a crossed-out
   declaration and see what beat it (lesson 02).

Level 4 · 08 turns this into a fuller debugging routine.

## How It Actually Works

The scanner is written in Rust and runs as a native Node.js add-on, which is why it's
fast enough to scan a large project on every rebuild. It respects `.gitignore` by
walking the file tree with the same ignore rules git uses. During extraction it doesn't
tokenise any particular language; it uses a set of heuristics for where class-like
strings start and end (quotes, whitespace, `=`, backticks, and so on) so that it works
across HTML, JSX, Vue, Svelte, PHP, Python templates, Markdown and more.

Extracted candidates go into a set. The compiler then tries to parse each one against
the design system. Parsing is cheap and failure is normal: most candidates (`const`,
`return`, `className`) are words that simply don't match any utility. In watch mode, the
scanner tracks which files changed, re-extracts only those, and the compiler generates
CSS only for **new** candidates, which is why incremental rebuilds take milliseconds.

The result depends only on the set of candidates and your CSS. It doesn't matter how
many times a class appears, in which file, or in what order.

## Common mistakes

- **Building class names with template strings or concatenation.** Write full class
  names, or use a CSS variable for truly dynamic values.
- **Expecting classes from a `node_modules` library** to be generated without `@source`.
- **Templates in a git-ignored folder** (generated or build output) that are never
  scanned.
- **Safelisting huge patterns** to "fix" dynamic class names, bloating every page.
- **Assuming a missing class is a cascade problem.** Check the output file first.
- **Getting `@source` paths wrong.** They're relative to the CSS file, not the project
  root.

## Exercise

1. Create a component that builds `` `text-${size}` `` from a prop. Confirm the classes
   are missing from the output, then fix it with a lookup object.
2. Put a file with a unique class in a folder, add the folder to `.gitignore`, and confirm
   the class disappears from the output. Add an `@source` line for it and see it return.
3. Safelist `bg-{red,green,blue}-{100,600}` with and without `hover:`, and count the
   rules generated.
4. Set up `source(none)` with two CSS entry points, each with its own `@source`, and
   confirm each output only contains its own classes.
