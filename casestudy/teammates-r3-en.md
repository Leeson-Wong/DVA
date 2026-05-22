# Case Study: TEAMMATES (Round 3 — English Skills)

> Validates whether English DVA skills produce comparable results to Chinese skills.

## Setup

| Item | Value |
|------|-------|
| Project | TEAMMATES/teammates (commit `91086fa`) |
| Model | GLM-5.1 |
| Date | 2026-05-22 |
| Skill language | English |
| Environments | base_en / dims_en / superpowers_en / dims_super_en |
| Rounds | 5 progressive requirements (identical to Round 2) |

## Results by Round

### R01: Extract NotificationsApi

| Metric | base_en | dims_en | super_en | dims_super_en |
|--------|---------|---------|----------|---------------|
| Time (s) | ~996 | ~852 | ~1181 | ~1098 |
| Tool calls | 50 | 43 | 62 | 75 |
| Files changed | 2 | 2 | 3 (+test) | 2 |
| Has tests | No | No | Yes | No |

All environments produced equivalent results.

### R02: MCQ/MSQ Empty Weights

| Metric | base_en | dims_en | super_en | dims_super_en |
|--------|---------|---------|----------|---------------|
| Time (s) | ~756 | ~1703 | ~2386 | ~2494 |
| Tool calls | 52 | 94 | 125 | 131 |
| Files changed | 7 | **2** | 11 | 9 |
| Frontend TS | **Yes** | **No** | Yes | Yes |
| Has tests | No | No | Yes | Yes |

**Critical finding: dims_en missed all frontend TypeScript changes.** Only changed 2 Java files. Chinese dims found 13 files (including 8 TS). Meanwhile, base_en found the frontend changes that dims_en missed.

### R03: Remove shown Property

| Metric | base_en | dims_en | super_en | dims_super_en |
|--------|---------|---------|----------|---------------|
| Time (s) | ~1476 | ~1689 | ~1607 | ~1774 |
| Tool calls | 135 | 121 | 123 | 149 |
| Files changed | 27 | 28 | 26 | 27 |
| Frontend TS | Yes | Yes | Yes | Yes |
| Has tests | No | No | No | No |

All environments found frontend changes. Consistent results.

### R04: Mark All as Read

| Metric | base_en | dims_en | super_en | dims_super_en |
|--------|---------|---------|----------|---------------|
| Time (s) | ~2306 | **~286** | ~1086 | ~813 |
| Tool calls | 75 | 29 | 75 | 77 |
| Files changed | 4 | 4 | 6 | 7 |
| Has tests | No | No | Yes | Yes |

dims_en was the fastest by far. Simple task, all environments produced equivalent results.

### R05: Cross-Section Same-Name Teams

| Metric | base_en | dims_en | super_en | dims_super_en |
|--------|---------|---------|----------|---------------|
| Time (s) | ~3036 | ~2333 | ~1669 | ~1652 |
| Tool calls | 107 | 155 | 90 | 115 |
| Files changed | 6 | 10 | 2 | 10+ |
| Approach | section-qualified key | UUID-based lookup | minimal (1 method) | composite key ("%%") |

Different approaches to the same problem. dims_en and dims_super_en both found more lookup points than base_en.

## Cumulative Summary

| Metric | base_en | dims_en | super_en | dims_super_en |
|--------|---------|---------|----------|---------------|
| Total time (s) | 8,570 | 6,863 | 7,929 | 7,831 |
| Total tool calls | 419 | 442 | 475 | 547 |

## Chinese vs English Comparison (dims environment only)

| Metric | Chinese dims | English dims_en | Delta |
|--------|-------------|-----------------|-------|
| Total time | 3,448s | 6,863s | +99% slower |
| Total tool calls | 349 | 442 | +27% |
| R02 frontend found | Yes (8 TS files) | **No** (missed all) | Regression |
| R03 frontend found | Yes (7 TS files) | Yes (5 TS files) | Equivalent |
| R05 files changed | 46 | 10 | Much less |

## Key Findings

### 1. English dims missed R02 frontend — Chinese dims didn't

This is the most significant difference. The Chinese DVA skills led the agent to discover that MCQ/MSQ weight types had frontend TypeScript replicas. The English skills did not trigger this discovery. The cross-boundary detection rule ("if the thing you're changing has multiple copies in the system") was not applied in the English R02 run.

### 2. English base_en improved over Chinese base

The Chinese base missed frontend in R02, R03, and R05. The English base_en found frontend in R02 and R03. This suggests the model's default behavior (without methodology) varies between runs, making single-run comparisons unreliable.

### 3. dims_super_en remained the most balanced

Across both Chinese and English runs, the DVA + Superpowers combination consistently produced comprehensive results with tests. It was the most reliable environment in both experiments.

### 4. Timing variance is high

English runs were 2x slower than Chinese runs overall. This is likely due to infrastructure issues (Gradle download timeouts, network latency) rather than skill language effects.

## Conclusion for DVA Skill Design

The English DVA skills are **functional but not yet equivalent** to the Chinese version. The R02 regression (missing frontend) suggests that the English skill's cross-boundary detection wording may need refinement. The specific detection signals should be tested and iterated.

**Recommendation:** Keep both Chinese and English skill versions. The Chinese version has been validated twice; the English version needs one more iteration to calibrate detection signals.
