---
description: "Motion in Tailwind v4: transitions, the separate translate/scale/rotate properties, entry animations with starting:, and reduced motion."
---

# 09 · Transitions, Transforms & Animation

Motion makes an interface feel responsive: a button that darkens smoothly, a menu that
fades in, a card that lifts on hover. Tailwind's motion utilities are thin wrappers
around CSS transitions, transforms and animations, but v4 changed how transforms work
under the hood, and that change explains a bug you'll probably meet if you mix Tailwind
with hand-written CSS. This lesson covers the utilities, the change, entry animations
with `starting:`, your own keyframes, and respecting users who turn motion off.

## Transitions

A transition animates a property from its old value to its new one whenever it changes.
You need two things: something that changes (usually a variant like `hover:`) and a
`transition-*` utility that says which properties to animate:

```html
<a class="rounded-md bg-sky-700 px-4 py-2 text-white transition-colors duration-200
          hover:bg-sky-800">Continue</a>
```

| Utility | `transition-property` |
|---|---|
| `transition` | a sensible default list: colours, `opacity`, `box-shadow`, `transform`, `translate`, `scale`, `rotate`, filters, `display`, … |
| `transition-colors` | colour properties only (`color`, `background-color`, `border-color`, `fill`, …) |
| `transition-opacity` | `opacity` |
| `transition-transform` | `transform, translate, scale, rotate` |
| `transition-shadow` | `box-shadow` |
| `transition-all` | `all` |
| `transition-[height]` | whatever you put in brackets |

Duration, easing and delay are separate: `duration-200`, `ease-out`, `delay-150`. Without
them, transitions use the theme defaults, `--default-transition-duration: 150ms` and
`--default-transition-timing-function: cubic-bezier(0.4, 0, 0.2, 1)`.

Prefer the narrow utilities. `transition-all` animates *every* property that changes,
including ones you didn't intend (a width that changes at a breakpoint, a colour change
from switching themes), and makes the browser check every property on every change.

## Transforms in v4: separate properties

In v3, `translate-x-4`, `scale-95` and `rotate-45` all wrote to the one `transform`
property. v4 uses the **individual** CSS properties `translate`, `scale` and `rotate`:

```css
.translate-x-4 {
  --tw-translate-x: calc(var(--spacing) * 4);
  translate: var(--tw-translate-x) var(--tw-translate-y);
}
.scale-95 { /* --tw-scale-x/y/z: 95% */ scale: var(--tw-scale-x) var(--tw-scale-y); }
.rotate-45 { rotate: 45deg; }
```

(Skews and 3D rotations like `rotate-x-12` still use `transform`, because there are no
individual properties for them.)

The independent properties compose cleanly: a `hover:scale-105` doesn't wipe out a
`-translate-y-1/2` on the same element, because they're different properties.

### The bug this causes

If you write your own CSS with `transition-property: transform`, a Tailwind `scale-*`
change **won't animate**, because the property that changed is `scale`, not `transform`.
We tested two elements, one with `transition-[transform]` and one with
`transition-transform`, and added `scale-150` to both. 100ms later:

```text
transition-[transform]   running animations: 0   scale: 1.5       (jumped instantly)
transition-transform     running animations: 1   scale: 1.01293   (animating)
```

Use `transition-transform` (or `transition`), which lists all four properties. The same
applies to any third-party CSS or JavaScript animation library that assumes Tailwind
uses `transform`.

## Easing that feels right

- **`ease-out`** (fast start, slow end) for things **entering**: menus, toasts,
  dialogs. They respond instantly and settle gently.
- **`ease-in`** for things **leaving**.
- **`ease-in-out`** for things moving from one place to another on screen.
- Keep UI transitions short: roughly 100–300ms. Longer feels sluggish for something the
  user triggered. Big, decorative motion can be longer.

Custom curves go in the theme as `--ease-*` tokens (lesson 01), e.g.
`--ease-snappy: cubic-bezier(0.2, 0, 0, 1);` gives `ease-snappy`.

## Entry animations with `starting:`

Transitions only run when a value *changes*. An element that appears (inserted into the
DOM, or switched from `display: none`) has no "before" value, so it just pops in. CSS's
`@starting-style` provides that before value, and Tailwind exposes it as `starting:`:

```html
<div class="opacity-100 transition-opacity duration-300 starting:opacity-0">…</div>
```

We tested two hidden elements, both `opacity-100 transition-opacity duration-1000`, one
with `starting:opacity-0`, and switched both from `hidden` to `block`. 100ms later:

```text
with starting:opacity-0     running animations: 1   opacity: 0.0196   (fading in)
without                     running animations: 0   opacity: 1        (popped in)
```

To animate an element **out** to `display: none` as well (popovers, dialogs), the
transition must include `display`, and the browser must be allowed to transition a
discrete property, which is what `transition-discrete` (`transition-behavior:
allow-discrete`) does:

```html
<div id="menu" popover
     class="rounded-lg bg-white p-2 shadow-lg ring-1 ring-black/5
            opacity-0 scale-95 transition transition-discrete duration-150 ease-out
            open:opacity-100 open:scale-100
            starting:open:opacity-0 starting:open:scale-95">
  …
</div>
<button popovertarget="menu">Options</button>
```

`open:` matches `:popover-open` (and `[open]` for `<details>`/`<dialog>`), and
`starting:open:` is the state it starts from on the way in. For the exit to animate,
`display` and `overlay` (which keeps a popover in the top layer while it fades out) must
be in the transition list. The default `transition` utility's list in 4.3.3 includes
both, along with `opacity` and `scale`, so it covers this case without
`transition-all`. Browser support for `@starting-style` and discrete transitions is
recent; without it, the menu simply appears and disappears instantly, which is a fine
fallback.

## Animations and keyframes

Four animations ship in the default theme:

| Utility | Defined as | Typical use |
|---|---|---|
| `animate-spin` | `spin 1s linear infinite` | Loading spinners |
| `animate-ping` | `ping 1s cubic-bezier(0, 0, 0.2, 1) infinite` | Notification dots |
| `animate-pulse` | `pulse 2s cubic-bezier(0.4, 0, 0.6, 1) infinite` | Skeleton screens |
| `animate-bounce` | `bounce 1s infinite` | "Scroll down" hints |

Your own go in `@theme`, with the keyframes alongside (emitted only if the animation is
used):

```css title="src/app.css"
@theme {
  --animate-slide-up: slide-up 0.4s var(--ease-out) both;
  @keyframes slide-up {
    from { opacity: 0; translate: 0 0.75rem; }
    to { opacity: 1; translate: 0 0; }
  }
}
```

```html
<li class="animate-slide-up">…</li>
<li class="animate-slide-up [animation-delay:60ms]">…</li>
<li class="animate-slide-up [animation-delay:120ms]">…</li>
```

`both` keeps the `from` state during the delay and the `to` state afterwards. Note that
`delay-*` utilities set `transition-delay`, not `animation-delay`, which is why the
stagger uses arbitrary properties.

## Reduced motion

Some people get dizzy or nauseous from motion, and operating systems have a "reduce
motion" setting. Respect it. Two variants map to `prefers-reduced-motion`:

- `motion-reduce:` applies when the user asked for less motion.
- `motion-safe:` applies only when they **haven't**.

```html
<!-- Opt out: animate by default, stop for reduced motion -->
<svg class="size-5 animate-spin motion-reduce:animate-none" …></svg>

<!-- Opt in: only animate when it's known to be OK -->
<div class="motion-safe:transition motion-safe:hover:-translate-y-1">…</div>
```

`motion-safe:` (opt in) is the safer default for decorative motion like hover lifts and
entrance animations. For a spinner, stopping it entirely can make the page look frozen.
Replace it with something static that still communicates "loading", for example
`motion-reduce:animate-none` plus visible "Loading…" text.

Reduced motion doesn't mean *no* motion. Fades and colour changes are usually fine; it's
large movement, zooming and parallax that cause problems.

## Worked example: a card that lifts on hover

```html
<a href="/guides/containers"
   class="group block rounded-xl border border-gray-200 bg-white p-5
          motion-safe:transition motion-safe:duration-200 motion-safe:ease-out
          hover:border-gray-300 hover:shadow-lg motion-safe:hover:-translate-y-0.5
          focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-sky-600">
  <h3 class="font-semibold text-gray-900">Container queries</h3>
  <p class="mt-1 text-sm text-gray-600">Make components respond to their own width.</p>
  <span class="mt-3 inline-flex items-center gap-1 text-sm font-medium text-sky-700">
    Read guide
    <span aria-hidden="true" class="motion-safe:transition-transform
                                    motion-safe:group-hover:translate-x-0.5">→</span>
  </span>
</a>
```

The border and shadow change for everyone (they're not motion). The lift and the arrow
nudge only happen for users who haven't asked for reduced motion. Animating
`translate` and `box-shadow` is cheap; avoid animating `width`, `height`, `top` or
`margin`, which force layout on every frame.

## How It Actually Works

A transition is a rule the browser checks whenever a computed style changes: if the
property is listed in `transition-property`, it interpolates between the old and new
values over the duration. Tailwind's `duration-*` and `ease-*` utilities set both the
property and a `--tw-duration` / `--tw-ease` variable, and the `transition` utilities
read `var(--tw-duration, var(--default-transition-duration))`. That's why
`duration-300 transition` works in either class order.

The transform utilities set `--tw-translate-x`, `--tw-scale-x` and similar variables and
then the individual `translate` / `scale` properties, so combining `translate-x-4` and
`-translate-y-1/2` fills both variables of one `translate` value. The variables are
registered with `@property` (you can see the `@property --tw-scale-x` blocks in any
compiled file) with `inherits: false` and an initial value, so a parent's
`--tw-scale-x` never leaks into a child that has its own transform utilities.

`@starting-style` defines the styles an element has "before" its first style
computation. When the element first renders (or leaves `display: none`), the browser
transitions from the starting style to the normal style.

## Common mistakes

- **`transition-property: transform` in your own CSS** for Tailwind `scale-`/`rotate-`/
  `translate-` changes. Use `transition-transform`.
- **`transition-all` everywhere**, which animates things you didn't mean to.
- **Using `delay-*` for an animation.** It sets `transition-delay`; use
  `[animation-delay:…]`.
- **Expecting an element to animate in from `display: none`** without `starting:`.
- **Animating layout properties** (`width`, `height`, `top`) for hover effects.
- **Ignoring reduced motion**, or stopping a spinner with nothing to replace it.
- **Long durations on UI feedback.** Keep them under about 300ms.

## Exercise

1. Build a button with `transition-colors` and a hover colour, and try different
   durations until it feels instant but smooth.
2. Recreate the `transition-[transform]` bug with `hover:scale-105`, then fix it.
3. Build the popover menu and check it fades in and out. Then test it in a browser
   without `@starting-style` support (or temporarily remove the `starting:` classes)
   and confirm it still works.
4. Add a custom `slide-up` animation with a staggered delay to a list of five items.
5. Turn on "reduce motion" in your operating system and check every animation on your
   page.
