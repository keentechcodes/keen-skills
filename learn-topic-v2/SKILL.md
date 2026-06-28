---
name: learn-topic-v2
description: >
  Research any topic and render a self-contained HTML primer — Mermaid diagrams,
  cited sources, saved to ~/learn-<topic>.html. Trigger when the user wants a
  digestible HTML guide on a topic. Distinct from /teach (skill transfer) and
  gstack /learn (project learnings).
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

# /learn-topic-v2 — Research + render a digestible HTML primer

Produce a **self-contained HTML primer** about a topic the user names. The file must:

1. Open in any modern browser (Chromium, Firefox, Safari) with no external install.
2. Render Mermaid diagrams and tables via CDN.
3. Read like a well-edited primer: clear sections, scannable headings, diagrams before prose where a diagram carries the idea.
4. Cite sources fetched during research at the bottom — _fetched-only_, never invented.
5. Live at `~/learn-<topic-slug>.html` unless the user named a different path.

## Workflow (6 phases, run in order)

### Phase 1 — Settle the contract

Confirm:
- Topic name (use the user's exact wording for the title).
- Any reference URLs/files the user supplied.
- Output path: `~/learn-<kebab-case-topic-slug>.html` by default. Slug rules: lowercase, kebab-case, drop articles ("the/a/an") — e.g. "Loop Engineering" → `loop-engineering`. Override only if the user named a path or said "current dir" (then `./learn-<slug>.html`).
- Scope: comprehensive primer (TL;DR + mental models + diagrams + examples + failure modes + references) by default. Narrow only if the user asks for a focused subset.

If the user attached a binary file (PDF, image, audio) this model cannot read, say so explicitly in the final HTML — never silently skip it. Run the research without that source and disclose it in a callout.

Done when topic name, output path, and scope are written down. Defaults accepted silently if the user gave a bare topic.

### Phase 2 — Parallel research

Fire in parallel:
- `exa_web_search_exa` — 2 queries from different angles (concept + "primer/explainer/example").
- `websearch_web_search_exa` — 1 backup query if exa returns thin coverage.
- For any GitHub repo in results → `exa_web_fetch_exa` on the repo URL + raw README URL.
- For any canonical essay/post in results → `exa_web_fetch_exa` on that URL.
- If the topic has a library/framework → `context7_resolve-library-id` + `context7_query-docs` for current docs.

Optional delegation when coverage is broad:
- `task(subagent_type="explore", run_in_background=true)` — only if the topic lives inside the user's repo AND the user wants repo-specific concepts included.
- `task(subagent_type="librarian", run_in_background=true)` — only when the topic is about a specific library/SDK with versioned behavior the LLM may not know.

Stop when sources _converge_ — usually 3–5 authoritative sources cover a concept. Do not keep pinging to feel safer.

### Phase 3 — Synthesize the structure

Every primer MUST have these sections (omittable only when truly N/A, with a one-line reason in the final HTML):

1. **TL;DR / What it is** — 1–2 paragraphs + 1–2 anchor quotes from named authorities.
2. **The shift / mental model** — why this concept exists and what it replaces or sits above. Use a visual stack (4 boxes or a ladder).
3. **Core cycle / primitives** — the repeating loop or building blocks. Mermaid flowchart preferred.
4. **Anatomy / how it fits together** — Mermaid flowchart that shows the whole system end-to-end.
5. **Design levers / tradeoffs** — table.
6. **Key patterns** — table or grid, with badges for cost/risk where relevant.
7. **Worked example** — Mermaid sequenceDiagram or numbered steps with code block. Minimal but concrete.
8. **Formal layer (optional)** — only if a spec / rubric / scoring system exists for the topic. YAML or JSON block.
9. **Failure modes / anti-patterns** — grid of cards in red/dark, each with name + symptom + mitigation.
10. **Cost / safety / defaults** — bulleted or callout.
11. **Getting started in 5 minutes** — copyable code block.
12. **Decision tree for the reader** — Mermaid flowchart: "should YOU do this? if X, then Y."
13. **Checklist before applying** — interactive-style checkboxes (visual only).
14. **Caveats / honest limits** — bulleted.
15. **References** — table with name + what it's good for; link URL as anchor text. _Fetched-only_ — every entry was pulled during Phase 2.
16. **Bottom line** — dark CTA-style card with a one-paragraph summary.

Sections are a menu, not a checklist — omit what a topic genuinely lacks (e.g. "formal spec" for a soft concept), don't pad.

### Phase 4 — Render HTML

Copy the scaffold verbatim from [`references/template.html`](references/template.html) — open it, swap only the content. It carries Tailwind CDN + Mermaid v11 ESM, the `max-w-5xl` container, `font-serif` headings, `bg-stone-50` body, white cards with `border border-slate-200`, rounded-xl, and the visual-convention classes (`deep`, `pipe`, `badge-strong`, `badge-worth`, `badge-spec`, `tag`, `punch`, `quote-pull`, `fail-bg`, `lift-bg`, `lift-bw`) — all documented inside the template as an HTML comment.

One `<section>` per content block from Phase 3. Close with the dark "Bottom line" card.

### Phase 5 — Pick diagrams

Use the heuristic in [`references/diagram-heuristic.md`](references/diagram-heuristic.md) — concept shape → Mermaid type (flowchart / sequenceDiagram / stateDiagram-v2) or HTML grid / table / `<pre class="code">`. Never default to ASCII when Mermaid works.

### Phase 6 — Verify and clean up

After writing the HTML, run:

```bash
ls -la ~/learn-<topic-slug>.html
wc -l ~/learn-<topic-slug>.html
grep -c "mermaid" ~/learn-<topic-slug>.html  # expect >=1 if you used any diagrams
```

Report the path and the section count to the user. Do not claim "renders correctly" without opening the file in a browser — if a browser is unavailable, say "static checks pass; browser render unverified."

Then close any background delegation tasks you launched. If the user attached files you couldn't read, disclose that in the HTML (Phase 1 rule) and in your final chat message.

## Output contract (binary pass conditions)

Done means ALL are true:

1. `~/learn-<topic-slug>.html` exists and is non-empty.
2. Opens in any browser with Tailwind + Mermaid rendering (CDN loaded, no install).
3. Contains the section list from Phase 3 (omitted ones disclosed).
4. References section lists _fetched-only_ sources from Phase 2 — no placeholders, no inventions.
5. Diagrams chosen per `references/diagram-heuristic.md`; no ASCII fallback used when Mermaid would have worked.
6. If user attached a file the model can't read → disclosed in a callout in the HTML AND in the final chat reply.

## Reference example

A complete, finished example of what this skill should produce lives at:

```
~/.config/opencode/skills/learn-topic-v2/references/example-loop-engineering.html
```

Open it. Study the section count, the diagram choices, the badge usage, and the references table. That is what "good" looks like. Match its density and visual discipline.
