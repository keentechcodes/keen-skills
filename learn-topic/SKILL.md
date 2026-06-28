---
name: learn-topic
description: >
  Research any topic, synthesize findings from authoritative sources, and render a human-digestible
  HTML guide with diagrams (Mermaid / ASCII / tables) — saved as a standalone HTML file the user
  opens in a browser. Trigger when the user says "learn X", "teach me X with a guide", "research X
  and make an HTML writeup", "primer on X", "digestible read on X", "explain X like a guide", or
  wants a self-contained, shareable HTML document about a topic after research. Distinct from /teach
  (in-workspace skill transfer) and from gstack /learn (managing project learnings).
allowed-tools:
  - read
  - write
  - bash
  - glob
  - grep
  - todowrite
  - exa_web_search_exa
  - exa_web_fetch_exa
  - websearch_web_search_exa
  - webfetch
  - task
---

# /learn-topic — Research + render a digestible HTML guide

Produce a **self-contained HTML file** about a topic the user names. The file must:

1. Be openable in any modern browser (Chromium, Firefox, Safari).
2. Render Mermaid diagrams and tables without external install.
3. Read like a well-edited primer: clear sections, scannable headings, diagrams before prose where a diagram carries the idea.
4. Cite sources used during research at the bottom.
5. Live at `~/learn-{topic-slug}.html` unless the user asked for a different path.

## When to invoke

- User asks: "learn X", "primer on X", "digestible guide on X", "research X and make an HTML writeup", "explain X like a guide", "teach me X with an HTML guide", "I want a readable writeup of X with diagrams".
- User cites a topic name + (optional) reference materials (URLs, files).
- User asks for output in HTML with diagrams and references.

## When NOT to invoke

- User asks for口头 explanation in chat → just answer in chat.
- User asks to write a markdown file specifically → use Write directly.
- User is asking about something they could just read in the repo → use explore first.
- User says "teach me" without wanting HTML output → that's the `/teach` skill, not this one.

## Workflow (5 phases, MUST run in order)

### Phase 1 — Settle the contract (30 seconds)

Confirm:
- Topic name (use user's exact wording for the title).
- Any reference URLs/files the user supplied.
- Default output path: `~/learn-{kebab-case-topic-slug}.html`. Override only if user named a path.
- Default scope: comprehensive primer (TL;DR + mental models + diagrams + examples + failure modes + references). Narrow only if user asks for a focused subset.

If the user attached a binary file (PDF, image, audio) and this model cannot read it, say so explicitly in the final HTML — never silently skip it. Run the research without that source and disclose it in a callout.

### Phase 2 — Parallel research (use tools aggressively)

Fire in parallel:

- `exa_web_search_exa` — 2 queries from different angles (concept + "primer/explainer/exampleFRAME").
- `websearch_web_search_exa` — 1 backup query if exa returns thin coverage.
- For any GitHub repo mentioned in results → `exa_web_fetch_exa` on the repo URL + raw README URL.
- For any canonical essay/post mentioned → `exa_web_fetch_exa` on that URL.
- If the topic has a library/framework → use `context7_resolve-library-id` + `context7_query-docs` for current docs.

Optional delegation when coverage is broad:

- `task(subagent_type="explore", run_in_background=true)` — only if the topic lives inside the user's repo AND the user wants repo-specific concepts included. Most topics won't need this.
- `task(subagent_type="librarian", run_in_background=true)` — only when the topic is about a specific library/SDK with versioned behavior the LLM may not know.

Stop researching when sources converge — usually 3–5 authoritative sources cover a concept. Do NOT keep pinging to feel safer.

### Phase 3 — Synthesize the structure

Every guide MUST have these sections (omittable only when truly N/A, with a one-line reason in the final HTML):

1. **TL;DR / What it is** — 1–2 paragraphs + 1–2 anchor quotes from named authorities.
2. **The shift / mental model** — why this concept exists and what it replaces or sits above. Use a visual stack (4 boxes or a ladder).
3. **Core cycle / primitives** — the repeating loop or building blocks. Mermaid flowchart preferred.
4. **Anatomy / how it fits together** — Mermaid flowchart that shows the whole system end-to-end.
5. **Design levers / tradeoffs** — table.
6. **Key patterns** — table or grid, with badges for cost/risk where relevant.
7. **Worked example** — Mermaid sequenceDiagram or numbered steps with code block. Should be minimal but concrete.
8. **Formal layer (optional)** — only if a spec / rubric / scoring system exists for the topic. YAML or JSON block.
9. **Failure modes / anti-patterns** — grid of cards in red/dark, each with name + symptom + mitigation.
10. **Cost / safety / defaults** — bulleted or callout.
11. **Getting started in 5 minutes** — copyable code block.
12. **Decision tree for the reader** — Mermaid flowchart: "should YOU do this? if X, then Y."
13. **Checklist before applying** — interactive-style checkboxes (visual only).
14. **Caveats / honest limits** — bulleted.
15. **References** — table with name + what it's good for; link URL as anchor text.
16. **Bottom line** — dark CTA-style card with a one-paragraph summary.

If a topic genuinely lacks one of these (e.g. "formal spec" for a soft concept), OMIT it — don't pad. Sections are a menu, not a checklist.

### Phase 4 — Render HTML (follow the template contract)

Use this EXACT scaffold (copy verbatim, only swap content). Tailwind CDN + Mermaid v11 ESM. `max-w-5xl` container, `font-serif` headings, `bg-stone-50` body, white cards with `border border-slate-200`, rounded-xl.

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>{Topic} — A Digestible Guide</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script type="module">
      import mermaid from "https://cdn.jsdelivr.net/npm/mermaid@11/dist/mermaid.esm.min.mjs";
      mermaid.initialize({ startOnLoad: true, theme: "neutral", securityLevel: "loose" });
    </script>
    <style>
      .deep { background: linear-gradient(135deg, #0f172a, #1e293b); color: #f8fafc; }
      .module-box { border: 2px solid #1e293b; border-radius: 6px; }
      .module-box-thin { border: 1px dashed #64748b; border-radius: 6px; }
      .badge-strong { background: #059669; color: #f0fdf4; }
      .badge-worth { background: #d97706; color: #fffbeb; }
      .badge-spec { background: #475569; color: #f1f5f9; }
      .tag { background: #e2e8f0; color: #0f172a; }
      .code { font-family: ui-monospace, SFMono-Regular, Menlo, monospace; }
      .pipe { background: linear-gradient(135deg, #0f172a, #1e293b); color: #f8fafc; }
      .lift-bg { background: linear-gradient(135deg, #1e3a8a, #0c4a6e); color: #f0f9ff; }
      .lift-bw { background: repeating-linear-gradient(45deg, #e0f2fe, #e0f2fe 6px, #bae6fd 6px, #bae6fd 12px); }
      .fail-bg { background: linear-gradient(135deg, #7f1d1d, #450a0a); color: #fef2f2; }
      .punch { background: #fef3c7; border-left: 4px solid #d97706; }
      .quote-pull { background: #f8fafc; border-left: 4px solid #0f172a; }
    </style>
  </head>
  <body class="bg-stone-50 text-slate-900 font-sans">
    <main class="max-w-5xl mx-auto px-6 py-12 space-y-12">

      <header class="space-y-4"> ... </header>

      <!-- one <section> per content block from Phase 3 -->
      <section id="..." class="rounded-xl border border-slate-200 bg-white p-6 space-y-5"> ... </section>
      ...

      <section class="rounded-xl border-2 border-slate-900 bg-slate-900 text-slate-50 p-6 space-y-3">
        <h2 class="text-2xl font-serif">Bottom line</h2>
        <p class="text-base"> ... </p>
      </section>

    </main>
  </body>
</html>
```

**Visual conventions (consistent across guides):**

| Class | Use for |
|---|---|
| `deep` | primary / durable concept (dark slate) |
| `pipe` | runtime / data spine |
| `badge-strong` (green) | recommended / strong signal |
| `badge-worth` (amber) | worth exploring / medium cost |
| `badge-spec` (slate) | speculative / very high cost |
| `tag` (light) | neutral label |
| `punch` (amber strip) | critical callout, "the one thing to remember" |
| `quote-pull` (white strip) | attributed quote |
| `fail-bg` (dark red) | failure mode / anti-pattern card |
| `lift-bg` (blue gradient) | pipeline / sequence strip |
| `lift-bw` (blue stripes) | repeating conventional pattern block |
| `quote-pull` | quoted authority |

### Phase 5 — Diagram selection heuristic

Pick the right diagram type per section. NEVER default to ASCII when Mermaid works.

| Concept shape | Use |
|---|---|
| Sequential pipeline / lifecycle | Mermaid `flowchart LR` with named nodes |
| Decision logic / branching | Mermaid `flowchart TD` with diamond `{}` nodes |
| Multi-actor conversation / events | Mermaid `sequenceDiagram` |
| State transitions | Mermaid `stateDiagram-v2` |
| Hierarchical relationships | Mermaid `flowchart TB` |
| Side-by-side comparison | HTML `<div class="grid md:grid-cols-2">` of cards |
| Tabular data (cost, cadence, properties) | HTML `<table>` with badge pills |
| Numeric matrix / 2D spectrum | ASCII grid as preformatted block (rare) |
| Code / config / commands | `<pre class="code">` inside styled box |
| One-sentence callout / pull quote | `quote-pull` div |

**Mermaid rules:**
- Always wrap in `<pre class="mermaid">` so Tailwind doesn't style-break it.
- Use `classDef` to color-code leaks/risks the way the template does.
- Keep node labels short — Mermaid wraps; long labels break rendering.
- Sequence diagrams: 4–8 actors max. More → split into two diagrams.
- Test mentally: does the graph close back to its start? Cycles are fine for "loops"; open chains for "pipelines."

**ASCII fallback:** Only when a structure can't be drawn as Mermaid (e.g. emulating a terminal session, embedding YAML inside a process flow). Use `<pre class="code">` blocks.

### Phase 6 — Verify

After writing the HTML, run:

```bash
ls -la ~/learn-{topic-slug}.html
wc -l ~/learn-{topic-slug}.html
grep -c "mermaid" ~/learn-{topic-slug}.html  # expect >=1 if you used any diagrams
```

Report the path and the section count to the user. Do NOT claim verification of "renders correctly" without opening it — if a browser is unavailable, say "unverified against browser; static checks pass."

### Phase 7 — Cleanup

- Close any background delegation tasks you launched.
- Do not commit, push, or share the file unless explicitly asked.
- If the user attached files you couldn't read, disclose that in the HTML (Phase 1 rule) and in your final chat message.

## Output contract (binary pass conditions)

Done means ALL are true:

1. `~/learn-{topic-slug}.html` exists and is non-empty.
2. Opens in any browser with Tailwind + Mermaid rendering (CDN loaded, no install).
3. Contains the section list from Phase 3 (omitted ones disclosed).
4. References section lists actual sources fetched during Phase 2 — not generic placeholders.
5. Diagrams chosen per Phase 5 heuristic; no ASCII fallback used when Mermaid would have worked.
6. If user attached a file the model can't read → disclosed in a callout in the HTML AND in the final chat reply.

## File output location

Default: `~/learn-{kebab-case-topic-slug}.html`

- TopicSlug rules: lowercase, kebab-case, drop articles ("the/a/an"), e.g. "Loop Engineering" → `loop-engineering`.
- If user named a path → use it verbatim.
- If user said "current dir" → use `./learn-{slug}.html`.
- NEVER write outside these paths without explicit permission.

## Reference example

A complete, finished example of what this skill should produce lives at:

```
~/.config/opencode/skills/learn-topic/references/example-loop-engineering.html
```

Open it. Study the section count, the diagram choices, the badge usage, and the references table. That is what "good" looks like. Match its density and visual discipline.

## Anti-patterns (DO NOT)

- Do NOT dump chat-shaped prose into `<pre>` blocks — render real HTML.
- Do NOT skip Phase 2 research and rely on pre-trained knowledge alone. Always fetch >= 3 sources.
- Do NOT use flowery filler ("In today's fast-paced world of..."). Lead with the definition in the first 2 sentences.
- Do NOT invent sources. Every reference at the bottom must have been fetched during this session.
- Do NOT use Tailwind classes that ship in v4-only if the CDN pulls v3 — stick to stable, well-known utility classes. Tailwind CDN pulls stable v3 by default.
- Do NOT use emojis in section headers. The template is plain text + badges.
- Do NOT skip Phase 6 verification.
- Do NOT claim "renders correctly" without opening the file in a browser. State static-checks-only if a browser is unavailable.

## Examples of trigger phrases that map to this skill

| User says | Run /learn-topic |
|---|---|
| "research X and make an HTML writeup" | yes |
| "primer on X with diagrams" | yes |
| "learn X" | yes |
| "make a digestible read on X" | yes |
| "teach me X with a guide" | yes |
| "explain X like a study guide" | yes |
| "I want a shareable doc on X" | yes |
| "explain X" (no HTML request) | NO — just answer in chat |
| "teach me X" (in-workspace skill transfer) | NO — that's `/teach` |
| "learn from past sessions" | NO — that's gstack `/learn` |