> **Note:** This document is AI-translated from the Chinese original ([变维拆分.md](../变维拆分.md)). If you find translation errors, please [open an issue](../../issues).

# DVA (Dimensional Variance Analysis)

> Internalize what changes together, externalize what doesn't.

## What Is This

DVA is a software design analysis tool. It helps you think clearly before writing code: **when requirements change, your code only needs one spot updated to keep up — not a cascade of breakage across the entire system.**

You don't need to know design patterns. You don't need to know architecture. You only need to understand two things.

## Two Primitives

Any software system, at its core, contains only two kinds of things:

- **Entity (node):** Holds state. It may have simple self-describing behavior (only touching its own data), or no behavior at all.
- **Relation (edge):** Connects entities. The complex behavior of a system is, in essence, relations at work.

The core design decision is just one: **where behavior lives.** Should a piece of logic live inside an entity, or outside it?

The criterion is **whether the rate of change is aligned**:

- Does this logic change at the same pace as the entity itself? → **Internalize** (fold it into the entity)
- Not the same, or frequently in conflict? → **Externalize** (split it out, into an independent entity or external configuration)

Example:
- `totalPrice = sum(items.price)` — simple summation, lives and dies with the order → **Internalize** into the Order entity
- Shipping rules change monthly, order structure hasn't changed in six months → paces are misaligned → must **Externalize**

## How to Use

DVA doesn't require a full walkthrough every time. Choose your depth based on the situation:

### Quick Scan (30 seconds)

**When to use:** Before writing any code. Covers 80% of scenarios.

**How:** Ask yourself — "what will change here?"

| What to ask yourself | Example |
|---------------------|---------|
| "Will the approach change often?" | Shipping rules adjusted monthly, discount algorithms frequently revised |
| "Will who does this change?" | Integrating a new payment channel, switching logistics providers |
| "Will there be more and more of these?" | Product categories expanding from 10 to 100 |
| "Will steps and their order change often?" | Approval flow going from 3 steps to 5 |
| "Will the calling convention change?" | Migrating from REST to gRPC |
| "Will business rules change often?" | Coupon threshold adjustments, membership tier rule changes |

Got an answer → the rate of change may be misaligned, so leave an opening in your code at that spot.
No answer → don't worry about it, just write the code.

**Example:**
> You're about to write an email sending feature. Quick Scan: recipient addresses will change, email templates will change, the sending channel might be swapped. Keep those three things in mind, and you'll naturally make them configurable when you write the code.

### Standard Analysis (2 minutes)

**When to use:** New features, new modules, or refactoring problematic existing modules.

**How:** Three steps — no documentation needed.

1. **Find entities and relations:** What entities does this feature involve? How are they connected?
2. **Mark change hotspots:** Which relation's pace of change is misaligned with its entity?
3. **Decide:** Externalize the misaligned relations. Start with the lightest approach.

Once you've mapped it out in your head, start writing code.

### Full Analysis (10-15 minutes)

**When to use:** Starting a new project, major refactoring, or a system where touching one thing breaks three others.

**How:** Walk through the complete process below. It's recommended to write down the variance spectrum table for future reference.

#### Step 1: Identify Entities and Relations

List all entities involved in the system and the relations between them.

Example — E-commerce order system:

| Entity | Relation | What the relation does |
|--------|----------|----------------------|
| Order | — calculates shipping → | Shipping rules |
| Order | — state transition → | State machine |
| Order | — calls payment → | Payment channel |
| Order | — syncs inventory → | Inventory system |
| Order | — contains → | Order items |

#### Step 2: Mark Rates of Change

For each relation, annotate its rate of change and trend:

| Relation | Rate | Trend |
|----------|------|-------|
| Order → Shipping rules | High | Oscillating |
| Order → State machine | Low | Stable |
| Order → Payment channel | Medium | Growing |
| Order → Inventory system | High | Oscillating |
| Order → Order items | Low | Stable |

#### Step 3: Group by Rate of Change

**Group together what shares the same rate and trend; split apart what doesn't.**

- **Order skeleton:** Order entity + Order items + State transitions → low rate, stable → group together, Internalize
- **Shipping strategy:** Shipping rules → high rate, oscillating → split from the skeleton, Externalize
- **Payment adaptation:** Payment channel → medium rate, growing → separate group, Externalize, will keep expanding
- **Inventory sync:** Inventory data → high rate, oscillating → separate group, Externalize, changes independently

#### Step 4: Choose Externalization Techniques

For each relation that needs externalization, start with the lightest option:

| What changes about the relation | Lightest approach | Heavier approach |
|-------------------------------|-------------------|------------------|
| A value changes often | Config file / environment variable | — |
| A piece of logic is swapped often | Callback / function parameter | Strategy pattern |
| Rules are frequently added or removed | Data table / JSON config | Rules engine |
| Implementation changes, interface stays the same | Add an interface layer | Bridge pattern |
| New types keep appearing | Registry | Factory pattern |
| Internally complex, needs a simple exterior | Wrap in a function | Facade pattern |
| Hierarchy will deepen | Unified interface, recursive composition | Composite pattern |
| Process steps change often | Pipeline / middleware chain | Chain of Responsibility |
| Operations need to be undoable | Record operation objects | Command pattern |

**Principle: If a config file solves it, don't write a Strategy. If adding a function layer solves it, don't build a class hierarchy. Use a heavy hammer only when necessary.**

#### Step 5: Validate

Check in two rounds.

**Round 1: Does it hold up? (protect the present)** — must pass all

1. **Functional viability:** When requirements change, can you keep up by changing one place?
2. **Stability:** Does changing A break B?
3. **Robustness:** When unexpected changes arrive, can the system absorb them?

**Round 2: How well does it hold up? (protect the future)** — calibrate based on project stage

4. **Isolation:** Does modifying this module require touching other modules?
5. **Evolvability:** If requirements shift dramatically in three months, can you keep iterating, or do you need a rewrite?
6. **Maintainability:** Will this code still be readable six months from now?
7. **Testability:** After a change, can you quickly verify nothing broke?

In the prototyping stage, Round 2 can be relaxed. For production systems, all items must pass.

**You can upgrade between levels — no need to get everything right in one shot.** Start with a Quick Scan and write code. If issues emerge, pause and do a Standard Analysis. If that's not enough, move up to a Full Analysis.

## Design Patterns and DVA

Most design patterns are **standard topologies for externalized Relations**. When you take an implicit relation and make it explicit — an independently variable entity — you naturally arrive at a particular design pattern.

| What changes about the relation | Topology after externalization | Corresponding pattern |
|-------------------------------|-------------------------------|----------------------|
| Who does it | Star: center delegates to replaceable leaf nodes | Strategy / State / Visitor |
| How to create | Triangle: consumer → factory → product | Factory / Builder / Prototype |
| Who gets notified | Radiating: center fans out to subscribers | Observer / Mediator |
| When to do it | Transient edge solidifies into a persistent node | Command / Memento |
| How to connect | Insert a middle node for translation or simplification | Facade / Adapter / Bridge / Proxy / Decorator |
| One or many | Tree: recursive composition, unified externally | Composite / Iterator |
| In what order | Chain: pass along the chain for processing | Template Method / Chain of Responsibility |

Not all 23 patterns are purely about externalization. Singleton manages instance count constraints, Flyweight manages memory optimization, Interpreter manages grammar expression — each has its own motivation. DVA explains where most patterns come from, but shouldn't be forced to explain all of them.

## Relationship to Existing Theories

DVA does not replace any existing method. It is an **upstream analysis step** that feeds into all of them:

```
DVA (analyze entities, relations, rates of change)
  ↓
Decide where behavior lives (Internalize or Externalize)
  ↓
Choose externalization technique:
  ├── Lightest: config, callbacks, data-driven
  ├── PEAA patterns: Transaction Script / Table Module / Domain Model
  ├── GoF design patterns: Strategy, Facade, Composite...
  ├── DDD tactics: Aggregates, Value Objects, Domain Events
  └── Any other suitable technique
```

Analyze first, then choose your tools. Don't presuppose a path.

## Instructions for AI Agents

If you use AI to write code (Claude, Cursor, Copilot, etc.), give the AI this prompt:

> Before writing code, perform a DVA analysis. Choose depth based on the situation:
>
> **Day-to-day coding (Quick Scan):** Think about "what will change here," and leave configuration hooks or callback parameters at the spots most likely to change. Do not over-engineer.
>
> **New features / new modules (Standard Analysis):** Find entities and relations, mark change hotspots (which relation's pace of change is misaligned with its entity), and isolate at the misalignment points. Do not default to any architectural framework.
>
> **New projects / major refactoring (Full Analysis):** List all entities and relations, annotate rates of change and trends, group by alignment, and for each split point choose the lightest technique available (config > callback > interface > design pattern). Validate for functional viability, stability, and robustness.
>
> Most of the time, a Quick Scan is enough. Do not perform a Full Analysis every time.

## Limits of the Model

DVA uses rate-of-change alignment as its **primary criterion**, but not the only one. Real-world design also requires considering:

- **Readability:** Sometimes externalization makes code harder to read, and it's worth internalizing even when the rate of change is misaligned
- **Team habits:** Patterns unfamiliar to the team can increase cognitive load rather than reduce it
- **System consistency:** Maintaining consistency with the existing architectural style is sometimes more important than local optimality
- **Constraint-driven patterns:** Singleton, Flyweight, and others solve constraint problems, not change problems

## Glossary

| Term | Meaning | In one sentence |
|------|---------|-----------------|
| Entity | A node that holds state | Order, User, Product |
| Relation | An edge connecting entities; system behavior runs along relations | "Order calculates shipping," "Order calls payment" |
| Internalize | Relation folds into the entity, enriching it | totalPrice written inside Order |
| Externalize | Relation is peeled off the entity, becoming an independent entity or configuration | Shipping calculation split into a standalone PricingService |
| Rate-of-change alignment | Whether the pace of behavior change syncs with the entity itself | Aligned → Internalize, Misaligned → Externalize |
| Atom | The smallest unit with a consistent rate of change | "Order skeleton" or "Shipping strategy" |
| DVA | Splitting relations with inconsistent rates of change along their boundaries | "Internalize what changes together, externalize what doesn't." |

---

*Software design is not about sculpting entities — it's about managing where behavior lives. Internalize what should be internalized, externalize what should be externalized. The criterion is whether the rate of change is aligned.*
