---
description: "Tailwind's escape hatches: arbitrary values, the (--var) shorthand for CSS variables, type hints, arbitrary properties and variants."
---

# 04 · Arbitrary Values, Properties & CSS Variables

The theme covers your design system's values. Real pages always have exceptions: a
background image, a width that must match a third-party embed, a `calc()`, a CSS property
Tailwind has no utility for. Tailwind's square-bracket syntax lets you write any value,
any property, or any selector inline and still get a normal generated class.

Used occasionally, these escape hatches save you from creating one-off CSS files. Used
constantly, they turn your markup into hard-to-read inline styles with extra steps. This
lesson covers how they work, the parsing rules that trip people up, and how to tell
which situation you're in.

## Arbitrary values: `utility-[value]`

Any utility that takes a value accepts one in brackets:

```html
<div class="w-[37rem] p-[3px] bg-[#1da1f2] text-[22px] top-[calc(100%-1px)]"></div>
```

Compiled (Tailwind 4.3.3):

```css
.top-\[calc\(100\%-1px\)\] { top: calc(100% - 1px); }
.w-\[37rem\] { width: 37rem; }
.bg-\[\#1da1f2\] { background-color: #1da1f2; }
.p-\[3px\] { padding: 3px; }
.text-\[22px\] { font-size: 22px; }
```

Notice the `calc()`: you wrote `100%-1px` with no spaces, and Tailwind added spaces
around the `-`. CSS requires spaces around `+` and `-` in `calc()`, and since a class
name can't contain spaces, Tailwind inserts them for you.

Arbitrary values also work with variants: `md:w-[42rem]`, `hover:bg-[#0c85d0]`.

## The space rule: use underscores

Class names are separated by spaces, so `p-[1rem 2rem]` is two broken tokens, `p-[1rem`
and `2rem]`. We compiled `w-[37 rem]` and `p-[1rem 2rem]` and got nothing at all for
either. Use underscores where CSS needs spaces:

```html
<div class="grid grid-cols-[1fr_auto]"></div>       <!-- 1fr auto -->
<div class="bg-[rgb(0_0_0/50%)]"></div>             <!-- rgb(0 0 0/50%) -->
<p class="before:content-['hello_world']"></p>       <!-- 'hello world' -->
```

When you need a real underscore, escape it with a backslash: `content-['snake\_case']`
compiled to `'snake_case'`. (Inside a URL, Tailwind keeps underscores as they are, since
underscores in URLs are common.)

Tailwind checks that brackets are balanced and the value looks like CSS. It **doesn't**
check that the value is valid for the property: `w-[100px_200px]` happily compiled to
`width: 100px 200px;`, which the browser then drops. If an arbitrary class "does
nothing", look at the Computed pane in DevTools to see whether the declaration was
rejected.

## CSS variables: the `(--var)` shorthand

v4 has a dedicated short form for using a CSS variable as the value: parentheses instead
of brackets, with no `var()`:

```html
<div class="bg-(--brand) gap-(--gap)"></div>
```

```css
.bg-\(--brand\) { background-color: var(--brand); }
.gap-\(--gap\) { gap: var(--gap); }
```

`bg-[var(--brand)]` produces the same thing; the parentheses form is just shorter. In
v3 you'd write `bg-[--brand]`. Don't carry that habit over: in 4.3.3, `bg-[--brand]`
compiled to `background-color: --brand;`, which is invalid CSS and silently ignored.
Level 3 · 08 covers this and the other migration changes.

This is the cleanest way to feed **runtime** values into Tailwind, such as a colour from a
database or a progress percentage computed in JavaScript. Set the variable with an
inline style and let a class use it:

```html
<div class="h-2 w-full rounded-full bg-gray-200">
  <div class="h-full w-(--progress) rounded-full bg-sky-600" style="--progress: 62%"></div>
</div>
```

You **can't** build class names dynamically (`w-[${pct}%]`), because the compiler never
sees the final string (Level 3 · 01 explains exactly why). Variables are the answer to
that.

## Type hints for ambiguous utilities

Some utilities serve several properties. `text-` is both `font-size` and `color`. With a
literal value Tailwind can tell which you mean: `text-[22px]` is clearly a length,
`text-[#333]` clearly a colour. With a variable it can't see the value, and it guesses
**colour**:

```css
.text-\(--size\) { color: var(--size); }   /* probably not what you wanted */
```

Add a **type hint** to say what the variable holds:

```html
<p class="text-(length:--size)">…</p>    <!-- font-size: var(--size) -->
<p class="text-(color:--ink)">…</p>      <!-- color: var(--ink) -->
<p class="text-[length:var(--y)]">…</p>  <!-- the bracket form works too -->
```

All three compiled as the comments say. Common hints are `length`, `color`, `percentage`,
`number`, `url`, `image` and `position`. You only need them when a utility is ambiguous
and the value is a variable.

## Arbitrary properties: `[property:value]`

When there's no utility for a CSS property at all, write the declaration in brackets as
the whole class:

```html
<svg class="[mask-type:luminance] hover:[mask-type:alpha]">…</svg>
<div class="[--gap:1rem] gap-(--gap) md:[--gap:2rem]">…</div>
```

```css
.\[mask-type\:luminance\] { mask-type: luminance; }
.\[--gap\:1rem\] { --gap: 1rem; }
```

The second line is a useful pattern: setting a CSS variable with variants. The variable
changes at `md`, and every child that uses `var(--gap)` follows.

## Arbitrary variants: `[selector]:utility`

You can also write the selector part yourself. `&` stands for the element with the class:

```html
<ul class="[&>li]:mt-2">…</ul>               <!-- direct li children -->
<article class="[&_p]:leading-7">…</article>  <!-- any p inside (_ is a space) -->
```

```css
.\[\&\>li\]\:mt-2 > li { margin-top: calc(var(--spacing) * 2); }
.\[\&_p\]\:leading-7 p { line-height: calc(var(--spacing) * 7); /* … */ }
```

These are handy for styling content you don't control, like HTML from a CMS. For direct
children there's also the built-in `*:` variant (`*:mt-2`), and Level 2 · 06 covers
custom variants you can name and reuse.

## `!` for important

Add `!` at the **end** of a class to make its declaration `!important`:

```html
<div class="bg-red-500!"></div>
```

```css
.bg-red-500\! { background-color: var(--color-red-500) !important; }
```

v4 also still accepts the v3 position at the start (`!bg-red-500`), and both compiled
the same way in 4.3.3, but the trailing form is the documented one. Reach for `!` only
when you're fighting CSS you don't control, such as a third-party widget's styles.
Inside your own code, needing it is a sign that two utilities conflict on the same
element (Level 3 · 06 covers merging classes properly).

## Worked example: a hero with a background image and a precise overlap

```html
<section class="relative isolate overflow-hidden
                bg-[url(/img/harbour.jpg)] bg-cover bg-center">
  <div class="absolute inset-0 -z-10 bg-[linear-gradient(to_top,rgb(0_0_0/70%),transparent_60%)]"></div>

  <div class="mx-auto max-w-6xl px-4 pt-[min(30vh,16rem)] pb-16 text-white">
    <h1 class="max-w-[18ch] text-4xl font-bold text-balance sm:text-5xl">
      Weekend sailing courses on the south coast
    </h1>
  </div>

  <!-- Card that overlaps the hero's bottom edge by exactly 3rem -->
  <div class="relative mx-auto -mb-12 max-w-3xl rounded-xl bg-white p-6
              [--card-shadow:0_10px_30px_rgb(0_0_0/0.15)] shadow-(--card-shadow)">
    …
  </div>
</section>
```

Arbitrary values earn their place here:

- The background image URL and the gradient are one-offs specific to this section.
- `pt-[min(30vh,16rem)]` uses a CSS function no utility covers.
- `max-w-[18ch]` limits the headline by characters, a unit Tailwind's scale doesn't use.

The last card shows the line to watch. `[--card-shadow:…] shadow-(--card-shadow)` works,
but it's hard to read, and the built-in `shadow-lg` would probably have done. If this shadow is a real part of
the design, it belongs in the theme as `--shadow-card` (lesson 01).

## When to use which

| Situation | Use |
|---|---|
| Value used once, and specific to this element | Arbitrary value `w-[37rem]` |
| Value repeated in several places | A theme token (`@theme`) |
| Value changes at runtime | A CSS variable + `(--var)` class |
| CSS property with no utility | Arbitrary property `[mask-type:…]` |
| Styling children you don't control | Arbitrary variant `[&_p]:…` or `*:` |
| A long or repeated arbitrary class | A custom utility (lesson 05) |

## How It Actually Works

When the compiler finds a candidate like `bg-[#1da1f2]`, it splits it into a root (`bg`)
and a value in brackets. For bracket values it skips the theme lookup and runs the value
through a decoder: underscores become spaces (except escaped ones and inside `url()`),
and spaces are added around math operators in `calc()`-style functions. Then it infers
the **type** of the value (colour, length, percentage, URL, …) to pick which property to
generate when the root is ambiguous, using the hint if you gave one. The parentheses form
`bg-(--brand)` is the same pipeline, with the value wrapped in `var()`.

An arbitrary property (`[mask-type:luminance]`) skips the utility plugins entirely. The
compiler just checks that it looks like `property:value` and emits it. An arbitrary
variant (`[&>li]:`) replaces `&` with the escaped class selector. Because the class name
must be a valid CSS selector, every special character is escaped with a backslash in the
output, which is why the compiled selectors look so busy.

## Common mistakes

- **Spaces inside brackets.** Use `_`. A class with a space silently generates nothing.
- **`text-(--size)` without a hint**, which sets `color`. Write `text-(length:--size)`.
- **Building class names with string interpolation**, e.g. `` `w-[${width}px]` ``. Use a
  CSS variable instead.
- **Using the v3 `bg-[--brand]` form in v4.** It compiles to invalid CSS. Use
  `bg-(--brand)`.
- **Repeating the same arbitrary value** in many files. Make it a theme token.
- **Using `!` to win against your own utilities.** Fix the conflicting class instead.
- **Trusting that a generated class is valid CSS.** Tailwind doesn't validate values;
  check DevTools.

## Exercise

1. Build a progress bar whose width comes from `style="--progress: 40%"` and a
   `w-(--progress)` class. Change the variable from the console and watch it update.
2. Create `--size: 1.75rem` on a parent and try `text-(--size)` and
   `text-(length:--size)` on a child. Explain the difference using DevTools.
3. Render some CMS-style HTML (`<h2>`, `<p>`, `<ul>`) inside an `<article>` and style it
   with arbitrary variants only. Then note which ones you'd rather make permanent.
4. Find three arbitrary values in the Level 1 landing page you could add, and decide for
   each whether it should be arbitrary or a theme token.
