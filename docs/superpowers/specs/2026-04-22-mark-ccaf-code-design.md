# Mark CCAF code in tutorial examples — design

**Date:** 2026-04-22
**Scope:** `index.html` only. `study-guide.html` has no code examples.

## Problem

Every Python tutorial example in `index.html` mixes two kinds of code: lines that teach the Anthropic Claude API / agentic-loop pattern (the content the CCA-F exam actually tests) and lines that are generic application plumbing a learner would replace with their own code (fake tool bodies, debug prints, user prompt strings). Today there is no visual signal distinguishing them, so a learner scanning a ~50-line file cannot tell at a glance which lines carry exam-relevant knowledge.

## Goal

Give every tutorial code block a consistent, low-noise visual mark that separates **CCAF code** (Claude API surface + agentic-loop scaffold) from **application logic** (demo plumbing), so a learner can skim a file and immediately see which lines are the cert content.

## Non-goals

- No change to `study-guide.html`.
- No JavaScript-based auto-detection. Lines are marked by hand once; the set changes rarely.
- No interactive toggle to hide/show application logic.
- No new block-level variant beyond Python per-line stripes and a header badge for pure-protocol blocks.

## Design

### Visual treatment

**Per-line stripe** on Python code blocks. A thin coloured left-gutter bar on each CCAF line, following the same structural pattern as the existing `.added` class but in a distinct colour so the two marks never get confused when they co-occur.

**Header badge** on code blocks whose content is 100% Claude protocol output (the `response` and `messages state` block types). A small uppercase `CCAF` pill in the codeblock header. No per-line marks inside — the whole block is CCAF by its nature.

**Nothing** on `bash` blocks. Install and shell commands are neither CCAF nor application logic.

### Colour

A new CSS custom property `--ccaf-stripe` holding a muted indigo (approximately `#6366F1` in light mode, a lighter variant in dark mode). Chosen because the existing `--accent` carries the "changed line" meaning on `.added`; using a second distinct hue keeps the two axes independent.

### Marking rules (the "Broad" rule)

A Python line is marked as CCAF if it touches the Claude / Anthropic API surface **or** the agentic-loop scaffold:

- SDK import and client init: `import anthropic`, `client = anthropic.Anthropic()`
- Model and iteration constants: `MODEL = "claude-sonnet-4-6"`, `MAX_ITERATIONS = N`
- Tool catalogue: the entire `tools = [{...}]` structure including `name`, `description`, `input_schema`
- Message construction: `messages = [...]`, any `messages.append({"role": ..., "content": ...})`
- API call: `client.messages.create(...)` and each argument line
- Response inspection: `resp.stop_reason`, `resp.content`, `block.type == "tool_use"`, `block.id`, `block.name`, `block.input`
- `tool_result` construction: the dict with `type: "tool_result"`, `tool_use_id`, and `content`
- Loop scaffold: `for turn in range(MAX_ITERATIONS):`, `if resp.stop_reason == "end_turn":`, `break`, the `for/else` safety branch

A Python line is **not** marked (application logic) if it is:

- The definition or body of a user-supplied tool function (e.g. the fake weather dict inside `run_tool`)
- User prompt text content (e.g. the literal `"Compare the weather…"` string)
- Debug `print(...)` calls
- Blank lines

**Boundary case:** `result = run_tool(block.name, block.input)` is marked. The *call* sits on the CCAF side of the boundary because it dispatches using fields from the API response. The *definition* of `run_tool` itself is not marked.

### Legend

A one-line inline key appears **once**, immediately before the first Python code block of Tutorial 01:

> ▎ Lines with a coloured left-stripe are **Claude API / agentic-loop code** (CCA-F exam content). Unmarked lines are application plumbing you'd swap out for your own logic.

No legend repeats elsewhere — once per page is enough.

## Implementation

### CSS

Add a new custom property and two classes to the existing `<style>` block:

```css
:root {
  --ccaf-stripe: #6366F1;
}

.codeblock pre .ccaf-line {
  display: block;
  box-shadow: inset 2px 0 0 var(--ccaf-stripe);
  padding-left: var(--s-2);
  margin-left: calc(-1 * var(--s-2));
}

.codeblock-badge {
  display: inline-flex;
  align-items: center;
  font-size: 0.7rem;
  font-weight: 600;
  letter-spacing: 0.06em;
  text-transform: uppercase;
  padding: 2px 8px;
  border: 1px solid var(--border);
  border-radius: 999px;
  color: var(--text-muted);
  margin-left: var(--s-2);
}
```

A dark-mode override for `--ccaf-stripe` is added alongside existing dark-mode variables.

### HTML

- For each of the 22 Python `<pre>` blocks, wrap each CCAF line in `<span class="ccaf-line">…</span>`. One span per logical source line. Leading whitespace stays inside the span so indentation is preserved.
- For each of the 3 pure-protocol blocks (2 `response`, 1 `messages state`), insert `<span class="codeblock-badge">CCAF</span>` inside the `.codeblock-head` after the `.codeblock-lang` element.
- Insert the legend line once, immediately before the first Python `.codeblock` in Tutorial 01.

### Copy-paste safety

The existing copy button uses `pre.textContent`, which strips HTML tags. Wrapping lines in `<span>` does not alter what learners get when they click copy. Verified at `index.html:4994`.

### Print

`.codeblock` already has print styles. The inset-box-shadow on `.ccaf-line` degrades gracefully in print (visible as a faint left edge on most browsers). No additional print rules needed.

## Files affected

- `index.html` — CSS additions, span wrappers on Python lines, badges on protocol blocks, legend insertion. Single file, single commit.

## Verification

After implementation, a reader should be able to:

1. Open `index.html` and see the legend once near Tutorial 01.
2. Scroll through any Python code block and visually separate CCAF lines from application plumbing without reading the code.
3. See a `CCAF` header badge on every response and messages-state block.
4. See no marks on bash blocks.
5. Copy any code block and paste it into an editor with no `<span>` artefacts.
6. Print the page and still see stripes on CCAF lines.

## Out of scope

- Auto-detection via JavaScript.
- An interactive toggle to dim or hide application-logic lines.
- A matching treatment for `study-guide.html` (no code blocks there).
- Retroactively marking future code blocks added after this change — each new block will need its lines wrapped by whoever adds it. The legend is the durable signal.
