---
description: "Tailwind's cascade layers and Preflight: what the reset changes, why utilities beat components, and why unlayered CSS beats them all."
---

# 02 · Cascade Layers, Preflight & Specificity

Lesson 01 was about whether a class is *generated*. This lesson is about whether a
generated class *wins*. Tailwind v4 is built on native CSS **cascade layers**, and once you
understand the four layers it creates, most "my class is there but doesn't apply" problems
become easy to explain. You'll also look closely at **Preflight**, the base reset that
quietly changes how headings, lists, images and buttons behave.

(For the cascade itself, see
[Cascade & Specificity](https://sigilipelli.github.io/html-css-mastery-path/level-1/08-cascade-specificity/)
and [Modern Selectors & Layers](https://sigilipelli.github.io/html-css-mastery-path/level-3/07-modern-selectors-layers/)
on the HTML & CSS course.)

## The four layers

The very first line of `tailwindcss/index.css` is:

```css
@layer theme, base, components, utilities;
```

That declares four layers in order. In the cascade, **a later layer beats an earlier one
regardless of selector specificity**. Specificity only matters *within* a layer. So:

| Layer | Contains | Beaten by |
|---|---|---|
| `theme` | `:root` variables from `@theme` | everything |
| `base` | Preflight, your `@layer base` rules, base styles from plugins | components, utilities |
| `components` | your `@layer components` rules | utilities |
| `utilities` | every generated utility, `@utility` definitions | nothing in a layer |
| *(no layer)* | any CSS not inside a layer | **beats all layers** |

The last row is the important surprise. CSS that isn't in any layer wins over every
layered rule, however specific. A plain `.title { color: red }` in an unlayered file
beats `text-sky-700`.

## Third-party CSS: the classic conflict

Say a widget library ships `widget.css`:

```css title="widget.css"
.widget-title { color: rgb(200, 0, 0); padding: 20px; }
```

You import it and try to adjust a title with utilities:

```html
<h2 class="widget-title text-sky-700 p-1">Ratings</h2>
```

We compiled two versions of the stylesheet and measured the `<h2>` in Chromium:

```css
/* Version 1 */
@import "./widget.css";
/* Version 2 */
@import "./widget.css" layer(components);
```

```text
Version 1 (unlayered):          color rgb(200, 0, 0)          padding 20px
Version 2 (layer(components)):  color oklch(0.5 0.134 242.7)  padding 4px
```

In version 1 the widget's unlayered rules beat the utilities entirely. In version 2,
importing it **into the `components` layer** put it below `utilities`, and `text-sky-700`
and `p-1` won. Wrapping third-party CSS in a layer is usually the right fix, and it's
much better than adding `!` to your utilities.

## Specificity inside the utilities layer

Within one layer, normal specificity and source order apply. Nearly all utilities are a
single class (`0,1,0`), and variants add pseudo-classes: `hover:bg-sky-700` is
`.hover\:bg-sky-700:hover`, which is `0,2,0` and beats `bg-sky-600` on hover. Where two
utilities have equal specificity, the one that comes **later in the generated file**
wins, and Tailwind controls that order (by variant, then by property). The order of
classes in your `class` attribute never matters.

That's why two conflicting utilities on one element, like `p-2 p-4`, give an answer you
can't control from the markup. Avoid putting conflicting utilities on the same element;
Level 3 · 06 covers merging classes when components accept overrides.

## What Preflight does

Preflight is Tailwind's base reset, in the `base` layer. Like normalize-style resets it
smooths over browser inconsistencies, but it goes further, removing most browser default styling so that elements look the
same in every browser and nothing is styled until you style it. The rules you'll notice
most (from 4.3.3):

```css
*, ::after, ::before, ::backdrop, ::file-selector-button {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
  border: 0 solid;
}
h1, h2, h3, h4, h5, h6 { font-size: inherit; font-weight: inherit; }
a { color: inherit; text-decoration: inherit; }
ol, ul, menu { list-style: none; }
img, svg, video, canvas, audio, iframe, embed, object { display: block; vertical-align: middle; }
img, video { max-width: 100%; height: auto; }
[hidden]:where(:not([hidden="until-found"])) { display: none !important; }
```

What each one means in practice:

- **No margins anywhere.** Paragraphs, headings and lists sit flush. You add spacing
  deliberately with utilities like `mt-4` or `space-y-4`.
- **`border: 0 solid`.** Borders start at zero width but with a style, which is why
  `border` (width 1px) alone shows a line: no need to also write `border-solid`.
- **Headings look like body text.** `<h1>` is semantic only until you add `text-3xl
  font-bold`. This separates meaning from appearance: choose the heading level for the
  document outline, not for its default size.
- **Links are unstyled.** They inherit colour and have no underline. Always restyle
  links in content, or users can't find them.
- **Lists have no bullets.** Add `list-disc ps-5` (or `list-decimal`) where you want
  them. A known side effect: Safari's VoiceOver has historically stopped announcing a
  list as a list when `list-style: none` is applied, so for navigation-style lists
  you want announced as lists, add `role="list"` to the `<ul>`. (We couldn't test
  VoiceOver for this course; check with your own screen reader.)
- **Images are block-level and responsive.** No mysterious gap under images, and they
  never overflow their container. Use `inline` or `inline-block` when you need an image
  in a line of text.
- **Buttons look plain** and keep `cursor: default`, like native buttons. Add
  `cursor-pointer` if your design wants a pointer.
- **`[hidden]` always wins.** `display: none !important` in the base layer means
  `<div hidden class="flex">` stays hidden; we checked and it computed to `display: none`.
  An important declaration in an earlier layer beats normal declarations in every later
  layer, so the attribute can't be overridden by a utility. To show the element, remove
  the attribute.

## Adding your own base styles

Put element-level defaults in `@layer base`, so utilities can still override them:

```css title="src/app.css"
@import "tailwindcss";

@layer base {
  h1 { font-size: var(--text-3xl); font-weight: var(--font-weight-bold); }
  a:not([class]) { color: var(--color-sky-700); text-decoration: underline; }
  ::selection { background-color: var(--color-amber-200); }
}
```

Using theme variables (`var(--text-3xl)`) keeps these consistent with your utilities.
The `a:not([class])` selector styles plain links in content (from Markdown, say) without
fighting links that you've styled with classes.

Keep base styles minimal. Every base rule is something you have to know about when a
utility doesn't seem to "start from nothing".

## Turning Preflight off

For an existing site where a global reset would break things, import only the theme and
utilities:

```css title="src/app.css"
@layer theme, base, components, utilities;
@import "tailwindcss/theme.css" layer(theme);
@import "tailwindcss/utilities.css" layer(utilities);
```

We compiled this (with scanning limited to one test file) and confirmed utilities like
`p-4` and `bg-sky-500` were generated while no Preflight rules were present (not even the
`box-sizing` reset). Without Preflight, utilities still work, but the starting point is
the browser's defaults: `<h1>` keeps its default size and margins, buttons have native
styling, and borders need `border-solid` explicitly. Level 4 · 06 covers introducing
Tailwind into an existing CSS codebase in more depth.

## How It Actually Works

Cascade layers were added to CSS so that the *source* of a rule, not just its selector,
decides precedence. The browser sorts declarations by origin and importance, then by
layer, then by specificity, then by order of appearance. For normal declarations, later
layers win and unlayered styles win over all layers. For `!important` declarations the
order **reverses**: earlier layers win, and an important rule in a layer beats an
important unlayered rule. That reversal is what makes Preflight's
`[hidden] { display: none !important }` impossible to override from `utilities`.

Tailwind relies on layers rather than specificity tricks. Every utility can be one class
with no `!important`, because the `utilities` layer is declared last. Everything you put
in `base` and `components` stays below it automatically.

## Common mistakes

- **Unlayered custom CSS** that silently beats every utility. Put it in `@layer base` or
  `@layer components`.
- **Importing third-party CSS without `layer(…)`**, then fighting it with `!`.
- **Expecting default heading sizes, link underlines or list bullets.** Preflight
  removes them; add them back where you want them.
- **`hidden` attribute plus a display utility** to show the element. Remove the
  attribute instead.
- **Choosing a heading level for its size.** Pick the level for the outline and style it.
- **Turning Preflight off on a new project.** You lose the consistent starting point for
  little gain.

## Exercise

1. Add an unlayered rule `.note { padding: 2rem; }` and an element with
   `class="note p-2"`. Check the computed padding. Move the rule into
   `@layer components` and check again.
2. Import a small third-party stylesheet with and without `layer(components)` and
   compare which of your utilities apply.
3. Write base styles so that plain links in a Markdown article are underlined and
   coloured, but your nav links (which have classes) aren't affected.
4. Put `hidden` and `flex` on the same element and confirm it stays hidden. Then show it
   by removing the attribute from the console.
