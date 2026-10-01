---
description: "Tailwind layouts for right-to-left languages: logical utilities like ms-, pe-, start- and border-s, rtl: and ltr: variants, and i18n text."
---

# 03 · RTL, Logical Utilities & Internationalization

Arabic, Hebrew, Persian and Urdu are written right to left. In those languages the
whole interface mirrors: the sidebar moves to the right, back arrows point right, text
aligns right, and the "start" of a row is on the right. If your CSS says "left" and
"right", every one of those decisions has to be written twice. If it says "start" and
"end", the browser handles most of it. This lesson covers Tailwind's logical utilities,
what they compiled to and how they behaved in a right-to-left test, the `rtl:` and `ltr:`
variants for the rest, and the text issues that come with translation.

(For the CSS side, see
[Logical Properties & i18n](https://sigilipelli.github.io/html-css-mastery-path/level-4/05-logical-properties-i18n/)
on the HTML & CSS course.)

## Direction comes from HTML

Set the document direction and language on `<html>`:

```html
<html lang="ar" dir="rtl">
```

`dir="rtl"` flips the inline direction for everything inside: text alignment, flex and
grid order, which side is "start". `lang` drives hyphenation, quotes, font selection and
screen reader pronunciation. Set both, and set them per element when a page mixes
languages (`<blockquote lang="he" dir="rtl">`).

## Logical utilities

Every physical spacing, inset, border and radius utility has a logical twin:

| Physical | Logical | CSS property (compiled) |
|---|---|---|
| `ml-4` / `mr-4` | `ms-4` / `me-4` | `margin-inline-start` / `margin-inline-end` |
| `pl-2` / `pr-2` | `ps-2` / `pe-2` | `padding-inline-start` / `-end` |
| `left-0` / `right-0` | `start-0` / `end-0` | `inset-inline-start` / `-end` |
| `border-l-4` | `border-s-4` | `border-inline-start-width` (+ style) |
| `rounded-l-lg` | `rounded-s-lg` | `border-start-start-radius` + `border-end-start-radius` |
| `text-left` | `text-start` | `text-align: start` |
| `float-left` | `float-start` | `float: inline-start` |
| `scroll-ml-4` | `scroll-ms-4` | `scroll-margin-inline-start` |

`mx-`, `px-`, `inset-x-` and `gap-` are already symmetrical, so they need no logical
version.

We rendered a few of these inside a `dir="rtl"` container in Chromium:

```text
ms-4         margin-left 0px, margin-right 16px    <- start is on the right
ml-4         margin-left 16px, margin-right 0px    <- physical stays left
text-start   computed "start" (renders right-aligned in RTL)
text-left    computed "left"  (stays left even in Arabic)
```

The rule for new code is simple: **write logical utilities by default**. Use physical
ones only for things that genuinely shouldn't mirror, and those are rarer than you'd
think. A progress bar, for example, *should* fill in reading direction, so it stays
logical. Real physical cases are things like a code block, a phone number field, a
video timeline or a map.

## Flex, grid and gap mirror automatically

Flex rows and grid columns follow the inline direction, so they mirror without any
extra classes. In the same RTL container, a 300px flex row with two 40px items:

```text
gap-4        first item x=360, second x=304
space-x-4    first item x=360, second x=304
```

The first item sits at the right edge and the second 16px to its left, for both. In v4,
`space-x-*` uses `margin-inline-end` (Level 3 · 08), so it mirrors correctly; in v3 it
used left margins and needed `rtl:space-x-reverse`. `gap` is still the better choice:
it doesn't touch the children's margins at all.

## The `rtl:` and `ltr:` variants

Some things can't be expressed logically. The classic one is **directional icons**: an
arrow pointing "forward" points right in English and left in Arabic.

```html
<a href="/next" class="inline-flex items-center gap-2">
  Next
  <svg class="size-4 rtl:rotate-180" aria-hidden="true" viewBox="0 0 20 20" fill="currentColor">
    <path d="M7.3 4.3a1 1 0 0 1 1.4 0l5 5a1 1 0 0 1 0 1.4l-5 5a1 1 0 0 1-1.4-1.4L11.6 10 7.3 5.7a1 1 0 0 1 0-1.4z"/>
  </svg>
</a>
```

```css
.rtl\:rotate-180:where(:dir(rtl), [dir="rtl"], [dir="rtl"] *) { rotate: 180deg; }
```

In our test the icon computed `rotate: 180deg` inside the RTL container. The variant
matches `:dir(rtl)` (the browser's computed direction) and the `dir` attribute on the
element or any ancestor.

Mirror only icons whose meaning is directional: arrows, "back"/"forward", chevrons in
breadcrumbs, a "reply" arrow. Don't mirror icons that represent real objects (a clock,
a play button for media, a checkmark, a logo).

Other uses of `rtl:`/`ltr:`: a `translate-x` animation on a slide-in drawer
(`-translate-x-full rtl:translate-x-full`), a gradient direction (`bg-linear-to-r
rtl:bg-linear-to-l`), or a box shadow offset.

## Text and translation

Layout isn't the only thing that changes with language:

- **Text length varies a lot.** A German or Finnish translation of a short English label
  is often much longer. Don't size buttons and tabs with fixed widths; use padding and
  let them grow, with `flex-wrap` on toolbars. Test with your longest language.
- **Long words.** `hyphens-auto` hyphenates only when the `lang` attribute is set and the
  browser has a dictionary for that language. `wrap-break-word` (`overflow-wrap:
  break-word`) breaks long unbreakable strings like URLs.
- **CJK text.** Chinese, Japanese and Korean don't use spaces between words.
  `break-keep` (`word-break: keep-all`) stops Korean words from breaking mid-word. Letter
  spacing designed for Latin headings (`tracking-tight`) often looks wrong in CJK.
- **Script-specific line height.** Arabic and Devanagari scripts often need more line
  height than Latin. Target a language with an arbitrary variant:
  `[&:lang(ar)]:leading-8` compiled to `.…:lang(ar) { line-height: … }`, and
  `[:lang(ja)_&]:tracking-normal` targets elements inside a Japanese section.
- **Fonts.** Make sure your font stack covers the scripts you support, or put a
  script-specific font first in a `--font-*` token used for that language.

## Worked example: a mirrored app header

```html
<header class="flex items-center gap-3 border-b border-gray-200 px-4 py-3">
  <a href="/" class="font-semibold">Acme</a>

  <nav aria-label="Main" class="ms-6 flex gap-4 text-sm max-md:hidden">
    <a href="/projects">Projects</a>
    <a href="/reports">Reports</a>
  </nav>

  <div class="ms-auto flex items-center gap-2">
    <input type="search" placeholder="Search"
           class="w-48 rounded-md border border-gray-300 py-1.5 ps-8 pe-3 text-sm">
    <a href="/back" class="inline-flex items-center gap-1 text-sm">
      <svg class="size-4 rtl:rotate-180" aria-hidden="true" viewBox="0 0 20 20" fill="currentColor">
        <path d="M12.7 15.7a1 1 0 0 1-1.4 0l-5-5a1 1 0 0 1 0-1.4l5-5a1 1 0 1 1 1.4 1.4L8.4 10l4.3 4.3a1 1 0 0 1 0 1.4z"/>
      </svg>
      Back
    </a>
  </div>
</header>
```

Switch `<html dir="rtl">` and: the logo moves to the right edge, the nav follows it with
its `ms-6` gap on the correct side, `ms-auto` pushes the search group to the left edge,
the search field's icon padding (`ps-8`) moves to the right side where the icon should
be, and the back arrow points right. Nothing in the markup changed except `dir`.

## How It Actually Works

Logical properties are defined relative to the **writing mode** and **direction** of
the element. `margin-inline-start` maps to `margin-left` when `direction` is `ltr` and to
`margin-right` when it's `rtl` (and to top or bottom in vertical writing modes). The
browser resolves the mapping per element at computed-value time, which is why the same
class works in both directions. Flexbox and grid lay out along the inline axis, so their
order follows direction automatically.

Tailwind's part is small: it generates the logical properties, and it provides the
`rtl`/`ltr` variants as selectors (`:dir()` plus the `dir` attribute) for the cases where
you need a physical change.

## Common mistakes

- **Writing `ml-`, `pr-`, `left-`, `text-left` by habit.** Use the logical versions.
- **Forgetting `dir` on the `<html>` element** and trying to mirror with CSS instead.
- **Mirroring every icon.** Only directional icons should flip.
- **Fixed widths for translated labels.**
- **Setting `hyphens-auto` without `lang`.** Nothing happens.
- **Using `rtl:space-x-reverse`** in v4 code, which no longer needs it (prefer `gap`).

## Exercise

1. Take your Level 2 dashboard, replace every physical utility with its logical
   equivalent, and switch `<html dir="rtl">`. List what still looks wrong and fix it with
   `rtl:` variants.
2. Find all directional icons in a project and add `rtl:rotate-180` only to those.
3. Replace the dashboard's button labels with long German translations and fix any
   layout that breaks.
4. Add `lang` attributes and test `hyphens-auto` on a narrow column of English and German
   text.
