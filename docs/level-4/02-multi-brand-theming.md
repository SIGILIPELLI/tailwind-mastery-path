---
description: "Multi-brand and runtime theming in Tailwind v4: brand scopes, why subtree overrides need @theme inline, tenant colours and readable text."
---

# 02 · Multi-Brand & Runtime Theming

Dark mode (Level 2 · 03) is one kind of theming: two value sets for the same tokens. Many
products need more: a white-label app where each customer has their own brand colour,
a company with several sub-brands sharing one component library, or a dashboard where
each workspace picks an accent. This lesson covers the three common setups, the variable
resolution rule that breaks the most obvious implementation, and how to keep
runtime-chosen colours readable.

## Setup 1: a fixed set of brands, switched by attribute

When the brands are known in advance, put each brand's values in CSS and switch with an
attribute. Components use semantic tokens only, as in Level 3 · 07:

```css title="src/app.css"
@import "tailwindcss";

@theme {
  --color-action: oklch(48% 0.17 255);         /* default brand */
  --color-action-hover: oklch(42% 0.16 255);
  --color-on-action: var(--color-white);
}

@layer base {
  [data-brand="forest"] {
    --color-action: oklch(45% 0.12 150);
    --color-action-hover: oklch(39% 0.11 150);
  }
  [data-brand="sunrise"] {
    --color-action: oklch(80% 0.16 85);
    --color-action-hover: oklch(74% 0.16 85);
    --color-on-action: var(--color-gray-950);  /* light brand: dark text */
  }
}
```

```html
<html data-brand="forest">…</html>
<!-- or scope a brand to part of a page -->
<section data-brand="sunrise"> <button class="bg-action text-on-action">…</button> </section>
```

The override rules redefine the **same semantic variables** the utilities read
(`--color-action`), on the element with the attribute. Everything inside inherits the
new values. This is the same mechanism as the token-based dark mode and the "light
island" from Level 2 · 03, and it works on any subtree.

Note the `sunrise` brand also changes `--color-on-action`. A light brand colour needs dark
text; one "text on brand" colour can't work for every brand.

## The trap: theme tokens that point at other variables

A tidier-looking structure is to keep raw brand values in their own variables and make
the semantic token point at them:

```css
@theme {
  --color-action: var(--brand-600);
}
:root { --brand-600: oklch(48% 0.17 255); }
[data-brand="forest"] { --brand-600: oklch(45% 0.12 150); }
```

Then only `--brand-600` needs overriding per brand. We tested an element with
`bg-action` at the root and another inside `<section data-brand="forest">`:

```text
@theme          root: oklch(0.48 0.17 255)   inside forest: oklch(0.48 0.17 255)   <- not switched
@theme inline   root: oklch(0.48 0.17 255)   inside forest: oklch(0.45 0.12 150)   <- switched
```

With plain `@theme`, the forest section stayed blue. The reason is how CSS variables
resolve: `--color-action: var(--brand-600)` is declared on `:root`, so it's resolved
**on `:root`**, using the root's `--brand-600`. Descendants inherit the already-resolved
value; changing `--brand-600` lower down doesn't re-resolve it.

`@theme inline` fixes it, because the utility then contains the reference itself:

```css
.bg-action { background-color: var(--brand-600); }
```

which is resolved on each element, picking up the nearest `--brand-600`. The rule: if
a theme token refers to another variable that you override **below the root**, declare
it with `@theme inline` (or override the semantic token itself, as in Setup 1).

## Setup 2: runtime colours from data

When customers choose their own colour, you can't list brands in CSS. Set a variable from
the server or JavaScript, and derive the rest in CSS with **relative colour syntax**:

```css title="src/app.css"
@theme inline {
  --color-brand: var(--brand);
  --color-brand-strong: oklch(from var(--brand) calc(l - 0.1) c h);
  --color-brand-soft: oklch(from var(--brand) 0.96 calc(c * 0.25) h);
}
:root { --brand: oklch(55% 0.2 255); }   /* fallback */
```

```html
<body style="--brand: {{ workspace.brand_hex }}">
  <a class="bg-brand-strong px-4 py-2 text-white hover:bg-brand">Upgrade</a>
  <div class="bg-brand-soft p-4">…</div>
</body>
```

`oklch(from var(--brand) calc(l - 0.1) c h)` means "the brand colour, converted to
oklch, with its lightness reduced by 0.1". With `--brand: #e11d48`, Chromium computed:

```text
bg-brand         rgb(225, 29, 72)
bg-brand-strong  oklch(0.486 0.222 17.6)
bg-brand-soft    oklch(0.96 0.056 17.6)
```

One input colour, a usable mini-scale out. `@theme inline` matters here too, because
`--brand` is set on `<body>`, not on `:root`.

Relative colour syntax is supported in current versions of the major browsers; check
your support targets, and keep the plain `--brand` fallback meaningful for older ones.
Never put user input into a `style` attribute without validating it as a colour on the
server first.

## Keeping runtime colours readable

A customer can pick any colour, including one that's unreadable with white text. Two
layers of defence.

**In CSS, where supported:** `contrast-color()` returns black or white, whichever
contrasts more with the given colour. We tested it in Chromium 149:

```html
<p class="bg-brand text-[contrast-color(var(--brand))]">…</p>
```

```text
--brand #e11d48 (rose):    contrast-color -> white   contrast 4.70
--brand #facc15 (yellow):  contrast-color -> black   contrast 13.71
white text on the yellow brand, for comparison:      contrast 1.53
```

`contrast-color()` is very new, so treat it as an enhancement; browsers that don't
support it drop the declaration.

**On the server, when the colour is saved:** compute the contrast once and store the
text colour alongside the brand:

```js title="lib/brand-check.mjs"
// Check a tenant's brand colour before saving it.
function luminance(hex) {
  const [r, g, b] = hex.replace("#", "").match(/../g).map((h) => parseInt(h, 16) / 255);
  const lin = (v) => (v <= 0.04045 ? v / 12.92 : ((v + 0.055) / 1.055) ** 2.4);
  return 0.2126 * lin(r) + 0.7152 * lin(g) + 0.0722 * lin(b);
}
function ratio(a, b) {
  const [hi, lo] = [luminance(a), luminance(b)].sort((x, y) => y - x);
  return +((hi + 0.05) / (lo + 0.05)).toFixed(2);
}

export function checkBrand(hex) {
  const white = ratio(hex, "#ffffff");
  const black = ratio(hex, "#000000");
  return {
    hex,
    textOnBrand: white >= black ? "white" : "black",   // the readable choice
    white,
    black,
    whiteTextOk: white >= 4.5,                         // if your design insists on white
    asTextOnWhite: white >= 4.5,                       // can it be used for links on white?
  };
}

for (const hex of ["#e11d48", "#facc15", "#2563eb", "#14b8a6", "#7c7c7c"]) {
  console.log(checkBrand(hex));
}
```

```text
{ hex: '#e11d48', textOnBrand: 'white', white: 4.7,  black: 4.47,  whiteTextOk: true,  asTextOnWhite: true }
{ hex: '#facc15', textOnBrand: 'black', white: 1.53, black: 13.71, whiteTextOk: false, asTextOnWhite: false }
{ hex: '#2563eb', textOnBrand: 'white', white: 5.17, black: 4.06,  whiteTextOk: true,  asTextOnWhite: true }
{ hex: '#14b8a6', textOnBrand: 'black', white: 2.49, black: 8.44,  whiteTextOk: false, asTextOnWhite: false }
{ hex: '#7c7c7c', textOnBrand: 'black', white: 4.17, black: 5.03,  whiteTextOk: false, asTextOnWhite: false }
```

(Output reformatted onto one line per colour.) The script's value for rose with white
text, 4.70, matches what we measured in the browser.

An interesting property shows up in the numbers: every colour has **at least about
4.58:1** against either black or white. (That's √21, the point where the two ratios are
equal.) So "pick black or white" always passes AA for normal text. Real problems only
appear when the design insists on white text, or uses the brand colour *as* text on a
white background, like `#14b8a6` links at 2.49:1. Store `textOnBrand` and use
`--brand-strong` (darker) for links, or fall back to your default action colour when
`asTextOnWhite` is false.

## Setup 3: a theme per component library consumer

If several apps share one library, ship the semantic token names as the library's API
and let each app define the values:

```css title="app-a/src/app.css"
@import "tailwindcss";
@import "@acme/ui/theme.css";   /* token names + defaults */

@layer base {
  :root {
    --color-action: oklch(52% 0.18 300);   /* app A's purple */
  }
}
```

The library's components keep using `bg-action`, and each app gets its own brand without
forking the components.

## How It Actually Works

Custom properties are resolved per element at computed-value time. When an element
declares `--a: var(--b)`, the browser substitutes `--b`'s value **for that element** and
stores the result; children inherit the result, not the formula. That's why where a
reference is *declared* decides which value it sees. `@theme` declares tokens on `:root`,
so references inside them see root values. `@theme inline` copies the reference into each
utility, so it's resolved on the element using the class.

Relative colour syntax (`oklch(from X l c h)`) converts X into the target colour space
and lets you use and modify its channels with `calc()`. It's computed by the browser for
each element, so it follows runtime changes to `--brand`. `contrast-color()` is likewise
resolved per element.

## Common mistakes

- **A semantic token in plain `@theme` pointing at a brand variable** that you override
  in a subtree. It won't switch; use `@theme inline` or override the semantic token.
- **One text colour for all brand colours.** Light brands need dark text.
- **Putting unvalidated user input in `style`.** Validate it as a colour first.
- **Relying on `contrast-color()` alone.** It's new; compute the text colour on the
  server as well.
- **Using a brand colour as link text** without checking it against the page background.
- **Duplicating components per brand** instead of switching tokens.

## Exercise

1. Add two `data-brand` themes to your Level 3 component library and render the gallery
   inside each. Check contrast for every variant (Level 3 · 10).
2. Reproduce the `@theme` vs `@theme inline` difference with a subtree override, and
   explain it using the browser's Computed pane.
3. Build a "workspace colour" picker that sets `--brand` on `<body>` and derives strong and
   soft shades with relative colour syntax.
4. Add the brand-check script to the server code that saves the colour, and use its
   `textOnBrand` result in the template.
