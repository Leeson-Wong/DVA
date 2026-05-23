---
name: change-dim-scan
description: "Quick Scan: 30-second DVA analysis. Ask one question — 'what will change here?' — judge if change rates are aligned, leave hooks where they aren't."
---

# Quick Scan — 30-Second DVA Analysis

**Covers 80% of scenarios.** A flash of thought before writing code is all it takes.

## How

Ask yourself one question: **"What will change here?"**

Then judge in business language:

| Question to ask yourself | Example |
|--------------------------|---------|
| "Will the approach change often?" | Shipping rules adjusted monthly, discount algorithms frequently revised |
| "Will who does it change?" | Integrating a new payment provider, switching logistics vendors |
| "Will things keep multiplying?" | Product categories growing from 10 to 100 |
| "Will steps or order change?" | Approval flow going from 3 steps to 5 |
| "Will the interface change?" | Migrating from REST to gRPC |
| "Will business rules change often?" | Coupon threshold adjustments, membership tier rule changes |

For each change point, quickly judge: **does this change at the same pace as the entity?**
- Same pace → leave it alone, internalize in the entity
- Different pace → **leave a hook** (make it configurable, a callback, or a parameter)

**Done thinking? Write code.** No tables to fill, no documents to write.

## Example

> Building an email sending feature.
>
> Quick scan: recipient addresses will change, email templates will change, sending channel might switch.
>
> Judgment: recipient and template follow the email entity — internalize. Sending channel (SMTP / API) has an independent change rate — externalize as configurable.
>
> Remember this judgment → write code.

## Anti-Patterns

- **Overthinking.** Quick scan is a flash of thought. If it takes more than 30 seconds, you should upgrade to Standard Analysis.
- **Scanning but not leaving hooks.** Identifying misaligned change rates but hard-coding them anyway is the same as not scanning.
- **Making everything configurable.** Don't over-abstract things with aligned change rates.

## When to Upgrade

Upgrade to `change-dim-split` (Standard Analysis) if Quick Scan reveals any of these:

- More than 3 misaligned change rates
- Multiple entities involved
- Uncertainty about whether changes affect each other
- The code being modified already has quality issues
- **你正在改的东西在系统里有副本** — a field, type, or logic you're modifying appears in more than one place (backend AND frontend, multiple services, duplicated across layers). Search the codebase (`grep`, `find`) for all occurrences before proceeding. Missing copies produces broken builds. Use at least Standard Analysis.
