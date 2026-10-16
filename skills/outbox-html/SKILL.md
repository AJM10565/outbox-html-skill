---
name: outbox-html
description: "Generate a self-contained click-to-copy HTML 'outbox' for any text the user must copy out of a terminal agent (Claude Code, pi, Codex, etc.) and paste elsewhere (chat messages, emails, form fields). Use whenever you hand the user prose or snippets to send that the agent can't deliver directly, especially multi-line text where terminal copy mangles whitespace."
---

# Outbox HTML: click-to-copy handoff page

## When to Use

Use whenever you produce text the user will copy out of the agent's terminal UI and paste somewhere the agent cannot write directly: chat/messenger replies, emails, web form fields, outreach messages, code snippets, etc. Especially valuable when the text is multi-line or whitespace-sensitive — terminal drag-select and copy commands often mangle indentation, reflow lines, or over-grab surrounding UI text. Default to this whenever handing over one or more discrete messages for the user to send.

## Procedure

1. Collect the discrete blocks to hand over (one per message/snippet), each with a short label (e.g. "Reply to Jordan", "Cover letter paragraph").
2. Write a self-contained HTML file (no external dependencies) to `<cwd>/outbox.html` (or a path the user names). Overwrite it each time; it is a scratch deliverable.
3. For each block, emit one card: a label, a **Copy** button, and a `<textarea readonly>` holding the EXACT text. Preserve real newlines; do not reflow.
4. Escape ONLY `&` → `&amp;` and `<` → `&lt;` inside each textarea (textarea content parses entities and the `</textarea>` tag; nothing else needs escaping, and whitespace is preserved literally). Set `rows` to roughly the line count.
5. Use a copy handler that tries `navigator.clipboard.writeText` first and ALWAYS falls back to `textarea.select()` + `document.execCommand('copy')`, because `file://` pages are an insecure context where `navigator.clipboard` often fails.
6. Open the file in the user's browser:
   - macOS: `open <path>`
   - Linux: `xdg-open <path>`
   - Windows: `start <path>` (or `explorer.exe <path>` from WSL)
   If the environment is headless or opening fails, just tell the user the file path to open.
7. Tell the user the page is ready: click **Copy** on a card, then paste. Keep chat commentary minimal — the HTML is the deliverable. Do not also paste the raw blocks in chat unless asked.

### Minimal template

```html
<!doctype html><meta charset="utf-8"><title>Outbox</title>
<style>
  body{font:15px/1.4 system-ui;max-width:720px;margin:2rem auto;padding:0 1rem}
  .card{border:1px solid #ccc;border-radius:8px;padding:1rem;margin:1rem 0}
  textarea{width:100%;font:13px/1.4 ui-monospace,monospace;white-space:pre}
  button{margin-bottom:.5rem}
</style>
<div class="card">
  <h3>LABEL HERE</h3>
  <button onclick="copyIt(this)">Copy</button>
  <textarea readonly rows="6">EXACT TEXT HERE (escape only &amp; and &lt;)</textarea>
</div>
<script>
async function copyIt(btn){
  const ta = btn.parentElement.querySelector('textarea');
  try { await navigator.clipboard.writeText(ta.value); }
  catch (e) { ta.focus(); ta.select(); document.execCommand('copy'); }
  btn.textContent = 'Copied'; setTimeout(()=>btn.textContent='Copy', 1500);
}
</script>
```

## Pitfalls

- `navigator.clipboard.writeText` FAILS on `file://` (insecure context) in Chrome and others — you MUST include the `document.execCommand('copy')` fallback via a real, focused, selected textarea, or the button silently does nothing.
- HTML-escaping the whole message breaks it: inside a `<textarea>`, escape ONLY `&` and `<`. Do NOT escape quotes, newlines, or apostrophes (that corrupts the pasted text).
- Do not use `<pre>` or `<div>` for the source text — a readonly `<textarea>` both preserves whitespace and is trivially selectable/copyable as a fallback.
- Do not reflow or "prettify" the message; the textarea must contain exactly what the user should send, including intended line breaks.

## Verification

1. The file opens in a browser and each Copy button changes to "Copied" on click.
2. Pasting into a plain-text field reproduces the exact text with newlines and indentation intact (test at least one multi-line block).
3. Works from `file://` (no clipboard-permission error in the console) thanks to the execCommand fallback.
