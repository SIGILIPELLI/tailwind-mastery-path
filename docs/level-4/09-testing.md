---
description: "Test Tailwind UIs where bugs live: rendered contrast checks, focus in forced colours, overflow, and visual regression with Playwright."
---

# 09 · Testing Tailwind UIs

Tailwind moves styling into markup, which changes what testing has to cover. The classes
can be exactly right and the UI still wrong: a token change makes muted text unreadable,
a refactor removes a focus outline, a long translation overflows on phones. Those bugs
are only visible in the **rendered** page. This lesson adds browser tests to the Level 3
component library with Playwright, then deliberately breaks the design twice to show
what each kind of test catches and what it misses.

## Three layers of tests

| Layer | Checks | Speed | Catches |
|---|---|---|---|
| Unit (Level 3 · 10) | the class strings recipes return | milliseconds | broken variants, merge conflicts |
| Rendered assertions | computed styles in a real browser | ~0.1–0.5 s each | contrast, focus, overflow, media variants |
| Visual regression | screenshots compared with a baseline | ~1 s each | anything visible, including things you didn't think to assert |

You want a few of each. Unit tests keep recipes honest. Rendered assertions encode the
rules you care about most (contrast, focus, no overflow) as explicit, readable checks.
Screenshots are the safety net for everything else.

## Setup

In the `ui-kit` project from Level 3 · 10, which already builds `gallery/index.html` and
`gallery/ui.css`:

```bash
npm install -D @playwright/test
npx playwright install chromium
```

(We used `@playwright/test` 1.61.0.)

```js title="playwright.config.js"
import { defineConfig, devices } from "@playwright/test";

export default defineConfig({
  testDir: "e2e",
  use: { baseURL: `file://${process.cwd()}/gallery/` },
  projects: [
    { name: "desktop", use: { ...devices["Desktop Chrome"] } },
    { name: "phone", use: { ...devices["Pixel 7"] } },
  ],
});
```

Two projects run every test twice: once at desktop size and once with a phone profile
(narrow viewport, touch, mobile user agent). The tests load the gallery straight from
disk; for an app you'd add a `webServer` entry to start your dev server.

## A reusable contrast helper

The same measurement used throughout this course, packaged as a function that runs in
the page:

```js title="e2e/contrast.js"
// WCAG contrast of each element's text against its effective background.
// Runs in the page; ancestor backgrounds are painted onto a canvas so
// translucent layers blend the way they do on screen.
export function contrastOf(selector) {
  const c = document.createElement("canvas");
  c.width = c.height = 1;
  const x = c.getContext("2d", { willReadFrequently: true });
  const px = () => [...x.getImageData(0, 0, 1, 1).data].slice(0, 3);
  const lum = ([r, g, b]) => {
    const f = (v) => ((v /= 255) <= 0.04045 ? v / 12.92 : ((v + 0.055) / 1.055) ** 2.4);
    return 0.2126 * f(r) + 0.7152 * f(g) + 0.0722 * f(b);
  };
  return [...document.querySelectorAll(selector)].map((el) => {
    const layers = [];
    for (let e = el; e; e = e.parentElement) {
      const bg = getComputedStyle(e).backgroundColor;
      layers.push(bg);
      x.clearRect(0, 0, 1, 1); x.fillStyle = bg; x.fillRect(0, 0, 1, 1);
      if (x.getImageData(0, 0, 1, 1).data[3] === 255) break;
    }
    x.fillStyle = "#fff"; x.fillRect(0, 0, 1, 1);
    for (const l of layers.reverse()) { x.fillStyle = l; x.fillRect(0, 0, 1, 1); }
    const bg = px();
    x.fillStyle = getComputedStyle(el).color; x.fillRect(0, 0, 1, 1);
    const fg = px();
    const [hi, lo] = [lum(fg), lum(bg)].sort((a, b) => b - a);
    return { name: el.dataset.test, ratio: +((hi + 0.05) / (lo + 0.05)).toFixed(2) };
  });
}
```

## The tests

```js title="e2e/gallery.spec.js"
import { test, expect } from "@playwright/test";
import { contrastOf } from "./contrast.js";

for (const theme of ["light", "dark"]) {
  test.describe(`${theme} theme`, () => {
    test.beforeEach(async ({ page }) => {
      await page.emulateMedia({ reducedMotion: "reduce" });
      await page.goto("index.html");
      if (theme === "dark") {
        await page.evaluate(() => document.documentElement.classList.add("dark"));
      }
      // let colour transitions finish before measuring or screenshotting
      await page.waitForTimeout(400);
    });

    test("every variant meets 4.5:1 text contrast", async ({ page }) => {
      const results = await page.evaluate(contrastOf, "[data-test]");
      const failing = results.filter((r) => r.ratio < 4.5);
      expect(failing, JSON.stringify(failing)).toEqual([]);
    });

    test("matches the visual baseline", async ({ page }) => {
      await expect(page).toHaveScreenshot(`gallery-${theme}.png`, { fullPage: true });
    });
  });
}

test("no horizontal overflow", async ({ page }) => {
  await page.goto("index.html");
  const overflow = await page.evaluate(
    () => document.documentElement.scrollWidth - document.documentElement.clientWidth
  );
  expect(overflow).toBe(0);
});

test("keyboard focus is visible, also in forced colours", async ({ page }) => {
  await page.emulateMedia({ forcedColors: "active" });
  await page.goto("index.html");
  await page.keyboard.press("Tab");
  const focused = page.locator(":focus");
  await expect(focused).toHaveAttribute("data-test", "button-primary-sm");
  await expect(focused).toHaveCSS("outline-style", "solid");
  await expect(focused).toHaveCSS("outline-width", "2px");
});
```

Details that matter:

- **Reduced motion and a short wait** before measuring. Colour transitions were the
  cause of the false 1:1 contrast result in Level 3 · 10; tests have to wait for the
  final state.
- **Both themes** run every check.
- **The focus test uses the keyboard** (`Tab`), so it checks `:focus-visible` the way a
  user triggers it, and it runs with forced colours emulated, the mode where ring-based
  focus disappears (Level 3 · 04).
- The overflow test measures `scrollWidth - clientWidth`, the same check used for the
  projects in Levels 1 and 2.

## First run: creating baselines

```bash
npx playwright test --update-snapshots
```

```text
A snapshot doesn't exist at …/gallery-dark-desktop-darwin.png, writing actual.
…
12 passed (3.5s)
```

Four baseline images were written: light and dark, desktop and phone. Look at them
before committing; a baseline is only useful if it's correct. Note the `darwin` in the
file names: screenshots are platform-specific, because fonts and anti-aliasing differ
between operating systems. Generate and compare baselines on the same platform as your CI
(commonly by running the tests in the official Playwright Docker image), or they'll fail
on every run.

After that, a normal `npx playwright test` reported `12 passed (3.0s)`.

## Breaking it on purpose

**Regression 1: a "harmless" token change.** We changed `--radius-control` from
`0.5rem` to `0.25rem` and rebuilt the CSS. Desktop results:

```text
✓ light theme › every variant meets 4.5:1 text contrast
✘ light theme › matches the visual baseline
✓ dark theme › every variant meets 4.5:1 text contrast
✘ dark theme › matches the visual baseline
✓ no horizontal overflow
✓ keyboard focus is visible, also in forced colours

Error: expect(page).toHaveScreenshot(expected) failed
  64 pixels (ratio 0.01 of all image pixels) are different.
…
  88 pixels (ratio 0.01 of all image pixels) are different.
```

Only the screenshots noticed. No assertion was about radius, and none needed to be. The
number of changed pixels was tiny (just the corners of the controls), but any difference
fails by default. Playwright saves the expected, actual and a diff image in
`test-results/`, so you can see exactly what changed and either fix the code or accept
the change with `--update-snapshots`.

**Regression 2: a contrast problem.** We changed the light theme's `--color-fg-muted` from
`slate-600` to `slate-400`:

```text
✘ light theme › every variant meets 4.5:1 text contrast
✓ dark theme › every variant meets 4.5:1 text contrast

Error: [{"name":"hint","ratio":2.63}]
```

The assertion named the element (the input hint) and its ratio. A screenshot test would
also have failed, but its message would only say "pixels are different", and a reviewer
might accept a lighter grey as an intended design tweak. The explicit check says *why*
it's wrong.

That's the case for both kinds: screenshots catch the unexpected, assertions explain the
important.

## What these tests don't cover

- **Elements the helper doesn't select.** The contrast check only measures `[data-test]`
  elements; the input placeholders also use `fg-muted` and weren't checked. Extend the
  selector deliberately.
- **Text over images or gradients.** The helper reads background colours, not images.
- **Hover states** unless you hover in the test (`await locator.hover()`), and on the
  phone project hover styles don't apply at all (`@media (hover: hover)`).
- **Screen reader output.** Use an accessibility scanner (such as axe, via
  `@axe-core/playwright`) for automated checks, and still test with a real screen reader.
- **Other browsers.** Add Firefox and WebKit projects to the config if your users need
  them; rendering differs.

## Testing real apps

The same patterns work for a full application:

- Run tests against a **component gallery** (Storybook, Histoire, or a generated page as
  here). It's faster and more stable than full user flows.
- Use **stable selectors** (`data-test`, roles, labels), never Tailwind classes. Classes
  change whenever the design does.
- **Mask dynamic content** in screenshots (`toHaveScreenshot({ mask: [locator] })`) for
  dates, avatars and random data.
- Keep screenshot tests **few and focused**: a gallery per component plus a handful of
  key pages. Hundreds of full-page screenshots become noise that people approve without
  looking.

## How It Actually Works

Playwright drives a real browser engine, so every test sees what users see: the actual
cascade, computed variable values, media query results and fonts. `emulateMedia`
changes what the engine reports for `prefers-reduced-motion`, `forced-colors`,
`prefers-color-scheme` and print, so the corresponding Tailwind variants are exercised
for real. `toHaveScreenshot` captures the page, waits until two consecutive captures are
identical (to avoid catching animations mid-frame), and compares the pixels with the
stored baseline, failing if more pixels differ than the allowed threshold (zero by
default).

## Common mistakes

- **Asserting on class names** (`toHaveClass("bg-sky-700")`) instead of rendered results.
  The class can be right and the result wrong.
- **Measuring before transitions finish.**
- **Baselines created on one OS and compared on another.**
- **Updating snapshots without looking** at the diff.
- **Thousands of screenshots** instead of a focused gallery.
- **Treating automated checks as a full accessibility audit.**

## Exercise

1. Add the Playwright setup to your Level 3 component library and get all tests passing.
2. Reproduce both regressions from this lesson and read Playwright's diff images.
3. Extend the contrast check to cover placeholders (compute their colour with
   `getComputedStyle(el, "::placeholder")`).
4. Add a test that hovers a primary button on the desktop project and checks that its
   background changes.
5. Add `@axe-core/playwright` and run an accessibility scan of the gallery in both
   themes.
