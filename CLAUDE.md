# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Scott R. Prelewicz's personal site: a static two-page site (`index.html` landing page, `resume.html` full résumé) with no build step, no package manager, and no test suite. There is no server-side code — everything renders client-side in the browser.

## Running it

There's no dev server or build command. Open `index.html` / `resume.html` directly in a browser, or serve the directory statically, e.g.:

```
python3 -m http.server
```

There is no linter or test command configured for this repo (the `oxlintrc.json` under `styles/` is metadata for the vendored design system, not a project lint config — see below).

## Architecture

### The `dc` component runtime (`support.js`)

`support.js` is a **generated, vendored runtime** — its header says `GENERATED from dc-runtime/src/*.ts — do not edit`. The `dc-runtime` TypeScript source isn't part of this repo, so treat `support.js` as a black box: never hand-edit it.

Both HTML pages follow the same pattern the runtime expects:

- The whole page body is wrapped in `<x-dc>...</x-dc>`.
- A `<helmet>` block inside it holds everything that belongs in `<head>` (title, meta tags, JSON-LD, the page's inline `<style>` token overrides) — the runtime hoists this into the real document head at boot.
- Template expressions use `{{ expr }}` interpolation against a `vals` object returned by the page's component.
- Conditional rendering uses `<sc-if value="{{ cond }}" hint-placeholder-val="{{ staticFallback }}">...</sc-if>`. The `hint-placeholder-val` is the value shown while the page is streaming/hydrating, before real state is available — always keep it in sync with a sensible default.
- List rendering uses `<sc-for list="{{ array }}" as="item" hint-placeholder-count="N">...</sc-for>`.
- Each page ends with `<script type="text/x-dc" data-dc-script data-props="...">` containing `class Component extends DCLogic { ... }`. This is the page's actual logic: `state`, data arrays (e.g. `workCards`, `faqs`), lifecycle methods (`componentDidMount`/`componentWillUnmount`), event handlers, and a `renderVals()` method that returns the `vals` object the template's `{{ }}` expressions read from.
- The `data-props` JSON attribute declares the page's editable props (with `editor` type, `default`, and `section`) — e.g. `defaultTheme`, `showSelectedWork`, `heroPhotoGrayscale`, `resumeUrl` on `index.html`.

To change page content or behavior, edit the `class Component extends DCLogic` block (and its matching template markup) in `index.html` or `resume.html` directly — there's no separate data/config file.

### Styling (`styles/`)

- `styles/styles.css` is the only stylesheet, from a vendored design system called "Modernist" (flat, architectural, near-mono red-on-white, zero border-radius, Archivo type, strong 2px dividers — see `styles/readme.md` for the full design language and component class reference). Both pages link it and pull all color/spacing/type values from its CSS custom properties (`--color-*`, `--space-*`, etc.) plus a small set of page-local `:root` variables defined inline in each HTML file's `<style>` block (`--bg`, `--accent`, `--link`, etc., with `[data-theme="dark"]` overrides for dark mode).
- `styles/_ds_bundle.js` and `styles/_ds_manifest.json` are tooling artifacts from the design-system generator (component registry/manifest), not code to edit by hand.
- `styles/_adherence.oxlintrc.json` is a lint-rule spec (no raw hex colors, no raw `px` values, no non-Archivo fonts) belonging to that same external design-system tool — it isn't wired up to run in this repo.
- `styles/readme.md` documents the full Modernist design system, including component pages (`components/*.html`), foundation pages (`foundations/*.html`), and `theme.json`/`templates/` — **none of those referenced files exist in this repo**; only `styles.css` and this readme were vendored in. Use the readme for the design vocabulary (tokens, component class names, "do/don't" rules) but don't expect the referenced example pages to be present.

### Theming

Both pages support light/dark mode via a `data-theme` attribute on the root wrapper div, toggled at runtime (`toggleTheme` in the `Component` class) and initialized from the `defaultTheme` prop. Dark-mode values live in a `[data-theme="dark"]` CSS block alongside the light defaults in each page's inline `<style>`.

### Content notes

- `assets/scott-photo.jpg` and `assets/Scott-Prelewicz-Resume.pdf` are referenced directly by path from `index.html`/`resume.html`.
- Structured data (`application/ld+json` for `Person` and `FAQPage`) in `index.html`'s `<helmet>` is duplicated content from the FAQ list in the `Component` class — keep both in sync if editing FAQ copy.
