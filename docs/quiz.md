---
title: "Final Quiz"
description: "Test what you have learned across all four levels of Tailwind CSS Mastery Path with a 20-question final quiz."
---

# Final Quiz

20 questions drawn from all four levels of this course. You need **70% (14 out of 20)** to pass —
you can retake it as many times as you like. Your best result is saved in this browser.

<div id="mp-quiz-root"></div>

<script type="application/json" id="mp-quiz-data">
[
  {
    "question": "What does this compile to?\n```html\n<div class=\"p-4 gap-2\">\n```\n",
    "options": [
      "padding: calc(var(--spacing) * 4); gap: calc(var(--spacing) * 2)",
      "padding: 4rem; gap: 2rem",
      "padding: 4px; gap: 2px",
      "padding: 1rem; gap: 0.5rem"
    ],
    "correctIndex": 0,
    "explanation": "Utilities multiply the --spacing theme variable, so the final size follows the theme. Verified by compiling it with Tailwind CSS v4.3 and checking the generated CSS."
  },
  {
    "question": "Which breakpoint rule does md:flex emit in v4?\n```html\n<div class=\"md:flex\">\n```\n",
    "options": [
      "@media (width >= 48rem) { .md\\:flex { display: flex",
      "@media (min-width: 768px)",
      "@media (max-width: 768px)",
      "@media screen and (width >= 768px)"
    ],
    "correctIndex": 0,
    "explanation": "Breakpoints are mobile-first and rem-based. Verified by compiling it with Tailwind CSS v4.3 and checking the generated CSS."
  },
  {
    "question": "How is dark mode applied by default?\n```html\n<div class=\"dark:bg-gray-900\">\n```\n",
    "options": [
      "@media (dark)",
      "&:where(.dark, .dark *)",
      "@media (prefers-color-scheme: dark)",
      "[data-theme=dark]"
    ],
    "correctIndex": 2,
    "explanation": "The default dark variant follows the OS preference; override it with @custom-variant for a manual toggle. Verified by compiling it with Tailwind CSS v4.3 and checking the generated CSS."
  },
  {
    "question": "What does w-[13px] produce?\n```html\n<div class=\"w-[13px]\">\n```\n",
    "options": [
      "width: var(--13px)",
      "width: 13",
      "width: 13px",
      "width: 13rem"
    ],
    "correctIndex": 2,
    "explanation": "Arbitrary values are emitted as written. Verified by compiling it with Tailwind CSS v4.3 and checking the generated CSS."
  },
  {
    "question": "What does bg-black/50 compile to?\n```html\n<div class=\"bg-black/50\">\n```\n",
    "options": [
      "rgba(0,0,0,.5) only",
      "background-opacity: 50",
      "opacity: 0.5",
      "background-color: color-mix(in oklab, var(--color-black) 50%, transparent)"
    ],
    "correctIndex": 3,
    "explanation": "The slash modifier sets the colour's alpha. Verified by compiling it with Tailwind CSS v4.3 and checking the generated CSS."
  },
  {
    "question": "What does hover:bg-red-500 do on touch devices?\n```html\n<div class=\"hover:bg-red-500\">\n```\n",
    "options": [
      "@media (hover: hover) { &:hover { background-color: var(--color-red-500)",
      "Applies on tap",
      "Never applies",
      "Always applies"
    ],
    "correctIndex": 0,
    "explanation": "v4 only applies hover styles where hover is supported. Verified by compiling it with Tailwind CSS v4.3 and checking the generated CSS."
  },
  {
    "question": "What do you put in a CSS file to start Tailwind v4?",
    "options": [
      "require('tailwindcss')",
      "@use tailwind;",
      "@tailwind base; @tailwind utilities;",
      "@import \"tailwindcss\";"
    ],
    "correctIndex": 3,
    "explanation": "One import replaces the three v3 directives."
  },
  {
    "question": "Why does `bg-${color}-500` fail to produce a style?",
    "options": [
      "It needs @apply",
      "It is deprecated",
      "Tailwind forbids variables",
      "The scanner looks for complete class names, so partial strings are not found"
    ],
    "correctIndex": 3,
    "explanation": "Map values to full class names or safelist them with @source inline()."
  },
  {
    "question": "When should you use @apply?",
    "options": [
      "Never",
      "Sparingly, for styling markup you cannot add classes to; prefer components for reuse",
      "For every component",
      "Always"
    ],
    "correctIndex": 1,
    "explanation": "Heavy @apply undermines utility-first."
  },
  {
    "question": "Where do design tokens live in v4?",
    "options": [
      "In CSS custom properties defined in @theme, which also generate utilities",
      "JSON",
      "tailwind.config.js only",
      "HTML"
    ],
    "correctIndex": 0,
    "explanation": "CSS-first configuration."
  },
  {
    "question": "How do you reuse a button style across an app?",
    "options": [
      "Copy the classes",
      "Extract a framework component or template partial and keep utilities there",
      "Global CSS",
      "Inline styles"
    ],
    "correctIndex": 1,
    "explanation": "One source of truth."
  },
  {
    "question": "Why use tailwind-merge with a component's class prop?",
    "options": [
      "Law",
      "Speed",
      "Conflicting utilities (p-2 vs p-4) are resolved by CSS order, not class order, so merging picks the intended one",
      "Colour"
    ],
    "correctIndex": 2,
    "explanation": "cn() = clsx + tailwind-merge."
  },
  {
    "question": "What does motion-reduce: let you do?",
    "options": [
      "Turn off or soften animations for users who prefer reduced motion",
      "Speed up",
      "Hide",
      "Preload"
    ],
    "correctIndex": 0,
    "explanation": "Accessibility."
  },
  {
    "question": "What makes a form control accessible?",
    "options": [
      "Placeholder only",
      "A real label tied to the input, visible focus styles and clear error text",
      "Colour only",
      "Icons"
    ],
    "correctIndex": 1,
    "explanation": "Utilities style, semantics inform."
  },
  {
    "question": "How should you keep utility class order consistent?",
    "options": [
      "By hand",
      "Alphabetically",
      "Use the official Prettier plugin to sort classes",
      "Random"
    ],
    "correctIndex": 2,
    "explanation": "Reduces noisy diffs."
  },
  {
    "question": "What does the container query variant @md: respond to?",
    "options": [
      "The viewport",
      "The screen",
      "The width of the nearest @container ancestor, not the viewport",
      "The device"
    ],
    "correctIndex": 2,
    "explanation": "Components adapt to the space they get."
  },
  {
    "question": "What does rtl: help with and what is better?",
    "options": [
      "Fonts",
      "Nothing",
      "Direction-specific overrides; prefer logical utilities such as ps-4 and ms-auto that adapt automatically",
      "Colours"
    ],
    "correctIndex": 2,
    "explanation": "Logical properties mirror for RTL."
  },
  {
    "question": "How do you theme several brands with the same components?",
    "options": [
      "Inline",
      "Separate builds",
      "Components use semantic tokens (such as bg-brand) whose variable values are switched per scope",
      "Copy components"
    ],
    "correctIndex": 2,
    "explanation": "CSS variables change at runtime."
  },
  {
    "question": "What should you check when a class does not apply?",
    "options": [
      "That it appears as a complete string in a scanned file, then inspect the generated rule in DevTools",
      "Reinstall",
      "Change browser",
      "Clear colours"
    ],
    "correctIndex": 0,
    "explanation": "Check @source and layer order."
  },
  {
    "question": "What shows a Tailwind design system is production-ready?",
    "options": [
      "Accessibility and visual-regression tests in CI, documentation and a versioned release process",
      "It looks good",
      "It builds",
      "Screenshots"
    ],
    "correctIndex": 0,
    "explanation": "Treat tokens as an API."
  }
]
</script>

## Coding challenges

Try these on your own computer, then compare your output. Nothing you write is sent anywhere.

### A responsive card: classes and the CSS they generate

Write the class list for a card (rounded, white, padded, shadowed; a row from the md breakpoint with a gap; gray hover and dark-mode backgrounds), then build it with the Tailwind v4 CLI and read the utilities layer of the generated CSS. The expected output below is the generated utilities layer from a real build; compare the order of rules and the shape of the variants.

**Starter code**

```css
/* input.css */
@import "tailwindcss" source(none);
@source "./index.html";

<!-- index.html: TODO - write the one-line class list for a card that
     - has extra-large rounded corners, a white background, 1.5rem padding (p-6) and a medium shadow
     - from the md breakpoint up is a row (flex) with vertically centred items and a 1.5rem gap (6 spacing steps)
     - gets a gray-50 background on hover and a gray-900 background in dark mode -->
<article class="TODO">
  ...
</article>

/* build */
npx @tailwindcss/cli -i input.css -o out.css

/* Then read the utilities layer of out.css: what rules did Tailwind generate for your classes, in what order? */
```

**Expected output**

```text
@layer utilities {
  .rounded-xl {
    border-radius: var(--radius-xl);
  }
  .bg-white {
    background-color: var(--color-white);
  }
  .p-6 {
    padding: calc(var(--spacing) * 6);
  }
  .shadow-md {
    --tw-shadow: 0 4px 6px -1px var(--tw-shadow-color, rgb(0 0 0 / 0.1)), 0 2px 4px -2px var(--tw-shadow-color, rgb(0 0 0 / 0.1));
    box-shadow: var(--tw-inset-shadow), var(--tw-inset-ring-shadow), var(--tw-ring-offset-shadow), var(--tw-ring-shadow), var(--tw-shadow);
  }
  @media (hover: hover) {
    .hover\:bg-gray-50:hover {
      background-color: var(--color-gray-50);
    }
  }
  @media (width >= 48rem) {
    .md\:flex {
      display: flex;
    }
    .md\:items-center {
      align-items: center;
    }
    .md\:gap-6 {
      gap: calc(var(--spacing) * 6);
    }
  }
  @media (prefers-color-scheme: dark) {
    .dark\:bg-gray-900 {
      background-color: var(--color-gray-900);
    }
  }
}
```

<details><summary>Show solution and explanation</summary>

<pre><code>/* input.css */
@import &quot;tailwindcss&quot; source(none);
@source &quot;./index.html&quot;;

&lt;!-- index.html --&gt;
&lt;article class=&quot;rounded-xl bg-white p-6 shadow-md md:flex md:items-center md:gap-6 hover:bg-gray-50 dark:bg-gray-900&quot;&gt;
  ...
&lt;/article&gt;

/* build */
npx @tailwindcss/cli -i input.css -o out.css</code></pre>

<p>Each class becomes one rule; variants wrap or qualify the selector (a media query for md:, :hover inside @media (hover: hover), and a prefers-color-scheme query for dark:). Verified by compiling the classes with Tailwind CSS v4.3 and printing the utilities layer.</p>
</details>

### Design tokens, a custom utility and a custom variant

Define theme tokens, a custom breakpoint and spacing base in @theme, add a custom utility with @utility and a custom variant with @custom-variant, then compile classes that use them. The expected output shows the generated utilities and the theme variables Tailwind kept because they were used.

**Starter code**

```css
/* input.css */
@import "tailwindcss" source(none);
@source "./index.html";

/* TODO: in an @theme block define
     --color-brand: oklch(0.62 0.19 255);  --color-brand-dark: oklch(0.45 0.17 255);
     --breakpoint-3xl: 120rem;             --spacing: 0.5rem;
   TODO: a custom utility text-pretty-balance that sets text-wrap: balance (@utility)
   TODO: a custom variant theme-night that matches [data-theme=night] and its descendants (@custom-variant)  */

<!-- index.html -->
<div class="bg-brand text-brand-dark p-4 3xl:p-8 theme-night:bg-brand-dark text-pretty-balance">...</div>

/* Then read out.css: which rules and which theme variables were generated, and what does p-4 become with the new --spacing? */
```

**Expected output**

```text
@layer utilities {
  .bg-brand {
    background-color: var(--color-brand);
  }
  .p-4 {
    padding: calc(var(--spacing) * 4);
  }
  .text-pretty-balance {
    text-wrap: balance;
  }
  .text-brand-dark {
    color: var(--color-brand-dark);
  }
  @media (width >= 120rem) {
    .\33 xl\:p-8 {
      padding: calc(var(--spacing) * 8);
    }
  }
  .theme-night\:bg-brand-dark:where([data-theme=night], [data-theme=night] *) {
    background-color: var(--color-brand-dark);
  }
}

/* theme variables emitted (only those used) */
--spacing: 0.5rem
--color-brand: oklch(0.62 0.19 255)
--color-brand-dark: oklch(0.45 0.17 255)
```

<details><summary>Show solution and explanation</summary>

<pre><code>/* input.css */
@import &quot;tailwindcss&quot; source(none);
@source &quot;./index.html&quot;;
@theme {
  --color-brand: oklch(0.62 0.19 255);
  --color-brand-dark: oklch(0.45 0.17 255);
  --breakpoint-3xl: 120rem;
  --spacing: 0.5rem;
}
@utility text-pretty-balance { text-wrap: balance; }
@custom-variant theme-night (&amp;:where([data-theme=night], [data-theme=night] *));

&lt;!-- index.html --&gt;
&lt;div class=&quot;bg-brand text-brand-dark p-4 3xl:p-8 theme-night:bg-brand-dark text-pretty-balance&quot;&gt;...&lt;/div&gt;</code></pre>

<p>Variables in @theme become real CSS custom properties and generate matching utilities (bg-brand, 3xl:p-8); @utility and @custom-variant add your own. Only the variables actually used are emitted, and p-4 is now calc(var(--spacing) * 4) with the 0.5rem base. Verified by compiling with Tailwind CSS v4.3.</p>
</details>
