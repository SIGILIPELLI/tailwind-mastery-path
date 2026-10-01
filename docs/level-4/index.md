---
description: "Level 4 of Tailwind CSS: conventions and linting, multi-brand theming, RTL, media variants, legacy CSS, server templates, debugging, testing."
---

# Level 4 · Master <span class="level-badge">Production</span>

Goal: run Tailwind in production for the long haul — in a team, across brands and
languages, alongside CSS you didn't write, in server-rendered stacks as well as
JavaScript apps — and catch problems with tools and tests before users do.

By now you can build almost any interface with Tailwind. Level 4 is about everything
around the building:

1. **Make the right thing automatic.** Class sorting, lint rules, token packages and
   test suites remove whole categories of mistakes and review comments.
2. **Design for more than one environment.** Other brands, right-to-left languages,
   print, forced colours, touch screens and reduced motion are all conditions your UI
   will meet. Each lesson shows how to handle one and how to test it.
3. **Prove it.** Every lesson in this level ran its examples: real linter output, real
   test failures from deliberately broken code, measured positions and contrast ratios.
   The capstone ends by breaking its own product to show that its tests notice.

## Modules

1. [Conventions at Scale: Class Order, Linting & Review](01-conventions-at-scale.md) — Prettier sorting and ESLint rules, run on real code
2. [Multi-Brand & Runtime Theming](02-multi-brand-theming.md) — brand scopes, the `@theme inline` trap, runtime colours and readable text
3. [RTL, Logical Utilities & Internationalization](03-rtl-logical.md) — logical utilities measured in RTL, `rtl:` for icons, translated text
4. [Print, Contrast, Motion & Other Media Variants](04-media-variants.md) — every environment variant, tested under emulation
5. [Modern CSS Through Tailwind](05-modern-css-features.md) — gradient interpolation, text balancing, masks, 3D, anchor positioning
6. [Working with Existing CSS & Third-Party Components](06-interop-existing-css.md) — layer order for legacy CSS, prefix, Shadow DOM
7. [Tailwind with Server-Rendered Templates & Static Sites](07-server-templates.md) — a Jinja site, classes in backend code, the interpolation trap
8. [Debugging Tailwind](08-debugging.md) — three questions, a class-checking script, and a catalogue of real bugs
9. [Testing Tailwind UIs](09-testing.md) — rendered assertions and visual regression with Playwright
10. [Capstone — A SaaS Marketing Site & App Shell](10-capstone.md) — tokens, dark mode, RTL, print and a 24-test suite

## What you need before starting

- **Levels 1–3**, especially theme tokens (L2·01), layers (L3·02), accessibility (L3·04)
  and the component library project (L3·10), which lessons 09 and 10 extend.
- **Node.js and npm** for the tools: Prettier, ESLint, Playwright. Lesson 07 also uses
  Python with Jinja2; any server-side template language works for the ideas.
- **Playwright's browsers** (`npx playwright install chromium`) for lessons 09 and 10.

Versions used in this level: Tailwind 4.3.3, Prettier 3.9.9 with
prettier-plugin-tailwindcss 0.8.1, ESLint 10.11.0 with eslint-plugin-better-tailwindcss
4.7.0, @playwright/test 1.61.0 (Chromium 149), and Jinja2 3.1.6.
