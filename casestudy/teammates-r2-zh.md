# Case Study: TEAMMATES (Round 2 — Chinese Skills)

> Re-run pending with English skills. This is the Chinese-skill baseline.

## Setup

| Item | Value |
|------|-------|
| Project | TEAMMATES/teammates (commit `91086fa`) |
| Model | GLM-5.1 |
| Date | 2026-05-22 |
| Skill language | Chinese |
| Environments | base / dims / superpowers / dims+superpowers |
| Rounds | 5 progressive requirements |

## Results Summary

| Metric | base | dims | superpowers | dims+super |
|--------|------|------|-------------|------------|
| Total time (s) | 1,636 | 3,448 | 2,184 | 2,993 |
| Total tool calls | 201 | 349 | 266 | 310 |
| Avg time/round | 327s | 690s | 437s | 599s |
| Has tests | No | No | Yes | Yes |
| Has methodology docs | No | Yes (DVA table) | Yes (log) | Both |

## Key Finding: dims Found Changes Others Missed

Post-experiment diff review found that dims' 22 extra file changes were **all necessary**:

- R02 (MCQ/MSQ empty weights): dims found 8 frontend TypeScript files that base missed
- R03 (Remove shown property): dims found 7 frontend TypeScript files that base missed
- R05 (Cross-section teams): dims implemented composite keys across 4 Java files that base missed

**Base environment produced incomplete implementations for R02, R03, and R05** — backend changed but frontend untouched, causing type mismatches and missing references at runtime.

## Skill Calibration Insights

This experiment produced two concrete calibration rules for DVA skills:

1. **Cross-boundary upgrade**: If a requirement affects data types replicated across multiple modules/layers/services, use at least Standard Analysis
2. **"Check existing before creating"**: The v2 SKILL's check-existing step prevented the over-engineering seen in Round 1 (Shopizer)

## Files

- Full report: `reports/teammates-compare.md`
- Requirements: `F:\exam\experiment\teammates\requirements\req-01.md` through `req-05.md`
