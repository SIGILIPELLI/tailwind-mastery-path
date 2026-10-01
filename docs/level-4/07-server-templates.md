---
description: "Tailwind with server-rendered templates: scanning Jinja, Django, Rails and Laravel templates, classes in backend code, and the CLI workflow."
---

# 07 · Tailwind with Server-Rendered Templates & Static Sites

Not every project is a JavaScript single-page app. Django, Rails, Laravel, Flask, Go and
static site generators render HTML on the server, and they work very well with Tailwind:
there's no client-side framework to integrate, just templates for the compiler to scan
and a CSS file to serve. This lesson builds a small Jinja2 site with the Tailwind CLI and
uses it to show the decisions that matter in a server-rendered setup: what to scan,
where class strings should live, how to reuse markup, and the one mistake that template
languages make especially easy.

## The setup

```text
site/
├── app/status.py              (data + class strings, Python)
├── templates/
│   ├── base.html
│   ├── macros.html
│   └── posts.html
├── static/src/app.css         (Tailwind entry)
├── static/app.css             (generated, git-ignored)
├── dist/                      (rendered HTML, git-ignored)
└── build.py
```

```css title="static/src/app.css"
@import "tailwindcss" source(none);
@source "../../templates";
@source "../../app";
```

`source(none)` turns off automatic detection, and the two `@source` lines say exactly
what to scan: the templates and the Python package. That matters more in server
projects than in front-end ones, because the project folder often contains large
things you don't want scanned: virtual environments, uploaded media, database dumps,
log files. (Automatic detection skips git-ignored paths, but an explicit list is
clearer and faster.) Paths are relative to the CSS file.

Run the CLI in watch mode next to your development server:

```bash
npx @tailwindcss/cli -i static/src/app.css -o static/app.css --watch
```

and with `--minify` in your deploy step. If your team doesn't use Node.js at all,
Tailwind also publishes a standalone CLI executable that needs no Node installation
(Level 1 · 02).

## Classes in backend code

Status badges, alert types and table row states are usually decided in backend code.
Keep the **full class strings** in that code, next to the data:

```python title="app/status.py"
# Full class strings live next to the data they style, so the scanner finds them.
STATUS_CLASSES = {
    "draft": "bg-amber-50 text-amber-900 ring-amber-300",
    "published": "bg-emerald-50 text-emerald-800 ring-emerald-300",
    "archived": "bg-slate-100 text-slate-700 ring-slate-300",
}

POSTS = [
    {"title": "Release notes for 2.4", "status": "published"},
    {"title": "Migrating the billing service", "status": "draft"},
    {"title": "Old onboarding guide", "status": "archived"},
]
```

The scanner reads `.py` files like any other text, so `bg-amber-50`, `text-emerald-800`
and the rest are found. In our build, all of them were in the generated CSS.

A template macro renders the badge:

```html title="templates/macros.html"
{% macro badge(status, classes) -%}
<span class="inline-flex rounded-full px-2 py-0.5 text-xs font-medium ring-1 ring-inset {{ classes[status] }}">
  {{ status|capitalize }}
</span>
{%- endmacro %}
```

```html title="templates/posts.html"
{% extends "base.html" %}
{% from "macros.html" import badge %}
{% block content %}
<h1 class="text-2xl font-bold tracking-tight">Posts</h1>
<ul class="mt-6 divide-y divide-slate-200">
  {% for post in posts %}
  <li class="flex items-center justify-between gap-4 py-3">
    <span class="font-medium">{{ post.title }}</span>
    {{ badge(post.status, status_classes) }}
  </li>
  {% endfor %}
</ul>
{# Broken on purpose: the scanner only sees "border-" and "-500" #}
<p class="mt-6 border-l-4 border-{{ accent }}-500 ps-3 text-sm">Accent note</p>
{% endblock %}
```

```html title="templates/base.html"
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>{% block title %}Posts{% endblock %}</title>
  <link rel="stylesheet" href="/static/app.css">
</head>
<body class="bg-white text-slate-900 antialiased">
  <main class="mx-auto max-w-2xl px-4 py-10">{% block content %}{% endblock %}</main>
</body>
</html>
```

```python title="build.py"
from pathlib import Path
from jinja2 import Environment, FileSystemLoader, select_autoescape
from app.status import STATUS_CLASSES, POSTS

env = Environment(loader=FileSystemLoader("templates"), autoescape=select_autoescape())
html = env.get_template("posts.html").render(posts=POSTS, status_classes=STATUS_CLASSES, accent="sky")
Path("dist").mkdir(exist_ok=True)
Path("dist/index.html").write_text(html)
print("wrote dist/index.html")
```

(In a Django or Flask app, the view does what `build.py` does here: renders a template
with a context.)

## The template-expression trap

`posts.html` contains a deliberate mistake:

```html
<p class="mt-6 border-l-4 border-{{ accent }}-500 ps-3 text-sm">Accent note</p>
```

After running the build:

```text
rendered HTML:   class="mt-6 border-l-4 border-sky-500 ps-3 text-sm"
generated CSS:   .border-l-4 present, .border-sky-500 missing
```

The rendered page *contains* `border-sky-500`, but the compiler scanned the template,
where it only saw `border-{{ accent }}-500`. This is the same problem as JavaScript
template strings (Level 3 · 01), and template languages make it very tempting:
`bg-{{ color }}-100`, `text-<%= size %>`, `col-span-{{ n }}`. The fixes are the same:

- **Map values to full class strings in code** (like `STATUS_CLASSES`) or in the
  template (`{% if accent == "sky" %}border-sky-500{% elif … %}`).
- **Use a CSS variable for truly dynamic values**:
  `style="--accent: {{ accent_hex }}" class="border-(--accent)"`.

Don't "fix" it by adding the rendered output folder (`dist/`, or a page cache) to
`@source`. It seems to work, but only after a page has been rendered once, the CSS then
depends on which pages happened to be built, and output folders can be huge.

## Reusing markup: partials, macros and components

The extraction rule from Level 2 · 05 applies to templates: reuse markup with the
template language's own tools, not with `@apply`.

| Stack | Reuse mechanism |
|---|---|
| Jinja / Django | macros (`{% macro %}`), `{% include %}` with `with` context, Django inclusion tags |
| Rails | partials (`render "shared/badge", status:`), ViewComponent |
| Laravel | Blade components (`<x-badge :status="$status" />`) |
| Go | `{{ template "badge" . }}` blocks |
| Static generators | includes / shortcodes (Hugo partials, Eleventy shortcodes) |

Each one keeps the classes in one file. For classes that must be adjustable by the
caller, pass extra classes as a parameter and append them, keeping overrides to layout
utilities (there's no tailwind-merge in most server languages, so avoid passing
conflicting utilities).

## Markdown and CMS content

Content from Markdown or a CMS has no classes. Style it with the Typography plugin
(Level 3 · 05):

```html
<article class="prose prose-slate max-w-none">{{ post.body_html|safe }}</article>
```

Only mark HTML as safe when it comes from a trusted source or has been sanitised; a
`prose` class doesn't make untrusted HTML safe.

## Serving the CSS

- **Cache busting:** frameworks have their own asset pipelines (Django's
  `ManifestStaticFilesStorage`, Rails' asset digests, Laravel's Vite integration). Let
  them fingerprint `static/app.css`. Without one, use a hashing script like the one in
  Level 3 · 09.
- **Don't commit the generated CSS**, unless your deployment can't run the build. If you
  must commit it, generate it in CI and fail the build when it's out of date.
- **One stylesheet** for the site is normally best.

## How It Actually Works

The compiler doesn't know or care what language a file is in. It reads each scanned file
as text and extracts class-shaped strings. In a Jinja template, `class="flex
items-center"` yields `flex` and `items-center`; `{{ classes[status] }}` yields nothing
useful, but that's fine because the values of `classes` are written in full in
`app/status.py`, which is also scanned. Template syntax around a class name (`{%`, `}}`,
`<%=`) breaks the string, which is exactly why `border-{{ accent }}-500` produces the
fragments `border-` and `-500` rather than a class.

The template engine and Tailwind never interact. The engine assembles HTML at request
time; Tailwind generated the CSS earlier, at build time, from the source files.

## Common mistakes

- **Building class names with template expressions** (`bg-{{ color }}-100`).
- **Scanning the rendered output folder** to work around that.
- **Scanning everything**, including virtual environments and uploads. Use
  `source(none)` and list your template and code folders.
- **Forgetting backend files** that contain class strings, so those classes are missing.
- **`@apply` component classes** instead of macros/partials/components.
- **Committing a stale generated CSS file.**
- **Marking untrusted HTML as safe** because it's styled with `prose`.

## Exercise

1. Recreate this project (or add Tailwind to an existing Django/Flask/Rails/Laravel
   app). Run the build and confirm the badge classes from backend code are generated.
2. Reproduce the `border-{{ accent }}-500` bug, then fix it twice: with a mapping, and
   with a CSS variable.
3. Turn the badge macro into your framework's component mechanism, with an optional
   extra-classes parameter.
4. Add the CLI to your framework's dev command so it watches alongside the server, and to
   the deploy step with `--minify`.
