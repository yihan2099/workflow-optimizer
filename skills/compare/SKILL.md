---
name: compare
description: Diff two metric snapshots (before vs after) and determine if a workflow improved, regressed, or stagnated. Standalone — does not invoke other skills.
argument-hint: "<before-path> <after-path>"
allowed-tools: Read
---

# Compare — Metric Diffing

## Input

Two metric files (baseline or iteration results):
- **before-path** — previous metrics
- **after-path** — current metrics

Read both files and extract: success rate, avg duration, failure distribution.

## Compute Deltas

For each metric: `delta = after - before`

| Metric | Before | After | Delta | Better if |
|--------|--------|-------|-------|-----------|
| Success rate | | | | Higher |
| Avg duration | | | | Lower |

## Verdict

**Improved** = success rate increased, OR same rate with lower duration.
**Regressed** = success rate decreased.
**Stagnant** = no meaningful change in any metric.

## Report

```
==============================================================
COMPARISON: {workflow-name}
==============================================================
Metric              Before         After          Delta
--------------------------------------------------------------
Success Rate        60.0%          80.0%          +20.0%
Avg Duration        185.0s         120.0s         -65.0s
--------------------------------------------------------------
  IMPROVED
==============================================================

Failure Distribution:
  TIMEOUT          3 → 1  (-2)
  TIMING           2 → 0  (-2)
  AUTH             1 → 1  ( 0)
==============================================================
```
