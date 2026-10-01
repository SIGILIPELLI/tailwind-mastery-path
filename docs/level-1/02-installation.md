---
description: "Install Tailwind CSS v4 with the standalone CLI or the Vite plugin, understand @import \"tailwindcss\", and learn exactly which files the compiler scans."
---

# 02 · Installing Tailwind v4: CLI, Vite & Source Detection

Tailwind needs something to run the compiler: read your source files, find class names,
write CSS. There are three common ways to run it — the **CLI**, a **build-tool plugin**
(Vite is the most common), or **PostCSS** — and all of them start from the same one-line
stylesheet. This lesson sets up the first two for real and then examines the part that
causes most "why isn't my class working?" questions: which files Tailwind actually scans.

You'll need **Node.js** (a current LTS release) and npm. Check with `node -v`.

## The stylesheet: one line

In v4 your main CSS file starts with:

```css title="src/input.css"
@import "tailwindcss";
```

That single import brings in the theme variables, the base styles (Preflight), and the
utilities layer. Configuration also goes in this file (Level 2 · 01) — v4 no longer needs
a `tailwind.config.js`.

## Option 1: the Tailwind CLI

Good for static sites, server-rendered apps (Django, Rails, Laravel, Go templates) and
learning.

```bash
mkdir tw-cli && cd tw-cli
npm init -y
npm install tailwindcss @tailwindcss/cli
```

Create `src/input.css` with the import above and an HTML page:

```html title="index.html"
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Tailwind CLI</title>
    <link rel="stylesheet" href="dist/output.css">
  </head>
  <body class="bg-slate-50 text-slate-900">
    <h1 class="p-8 text-4xl font-bold tracking-tight">It works</h1>
  </body>
</html>
```

Build once:

```bash
npx @tailwindcss/cli -i src/input.css -o dist/output.css
```

```text
≈ tailwindcss v4.3.3

Done in 33ms
```

Or rebuild on every change while you work:

```bash
npx @tailwindcss/cli -i src/input.css -o dist/output.css --watch
```

The options you'll actually use, from `tailwindcss --help` in 4.3.3: `-i/--input`,
`-o/--output`, `-w/--watch`, `-m/--minify` (for production), `--map` (source maps),
`--cwd` (which directory to treat as the project root) and `--poll` (for file systems
where change events don't work, such as some network drives and containers).

Add scripts so nobody has to remember the flags:

```json title="package.json (excerpt)"
{
  "scripts": {
    "dev": "tailwindcss -i src/input.css -o dist/output.css --watch",
    "build": "tailwindcss -i src/input.css -o dist/output.css --minify"
  }
}
```

There is also a **standalone executable** (a single binary with no Node.js required),
downloadable from the Tailwind CSS GitHub releases page — handy for projects in other
language ecosystems that don't otherwise use npm.

## Option 2: Vite

For JavaScript apps (React, Vue, Svelte, plain JS), the Vite plugin is the smoothest
setup — CSS updates instantly in the browser without a page reload.

```bash
npm create vite@latest my-app    # or start from an existing Vite project
cd my-app
npm install tailwindcss @tailwindcss/vite
```

```js title="vite.config.js"
import { defineConfig } from 'vite';
import tailwindcss from '@tailwindcss/vite';

export default defineConfig({
  plugins: [tailwindcss()],
});
```

```css title="src/style.css"
@import "tailwindcss";
```

Import that stylesheet from your HTML or your JavaScript entry point. We built this
minimal project — an HTML page plus a script that creates an element with classes at
runtime:

```js title="src/main.js"
const badge = document.createElement('span');
badge.className = 'ml-2 rounded-full bg-emerald-100 px-2 py-0.5 text-sm text-emerald-800';
badge.textContent = 'from JS';
document.querySelector('h1').append(badge);
```

`npx vite build` (Vite 8.3.1, `@tailwindcss/vite` 4.3.3) produced:

```text
dist/index.html                 0.47 kB │ gzip: 0.31 kB
dist/assets/index-DFnvEe5m.css  5.68 kB │ gzip: 1.91 kB
dist/assets/index-D5r8bB6c.js   0.85 kB │ gzip: 0.48 kB

✓ built in 196ms
```

The CSS contained `.bg-emerald-100` and `.ml-2` — classes that appear only inside a
JavaScript string. Tailwind found them because it scans source text, not the rendered DOM.

## Option 3: PostCSS

If your toolchain already uses PostCSS (some frameworks do), add `@tailwindcss/postcss`
to your PostCSS config instead of the Vite plugin. The CSS side is identical. Framework
integration guides on the Tailwind website list the recommended option for each framework;
follow those rather than mixing approaches.

## Editor setup

Install the official **Tailwind CSS IntelliSense** extension for VS Code (or the
equivalent for your editor). It autocompletes class names, shows the generated CSS on
hover, and warns about conflicting classes. On hover over `p-6` you'll see the same
`padding: calc(var(--spacing) * 6)` the compiler produces — the quickest way to learn
what a class does.

## Which files get scanned

Tailwind v4 detects your source files automatically — there's no `content` array to
configure as there was in v3. The rules matter, so we tested them. A project with this
layout, each file containing one distinct padding class:

```text
src/page.html            <div class="p-1">
src/app.js               const c = "p-2";
build/generated.html     <div class="p-3">      (build/ is listed in .gitignore)
node_modules/some-ui/    <div class="p-4">
vendor/lib/widget.html   <div class="p-5">
src/notes.md             p-6 as plain text in markdown
src/data.php             <div class="p-7">
```

Running the CLI from the project root generated:

```text
.p-1 .p-2 .p-5 .p-6 .p-7
```

So, by default:

- **Every text file in the project is a candidate** — HTML, JS, PHP, even Markdown
  prose. Tailwind doesn't care about file type; it scans text.
- **Files ignored by `.gitignore` are skipped** (`p-3` in `build/`).
- **`node_modules` is skipped** (`p-4`).
- **Other folders, like `vendor/`, are included** (`p-5`).
- A word in prose that happens to be a valid class (`p-6` in the notes) **is generated**.
  Harmless — a few extra bytes — but it explains the odd surprise class in your output.

### `@source`: telling the compiler more

```css title="src/input.css"
@import "tailwindcss";

@source "../node_modules/some-ui";   /* scan a package that ships Tailwind classes */
@source not "../vendor";             /* never scan this folder */
@source inline("p-8 hover:p-9");     /* always generate these, even if unused */
```

With those three lines, the same project generated:

```text
.p-1 .p-2 .p-4 .p-6 .p-7 .p-8 hover:p-9
```

`p-4` from the UI package is now included, `p-5` from `vendor/` is gone, and the inline
"safelist" classes are generated even though no file uses them. Paths in `@source` are
relative to the CSS file they're written in.

The most common real use of `@source` is a component library installed in
`node_modules` whose templates use Tailwind classes, and a monorepo where the CSS entry
file lives in a different package from the templates.

## How It Actually Works

The scanner starts from a **base directory** — the directory the CLI runs in (or `--cwd`),
or the project root for the Vite plugin — and walks it, applying ignore rules: `.gitignore`
entries, `node_modules`, lock files, binary files and files with extensions that can't
contain classes (images, fonts, archives). Explicit `@source` paths are added on top,
and `@source not` paths removed.

For each file, a fast tokenizer written in Rust extracts **candidates**: runs of characters
that could form a class name, including brackets, slashes, colons and dots
(`md:grid-cols-[200px_1fr]`). It doesn't know HTML from PHP; it just splits text on
characters that can't appear in a class. Candidates then go to the matching step described
in lesson 01. That's why a class inside a JavaScript string counts, why a class name built
by concatenation doesn't, and why an English word like "flex" in a paragraph produces a
`.flex` rule.

In watch mode and in the Vite plugin, changed files are re-scanned individually and only
new candidates trigger regeneration, which is why rebuilds stay in the millisecond range.

## Common mistakes

- **Following v3 instructions**: `npx tailwindcss init`, `tailwind.config.js` with a
  `content` array, `@tailwind base; @tailwind utilities;` directives. In v4 it's
  `@import "tailwindcss"` and automatic detection.
- **Linking the input CSS** (`src/input.css`) instead of the generated output in plain HTML
  projects.
- **Generated files not scanned** because they're gitignored — add an `@source` if a build
  step produces templates Tailwind must see.
- **Classes from a UI package missing** because `node_modules` isn't scanned by default.
- **Running the CLI from the wrong directory**, so the scan misses your templates (use
  `--cwd`).

## Exercise

1. Set up the CLI project above, with `dev` and `build` scripts. Build the card from lesson
   01 and compare the generated CSS with your predictions.
2. Set up the Vite project, add a class from JavaScript, and confirm it appears in the
   built CSS.
3. Reproduce the scanning experiment: create a gitignored folder with a class in it and
   confirm it's missing from the output; then bring it back with `@source`.
4. Install the IntelliSense extension and hover over five classes you haven't used before.
