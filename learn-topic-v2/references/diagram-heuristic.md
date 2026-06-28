# Diagram heuristic — learn-topic-v2

Pick the right diagram type per section. NEVER default to ASCII when Mermaid works.

## Concept shape → diagram type

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

## Mermaid rules

- Always wrap in `<pre class="mermaid">` so Tailwind doesn't style-break it.
- Use `classDef` to color-code leaks/risks the way the template does.
- Keep node labels short — Mermaid wraps; long labels break rendering.
- Sequence diagrams: 4–8 actors max. More → split into two diagrams.
- Test mentally: does the graph close back to its start? Cycles are fine for "loops"; open chains for "pipelines."

## ASCII fallback

Only when a structure can't be drawn as Mermaid (e.g. emulating a terminal session, embedding YAML inside a process flow). Use `<pre class="code">` blocks.
