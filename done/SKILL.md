---
name: done
description: >
  Save a session handoff to docs/sessions/. Trigger when the user says done, finished, wrap up,
  handoff, session notes, or wants context saved.
license: MIT
allowed-tools:
  - read
  - write
  - bash
  - glob
---

# Session Handoff — /done

When triggered, produce a structured markdown handoff document and save it to `docs/sessions/`.
A fresh agent or the user should be able to read it and restore full session context —
what was being done, why, what was learned, and what's next.

## What to capture

Review the **entire** conversation — not just the last few messages — and extract:

1. **Goal** — one clear sentence stating what the session was about
2. **Instructions** — persistent rules, preferences, and constraints the user stated that should carry forward to future sessions (coding style, validation commands, workflow preferences, tools to use or avoid)
3. **Discoveries** — technical findings, gotchas, and insights uncovered during the session. These are the things that would save someone hours if they knew them upfront. Use subheadings for distinct findings. Include evidence (specific values, line numbers, error messages, test results)
4. **Accomplished** — split into **Completed** (numbered, with status markers) and **Pending / Next Steps** (actionable items remaining)
5. **Relevant files / directories** — split into **Modified** (files changed, with line numbers and what changed) and **Read / Referenced** (files consulted but not changed)

If a section has no content, omit it entirely.

## File format

```markdown
# Session: {topic}

**Date:** {YYYY-MM-DD}
**Time:** {HH:MM} PHT (UTC+8)
**Model:** {model name} · {provider/client}
**Working directory:** {cwd}
**Branch:** {branch or "N/A"}

## Goal

{One specific sentence — not "worked on the app" but "Fixing bugs in the Fitness Activity Export feature in the SuperAdmin dashboard"}

## Instructions

{Persistent rules and preferences the user stated. These carry forward across sessions.
Write them as directives a future agent should follow.}

- {e.g., "Always run `pnpm check-types` after changes"}
- {e.g., "Use context7 MCP to validate against official docs before implementing fixes"}
- {e.g., "The user prefers extracting constants over magic numbers"}

## Discoveries

{Technical findings, gotchas, and insights. Use subheadings for distinct topics.
Include specific evidence — numbers, line numbers, error messages, before/after comparisons.
This is the most valuable section for future context restoration.}

### {Discovery Title — descriptive, e.g. "PostHog HogQL Default LIMIT Behavior" not "LIMIT issue"}

- {Specific finding with evidence}
- {Why it matters}

## Accomplished

### Completed

1. {Task description} ✅
2. {Task description} ✅
3. {Task that failed first, then succeeded} — {brief note on what went wrong and the fix}

### Pending / Next Steps

- {Actionable next step with enough context to act on it without re-reading the whole doc}
- {Another next step}

## Relevant Files

### Modified

- `{file/path}` (lines ~{N}-{M}) — {what changed and why}

### Read / Referenced

- `{file/path}` — {why it was relevant}
```

## Filename convention

```
docs/sessions/{YYYY-MM-DD}_{HHMM}-{topic-slug}.md
```

- Date and time are in PHT (UTC+8). Get local time via `TZ=Asia/Manila date '+%Y-%m-%d_%H%M'`
- Topic slug is lowercase, hyphenated, max 5 words derived from the session goal
- Example: `docs/sessions/2026-02-27_1430-fitness-export-bugfix.md`
- If a file with that name already exists, append a counter: `-2`, `-3`, etc.

## Steps

1. Run `mkdir -p docs/sessions/` to ensure the directory exists
2. Get the current local time in PHT (UTC+8) using `TZ=Asia/Manila date`
3. Determine the current git branch (if in a git repo). Use "N/A" otherwise.
4. Identify the model and provider/client powering this session (e.g., "Claude Opus 4.6 · OpenCode", "Claude Sonnet 4 · Github Copilot", "Gemini 2.5 Pro · Cursor"). Check your system prompt or model metadata if available.
 5. Review the **full** conversation history from start to finish, until you can enumerate every distinct topic, decision, and discovery the session covered
 6. Extract all items described in "What to capture" — every section filled with concrete content pulled from the conversation, none left skeletal
 7. Write the file using the template
 8. Check the output against the quality bar below; if any criterion fails, revise, then tell the user what was saved, the full path, and remind them they can use `/pickup` to restore context in a future session

## Quality bar

Before saving, check your output against these criteria:

- Could a fresh agent read this and start working without asking clarifying questions?
- Are discoveries backed by specific evidence, not vague statements?
- Are file references precise enough (with line numbers) to navigate directly?
- Are pending tasks actionable without re-reading the whole conversation?
- Are user instructions written as directives, not descriptions?
