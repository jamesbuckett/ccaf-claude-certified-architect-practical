# Typography tightening — design

**Date:** 2026-04-19
**Scope:** `index.html` (single-file site)
**Type:** CSS-only refactor. No markup, no JS, no new dependencies.

## Goal

Tighten the site's typography and move it toward a modern product/tech aesthetic (direction "A1" from brainstorming). Two concrete rules:

1. **One display/body family.** System sans stack for every heading and every body surface. Drop Fraunces entirely.
2. **Mono only for code.** `JetBrains Mono` is used exclusively inside code-rendering surfaces. Every decorative use of mono (eyebrows, stats, brand marks, step counters, labels) becomes sans.

## Non-goals

- No layout, spacing, or color changes.
- No change to uppercase treatment, letter-spacing, or font weights on decorative labels — only the `font-family` declaration is stripped. Labels that were uppercased mono stay uppercased sans.
- No per-class weight or italic rebalancing beyond what the `.step-why` commit already did.
- No markup edits. The refactor touches only the `<style>` block and the single Google Fonts `<link>` URL.

## Changes

### 1. Webfont load (line 98)

Remove `Fraunces:opsz,wght@…` from the Google Fonts URL. Keep the JetBrains Mono fragment and the `preconnect` hints.

Before:

```html
<link href="https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@400;500;600;700&family=Fraunces:opsz,wght@9..144,400;9..144,500;9..144,600;9..144,700&display=swap" rel="stylesheet">
```

After:

```html
<link href="https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@400;500;600;700&display=swap" rel="stylesheet">
```

### 2. CSS tokens

- Remove the `--serif` custom property from `:root` (line ~131).
- `--sans` and `--mono` are unchanged.

### 3. Headings

- `h1, h2, h3, h4 { font-family: var(--serif); … }` → remove the `font-family` declaration so they inherit sans from `body`.
- `h4` currently has its own `font-family: var(--sans)` override — drop it too since the grouped rule now inherits sans. Keeps the CSS uniform.
- Keep letter-spacing `-0.02em` and weight 600. These read fine with system sans at display sizes.
- `.toc-title` drops serif → sans.

### 4. Strip `font-family: var(--mono)` from decorative classes

The following selectors currently set `font-family: var(--mono)` for non-code purposes. The `font-family` line is removed; other properties on those rules (size, weight, text-transform, letter-spacing, color) are preserved.

The authoritative enumeration is the output of `grep -n "font-family: var(--mono)"` in `index.html`; every hit is stripped except those listed in section 5 below. The list that follows is the expected set as of 2026-04-19:

- Brand: `.brand-name`, `.brand-slash`, `.brand-sub`
- Eyebrows / labels: `.toc-eyebrow`, `.eyebrow-label`, any other `.eyebrow-*` selector
- Hero: `.hero-stat-v`, `.hero-stat-l`, `.hero-author-role`, hero meta selectors
- TOC/nav labels in the left sidebar
- Step counter: `.steps > li::before`
- Callout: `.callout-head`
- Tables: `th`, `td.mono-cell`
- Code-block chrome: `.codeblock-lang`, `.copy-btn`, `.copy-label`
- Glossary: glossary term labels
- Footer: `.foot-meta` and any badge/metadata selectors in the footer
- Tutorial cards: any mono-set metadata

The implementation step will start from `grep -n "font-family: var(--mono)"` and process every hit, keeping only the ones listed in section 5.

### 5. Mono preserved (code-only)

These keep `font-family: var(--mono)` because they render code or terminal output:

- `code.inline` — inline code spans in prose
- `.codeblock pre` — fenced code blocks
- `.mono` utility class — escape hatch for incidental inline code-like content
- `.gloss-term code` — `<code>` elements inside glossary terms (the term *is* code)
- `.trouble-symptom` — renders terminal/error output (e.g., `ImportError: No module named 'anthropic'`), which the A1 principle explicitly treats as a code-rendering surface

Nothing else.

## Verification

1. `grep -n "var(--serif)"` — must return zero hits.
2. `grep -n "Fraunces"` — must return zero hits.
3. `grep -n "font-family: var(--mono)"` — remaining hits must only be `code.inline`, `.codeblock pre`, and `.mono`.
4. Open `index.html` in a browser and scan: hero, left TOC, a tutorial section, a callout, a table, a code block, footer. Repeat with dark mode toggled.
5. No console errors, no missing-font FOUT beyond system sans swap.

## Risks and notes

- **Step-number badges** currently read "code-y" in mono. In sans they'll look like regular counters. This is the intended direction but is the most visible single change.
- **Hero stats** lose monospaced-digit alignment. The numbers are short enough that rag shouldn't matter visually.
- **Dark mode** is unaffected — it only overrides colors, not font families.
- **Fraunces removal** shortens the Google Fonts request and drops one network dependency. Small perf win.

## Out of scope

- Any rework of the light/dark palette.
- Any rework of the spacing scale.
- Reducing weights beyond what the change naturally produces (still 400/500/600/700 as authored).
- Italic rebalancing — already handled in the `.step-why` commit.
