---
description: "Style hover, focus, disabled and form states with Tailwind variants, react to parents with group and siblings with peer, and avoid the unnamed-peer leak."
---

# 08 · Hover, Focus & Other States: group and peer

Every utility you've used so far applies all the time. Real interfaces change on hover,
when a field gets focus, when a checkbox is ticked, when a form value is invalid. In plain
CSS you'd write a second rule with a pseudo-class. In Tailwind you put a **variant** in
front of the utility: `hover:bg-sky-600` means "`bg-sky-600`, but only while hovered".

This lesson covers the state variants you'll use every day, the two that let one element
react to *another* element (`group` and `peer`), and one trap with `peer` that's easy to
miss in testing.

## A variant is a prefix that wraps the rule

```html
<button class="rounded-md bg-sky-700 px-4 py-2 font-medium text-white
               hover:bg-sky-800 focus-visible:outline-2 focus-visible:outline-offset-2
               focus-visible:outline-sky-600 active:scale-95
               disabled:opacity-50">
  Save
</button>
```

Here is what the compiler produced for some of those classes (Tailwind v4.3, utilities
layer, trimmed):

```css
@media (hover: hover) {
  .hover\:bg-sky-800:hover { background-color: var(--color-sky-800); }
}
.focus-visible\:outline-2:focus-visible {
  outline-style: var(--tw-outline-style);
  outline-width: 2px;
}
.active\:scale-95:active { /* … */ scale: var(--tw-scale-x) var(--tw-scale-y); }
.disabled\:opacity-50:disabled { opacity: 50%; }
```

Two things to notice:

1. The class name contains the colon, escaped as `\:`, so `hover:bg-sky-800` is one
   ordinary class. Nothing happens at runtime; it's plain CSS.
2. **`hover:` is wrapped in `@media (hover: hover)`.** In v4, hover styles only apply on
   devices whose main pointer can actually hover. On a phone, tapping a button no longer
   leaves it stuck in its hover colour. If you were used to v3 this is a behaviour change.
   Don't rely on hover alone to reveal anything important, because touch users never
   see it.

## The everyday state variants

| Variant | CSS it adds | Typical use |
|---|---|---|
| `hover:` | `:hover` inside `@media (hover: hover)` | Buttons, links, rows |
| `focus:` | `:focus` | Inputs: style while editing |
| `focus-visible:` | `:focus-visible` | Focus rings on buttons and links (keyboard focus only) |
| `focus-within:` | `:focus-within` | Highlight a wrapper when any field inside has focus |
| `active:` | `:active` | Pressed feedback |
| `disabled:` | `:disabled` | Dimmed, non-interactive controls |
| `first:` / `last:` / `only:` | `:first-child`, `:last-child`, `:only-child` | Removing the border on the last list item |
| `odd:` / `even:` | `:nth-child(odd/even)` | Striped table rows |
| `empty:` | `:empty` | Hide an empty container |
| `required:` / `invalid:` / `user-invalid:` | matching pseudo-classes | Form validation styling |
| `placeholder:` / `file:` / `marker:` / `selection:` | `::placeholder`, `::file-selector-button`, `::marker`, `::selection` | Pseudo-elements |

A pattern worth copying for lists. The variant goes on the *child* because the pseudo-class
describes the child:

```html
<ul class="divide-y divide-gray-200 rounded-lg border border-gray-200">
  <li class="px-4 py-3 odd:bg-white even:bg-gray-50">Invoices</li>
  <li class="px-4 py-3 odd:bg-white even:bg-gray-50">Payments</li>
  <li class="px-4 py-3 odd:bg-white even:bg-gray-50">Refunds</li>
</ul>
```

### `focus` vs `focus-visible`

`focus:` matches whenever the element has focus, including after a mouse click.
`focus-visible:` matches when the browser decides a visible indicator is needed: keyboard
navigation, and *always* for text inputs. Use `focus-visible:` for the ring on buttons
and links, so mouse users don't get a ring after every click. For text fields either
works, and many designs use `focus:` there.

Never remove the outline without replacing it. `outline-none` with nothing else leaves
keyboard users with no idea where they are.

### `invalid` vs `user-invalid`

`invalid:` applies as soon as the value is invalid, so an empty `required` field is red
the moment the page loads, before the user has typed anything. `user-invalid:` maps to the
newer `:user-invalid` pseudo-class, which only matches after the user has interacted with
the field (for example, edited it and moved away, or tried to submit). For most forms,
`user-invalid:` is the friendlier choice. Check your browser support targets before
relying on it; it's a recent addition to CSS.

## Stacking variants

Variants combine. `md:hover:bg-sky-700` means "at the `md` breakpoint and up, while
hovered". In v4, stacked variants apply **left to right**, outermost first. The
compiler output shows the nesting:

```css
@media (width >= 48rem) {
  @media (hover: hover) {
    .md\:hover\:bg-red-600:hover { background-color: var(--color-red-600); }
  }
}
```

For media-query and pseudo-class variants the order rarely changes the result. It *does*
matter when a variant targets a different element (like `*:` below, or the `before:`
pseudo-element in later lessons). There, read the class left to right as a path. The
convention is to put responsive variants first (`md:hover:…`) so classes scan
consistently.

You can also negate a state: `hover:not-disabled:bg-sky-700` compiles to
`:hover:not(:disabled)`, so a disabled button won't darken on hover.

## `group`: style a child based on its parent's state

A whole card is a link. When you hover the card, the title should underline and an arrow
should slide. The hovered element is the card, but the styled elements are its children.
Mark the parent with `group` and use `group-*` variants on the children:

```html
<a href="/guides/grid" class="group block rounded-xl border border-gray-200 p-5
                              hover:border-sky-300 hover:shadow-md">
  <h3 class="font-semibold text-gray-900 group-hover:underline">Grid guide</h3>
  <p class="mt-1 text-sm text-gray-600">Columns, spans and subgrid.</p>
  <span class="mt-3 inline-block text-sky-700 transition-transform
               group-hover:translate-x-1" aria-hidden="true">→</span>
</a>
```

`group` itself generates no CSS. It's a marker class. The child's utility compiles to:

```css
@media (hover: hover) {
  .group-hover\:underline:is(:where(.group):hover *) { text-decoration-line: underline; }
}
```

In words: "this element, when it's a descendant of *any* hovered `.group`." Every state
variant has a `group-` form: `group-focus-within:`, `group-has-checked:`,
`group-disabled:` and so on.

### Named groups for nested groups

Because the selector is "any `.group` ancestor", nesting two groups causes crosstalk:
hovering the outer card triggers `group-hover:` styles in the inner one too. Give each
group a name with a slash:

```html
<li class="group/item flex items-center justify-between p-3 hover:bg-gray-50">
  <span>Quarterly report.pdf</span>
  <div class="group/actions flex gap-2">
    <button class="invisible group-hover/item:visible">Rename</button>
    <button class="invisible group-hover/item:visible
                   group-hover/actions:text-red-600">Delete</button>
  </div>
</li>
```

`group-hover/item:` compiles to `:where(.group\/item):hover *`, so it only listens to the
element marked `group/item`.

(Hover-only controls like these are invisible to touch users. On a real product, keep
them visible on small screens or reveal them with `focus-within` too.)

## `peer`: style an element based on a previous sibling

`peer` is the sibling version. Mark an element with `peer` and a *later* sibling can react
to its state. The classic use is form validation messages that need no JavaScript:

```html
<label class="block">
  <span class="text-sm font-medium">Email</span>
  <input type="email" class="peer mt-1 block w-full rounded-md border-gray-300
                             user-invalid:border-red-500">
  <p class="mt-1 hidden text-sm text-red-600 peer-user-invalid:block">
    Enter a valid email address.
  </p>
</label>
```

The generated selector uses the general sibling combinator `~`:

```css
.peer-invalid\:block:is(:where(.peer):invalid ~ *) { display: block; }
```

Because CSS can only look *forward* through siblings, the `peer` element must come
**before** the element using `peer-*`. Put the message after the input in the markup.
If you need the label visually after the input (a floating label, for example), keep the
DOM order and reposition it with layout utilities.

## The unnamed-peer leak

That `~` has a consequence you won't see with one field but will see with two. `~ *`
matches **every** later sibling, not just the next one. So a message is shown if *any*
earlier `.peer` in the same parent is invalid, not just "its" field.

We tested this in Chromium with two fields in the same parent: an invalid email followed
by a valid, filled-in name:

```html
<form>
  <input type="email" class="peer" value="bad">
  <p id="m1" class="hidden peer-invalid:block">Email invalid</p>
  <input type="text" required class="peer" value="ok">
  <p id="m2" class="hidden peer-invalid:block">Name required</p>
</form>
```

Computed `display` for the messages:

```text
m1: block
m2: block   <- shown, although the name field is valid
```

The "Name required" message appeared because the *email* input is an earlier `.peer`
sibling and it's invalid. The fix is **named peers**, the same slash syntax as groups:

```html
<input type="email" class="peer/email" value="bad">
<p class="hidden peer-invalid/email:block">Email invalid</p>
<input type="text" required class="peer/name" value="ok">
<p class="hidden peer-invalid/name:block">Name required</p>
```

Result: the email message shows (`block`), the name message stays hidden (`none`). The
other fix is to wrap each field and its message in its own element, so they aren't
siblings of the other fields. That's often the better structure anyway.

## Worked example: a settings row with a toggle

A checkbox styled as a switch, with a status label that changes text colour when it's on,
using only `peer` and `has-`:

```html
<label class="flex cursor-pointer items-center justify-between gap-4 rounded-lg
              border border-gray-200 p-4 has-checked:border-sky-300 has-checked:bg-sky-50">
  <span>
    <span class="block font-medium text-gray-900">Email notifications</span>
    <span class="block text-sm text-gray-500">Weekly summary of activity.</span>
  </span>

  <input type="checkbox" class="peer sr-only">
  <span aria-hidden="true"
        class="relative h-6 w-11 shrink-0 rounded-full bg-gray-300 transition-colors
               peer-checked:bg-sky-600 peer-focus-visible:outline-2
               peer-focus-visible:outline-offset-2 peer-focus-visible:outline-sky-600
               after:absolute after:top-0.5 after:left-0.5 after:size-5 after:rounded-full
               after:bg-white after:transition-transform peer-checked:after:translate-x-5"></span>
</label>
```

- The real checkbox is `sr-only`: visually hidden, still focusable and announced by
  screen readers. The visible track is a decorative `<span>` after it.
- `peer-checked:bg-sky-600` colours the track; `peer-checked:after:translate-x-5` moves the
  knob. That's a stacked variant read left to right: when the peer is checked, on this
  element's `::after`, translate it.
- `peer-focus-visible:outline-*` puts the keyboard focus ring on the visible track, since
  the real input is invisible.
- `has-checked:` on the `<label>` tints the whole row. It compiles to `:has(:checked)`,
  a parent selector, so no `group` is needed. More on `has-` in Level 2 · 06.

## How It Actually Works

Every variant is a function that takes a generated rule and rewrites its selector or wraps
it in an at-rule. `hover` appends `:hover` and wraps the rule in `@media (hover: hover)`.
`disabled` appends `:disabled`. `md` wraps in `@media (width >= 48rem)`. Stacking runs the
functions in order, which is why `md:hover:` nests the hover media query inside the
breakpoint one.

`group-*` and `peer-*` are *compound* variants. They take another variant's selector
(`:hover`, `:invalid`, `:checked`, or an arbitrary one like `group-[.is-open]:`) and
attach it to a marker class in a relational selector: `:where(.group):hover *` for
ancestors, `:where(.peer):invalid ~ *` for earlier siblings. The `:where()` keeps the
marker class at zero specificity, so `group-hover:underline` has the same specificity as
`hover:underline` and doesn't win battles it shouldn't. Named versions just change the
marker to `.group\/item` or `.peer\/email`.

Since the compiler only does string work, there are no limits on which states can be
combined, and there's no JavaScript. The browser's normal selector matching does all of
it, which is also why the peer leak happens: `~` means exactly what CSS says it means.

Variant output is ordered in the stylesheet so that, for the same property, a stateful
class generally comes after the plain one, and a `md:` rule comes after the unprefixed
rule. You don't need `!important` to make `hover:bg-sky-700` beat `bg-sky-600`.

## Common mistakes

- **Putting `peer` after the element that uses it.** `~` only looks forward; the styles
  silently never apply.
- **Multiple unnamed peers in one parent.** Messages appear for the wrong field. Name
  them (`peer/email`) or wrap each field.
- **Nested `group`s without names.** Hovering the outer element triggers the inner one's
  styles.
- **Using `focus:` for button rings.** Mouse users see a ring after every click; use
  `focus-visible:`.
- **Showing errors with `invalid:` on load.** Empty required fields look broken before
  the user has typed. Prefer `user-invalid:`.
- **Hover-only affordances.** Since v4 they don't fire on touch-only devices, so the
  feature is simply hidden there.
- **Expecting `group` or `peer` to produce CSS.** They're markers; check the child's
  class for the rule.

## Exercise

1. Build a button with hover, `focus-visible` ring, active scale and disabled styles.
   Disable it and confirm hover no longer changes the colour (use `hover:not-disabled:`).
2. Build a card link where hovering anywhere on the card underlines the title and moves
   an arrow. Then nest a second card with its own hover effect inside it, see the
   crosstalk, and fix it with named groups.
3. Recreate the peer leak: two inputs and two messages as siblings in one `<form>`. Make
   the first invalid and confirm both messages show. Fix it twice: once with named peers,
   once by wrapping each field.
4. Build the toggle row. Tab to it with the keyboard and confirm the focus ring appears
   on the visible track.
