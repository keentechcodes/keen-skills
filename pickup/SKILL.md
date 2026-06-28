---
name: pickup
description: >
  Restore context from a previous session handoff document (created by `/done`). Use when
  the user wants to resume previous work — "pickup", "continue from last time", "what was
  I working on" — and orient on what's pending.
license: MIT
allowed-tools:
  - read
  - write
  - bash
  - glob
  - question
---

# Restore Session Context — /pickup

When triggered, read a previous session's handoff document (created by `/done`) and
reconstruct the context so work can continue seamlessly. The goal is for the user (or you,
the agent) to be fully oriented on what happened, what's pending, and what to do next —
without the user having to re-explain anything.

## Usage

```
/pickup                                          # loads the most recent doc from docs/sessions/
/pickup docs/sessions/2026-02-27_1430-auth.md    # loads a specific file
```

## Steps

### 1. Find the handoff document

- If the user provides a file path as an argument (`$ARGUMENTS`), use that file
- If no argument is given, scan `docs/sessions/` for markdown files and pick the most
  recently modified one
- If `docs/sessions/` doesn't exist or is empty, tell the user there are no session docs
  and suggest they use `/done` at the end of a session to create one

### 2. Read and parse the document

Read the full contents of the handoff markdown file. Extract all of the following, none skipped:

- **Goal** — what the session was about
- **Instructions** — persistent rules and preferences to respect going forward
- **Decisions** — choices already made, so you don't re-debate settled questions
- **Discoveries** — gotchas and patterns to keep in mind
- **Accomplished** — split into **Completed** (what's done) and **Pending / Next Steps** (what's left to work on)
- **Open questions** — unresolved questions that still need answers
- **Relevant files** — which files were modified or referenced

### 3. Read the referenced files

Read the files listed under **Relevant Files → Modified** in the handoff doc. This is what
makes `/pickup` more than just showing notes — it loads the relevant code into context so
you're ready to work, not just aware of what happened. Never skip this step: the whole
point is to restore working context, not show a summary the user could read themselves.

If there are many files, prioritize the ones related to **Pending / Next Steps**. You don't
need to read every referenced file — but always read the modified files central to the
next tasks.

### 4. Present a context briefing

After reading everything, present a concise briefing to the user:

```
## Resuming: {topic}

**Last session:** {date}
**Branch:** {branch}

### Where we left off
{1-2 sentences on current status, from Accomplished → Pending / Next Steps}

### Key context
- {most important Decisions and Discoveries to keep in mind}

### Ready to continue
{list the items from Pending / Next Steps}

### Open questions from last session
- {items from the Open Questions section, if any}
```

### 5. Surface open questions

If the handoff document has an **Open Questions** section, present each question and ask
whether the user wants to resolve any before continuing. This is the moment to clear
ambiguity from the previous session before diving into new work.

## Important behavior

- Respect **Instructions** from the handoff doc — coding style, workflow preferences, tools
  to use or avoid. Carry them forward as directives, not descriptions.
- If the handoff doc references a git branch, check if that branch still exists and is
  checked out. Mention it if there's a mismatch.
- After the briefing, suggest starting on the first item in **Pending / Next Steps** — don't
  ask "what would you like to do?" unless the user redirects.
