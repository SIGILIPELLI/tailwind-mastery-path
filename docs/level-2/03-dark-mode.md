---
description: "Dark mode in Tailwind v4: the system default, a class toggle with @custom-variant, flash-free switching, and token-based themes."
---

# 03 · Dark Mode

Supporting dark mode means every surface, text colour, border and shadow needs a second
value. Tailwind gives you the `dark:` variant to set those values class by class. Out of
the box, `dark:` follows the operating system setting. Most real sites also want a
manual toggle, and larger ones avoid writing `dark:` everywhere by switching **tokens**
instead. This lesson builds all three approaches and explains when each one fits.

## The default: follow the system

With no configuration, `dark:` compiles to a `prefers-color-scheme` media query:

```html
<body class="bg-white text-gray-900 dark:bg-gray-950 dark:text-gray-100">
  <div class="rounded-xl border border-gray-200 p-6 dark:border-gray-800">…</div>
</body>
```

```css
@media (prefers-color-scheme: dark) {
  .dark\:bg-gray-950 { background-color: var(--color-gray-950); }
}
```

We rendered a `bg-white dark:bg-gray-900` element in Chromium with emulated colour
schemes. Under `light` it computed to white, and under `dark` to gray-900. No JavaScript,
no toggle. If your users are happy with "same as my OS", stop here.

## Adding a manual toggle with `@custom-variant`

To let users choose, redefine what `dark:` means. In v4 that's one line in your CSS:

```css title="src/app.css"
@import "tailwindcss";

@custom-variant dark (&:where(.dark, .dark *));
```

Now `dark:` means "this element has the class `dark`, or is inside an element that does".
The compiled rule:

```css
.dark\:bg-gray-900:where(.dark, .dark *) {
  background-color: var(--color-gray-900);
}
```

Put `class="dark"` on `<html>` and the whole page switches. If you prefer a data
attribute (handy when you also have other themes), use that instead:

```css
@custom-variant dark (&:where([data-theme=dark], [data-theme=dark] *));
```

### Why `:where()` matters

`:where()` has zero specificity, so `dark:bg-gray-900` has exactly the specificity of
`bg-white`: one class. It wins because dark-variant rules come later in the stylesheet,
not because of a stronger selector. We confirmed that with `<html class="dark">` both
`class="bg-white dark:bg-gray-900"` and `class="dark:bg-gray-900 bg-white"` computed to
gray-900. Class order in the attribute never matters.

The flip side: you can't make a "light island" inside a dark page with a `light` class.
`.dark *` still matches every descendant. Our test put
`<div class="light"><div class="bg-white dark:bg-gray-900">` inside a dark page, and the
inner element was still gray-900. If you need islands, use the token approach below.

## A three-way toggle without a flash

Users expect three choices: Light, Dark and System. Store the choice in `localStorage`
and apply the class **before the page paints**, otherwise a dark-mode user sees a white
flash on every load. That means a small blocking script in `<head>`, before your CSS:

```html title="index.html (in <head>)"
<script>
  (function () {
    var stored = null;
    try { stored = localStorage.getItem("theme"); } catch (e) {}
    var dark = stored === "dark" ||
      (stored !== "light" && matchMedia("(prefers-color-scheme: dark)").matches);
    document.documentElement.classList.toggle("dark", dark);
  })();
</script>
```

And the control:

```html
<label class="text-sm">
  Theme
  <select id="theme" class="ms-2 rounded-md border border-gray-300 bg-white px-2 py-1
                            dark:border-gray-700 dark:bg-gray-900">
    <option value="system">System</option>
    <option value="light">Light</option>
    <option value="dark">Dark</option>
  </select>
</label>

<script>
  const select = document.getElementById("theme");
  const media = matchMedia("(prefers-color-scheme: dark)");

  function apply(choice) {
    const dark = choice === "dark" || (choice === "system" && media.matches);
    document.documentElement.classList.toggle("dark", dark);
  }

  let current = "system";
  try { current = localStorage.getItem("theme") || "system"; } catch (e) {}
  select.value = current;

  select.addEventListener("change", () => {
    current = select.value;
    try {
      if (current === "system") localStorage.removeItem("theme");
      else localStorage.setItem("theme", current);
    } catch (e) {}
    apply(current);
  });

  // Follow OS changes while "System" is selected
  media.addEventListener("change", () => { if (current === "system") apply("system"); });
</script>
```

The `try`/`catch` blocks matter: in some private-browsing modes and with storage blocked,
touching `localStorage` throws. The page should still work, just without remembering the
choice.

A `<select>` is used here because it's accessible with no extra work. A pretty icon
button is fine too, as long as it's a real `<button>` with a label that says what it
does.

## The token approach: switch variables, not classes

Writing `dark:` next to every colour gets heavy: a card might need `bg-`, `border-`,
`text-` and `shadow-` overrides, each repeated for every component. The alternative is
semantic theme colours (lesson 02) whose **values** change in dark mode:

```css title="src/app.css"
@import "tailwindcss";

@custom-variant dark (&:where(.dark, .dark *));

@theme {
  --color-bg: var(--color-white);
  --color-bg-raised: var(--color-gray-50);
  --color-fg: var(--color-gray-900);
  --color-fg-muted: var(--color-gray-600);
  --color-line: var(--color-gray-200);
}

@layer base {
  .dark {
    --color-bg: var(--color-gray-950);
    --color-bg-raised: var(--color-gray-900);
    --color-fg: var(--color-gray-100);
    --color-fg-muted: var(--color-gray-400);
    --color-line: var(--color-gray-800);
  }
}
```

```html
<body class="bg-bg text-fg">
  <article class="rounded-xl border border-line bg-bg-raised p-6">
    <h2 class="font-semibold">Usage this month</h2>
    <p class="mt-1 text-fg-muted">You've used 62% of your quota.</p>
  </article>
</body>
```

No `dark:` classes at all. The utilities read `var(--color-bg)` and friends, and the
`.dark` rule redefines those variables for everything inside it. In our test, a
`bg-bg text-fg` element under `<html class="dark">` computed to the dark values.

Because it's just CSS variables, **islands work**: add a `.light` rule that sets the
light values again, and anything inside `class="light"` goes back to light.

Use `@layer base` for the override so it sits below the utilities in the cascade, and
make sure the `.dark` variables use the same names as the `@theme` ones. Most teams
combine both approaches: tokens for the 90% of surfaces and text, `dark:` for one-off
tweaks such as a hero image or a shadow that should disappear.

## Don't forget `color-scheme`

Scrollbars, form controls and the default canvas colour are drawn by the browser, not by
your CSS. Tell it which scheme the page supports:

```html
<html class="scheme-light dark:scheme-dark">
```

`scheme-light-dark` compiles to `color-scheme: light dark`, and `dark:scheme-dark` to
`color-scheme: dark` under your dark variant. Without it, a dark page can show bright
white scrollbars and native date pickers.

## Images and shadows

- **Images:** swap them with `dark:hidden` / `hidden dark:block` on two `<img>` elements,
  or, for the system-only approach, a `<picture>` with
  `<source media="(prefers-color-scheme: dark)">` so only one image downloads.
- **Shadows** barely show on dark backgrounds. Use borders or a lighter raised surface
  instead: `shadow-md dark:shadow-none dark:ring-1 dark:ring-white/10`.
- **Pure black** (`#000`) behind white text can be tiring to read. Most dark designs use a
  very dark grey, like `gray-950`.

## How It Actually Works

`dark` is a built-in variant whose default definition is the media query
`@media (prefers-color-scheme: dark)`. `@custom-variant dark (…)` replaces that
definition. The `&` stands for the generated utility's selector, so
`&:where(.dark, .dark *)` turns `.dark\:bg-gray-900` into
`.dark\:bg-gray-900:where(.dark, .dark *)`. Every `dark:` class, including stacked ones
like `dark:md:hover:`, is generated through that template.

Variant rules are sorted after plain utilities in the utilities layer. With `:where()`
keeping specificity equal, the later rule wins whenever the dark selector matches. The
token approach doesn't use variants at all. It relies on CSS custom property inheritance:
the utility always says `var(--color-bg)`, and the value is whatever the nearest ancestor
(or the element itself) defined last in the cascade.

## Common mistakes

- **Adding a toggle without the `@custom-variant` line.** `dark:` keeps following the OS
  and the class does nothing.
- **Applying the class in a script at the end of `<body>`.** The page paints light first,
  then flashes dark. Use a tiny blocking script in `<head>`.
- **Not handling storage errors.** The theme script throws and the page breaks in
  locked-down browsers.
- **Expecting `class="light"` to reset part of a dark page** with the `dark:` variant.
  Use tokens for islands.
- **Forgetting `color-scheme`**, leaving bright native scrollbars and inputs.
- **Duplicating a whole palette in `dark:` classes** across hundreds of components. Use
  semantic tokens for surfaces and text.
- **Only testing one mode.** Check focus rings, disabled states and borders in both.

## Exercise

1. Take the Level 1 landing page and add `dark:` classes so it follows the OS setting.
   Test with DevTools' "Emulate CSS prefers-color-scheme".
2. Switch to a class-based variant and build the three-way select with the flash-free
   `<head>` script. Reload in dark mode and confirm there's no white flash.
3. Convert the page's surfaces and text to semantic tokens and delete as many `dark:`
   classes as you can. Count them before and after.
4. Add a "light island" (a code sample card that should stay light) using a `.light`
   token rule.
