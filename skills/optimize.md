---
name: optimize
description: Full optimization loop. Measures a workflow, categorizes failures, applies fixes, and re-measures until target metrics are hit or max iterations reached.
argument-hint: "<workflow-path> [--runs 5] [--target-rate 0.8] [--max-iterations 5] [--baseline-only]"
---

# Optimize — Closed-Loop Workflow Improvement

## Input Parsing

1. **workflow-path** — path to the workflow definition (workflow.md)
2. **--runs N** (default: 5) — runs per iteration
3. **--target-rate N** (default: 0.8) — target success rate (0.0–1.0)
4. **--max-iterations N** (default: 5) — max fix-and-retest cycles
5. **--stagnation-limit N** (default: 3) — stop after N iterations with no improvement
6. **--baseline-only** — measure once and stop, no optimization loop

## The Loop

```
1. BASELINE: measure(workflow, runs=N)
2. CHECK: if success_rate >= target → DONE
3. CATEGORIZE: bucket failures
4. FILTER: only fixable categories
5. FIX: apply fixes from workflow's fixer.md
6. REBUILD: if workflow requires rebuild (e.g., Docker)
7. REMEASURE: measure(updated_workflow, runs=N)
8. COMPARE: diff(previous_metrics, new_metrics)
9. SAVE: persist iteration result
10. CHECK STAGNATION: if no improvement for K iterations → STOP
11. LOOP back to step 2
```

## Phase 1: Baseline

Use the `measure` skill to run the workflow N times.

```
/measure <workflow-path> --runs N
```

Save the result as the baseline. If `--baseline-only`, print the report and STOP.

## Phase 2: Check Target

If `success_rate >= target_rate`, the workflow already meets the target. Print summary and STOP.

## Phase 3: Categorize

Use the `categorize` skill on the failed runs.

```
/categorize <results>
```

If all failures are non-fixable (AUTH, RATE_LIMIT), STOP with explanation.

## Phase 4: Fix

Read the workflow's `fixer.md` file. This file defines how to fix each failure category for this specific workflow. The fixer is where domain knowledge lives.

Example fixer.md structure:
```markdown
# Fixer: post-to-twitter

## TIMEOUT
- Chain sequential commands with &&
- Replace state+click with eval-based clicks
- Remove unnecessary screenshots

## TIMING
- Add sleep 2 before button clicks
- Add retry loop for NOT_READY elements

## UI_CHANGE
- Inspect current DOM structure
- Update querySelector selectors in prompt

## CONTENT
- Switch from IIFE to var-assignment pattern
- Use Array.from() instead of spread
```

Apply the fixes described for the dominant failure category. Make surgical changes — don't rewrite everything.

## Phase 5: Rebuild (If Needed)

If the workflow requires a rebuild step (e.g., Docker container), run it:

```
# Defined in workflow.md under "Rebuild Command"
docker stop my-container && docker rm my-container
docker build -t my-image .
docker run -d --name my-container ...
```

## Phase 6: Remeasure

Use the `measure` skill again with the updated workflow.

## Phase 7: Compare

Use the `compare` skill to diff previous vs current metrics.

```
/compare <previous-metrics> <current-metrics>
```

## Phase 8: Save Iteration

Persist the iteration result:

```
.skill-optimizer/
  baselines/<workflow-id>/<date>.md
  iterations/<workflow-id>/<date>-iter-<N>.md
  learnings/<workflow-id>/failure-catalog.md
```

Iteration file format:
```markdown
# Iteration N — <date>

## Changes Made
- [Description of fixes applied]

## Results
| Metric | Before | After | Delta |
|--------|--------|-------|-------|
| Success Rate | 60% | 80% | +20% |
| Avg Turns | 28.5 | 20.2 | -8.3 |
| ...

## Failure Distribution
| Category | Before | After | Delta |
|----------|--------|-------|-------|
| TIMEOUT | 3 | 1 | -2 |
| ...
```

Append new failure patterns to `failure-catalog.md` for future reference.

## Phase 9: Stagnation Check

Track consecutive iterations with no improvement.

If `stagnation_count >= stagnation_limit`:
- STOP — diminishing returns
- Report what was achieved vs target
- Suggest manual review of remaining failures

## Final Report

```
============================================================
OPTIMIZATION SUMMARY
============================================================
  Converged    : YES/NO
  Iterations   : N
  Total Runs   : M
  Initial Rate : XX.X%
  Final Rate   : XX.X%
  Improvement  : +XX.X%
============================================================
```

If converged, suggest committing the changes. If not, list remaining failure categories that need attention.
