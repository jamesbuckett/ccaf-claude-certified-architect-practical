# Mark CCAF Code in Tutorial Examples — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Give every tutorial code block in `index.html` a low-noise visual mark that separates Claude-API / agentic-loop code (CCAF) from application plumbing, so a learner scanning a file can see at a glance which lines carry CCA-F exam content.

**Architecture:** Two visual mechanisms. (1) A new `.ccaf-line` CSS class applied via `<span>` wrappers inside `<pre>` blocks paints a coloured left-stripe on each CCAF Python line. (2) A new `.codeblock-badge` CSS class adds a small `CCAF` pill to the header of blocks that are 100% Claude protocol output (`response`, `messages state`). Bash blocks stay unmarked. One legend line above the first Python block in Tutorial 01 explains the stripe.

**Tech Stack:** Plain HTML5, vanilla CSS, single-file website. No build step, no framework, no test runner. Verification is visual: open `index.html` in a browser and scan.

**Source of truth:** `docs/superpowers/specs/2026-04-22-mark-ccaf-code-design.md` — includes the full marking rule and colour choice.

**Testing adaptation:** This is a static HTML/CSS change with no automated test harness in the project. TDD adapts to **concrete expected outcomes** verified in a browser at each task boundary. Every task ends with a visual check against a named expectation, then a commit.

---

## File Structure

One file is touched: `index.html`. No new files created, no files deleted.

Work is sequenced so CSS lands first (a broken selector would hide the whole change), legend and protocol-badge scaffolding second (low-count, high-signal), then per-tutorial Python span wrapping in source order.

Code-block inventory (line numbers are approximate — start of the `<div class="codeblock">` wrapper):

| Tutorial | Block type | Line (start) | File label |
|---|---|---|---|
| 01 | python | ~2031 | `python · loop.py` |
| 01 | response | ~2108 | `response` |
| 01 | response | ~2125 | `response` |
| 01 | messages state | ~2138 | `messages state` |
| 01 | python | ~2181 | `python · loop.py` |
| 01 | python | ~2240 | `python · loop.py` |
| 01 | python | ~2313 | `python · loop.py · build-and-break variant` |
| 01 | python | ~2414 | `python` |
| 02 | python | ~2482 | `python · orchestrator.py` |
| 02 | python | ~2582 | `python · orchestrator.py` |
| 02 | python | ~2646 | `python · orchestrator.py · build-and-break variant` |
| 02 | python | ~2741 | `python` |
| 03 | python | ~2816 | `python · server.py` |
| 04 | python | ~3066 | `python · describe.py` |
| 04 | python | ~3249 | `python` |
| 07 | python | ~3780 | `python · extract.py` |
| 07 | python | ~3929 | `python` |
| 07 | python | ~3991 | `python` |
| 08 | python | ~4062 | `python · prep.py` |
| 08 | python | ~4081 | `python · cache.py` |
| 08 | python | ~4144 | `python · cache.py · TTL variant` |
| 08 | python | ~4188 | `python · middle.py` |
| 08 | python | ~4308 | `python` |
| Utilities | python | ~4766 | `python` |
| Utilities | python | ~4836 | `python · usage.py` |

Total: **22 Python blocks, 3 protocol blocks.** Tutorials 05 and 06 have no Python blocks.

---

## The marking rule (copy-reference)

Every Python-block task applies this rule. Keep it visible while wrapping lines.

**Mark as CCAF** (wrap the line in `<span class="ccaf-line">…</span>`):

- SDK import / client init: `import anthropic`, `client = anthropic.Anthropic()`
- Model / iteration constants: `MODEL = "claude-…"`, `MAX_ITERATIONS = N`, `MAX_TOKENS = N`
- Tool catalogue: the entire `tools = [{...}]` structure — every line from `tools = [{` through the closing `}]`, including `name`, `description`, `input_schema`, nested schema fields
- Message construction: `messages = [...]`, any `messages.append({"role": ..., "content": ...})`
- API call: `client.messages.create(...)` and each argument line within it
- Response inspection: `resp.stop_reason`, `resp.content`, `block.type == "tool_use"`, `block.id`, `block.name`, `block.input`
- `tool_result` construction: the dict containing `type: "tool_result"`, `tool_use_id`, `content`
- Loop scaffold: `for turn in range(MAX_ITERATIONS):`, `if resp.stop_reason == "end_turn":`, `break`, the `for/else` safety branch
- Any line that directly calls, constructs, or inspects an Anthropic SDK type (`ToolUseBlock`, `TextBlock`, `Message`, etc.)
- The boundary line `result = run_tool(block.name, block.input)` — the **call** sits on the CCAF side because it dispatches using fields from the API response

**Do NOT mark** (application logic — leave the line as plain text inside `<pre>`):

- The **body** of any user-supplied tool function (e.g. the fake weather dict inside `run_tool`, the business-logic return values)
- User prompt text content (the literal `"Compare the weather…"` or similar strings)
- Debug `print(...)` calls
- Blank lines
- The `def run_tool(...)` signature line itself (not the call — only the definition)

**Whitespace preservation:** Wrap the full line including leading indentation inside the span. Do not add or remove whitespace. Each logical source line becomes one `<span class="ccaf-line">…</span>`. Because `.ccaf-line` uses `display: block`, each span renders as its own visual line.

**Example:** For the line `    MAX_ITERATIONS = 5`, write `<span class="ccaf-line">    MAX_ITERATIONS = 5</span>`.

---

## Task 1: Add CSS — `.ccaf-line`, `.codeblock-badge`, `--ccaf-stripe`

**Files:**
- Modify: `index.html` — `<style>` block, near the existing `.codeblock pre .added` rule (currently around line 868) and the `:root` custom-property block (near line 140 in the CSS section)

- [ ] **Step 1: Locate the existing `.added` rule in the `<style>` block**

Run: `grep -n '\.codeblock pre \.added' index.html`
Expected: one line reference around 868. This is the anchor — the new `.ccaf-line` rule goes directly after it.

- [ ] **Step 2: Locate the `:root` custom-property block**

Run: `grep -n '^:root' index.html`
Expected: one or more matches. Identify the `:root` block that holds light-mode colour tokens (e.g. `--accent`, `--border`). The new `--ccaf-stripe` variable is added there.

- [ ] **Step 3: Locate the dark-mode override block**

Run: `grep -n 'data-theme="dark"\|\[data-theme=dark\]\|prefers-color-scheme' index.html`
Expected: a block that redefines colour tokens for dark mode. The dark-mode override for `--ccaf-stripe` goes there.

- [ ] **Step 4: Add `--ccaf-stripe` to light `:root` block**

Insert inside the `:root` block, alongside other colour tokens:

```css
--ccaf-stripe: #6366F1;
```

- [ ] **Step 5: Add dark-mode override for `--ccaf-stripe`**

Insert inside the existing dark-mode override block, alongside the other dark-mode token redefinitions:

```css
--ccaf-stripe: #818CF8;
```

Lighter indigo so the stripe still reads against a dark background.

- [ ] **Step 6: Add `.ccaf-line` rule directly after `.codeblock pre .added`**

Insert after the closing `}` of `.codeblock pre .added`:

```css
.codeblock pre .ccaf-line {
  display: block;
  box-shadow: inset 2px 0 0 var(--ccaf-stripe);
  padding-left: var(--s-2);
  margin-left: calc(-1 * var(--s-2));
}
```

- [ ] **Step 7: Add `.codeblock-badge` rule directly after `.ccaf-line`**

Insert immediately after the `.ccaf-line` rule:

```css
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

- [ ] **Step 8: Visual check — no visible change yet**

Open `index.html` in a browser and hard-reload. Expected: the page renders exactly as before. No stripes anywhere (no `<span class="ccaf-line">` exist yet), no badges (no `<span class="codeblock-badge">` exist yet). If anything *changed* visually, a selector is wrong — fix before continuing.

- [ ] **Step 9: Commit**

```bash
git add index.html
git commit -m "Add CSS for .ccaf-line stripes and .codeblock-badge pill"
```

---

## Task 2: Add `CCAF` badges to the 3 protocol blocks

**Files:**
- Modify: `index.html` lines ~2108, ~2125, ~2138

- [ ] **Step 1: Inspect the first protocol block**

Read lines 2105–2115 of `index.html`. Expected: a `<div class="codeblock-head">` containing `<span class="codeblock-lang">response</span>` followed by a copy button.

- [ ] **Step 2: Insert a `CCAF` badge after the first `response` label**

In the line at ~2108, change:

```html
<div class="codeblock-head"><span class="codeblock-lang">response</span><button class="copy-btn"
```

to:

```html
<div class="codeblock-head"><span class="codeblock-lang">response</span><span class="codeblock-badge">CCAF</span><button class="copy-btn"
```

- [ ] **Step 3: Insert a `CCAF` badge after the second `response` label**

Apply the same edit to the line at ~2125.

- [ ] **Step 4: Insert a `CCAF` badge after the `messages state` label**

In the line at ~2138, change:

```html
<div class="codeblock-head"><span class="codeblock-lang">messages state</span><button class="copy-btn"
```

to:

```html
<div class="codeblock-head"><span class="codeblock-lang">messages state</span><span class="codeblock-badge">CCAF</span><button class="copy-btn"
```

- [ ] **Step 5: Visual check — three `CCAF` pills visible**

Reload `index.html`. Scroll to Tutorial 01's "Code trace" section. Expected: three `CCAF` pills visible in the headers of the two `response` blocks and the `messages state` block. No stripes anywhere yet. No other badges.

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "Add CCAF header badges to response and messages-state blocks"
```

---

## Task 3: Insert the legend before Tutorial 01's first Python block

**Files:**
- Modify: `index.html` — insert a new paragraph immediately before the `<div class="codeblock">` that wraps the Python block at line ~2031

- [ ] **Step 1: Locate the insertion point**

Read lines 2020–2035 of `index.html`. Find the last `</p>` or closing tag immediately before `<div class="codeblock">` at line ~2031. The legend goes on its own line between them.

- [ ] **Step 2: Insert the legend paragraph**

Directly above `<div class="codeblock">` at ~2031, insert:

```html
<p class="muted" style="margin-top: var(--s-4); font-size: 0.88rem;">
  <span class="ccaf-line" style="display:inline-block; padding-left: var(--s-2); margin-left: 0;">&nbsp;</span>
  Lines with a coloured left-stripe are <strong>Claude API / agentic-loop code</strong> (CCA-F exam content). Unmarked lines are application plumbing you'd swap out for your own logic.
</p>
```

The inline `.ccaf-line` swatch renders as a tiny visual sample of the stripe so the reader immediately knows what colour to look for.

- [ ] **Step 3: Visual check — legend renders correctly**

Reload `index.html`. Scroll to Tutorial 01. Expected: one short paragraph above the first `python · loop.py` block, with a small coloured stripe swatch at its left edge and the wording "Lines with a coloured left-stripe are Claude API / agentic-loop code…". The paragraph appears **once**, immediately before the first Python code block.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "Add legend explaining CCAF left-stripe convention"
```

---

## Task 4: Wrap CCAF lines in Tutorial 01 Python blocks

**Files:**
- Modify: `index.html` — Python blocks at ~2031, ~2181, ~2240, ~2313, ~2414

**Reference:** Apply the marking rule from the top of this plan. Fully-worked template for the first block follows; later blocks use the same rule.

- [ ] **Step 1: Mark block at ~2031 (`python · loop.py`)**

Read lines 2033–2054. Inside the `<pre>…</pre>`, wrap each CCAF line in `<span class="ccaf-line">…</span>`. The result should look like:

```html
<pre><span class="ccaf-line">import anthropic, json, os</span>

<span class="ccaf-line">client = anthropic.Anthropic()</span>
<span class="ccaf-line">MODEL = "claude-sonnet-4-6"</span>
<span class="ccaf-line">MAX_ITERATIONS = 5</span>

<span class="ccaf-line">tools = [{</span>
<span class="ccaf-line">    "name": "get_weather",</span>
<span class="ccaf-line">    "description": "Return the current temperature in Celsius for a given city.",</span>
<span class="ccaf-line">    "input_schema": {</span>
<span class="ccaf-line">        "type": "object",</span>
<span class="ccaf-line">        "properties": {"city": {"type": "string"}},</span>
<span class="ccaf-line">        "required": ["city"],</span>
<span class="ccaf-line">    },</span>
<span class="ccaf-line">}]</span>

def run_tool(name, args):
    if name == "get_weather":
        fake = {"Paris": 14, "Tokyo": 22, "Cape Town": 27}
        return {"temp_c": fake.get(args["city"], 20)}
    raise ValueError(f"unknown tool {name}")

<span class="ccaf-line">messages = [{"role": "user",</span>
<span class="ccaf-line">             "content": "Compare the weather in Paris and Tokyo right now."}]</span>

<span class="ccaf-line">for turn in range(MAX_ITERATIONS):</span>
<span class="ccaf-line">    resp = client.messages.create(</span>
<span class="ccaf-line">        model=MODEL, max_tokens=1024, tools=tools, messages=messages</span>
<span class="ccaf-line">    )</span>
    print(f"--- turn {turn} · stop_reason={resp.stop_reason} ---")
<span class="ccaf-line">    messages.append({"role": "assistant", "content": resp.content})</span>

<span class="ccaf-line">    if resp.stop_reason == "end_turn":</span>
        print("FINAL:", resp.content[-1].text)
<span class="ccaf-line">        break</span>

<span class="ccaf-line">    tool_results = []</span>
<span class="ccaf-line">    for block in resp.content:</span>
<span class="ccaf-line">        if block.type == "tool_use":</span>
<span class="ccaf-line">            result = run_tool(block.name, block.input)</span>
            print(f"  → {block.name}({block.input}) = {result}")
<span class="ccaf-line">            tool_results.append({</span>
<span class="ccaf-line">                "type": "tool_result",</span>
<span class="ccaf-line">                "tool_use_id": block.id,</span>
<span class="ccaf-line">                "content": json.dumps(result),</span>
<span class="ccaf-line">            })</span>
<span class="ccaf-line">    messages.append({"role": "user", "content": tool_results})</span>
<span class="ccaf-line">else:</span>
    print("!! loop exhausted without end_turn — safety cap hit")</pre>
```

Note the unmarked lines: the `def run_tool` body, the user-prompt string content is *marked* (it's part of the message construction — the whole `messages = [...]` block is CCAF), every `print(...)` is unmarked, blank lines are unmarked.

Wait — the user prompt text on its own line is part of a CCAF `messages = [...]` construction. The spec's rule explicitly says "User prompt text content (e.g. the literal `"Compare the weather…"` string)" is **NOT** marked. Resolve: when a single line contains *only* the prompt string (e.g. a multi-line string split across lines), that line is unmarked. When the line also contains dict-structural elements (brackets, `"role":`, `"content":`), the line is marked. In the template above, line `             "content": "Compare the weather in Paris and Tokyo right now."}]` contains both — mark it, because the structural element dominates. Err on the side of marking in ambiguous cases; the whole point is to show where the message shape is defined.

- [ ] **Step 2: Visual check — first block stripes render correctly**

Reload `index.html`. Scroll to the first Python block. Expected: stripes visible on every CCAF line, no stripes on the `def run_tool` body, no stripes on `print(...)` lines, no stripes on blank lines. The `run_tool` body (4 lines) is clearly unmarked — you can see the gap between the `tools = [{…}]` stripes above and the `messages = [...]` stripes below.

- [ ] **Step 3: Copy-paste test on first block**

Click the copy button on the first Python block. Paste into a plain-text editor. Expected: clean Python source with no `<span>` tags. If HTML tags appear, `navigator.clipboard.writeText(pre.textContent)` is behaving differently than expected — stop and investigate before continuing.

- [ ] **Step 4: Mark block at ~2181 (`python · loop.py`)**

Read the block. Apply the marking rule. This is the same `loop.py` file but with one line added (the inspection line the learner adds). Mark the same lines as Step 1, plus any newly-added inspection line that touches `block.type`, `block.content`, `resp.content`, etc. The existing `.added` span on the new line should sit **inside** the `.ccaf-line` span so both marks stack (a changed CCAF line shows both the stronger `.added` highlight and the CCAF stripe — if they visually conflict, the `.added` rule wins because its box-shadow is on top).

- [ ] **Step 5: Mark block at ~2240 (`python · loop.py`)**

Read the block. Apply the marking rule.

- [ ] **Step 6: Mark block at ~2313 (`python · loop.py · build-and-break variant`)**

Read the block. Apply the marking rule. This variant deletes `MAX_ITERATIONS`, so that line is absent; the loop is `while True:` — still mark it (it's the scaffold concept, now broken).

- [ ] **Step 7: Mark block at ~2414 (`python`)**

Read the block. Apply the marking rule.

- [ ] **Step 8: Visual check — all five Tutorial 01 Python blocks**

Reload `index.html`. Walk through Tutorial 01 top to bottom. Expected: every Python block shows stripes consistent with the rule. No stripes on bash blocks. The three protocol badges from Task 2 still render. The legend from Task 3 still renders.

- [ ] **Step 9: Commit**

```bash
git add index.html
git commit -m "Mark CCAF lines in Tutorial 01 Python blocks"
```

---

## Task 5: Wrap CCAF lines in Tutorial 02 Python blocks

**Files:**
- Modify: `index.html` — Python blocks at ~2482, ~2582, ~2646, ~2741

- [ ] **Step 1: Mark block at ~2482 (`python · orchestrator.py`)**

Read the block. Apply the marking rule from the top of this plan. Tutorial 02 is hub-and-spoke orchestration — in addition to the usual API-surface lines, mark any sub-agent client init, any `client.messages.create` call per sub-agent, and the result-aggregation lines that build a final `messages` list for the parent agent. User-supplied logic (domain-specific aggregation functions, prints) stays unmarked.

- [ ] **Step 2: Mark block at ~2582 (`python · orchestrator.py`)**

Read the block. Apply the marking rule.

- [ ] **Step 3: Mark block at ~2646 (`python · orchestrator.py · build-and-break variant`)**

Read the block. Apply the marking rule.

- [ ] **Step 4: Mark block at ~2741 (`python`)**

Read the block. Apply the marking rule.

- [ ] **Step 5: Visual check — Tutorial 02**

Reload. Walk through Tutorial 02. Expected: stripes on SDK-surface and loop-scaffold lines across all four Python blocks. Domain-specific aggregation logic is unmarked.

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "Mark CCAF lines in Tutorial 02 Python blocks"
```

---

## Task 6: Wrap CCAF lines in Tutorials 03 and 04 Python blocks

**Files:**
- Modify: `index.html` — Python blocks at ~2816 (Tutorial 03), ~3066 (Tutorial 04), ~3249 (Tutorial 04)

- [ ] **Step 1: Mark block at ~2816 (`python · server.py`)**

Tutorial 03 builds an MCP server. MCP is in the CCA-F scope — **mark** any MCP SDK imports (`from mcp.server …`), server decorators (`@server.list_tools()`, `@server.call_tool()`, etc.), protocol type constructions (`Tool(...)`, `TextContent(...)`), and server lifecycle calls. Application logic inside tool handlers (the actual data fetching or computation the tool performs) stays **unmarked**.

- [ ] **Step 2: Mark block at ~3066 (`python · describe.py`)**

Tutorial 04 is tool-description discipline. Mark SDK surface, the `tools = [{...}]` block including the `description` field (that field is the entire point of this tutorial), and the `client.messages.create(...)` call. Domain-specific tool body stays unmarked.

- [ ] **Step 3: Mark block at ~3249 (`python`)**

Read the block. Apply the marking rule.

- [ ] **Step 4: Visual check — Tutorials 03 and 04**

Reload. Walk through both tutorials. Expected: MCP decorators and types marked in Tutorial 03; tool-description lines clearly marked in Tutorial 04.

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "Mark CCAF lines in Tutorial 03 and 04 Python blocks"
```

---

## Task 7: Wrap CCAF lines in Tutorial 07 Python blocks

**Files:**
- Modify: `index.html` — Python blocks at ~3780, ~3929, ~3991

(Tutorials 05 and 06 have no Python blocks.)

- [ ] **Step 1: Mark block at ~3780 (`python · extract.py`)**

Tutorial 07 is structured output with tool-use + JSON Schema retry. Mark SDK surface, the JSON-Schema-bearing `tools = [{...}]` definition, the `client.messages.create(...)` call, response inspection, and the retry loop scaffold (including any `try/except jsonschema.ValidationError` branch that triggers a retry — the retry **pattern** is CCA-F content). Schema-validation library internals and domain-specific field names in the business payload stay unmarked.

- [ ] **Step 2: Mark block at ~3929 (`python`)**

Read the block. Apply the marking rule.

- [ ] **Step 3: Mark block at ~3991 (`python`)**

Read the block. Apply the marking rule.

- [ ] **Step 4: Visual check — Tutorial 07**

Reload. Walk through Tutorial 07. Expected: stripes on the retry-loop scaffold and the `tools = [...]` schema definition.

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "Mark CCAF lines in Tutorial 07 Python blocks"
```

---

## Task 8: Wrap CCAF lines in Tutorial 08 Python blocks

**Files:**
- Modify: `index.html` — Python blocks at ~4062, ~4081, ~4144, ~4188, ~4308

- [ ] **Step 1: Mark block at ~4062 (`python · prep.py`)**

Tutorial 08 is prompt caching. Mark SDK surface and any `cache_control` dict fragments (e.g. `{"type": "ephemeral"}`) — those are the cert-relevant bits.

- [ ] **Step 2: Mark block at ~4081 (`python · cache.py`)**

Read the block. Apply the marking rule. Mark `cache_control` fields, `client.messages.create(...)` call, `extra_headers={"anthropic-beta": …}` if present, and any `usage.cache_read_input_tokens` / `usage.cache_creation_input_tokens` inspection — those usage fields are CCA-F content.

- [ ] **Step 3: Mark block at ~4144 (`python · cache.py · TTL variant`)**

Read the block. Apply the marking rule. Same as the base `cache.py` plus the TTL-specific fields (e.g. `"ttl": "1h"`).

- [ ] **Step 4: Mark block at ~4188 (`python · middle.py`)**

Read the block. Apply the marking rule. "Lost in the middle" demonstrations still use `client.messages.create(...)` — mark the API surface, leave the corpus-construction plumbing unmarked.

- [ ] **Step 5: Mark block at ~4308 (`python`)**

Read the block. Apply the marking rule.

- [ ] **Step 6: Visual check — Tutorial 08**

Reload. Walk through Tutorial 08. Expected: `cache_control` dict entries clearly striped; usage-inspection lines striped.

- [ ] **Step 7: Commit**

```bash
git add index.html
git commit -m "Mark CCAF lines in Tutorial 08 Python blocks"
```

---

## Task 9: Wrap CCAF lines in Utilities Python blocks

**Files:**
- Modify: `index.html` — Python blocks at ~4766, ~4836

- [ ] **Step 1: Mark block at ~4766 (`python`)**

Read the block. Apply the marking rule.

- [ ] **Step 2: Mark block at ~4836 (`python · usage.py`)**

`usage.py` is the smoke-test utility. It's almost entirely SDK surface — a minimal `client.messages.create(...)` call plus a `usage`-field print. Mark nearly every line; leave only the debug `print(...)` formatting unmarked.

- [ ] **Step 3: Visual check — Utilities**

Reload. Scroll to the Utilities section. Expected: stripes on both blocks.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "Mark CCAF lines in Utilities Python blocks"
```

---

## Task 10: Final end-to-end verification

**Files:**
- No edits. This task is a pure verification pass against the spec's acceptance criteria.

- [ ] **Step 1: Visual scan — full page**

Open `index.html`. Scroll from top to bottom. Confirm each of the following:

1. Legend appears **exactly once**, above Tutorial 01's first Python block.
2. Every one of the 22 Python blocks has at least some striped lines.
3. No stripes on any bash block.
4. Three `CCAF` header badges on the two `response` blocks and one `messages state` block in Tutorial 01.
5. No stray `<span class="ccaf-line">` markup visible as text anywhere (a missed `</span>` would leak).

- [ ] **Step 2: Copy-paste test — three random Python blocks**

Pick three Python blocks at random (e.g. Tutorial 01 first block, Tutorial 05 `orchestrator.py`, Tutorial 08 `cache.py`). Click copy, paste into a plain-text editor. Expected for each: clean Python source, no HTML tags, original indentation preserved.

- [ ] **Step 3: Dark-mode test**

Toggle dark mode using the page's existing toggle. Expected: stripes still visible (lighter indigo), badges still readable, no contrast failures. If stripes disappear against the dark background, the dark-mode `--ccaf-stripe` override from Task 1 Step 5 is missing or wrong — go back and fix.

- [ ] **Step 4: Print-preview test**

Open the browser's print preview. Expected: stripes render on paper (at least as a faint left edge). Badges render as outlined pills. The page is still printable.

- [ ] **Step 5: Interaction with `.added` lines**

Find a block that has both `.added` and `.ccaf-line` on the same line (Tutorial 01 block at ~2181 — the "add one inspection line" step). Expected: the `.added` highlight wins visually (stronger background + stripe) and the `.ccaf-line` stripe is either hidden behind it or visually subordinate. Acceptable either way — just confirm the line is not unreadable.

- [ ] **Step 6: Commit the verification outcome**

If any fix was required during Steps 1–5, commit it with a descriptive message and re-run the relevant step. Otherwise, there is nothing to commit for this task — continue to the plan-completion note.

- [ ] **Step 7: Plan complete**

Confirm all 25 code blocks are marked per the spec and the page passes every acceptance criterion. If yes, plan is done.

---

## Self-Review Results

**Spec coverage:** Every element of `docs/superpowers/specs/2026-04-22-mark-ccaf-code-design.md` maps to a task:
- Per-line stripe on Python blocks → Tasks 4–9
- `CCAF` header badge on protocol blocks → Task 2
- Bash blocks unmarked → enforced by scope; verified in Task 10 Step 1 #3
- Colour (`--ccaf-stripe` light + dark) → Task 1 Steps 4–5
- Legend once near Tutorial 01 → Task 3
- Broad marking rule → restated in the "marking rule" section; referenced from every Python task
- Copy-paste safety → verified in Task 4 Step 3 and Task 10 Step 2
- Print-friendly → verified in Task 10 Step 4
- Interaction with existing `.added` rule → Task 4 Step 4 notes the stacking behaviour; Task 10 Step 5 verifies

**Placeholder scan:** No "TBD", no "implement later", no "add error handling", no "similar to Task N without repeating the code". Task 4 includes a fully-worked template; later tasks cite the same rule and the line ranges.

**Type consistency:** Class names used consistently throughout — `.ccaf-line`, `.codeblock-badge`, `--ccaf-stripe`. No drift.

**Scope check:** Single file, single feature, ~10 commits. One implementation plan is appropriate.
