# DVA — Dimensional Variance Analysis

> 同变的归一，异变的拆开。
> *Internalize what changes together, externalize what doesn't.*

A software design analysis framework. One question before you write code: **should this logic live inside the entity, or be split out?**

The criterion is simple — is the rate of change aligned?

## Core Idea

Any software system has exactly two things:

- **Entities (nodes)**: hold state
- **Relations (edges)**: connect entities; complex behavior runs on relations

The core design decision: **where does behavior live?**

- Change rate matches entity → **Internalize** (keep inside)
- Change rate diverges from entity → **Externalize** (split out)

## Three Depths

| Depth | Time | When to use |
|-------|------|-------------|
| Quick Scan | 30 seconds | Before writing any code |
| Standard Analysis | 2 minutes | New features / new modules |
| Full Analysis | 10-15 minutes | New projects / major refactors |

Most of the time, a quick scan is enough. Don't do full analysis every time.

## Quick Start

### For humans

Read [DVA.md](docs/DVA.md) — complete framework description with examples, verification steps, and design pattern derivation.

Chinese version: [变维拆分.md](变维拆分.md)

### For AI Agents

Skills ready for Claude Code / Gemini:

```
skills/
  using-dims/SKILL.md        # Entry: pick the right depth
  change-dim-scan/SKILL.md   # Quick scan (30s)
  change-dim-split/SKILL.md  # Standard + Full analysis
```

### Prompt for any AI

> Before writing code, do a DVA analysis. Pick depth by scenario:
>
> **Daily coding (quick scan):** Think "what will change here?" Leave hooks at unstable points. Don't over-engineer.
>
> **New features (standard analysis):** Find entities and relations. Mark where change rates diverge. Isolate at those points.
>
> **New projects (full analysis):** List all entities and relations. Annotate change rates and trends. Group by consistency. Pick the lightest externalization tool for each split point (config > callback > interface > design pattern).

## Design Patterns = Standard Topologies of Relation Externalization

| What changes in the relation | Topology | Corresponding pattern |
|-----------------------------|----------|-----------------------|
| Who does it | Star | Strategy / State / Visitor |
| How to create | Triangle | Factory / Builder / Prototype |
| Who gets notified | Broadcast | Observer / Mediator |
| When to do it | Transient→Persistent | Command / Memento |
| How to connect | Insert middle node | Facade / Adapter / Bridge / Proxy / Decorator |
| One or many | Tree | Composite / Iterator |
| In what order | Chain | Template Method / Chain of Responsibility |

DVA is the **upstream analysis step** for all these patterns — analyze change rates first, then pick from the toolbox.

## Experiment Validation

Controlled experiments on two open-source projects (4 environments × 5 rounds × 1 model):

| Project | God Class Type | Finding |
|---------|---------------|---------|
| Shopizer (e-commerce) | if-else chains | DVA found comprehensive change scope; v1 over-designed, v2 corrected |
| TEAMMATES (education) | Pure delegation façade | DVA's 22 extra files were all necessary; base missed frontend changes, producing undeployable output |

Detailed reports in [reports/](reports/).

## Project Structure

```
├── README.md              # This file
├── docs/
│   └── DVA.md             # English documentation (AI-translated)
├── 变维拆分.md             # Chinese documentation (original)
├── CLAUDE.md              # Claude Code entry
├── AGENTS.md              # Agents entry
├── GEMINI.md              # Gemini entry
├── skills/                # AI Agent skills
│   ├── using-dims/
│   ├── change-dim-scan/
│   └── change-dim-split/
└── reports/               # Experiment reports
    ├── shopizer-experiment.md
    └── teammates-compare.md
```

## Translation Notice

English documentation (`docs/DVA.md`) is AI-translated from the Chinese original (`变维拆分.md`). If you find translation errors, please [open an issue](../../issues).

## License

MIT
