# keen-skills

A collection of my personal skills for coding agents.

These skills are agent-agnostic — they work with any coding agent that supports a `SKILL.md` / slash-command style skill format. I mainly use them with:

- **Claude Code**
- **OpenCode**
- **Hermes**

Use whichever agent you prefer; the skills don't depend on a specific one.

## Skills

| Skill | What it does |
|-------|--------------|
| [`done`](./done) | Save a session handoff to `docs/sessions/` so a fresh agent can pick up where you left off. |
| [`pickup`](./pickup) | Restore context from a previous session's handoff document. |
| [`learn-topic`](./learn-topic) | Research a topic and render a self-contained HTML primer with diagrams and cited sources. |
| [`learn-topic-v2`](./learn-topic-v2) | Iteration on `learn-topic` — adds stricter sourcing, a diagram heuristic, and an HTML template. |

## Layout

```
keen-skills/
├── done/
│   └── SKILL.md
├── pickup/
│   └── SKILL.md
├── learn-topic/
│   ├── SKILL.md
│   └── references/
│       └── example-loop-engineering.html
└── learn-topic-v2/
    ├── SKILL.md
    └── references/
        ├── diagram-heuristic.md
        ├── example-loop-engineering.html
        └── template.html
```

Each skill lives in its own folder with a `SKILL.md` at the root. Some skills include a `references/` directory with examples, templates, or heuristics the skill pulls in at runtime.

## License

MIT
