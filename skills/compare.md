---
name: compare
description: Diff two metric snapshots (before vs after) and determine if the workflow improved.
argument-hint: "<before-metrics> <after-metrics>"
---

# Compare — Baseline Diffing

## Input

Two aggregate metric sets (from `measure` runs):
- **Before** — previous baseline or iteration
- **After** — current measurement

## Compute Deltas

For each metric, compute: `delta = after - before`

| Metric | Before | After | Delta | Direction |
|--------|--------|-------|-------|-----------|
| Success rate | | | | Higher is better |
| Avg turns | | | | Lower is better |
| Avg duration | | | | Lower is better |
| Avg efficiency | | | | Higher is better |
| Avg cost | | | | Lower is better |
| Avg fallbacks | | | | Lower is better |

## Improvement Check

The workflow **improved** if any of these are true:
1. Success rate increased
2. Success rate unchanged AND (avg turns decreased OR avg cost decreased)

The workflow **regressed** if:
1. Success rate decreased

The workflow is **stagnant** if:
1. No meaningful change in any metric

## Report

```
==============================================================
COMPARISON: <workflow-name>
==============================================================
Metric              Before         After          Delta
--------------------------------------------------------------
Success Rate        60.0%          80.0%          +20.0%
Avg Turns           28.5           18.2           -10.3
Avg Duration        185.0s         120.0s         -65.0s
Avg Efficiency      45%            78%            +33.0%
Avg Cost            $0.4500        $0.2200        -$0.2300
--------------------------------------------------------------
  IMPROVED
==============================================================

Failure Distribution:
  TIMEOUT          3 → 1  (-2)
  TIMING           2 → 0  (-2)
  AUTH             1 → 1  ( 0)
==============================================================
```

## Output

Return the delta object for use by the `optimize` loop to check convergence and stagnation.
