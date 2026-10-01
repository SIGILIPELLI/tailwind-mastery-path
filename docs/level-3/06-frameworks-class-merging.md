---
description: "Tailwind components that accept overrides: why class order doesn't matter, clsx, tailwind-merge, cva variant APIs and custom-token gotchas."
---

# 06 · Tailwind in Component Frameworks: Class Merging & Variant APIs

Once your buttons, badges and inputs are components (Level 2 · 05), two needs show up
quickly. Components need **variants**: primary/secondary/danger, small/medium/large. And
callers want to **override** a style for one use: "this button, but with more padding".
The naive approach of concatenating class strings breaks in a way that surprises almost
everyone, so this lesson starts there, then builds a proper component API with three
small, widely used libraries: `clsx`, `tailwind-merge` and `class-variance-authority`.

The examples use React, but the ideas (and the libraries, which are framework-agnostic
JavaScript) apply equally to Vue, Svelte, Solid, Astro and server templates.

## Class order in the attribute does nothing

Here's a button component that accepts extra classes:

```jsx
function Button({ className = "", ...props }) {
  return <button className={`rounded-md bg-sky-700 p-2 text-white ${className}`} {...props} />;
}

<Button className="p-4 bg-red-700">Delete</Button>
```

The resulting attribute is `"rounded-md bg-sky-700 p-2 text-white p-4 bg-red-700"`. It
*looks* like the later classes should win. They don't necessarily. In CSS, the winner
between two equal-specificity rules is the one **later in the stylesheet**, and Tailwind
decides that order, not your attribute. We rendered pairs in both orders:

```text
class="p-4 p-2"              -> padding 16px
class="p-2 p-4"              -> padding 16px
class="bg-red-500 bg-blue-500" -> red
class="bg-blue-500 bg-red-500" -> red
class="px-2 p-6"             -> padding-left 8px
```

`p-4` beat `p-2` both ways, and red beat blue both ways, because of where those rules
happen to sit in the generated file. The last line is the nastiest: `px-2` beat `p-6` on
the left side because Tailwind emits `px-*` after `p-*`. The rule: **never put two
utilities that set the same property on one element** and expect the order to choose.

## `clsx`: conditional classes

Before merging, a smaller problem: building class strings from conditions without
messy template literals. `clsx`, a tiny utility, takes strings, objects and arrays and
drops anything falsy:

```js
import { clsx } from "clsx";
clsx("btn", { "opacity-50": true, hidden: false }, ["p-2", null]);
// "btn opacity-50 p-2"   (output from clsx 2.1.1)
```

## `tailwind-merge`: resolve conflicts

`tailwind-merge` knows which Tailwind classes conflict and keeps only the **last** one
of each conflicting group:

```js
import { twMerge } from "tailwind-merge";

twMerge("p-4 p-2");
// "p-2"
twMerge("px-2 py-1 bg-red-500 hover:bg-red-600", "p-3 bg-sky-700");
// "hover:bg-red-600 p-3 bg-sky-700"
twMerge("text-sm text-gray-600", "text-sky-700");
// "text-sm text-sky-700"
```

(Outputs from tailwind-merge 3.7.0.) Notice what it understands: `p-3` replaces both
`px-2` and `py-1`; `hover:bg-red-600` survives because it's a different variant;
`text-sky-700` replaces the colour but not the size, even though both start with
`text-`. Combine it with `clsx` in a small helper, conventionally called `cn`:

```js title="src/lib/cn.js"
import { clsx } from "clsx";
import { twMerge } from "tailwind-merge";

export function cn(...inputs) {
  return twMerge(clsx(inputs));
}
```

Now components can merge reliably:

```jsx
function Button({ className, ...props }) {
  return <button className={cn("rounded-md bg-sky-700 p-2 text-white", className)} {...props} />;
}
<Button className="p-4 bg-red-700">Delete</Button>   // p-4 and bg-red-700 win
```

### The custom-token gotcha

`tailwind-merge` works from a built-in model of Tailwind's **default** theme. It can't
see your `@theme` file. With custom tokens we got these results:

```js
twMerge("shadow-sm", "shadow-card");      // "shadow-sm shadow-card"   (conflict not resolved)
twMerge("rounded-md", "rounded-card");    // "rounded-md rounded-card" (conflict not resolved)
twMerge("text-sky-700", "text-hero");     // "text-hero"               (colour removed!)
twMerge("text-brand-600", "text-sky-700"); // "text-sky-700"           (fine)
```

The third line is a real bug: `text-hero` is a custom **font size**, but tailwind-merge
assumed an unknown `text-*` is a **colour**, so it dropped `text-sky-700`. Tell it about
your tokens with `extendTailwindMerge`:

```js title="src/lib/cn.js"
import { clsx } from "clsx";
import { extendTailwindMerge } from "tailwind-merge";

const twMerge = extendTailwindMerge({
  extend: {
    theme: {
      text: ["hero"],     // --text-hero
      shadow: ["card"],   // --shadow-card
      radius: ["card"],   // --radius-card
    },
  },
});

export function cn(...inputs) {
  return twMerge(clsx(inputs));
}
```

With that configuration, `text-sky-700 text-hero` kept both, `text-sm text-hero` became
`text-hero`, and the shadow and radius pairs resolved to `shadow-card` and
`rounded-card`. Keep this list in sync with your theme; it's the price of the library.

## `cva`: a variant API

`class-variance-authority` turns a component's variants into a typed function:

```js title="src/components/button.js"
import { cva } from "class-variance-authority";

export const button = cva(
  "inline-flex items-center justify-center rounded-md font-medium focus-visible:outline-2 " +
    "focus-visible:outline-offset-2 disabled:opacity-50",
  {
    variants: {
      intent: {
        primary: "bg-sky-700 text-white hover:bg-sky-800 focus-visible:outline-sky-700",
        secondary: "bg-white text-gray-900 ring-1 ring-gray-300 hover:bg-gray-50 focus-visible:outline-gray-900",
        danger: "bg-red-700 text-white hover:bg-red-800 focus-visible:outline-red-700",
      },
      size: {
        sm: "h-8 px-3 text-sm",
        md: "h-10 px-4 text-sm",
        lg: "h-12 px-6 text-base",
      },
    },
    compoundVariants: [{ intent: "danger", size: "lg", class: "uppercase tracking-wide" }],
    defaultVariants: { intent: "primary", size: "md" },
  }
);
```

Calling it (cva 0.7.1):

```js
button();
// "inline-flex … disabled:opacity-50 bg-sky-700 text-white hover:bg-sky-800
//  focus-visible:outline-sky-700 h-10 px-4 text-sm"
button({ intent: "danger", size: "lg" });
// "… bg-red-700 text-white hover:bg-red-800 focus-visible:outline-red-700
//  h-12 px-6 text-base uppercase tracking-wide"
```

`compoundVariants` adds classes only for a specific combination. Every class name is
written out in full in the source file, so the Tailwind scanner sees all of them
(lesson 01). That's an important property of this pattern compared with building names
from props.

Wrap it in a component, merging caller overrides last:

```jsx title="src/components/Button.jsx"
import { button } from "./button";
import { cn } from "../lib/cn";

export function Button({ intent, size, className, ...props }) {
  return <button className={cn(button({ intent, size }), className)} {...props} />;
}

<Button size="sm" className="px-6 bg-emerald-700">Save draft</Button>
```

For that last call, the merged result was:

```text
inline-flex items-center justify-center rounded-md font-medium focus-visible:outline-2
focus-visible:outline-offset-2 disabled:opacity-50 text-white hover:bg-sky-800
focus-visible:outline-sky-700 h-8 text-sm px-6 bg-emerald-700
```

`px-6` replaced `px-3` and `bg-emerald-700` replaced `bg-sky-700`. But look:
`hover:bg-sky-800` and `focus-visible:outline-sky-700` are still there, because the
caller didn't override them. The button is now emerald that turns *sky blue* on hover.
That's correct merging behaviour and a design problem: **overrides should be small
tweaks** (spacing, width, margin), not recolouring. If callers need a different colour,
add an `intent` variant.

## The same idea in other frameworks

The libraries are plain JavaScript, so the pattern carries over:

```vue title="Button.vue"
<script setup>
import { computed } from "vue";
import { button } from "./button";
import { cn } from "../lib/cn";
const props = defineProps({ intent: String, size: String, class: String });
const classes = computed(() => cn(button({ intent: props.intent, size: props.size }), props.class));
</script>

<template>
  <button :class="classes"><slot /></button>
</template>
```

In server templates (Django, Rails, Laravel, Go), the same rule applies even without
libraries: keep each variant's full class list in one place (a partial, a helper, a
dictionary) and never build class names by string interpolation.

## How It Actually Works

`clsx` is pure string assembly; it knows nothing about Tailwind. `tailwind-merge` parses
each class into a **variant prefix** (`hover:`, `md:`), an **important** marker, and a
**base class**, then looks up the base class in its own table of "class groups" (all
padding classes, all font-size classes, all text-colour classes, …) along with a list of
which groups conflict (`p` conflicts with `px` and `py`, but `px` doesn't conflict with
`py`). Walking the list from the end, it keeps a class unless a later class with the same
variants and important flag already claimed its group. Because that table is built from
Tailwind's default theme, unknown values are guessed by shape, which is how `text-hero`
ended up in the colour group.

`cva` is a lookup: it concatenates the base string, the string for each chosen variant,
and any matching compound variants. None of these libraries generate CSS. They only
choose which class names end up in the attribute; Tailwind's compiler has already
generated CSS for every name, because every name appears in full in your source.

## Common mistakes

- **Relying on class order** to resolve conflicts (`p-2 p-4`). The stylesheet decides.
- **Concatenating caller classes without merging**, so overrides randomly fail.
- **Forgetting to configure tailwind-merge for custom tokens**, which can silently drop
  classes.
- **Using `className` overrides to recolour components**, leaving the old hover and
  focus colours behind. Add a variant instead.
- **Building variant classes from props** (`` `bg-${intent}-700` ``), which the scanner
  can't see.
- **Running twMerge on every render of thousands of elements** without need. It's fast
  and caches results, but plain static class strings are faster still for components
  that take no overrides.

## Exercise

1. Render `class="px-2 p-6"` and `class="p-6 px-2"` and confirm both have 8px left
   padding. Explain why using the compiled CSS.
2. Build the `cn` helper and a `Button` with `cva` (intent × size, with a compound
   variant). Use it in three places with small overrides.
3. Add a custom `--text-hero` token to your theme, reproduce the
   `twMerge("text-sky-700", "text-hero")` bug, then fix it with `extendTailwindMerge`.
4. Build a `Badge` component with `tone` (neutral/success/warning/danger) and `size`
   variants. Check every tone's contrast (lesson 04).
