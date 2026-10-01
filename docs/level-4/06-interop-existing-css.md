---
description: "Add Tailwind to an existing CSS codebase: layer order for legacy CSS, the prefix and important options, third-party widgets, Shadow DOM."
---

# 06 · Working with Existing CSS & Third-Party Components

Most Tailwind adoption doesn't start from an empty folder. It starts with a five-year-old
site that has Bootstrap, a hand-written `main.css` and a date picker from npm, and a team
that wants to use utilities for new work without breaking old pages. This lesson covers
how the existing CSS and Tailwind interact in the cascade (tested with three
configurations), the `prefix` and `important` options, and how to style third-party
widgets, including ones inside Shadow DOM.

## The core problem: who wins?

Level 3 · 02 showed the rule: **unlayered CSS beats every layer**, and among layers the
later one wins. A legacy stylesheet is almost always unlayered, so once you import it
next to Tailwind, a legacy `.card { padding: 20px }` beats `p-2` every time. The fix is to
put the legacy CSS into a layer, and the interesting decision is *where* in the order.

We tested a legacy file with an element rule and a class rule:

```css title="legacy.css"
.card { padding: 20px; border: 1px solid #ccc; }
h1 { font-size: 2.5rem; margin-bottom: 1rem; }
```

against `<h1>Title</h1>` and `<div class="card p-2">`, in three setups:

```css
/* A: legacy layer first (lowest priority) */
@layer legacy;
@import "tailwindcss";
@import "./legacy.css" layer(legacy);

/* B: legacy between base and components */
@layer theme, base, legacy, components, utilities;
@import "tailwindcss";
@import "./legacy.css" layer(legacy);

/* C: no Preflight, legacy between theme and components */
@layer theme, legacy, components, utilities;
@import "tailwindcss/theme.css" layer(theme);
@import "tailwindcss/utilities.css" layer(utilities);
@import "./legacy.css" layer(legacy);
```

Computed in Chromium:

```text
                         h1 font-size   h1 margin-bottom   .card.p-2 padding
A (legacy first)         16px           0px                8px
B (legacy after base)    40px           16px               8px
C (no Preflight)         40px           16px               8px
```

- In **all three**, `p-2` won over `.card`. Putting legacy CSS in any layer below
  `utilities` makes utilities reliable.
- In **A**, Preflight (in `base`) came *after* the legacy layer and reset the legacy
  heading styles: the `<h1>` lost its size and margin. Old pages would visibly change.
- In **B**, the legacy layer sits after `base`, so legacy element styles survive while
  utilities still win. This is usually the right order for a gradual migration.
- **C** drops Preflight entirely. It's the most conservative: old pages keep the
  browser's defaults plus the legacy CSS, exactly as before.

Declaring the full order with `@layer theme, base, legacy, components, utilities;`
before the import is what controls it; the first declaration of a layer name fixes its
position.

One caution: if your legacy CSS uses `!important` (Bootstrap's utility classes do),
those declarations *reverse* the layer order and beat your utilities' normal
declarations. Search the legacy CSS for `!important` before deciding.

## `prefix()`: avoiding class name collisions

If the legacy CSS already has classes named `flex`, `hidden` or `container` with
different meanings, Tailwind's classes would collide with them. A prefix keeps them
apart:

```css
@import "tailwindcss" prefix(tw);
```

```html
<div class="tw:flex tw:md:p-4 tw:hover:bg-sky-700">…</div>
```

Compiled output (with the same HTML also containing unprefixed `flex p-4`):

```css
.tw\:flex { display: flex; }
@media (hover: hover) {
  .tw\:hover\:bg-sky-700:hover { background-color: var(--tw-color-sky-700); }
}
@media (width >= 48rem) {
  .tw\:md\:p-4 { padding: calc(var(--tw-spacing) * 4); }
}
```

Things to notice:

- In v4 the prefix looks like a variant and always comes **first**: `tw:md:p-4`, not
  `md:tw-p-4` as in v3.
- The unprefixed `flex` and `p-4` generated nothing, so they can't clash.
- Theme variables are prefixed too (`--tw-color-sky-700`, `--tw-spacing`), so they won't
  collide with variables in the legacy CSS.

A prefix costs readability on every element. Use it when collisions are real, not by
default.

## `important`: the blunt instrument

```css
@import "tailwindcss" important;
```

This marks every utility declaration `!important` (we compiled `flex`, `p-4` and
`bg-sky-700` and each declaration ended in `!important`). It makes utilities win against
almost anything, including highly specific legacy selectors like
`#app .sidebar ul li a`. It also makes it impossible to override a utility from your own
CSS without another `!important`, and it interacts with layers in the reversed way
described above. Prefer layering. Reach for `important` only when legacy CSS uses very
specific selectors or IDs and you can't layer it.

## Styling third-party widgets

Third-party components ship their own markup and class names, so you can't put utilities
on their inner elements. Options, from best to worst:

1. **The component's own API.** Many libraries accept `className` props for parts, or
   expose CSS variables (`--rdp-accent-color` and similar). Feed those from your tokens:
   `[--rdp-accent-color:var(--color-action)]` on a wrapper.
2. **Arbitrary variants on a wrapper**, targeting the library's class names:

    ```html
    <div class="[&_.vendor-day]:rounded-md [&_.vendor-day[aria-selected=true]]:bg-action
                [&_.vendor-day[aria-selected=true]]:text-on-action">
      <!-- third-party calendar renders here -->
    </div>
    ```

    This depends on the library's internal class names, which can change in any release.
    Keep these overrides in one wrapper component so there's one place to fix.

3. **A layered override file** for large amounts of restyling, imported with
   `layer(components)` so utilities can still adjust it.

## Shadow DOM: utilities don't reach inside

Web components with Shadow DOM encapsulate their styles. Your page stylesheet doesn't
apply inside the shadow root at all. We defined a custom element whose shadow root
contained `<h2 part="title" class="text-red-700">` and `<p class="text-red-700">`:

```text
host element p-4            padding 16px            <- the host is in the page: styled
shadow <p class="text-red-700">   rgb(0, 0, 0)      <- class ignored: page CSS can't see it
shadow <h2 part="title"> + host [&::part(title)]:text-sky-700   sky-700   <- ::part works
```

The class inside the shadow root did nothing, because the page's `.text-red-700` rule
doesn't apply across the shadow boundary. What does work:

- **Utilities on the host element** (it's in the light DOM).
- **`::part()`** for elements the component exposes with a `part` attribute. In Tailwind:
  `[&::part(title)]:text-sky-700` on the host.
- **Inherited properties** (`color`, `font-*`) and **CSS custom properties**, which pass
  into shadow roots. Many design-system web components are themed entirely through
  variables.

If you're the author of the web component and want Tailwind inside it, the component
must include Tailwind's CSS in its own shadow root (for example via a constructed
stylesheet), which means shipping a copy of the generated CSS with the component.

## A migration plan

1. Import Tailwind with the legacy CSS in a layer between `base` and `components` (or
   without Preflight if old pages must not change at all).
2. Use utilities for new components only. Leave old pages alone.
3. When you touch an old component, convert it and delete its legacy rules.
4. Track remaining legacy CSS size as a metric; when it's small, remove the layer.
5. Only then consider enabling Preflight globally, and check old pages visually.

## How It Actually Works

`@layer` names are ordered by their **first appearance** in the stylesheet. Tailwind's
`index.css` declares `theme, base, components, utilities`, but if your file declares a
longer list first, that list wins and Tailwind's later declaration just reuses the
existing names. `@import "…" layer(name)` wraps the whole imported file in that layer.

`prefix(tw)` changes how the compiler parses candidates: every candidate must start with
`tw:`, and theme variables are emitted with the prefix. `important` adds `!important`
to every generated declaration. Shadow DOM is a browser boundary: each shadow root has
its own style scope, and only inherited properties, custom properties and `::part`/
`::slotted` selectors cross it.

## Common mistakes

- **Importing legacy CSS unlayered**, so it silently beats utilities.
- **Putting the legacy layer before `base`**, so Preflight resets old pages.
- **Forgetting legacy `!important` rules**, which beat normal utility declarations.
- **Using `prefix()` without a real collision**, paying readability for nothing.
- **Turning on `important` as a first resort.**
- **Styling a vendor's internal class names all over the codebase** instead of in one
  wrapper.
- **Expecting utility classes to work inside Shadow DOM.**

## Exercise

1. Take a small page styled with an old stylesheet, import Tailwind with the legacy CSS in
   each of the three layer orders above, and compare headings and spacing.
2. Find a class-name collision between your legacy CSS and Tailwind and resolve it with
   `prefix(tw)`. Then decide whether the collision was worth a prefix.
3. Wrap a third-party component and restyle it using its CSS variables first, then
   arbitrary variants only where needed.
4. Build a tiny web component with a `part="title"` element and style it from the page
   with `[&::part(title)]:…`.
