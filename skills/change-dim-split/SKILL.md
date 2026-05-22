---
name: change-dim-split
description: "DVA analysis. Standard (2 min) or Full (15 min): identify entities and relations, decide internalize or externalize by change rate alignment, pick the lightest tool."
---

# DVA — Standard Analysis & Full Analysis

**Internalize what changes together, externalize what doesn't.**

## Two Primitives

- **Entity (node)**: holds state, may have simple self-describing behavior
- **Relation (edge)**: connects entities; complex system behavior runs on relations

Core decision: **where does behavior live?** Aligned change rates internalize into entities; misaligned ones externalize out.

## Two Modes

### Standard Analysis (2 minutes)

Run three steps mentally — no need to fill tables:

1. **Find entities and relations**: What entities does this feature involve? How do they connect?
2. **Mark change hotspots**: Which relation changes at a different pace than its entity?
3. **Decide**: Plan to externalize misaligned relations. Start with the lightest tool.

Done marking? Write code.

### Full Analysis (10-15 minutes)

The system is so tangled that changing one thing breaks three others, or it's a new project. Full five-step process needed.

---

## Full Process

### Step 1: Identify Entities and Relations

List all entities and their relationships in the system.

Example — e-commerce order system:

| Entity | Relation | What the relation does |
|--------|----------|----------------------|
| Order | — calculates shipping → | Shipping rules |
| Order | — state transitions → | State machine |
| Order | — calls payment → | Payment provider |
| Order | — syncs inventory → | Inventory system |
| Order | — contains → | Order items |

### Step 2: Annotate Change Rates

For each relation, annotate its change rate and trend:

| Relation | Rate | Trend |
|----------|------|-------|
| Order → Shipping rules | High | Oscillating |
| Order → State machine | Low | Stable |
| Order → Payment provider | Medium | Growing |
| Order → Inventory system | High | Oscillating |
| Order → Order items | Low | Stable |

### Step 2.5: Check Existing Code

**Before creating anything new, check what already exists.** For each misaligned relation, ask:

> Does existing code already have an interface, abstraction layer, or configuration mechanism covering this rate of change?

- Yes, and it covers the current need → **skip, don't create**. Note "already externalized, no new work needed."
- Yes, but insufficient → evaluate extending the existing one vs. creating new. Prefer extending.
- No → proceed to Step 3, prepare to create.

**This is the critical step to prevent over-engineering.** DVA's value isn't just "telling you how to split" — it also tells you "you don't need to split."

### Step 3: Group by Change Rate

**Group together what shares the same rate and trend; separate what doesn't.**

- **Order backbone**: Order entity + Order items + State transitions → low rate, stable → group together, internalize
- **Shipping strategy**: Shipping rules → high rate, oscillating → split from backbone, externalize
- **Payment adapter**: Payment provider → medium rate, growing → separate group, externalize, will keep expanding
- **Inventory sync**: Inventory data → high rate, oscillating → separate group, externalize, independent changes

### Step 4: Pick Externalization Tools

For each relation that needs externalization, **pick from lightest to heaviest**:

| What changes in the relation | Lightest tool | Heavier tool |
|-----------------------------|---------------|-------------|
| A value changes often | Config file / env variable | — |
| A piece of logic swaps often | Callback / function parameter | Strategy pattern |
| Rules grow and shrink | Data table / JSON config | Rule engine |
| Implementation swaps, interface stays | Abstract an interface | Bridge pattern |
| New types keep appearing | Registry | Factory pattern |
| Internal complexity, external simplicity | Wrap in a function | Facade pattern |
| Hierarchy deepens | Uniform interface, recursive composition | Composite pattern |
| Process steps adjust often | Pipeline / middleware chain | Chain of Responsibility |
| Operations need undo | Record the operation object | Command pattern |

**Principle: if a config file solves it, don't write a Strategy. If a wrapper function solves it, don't build a class hierarchy. Use the heavy hammer only when necessary.**

**Must-answer before creating: every proposed new file, interface, or class must answer "why don't existing ones work?" If you can't answer, don't create it.**

### Step 5: Verify

**Round 1: Did it hold? (Protect the present)** — must pass all

1. **Functional**: When requirements change, can you adapt by changing one place?
2. **Stability**: Does changing A break B?
3. **Robustness**: Can it handle unexpected changes?

**Round 2: How well did it hold? (Protect the future)** — calibrate by project stage

4. **Isolation**: Does modifying this module require touching others?
5. **Evolvability**: After major requirement changes in 3 months, can you iterate or must you rewrite?
6. **Maintainability**: Can you understand this code 6 months from now?
7. **Testability**: After changes, can you quickly verify nothing broke?

Relax Round 2 for prototypes; production systems must pass all.

---

## Anti-Patterns

- **Defaulting to DDD.** DVA analysis doesn't always lead to domain models. Config files, callbacks, data-driven approaches are all valid. Don't use a sledgehammer to crack a nut.
- **Analyzing but not implementing.** Identifying what to externalize but not doing it is wasted analysis.
- **Chasing perfect boundaries.** No boundary is perfect. Holding against current changes is enough; future changes can be handled when they arrive.
- **Designing for hypothetical futures.** "What if we need to add X later" — wait until you actually do. YAGNI.
- **Externalizing what doesn't change differently.** Don't force-split relations with aligned change rates; internalizing them keeps things clearer.

## Key Principles

- **Internalize what changes together, externalize what doesn't** — same rate and trend → internalize; different → externalize
- **Lightest tool first** — config > callback > data-driven > interface abstraction > design pattern
- **Present over future** — Round 1 verification must pass; Round 2 is calibrated by project stage
- **Don't preset the path** — DVA is an upstream step; analyze first, then pick from the toolbox

## Design Patterns = Standard Topologies of Relation Externalization

Most design patterns are standard topological shapes produced by externalizing relations:

| What changes in the relation | Topology after externalization | Corresponding pattern |
|-----------------------------|-------------------------------|-----------------------|
| Who does it | Star (delegates to swappable leaves) | Strategy / State / Visitor |
| How to create | Triangle (consumer → factory → product) | Factory / Builder / Prototype |
| Who gets notified | Broadcast (fan-out from center to subscribers) | Observer / Mediator |
| When to do it | Transient edge → persistent node | Command / Memento |
| How to connect | Insert middle node for translation or simplification | Facade / Adapter / Bridge / Proxy / Decorator |
| One or many | Tree (recursive composition, uniform interface) | Composite / Iterator |
| In what order | Chain (pass along the chain) | Template Method / Chain of Responsibility |

Not all patterns are purely about externalization (Singleton manages instance constraints, Flyweight manages memory, Interpreter manages syntax). But most patterns can be derived from "what changes in the relation."

## Glossary

| Term | Meaning |
|------|---------|
| Entity | A node that holds state |
| Relation | An edge connecting entities; system behavior runs on relations |
| Internalize | Relation converges into the entity; entity becomes richer |
| Externalize | Relation splits from the entity, becoming an independent entity or configuration |
| Change rate alignment | Whether behavior changes at the same pace as the entity itself |
| Atom | The smallest unit with consistent change rates |
| DVA (Dimensional Variance Analysis) | Splitting relations with inconsistent change rates along their boundaries |
