---
name: pickup
description: >
  Restore context from a previous session handoff document. Use this skill whenever the user
  types /pickup, wants to resume previous work, asks to load a session, or mentions picking
  up where they left off. Trigger this even if the user just says "continue from last time"
  or "what was I working on". If no file argument is given, automatically find the most
  recent handoff doc in docs/sessions/.
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

Read the full contents of the handoff markdown file. Extract:

- **Summary** — what the session was about
- **Key decisions** — so you don't re-debate settled questions
- **Learnings** — gotchas and patterns to keep in mind
- **User preferences** — coding style, workflow choices to respect going forward
- **Changes made** — which files were touched
- **Current task status** — where things left off
- **Next tasks** — what to work on
- **Open questions** — what still needs answers

### 3. Read the referenced files

Look at the "Changes Made" section and read the key files listed there. This is what
makes `/pickup` more than just showing notes — it actually loads the relevant code into
context so you're ready to work, not just aware of what happened.

Be selective: read files that are central to the next tasks. If there are many files listed,
prioritize the ones related to "Next Tasks" and "Current Task Status". Don't read every
file blindly — use judgment.

### 4. Present a context briefing

After reading everything, present a concise briefing to the user:

```
## Resuming: {topic}

**Last session:** {date}
**Branch:** {branch}

### Where we left off
{1-2 sentences about current task status}

### Key context
- {most important decisions/learnings to keep in mind}

### Ready to continue
{list the next tasks from the handoff doc}

### Open questions from last session
- {any unresolved items that need input}
```

### 5. Surface open questions

If the handoff document contains open questions, present them to the user and ask if they
want to address any of them before continuing. This is the moment to resolve ambiguity
from the previous session before diving into new work.

## Important behavior

- Never skip reading the referenced files. The whole point is to restore working context,
  not just show a summary the user could read themselves.
- Respect user preferences captured in the handoff doc. If the previous session noted that
  the user prefers a specific coding style or tool, carry that forward.
- If the handoff doc references a git branch, check if that branch still exists and is
  checked out. Mention it if there's a mismatch.
- After the briefing, you're ready to work. Don't ask "what would you like to do?" — instead,
  suggest starting on the first item in "Next Tasks" unless the user directs otherwise.
