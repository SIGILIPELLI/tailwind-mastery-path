---
description: "Debug Tailwind systematically: is the class generated, does it win, does its variable resolve? With a class-checking script and real bugs."
---

# 08 · Debugging Tailwind

Every Tailwind bug looks the same from the outside: "I added a class and nothing
happened." The causes are very different, and guessing wastes hours. This lesson gives
you a fixed routine of three questions, in order, with the tool that answers each one,
then a catalogue of the actual bugs met while writing this course, each with its
symptom, diagnosis and fix.

## The three questions

1. **Was CSS generated for the class?** If not, it's a scanning, naming or
   configuration problem. Look at the compiled CSS.
2. **Does the rule apply to the element, and does it win?** If it's generated but not
   winning, it's a cascade problem. Look at DevTools' Styles pane.
3. **Does the value resolve?** If it wins but looks wrong, a variable or a function
   inside the value didn't resolve as you expected. Look at the Computed pane.

Answer them strictly in order. Most wasted debugging time comes from investigating
question 2 when the answer to question 1 was "no".

## Question 1: was it generated?

**Search the output.** Open the generated CSS (the CLI output file, or DevTools → Sources
for the stylesheet) and search for the class. Remember it's escaped in the CSS:
`md:hover:p-4` appears as `.md\:hover\:p-4`.

**Ask the compiler directly.** This script compiles individual classes against your real
stylesheet, including your theme and plugins, and prints what Tailwind generates for
each:

```js title="scripts/check-class.mjs"
// Usage: node check-class.mjs src/app.css bg-sky-650 bg-brand-600 md:hover:p-4
// Compiles each class on its own against your real stylesheet and prints the CSS
// Tailwind generates for it, or UNKNOWN if it generates nothing.
// @tailwindcss/node is Tailwind's internal Node API: fine for a debugging script,
// but it isn't a documented public API and may change between releases.
import { compile } from "@tailwindcss/node";
import { readFileSync } from "node:fs";
import path from "node:path";

const [cssFile, ...classes] = process.argv.slice(2);
const css = readFileSync(cssFile, "utf8").replace(/@import\s+["']tailwindcss["'][^;]*;/, '@import "tailwindcss" source(none);');
const base = path.dirname(path.resolve(cssFile));

for (const cls of classes) {
  const compiler = await compile(css, { base, onDependency() {} });
  const out = compiler.build([cls]);
  const start = out.indexOf("@layer utilities {");
  const utilities = start === -1 ? "" : out.slice(start + 18, out.indexOf("\n}\n", start)).trim();
  console.log(utilities ? `OK       ${cls}\n${utilities}\n` : `UNKNOWN  ${cls}\n`);
}
```

```bash
node scripts/check-class.mjs src/app.css bg-sky-650 bg-brand-600 md:hover:p-4 "bg-[--x]" "text-(--s)"
```

Output against a stylesheet with a custom `--color-brand-600`:

```text
UNKNOWN  bg-sky-650

OK       bg-brand-600
.bg-brand-600 {
    background-color: var(--color-brand-600);
  }

OK       md:hover:p-4
@media (width >= 48rem) {
    @media (hover: hover) {
      .md\:hover\:p-4:hover {
        padding: calc(var(--spacing) * 4);
      }
    }
  }

OK       bg-[--x]
.bg-\[--x\] {
    background-color: --x;
  }

OK       text-(--s)
.text-\(--s\) {
    color: var(--s);
  }
```

Each line answers a different question:

- `bg-sky-650` is a typo (there's no 650 shade), so it generates nothing.
- `bg-brand-600` proves the custom theme was loaded.
- `md:hover:p-4` shows the exact nesting of variants.
- `bg-[--x]` is generated but **invalid** (`background-color: --x`): the v3 variable
  syntax (Level 3 · 08).
- `text-(--s)` generated `color`, not `font-size`: a missing type hint (Level 2 · 04).

`@tailwindcss/node` comes with the CLI and the Vite plugin. If your package manager
doesn't make it importable, install it as a dev dependency. Your editor's Tailwind
extension gives the same information on hover.

**If the class is valid but missing from the output,** the scanner didn't see it:

- Is the full class name written literally in a scanned file? (No template strings,
  no `{{ }}` interpolation.)
- Is the file git-ignored, in `node_modules`, or outside the `@source` paths?
- Is it in a file type you excluded with `@source not`?

## Question 2: does it apply and win?

Select the element in DevTools and look at the **Styles** pane:

- **Rule not listed at all:** the selector doesn't match this element. Check the variant:
  `group-hover:` needs a `group` ancestor; `peer-*` needs an *earlier* sibling;
  `dark:` needs your custom variant's class; `@md:` needs an `@container` *ancestor*.
- **Rule listed but crossed out:** something else won. DevTools shows the winner above
  it. Check whether the winner is in a **later layer**, **unlayered** (legacy CSS,
  third-party CSS), or uses `!important`.
- **Two utilities for the same property on one element:** the one later in the generated
  CSS wins, regardless of class order (Level 3 · 06).

## Question 3: does the value resolve?

Look at the **Computed** pane for the property. A CSS variable that's undefined makes
the declaration "invalid at computed-value time", and the property falls back to
**inherit** (for inherited properties) or its **initial value**. We tested both with no
variables defined, inside a parent with `bg-amber-100 text-sky-700`:

```text
bg-(--brand)    background-color: rgba(0, 0, 0, 0)   <- initial value (transparent)
text-(--ink)    color: oklch(0.5 0.134 242.7)        <- inherited sky-700 from parent
```

Neither looks like an error; the element just looks unstyled or takes its parent's
colour. In DevTools, hover the `var(--brand)` in the Styles pane to see its value, or
type `getComputedStyle($0).getPropertyValue('--brand')` in the console.

## A catalogue of real bugs from this course

Each of these happened while building and checking this course's examples:

| Symptom | Question | Cause | Fix | Lesson |
|---|---|---|---|---|
| Classes built as `` `bg-${color}-500` `` missing | 1 | scanner sees fragments | full class names or a variable | L3·01 |
| `border-{{ accent }}-500` missing, though in rendered HTML | 1 | template interpolation | mapping in code | L4·07 |
| `bg-[--brand]` does nothing | 3 | v3 syntax compiles to `background-color: --brand` | `bg-(--brand)` | L2·04 |
| `text-(--size)` changes colour, not size | 1 | no type hint | `text-(length:--size)` | L2·04 |
| Error message shown for a *valid* field | 2 | unnamed `peer` matches any earlier invalid sibling | `peer/name` | L1·08 |
| `class="note p-2"` ignores `p-2` | 2 | unlayered CSS beats layers | `@layer components` | L2·05 |
| Third-party widget ignores utilities | 2 | its CSS is unlayered | `@import … layer(components)` | L3·02 |
| `hidden` element won't show with `flex` | 2 | Preflight's important `[hidden]` rule | remove the attribute | L3·02 |
| Brand override in a section doesn't apply | 3 | theme token resolved on `:root` | `@theme inline` | L4·02 |
| `scale-*` doesn't animate | 2 | own CSS transitions `transform`, v4 uses `scale` | `transition-transform` | L2·09 |
| `@md:` styles never apply | 2 | `@container` on the same element | container on a wrapper | L2·07 |
| `@container` element has zero width | 2 | size containment on a shrink-to-fit box | give it a width | L2·07 |
| No focus ring in high-contrast mode | 2 | rings are box-shadows, removed in forced colours | `outline-hidden` | L3·04 |
| `tailwind-merge` drops a colour | – | unknown custom token guessed as colour | `extendTailwindMerge` | L3·06 |
| Audit reports 1:1 contrast after theme switch | – | measured mid-transition | wait, or disable transitions while switching | L3·10 |
| Opacity lost after v3→v4 upgrade | 1 | `bg-opacity-*` removed, not migrated | `bg-x/90` | L3·08 |

Notice how many are question 2 or 3 problems that *look* like question 1 problems, and
vice versa. The routine is what tells them apart.

## Debugging tools worth knowing

- **DevTools Styles and Computed panes**, for questions 2 and 3.
- **DevTools Rendering pane**: emulate `prefers-color-scheme`, `forced-colors`,
  `prefers-reduced-motion`, `prefers-contrast` and print media.
- **The Elements panel's container badges**, showing which elements are query
  containers.
- **The editor extension**: hover a class to see its CSS, and it flags unknown classes.
- **ESLint with the Tailwind plugin** (Level 4 · 01): catches typos, conflicts and
  concatenation before you run anything.
- **`getComputedStyle($0)`** in the console, for exact values of properties and
  variables.

## How It Actually Works

The three questions map onto the three stages a class goes through. **Generation**
happens at build time: the scanner finds the string and the compiler turns it into a
rule (or doesn't). **Matching and cascade** happen in the browser: the selector must
match the element, and among matching declarations for a property the cascade picks one
by origin, importance, layer, specificity and order. **Value computation** happens last:
`var()` references are substituted, functions like `calc()` and `color-mix()` are
evaluated, and an invalid result turns into the inherited or initial value. A bug lives
in exactly one of those stages, and each stage has its own tool for inspecting it.

## Common mistakes

- **Starting at the cascade** before checking the class was generated.
- **Searching the CSS for the unescaped class name** (`md:p-4` instead of `md\:p-4`).
- **Adding `!important`** to fix a problem you haven't diagnosed.
- **Restarting the dev server and hoping**, when the real issue is a template string.
- **Ignoring the Computed pane** when a value "looks unstyled".
- **Debugging in one mode only**: check light and dark, desktop and touch, normal and
  forced colours.

## Exercise

1. Add `check-class.mjs` to a project and run it on five classes from your templates,
   including one custom token and one arbitrary value.
2. Pick three rows from the bug table, reproduce each in a scratch file, and walk
   through the three questions to diagnose it before reading the fix.
3. Put a `bg-(--undefined)` and a `text-(--undefined)` on two elements and explain the
   computed values.
4. Write your own one-page debugging checklist for your team, based on the three
   questions.
