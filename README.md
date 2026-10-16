# outbox-html — a click-to-copy handoff skill for coding agents

A tiny, harness-agnostic [agent skill](https://code.claude.com/docs/en/skills) that teaches your coding agent (Claude Code, pi, Codex, or any SKILL.md-aware harness) to hand you text via a **self-contained HTML "outbox" page** instead of pasting it into the terminal.

## The problem

Agents often draft text you need to send elsewhere — chat replies, emails, form fields, code snippets. Copying that text out of a terminal UI is lossy: drag-select grabs UI chrome, multi-line text gets reflowed, and indentation gets mangled.

## The solution

The skill has the agent write `outbox.html` — a dependency-free page with one card per message: a label, a **Copy** button, and the exact text in a readonly textarea. Click Copy, paste anywhere, whitespace intact. It includes the `document.execCommand('copy')` fallback so the Copy button actually works on `file://` pages (where `navigator.clipboard` fails).

## Install

The skill is just `skills/outbox-html/SKILL.md`. Copy it to wherever your agent discovers skills:

| Harness | Location |
|---|---|
| Claude Code (personal) | `~/.claude/skills/outbox-html/SKILL.md` |
| Claude Code (project) | `.claude/skills/outbox-html/SKILL.md` |
| pi | `~/.pi/agent/skills/outbox-html/SKILL.md` |
| Codex / others | Wherever the harness loads skills or project instructions; the file is plain markdown with YAML frontmatter |

Example:

```bash
git clone https://github.com/AJM10565/outbox-html-skill
mkdir -p ~/.claude/skills
cp -r outbox-html-skill/skills/outbox-html ~/.claude/skills/
```

## Usage

Once installed, the agent triggers it automatically when it hands you text to send. You can also ask explicitly: *"put those messages in an outbox page"*.

## License

MIT
