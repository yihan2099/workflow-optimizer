---
name: baseline
description: Persist and load metric snapshots for tracking progress over time.
argument-hint: "<workflow-id> [save|load|list|compare-latest]"
---

# Baseline — Metric Persistence

## Commands

### save

Save the current measurement as a baseline snapshot.

```
/baseline my-workflow save
```

Writes to: `.skill-optimizer/baselines/my-workflow/<date>.md`

Format:
```markdown
# Baseline: my-workflow — <date>

| Metric | Value |
|--------|-------|
| Runs | 5 |
| Success Rate | 80.0% |
| Avg Turns | 18.2 |
| Avg Duration | 120.0s |
| Avg Efficiency | 78% |
| Avg Cost | $0.22 |

## Failure Distribution
| Category | Count |
|----------|-------|
| TIMEOUT | 1 |

## Run Details
| # | Success | Turns | Duration | Cost | Error |
|---|---------|-------|----------|------|-------|
| 1 | PASS | 15 | 95s | $0.18 | |
| 2 | PASS | 20 | 130s | $0.25 | |
| 3 | FAIL | 30 | 300s | $0.35 | TIMEOUT |
| 4 | PASS | 18 | 110s | $0.20 | |
| 5 | PASS | 16 | 100s | $0.19 | |
```

### load

Load the most recent baseline for a workflow.

```
/baseline my-workflow load
```

Reads the latest file from `.skill-optimizer/baselines/my-workflow/`.

### list

List all saved baselines for a workflow.

```
/baseline my-workflow list
```

Output:
```
Baselines for my-workflow:
  2026-02-25.md  (60.0% success, 5 runs)
  2026-02-26.md  (70.0% success, 5 runs)
  2026-02-28.md  (80.0% success, 5 runs)
```

### compare-latest

Compare the two most recent baselines.

```
/baseline my-workflow compare-latest
```

Uses the `compare` skill to diff the last two snapshots.

## Storage Layout

```
.skill-optimizer/
  baselines/
    <workflow-id>/
      <date>.md          # One file per measurement session
  iterations/
    <workflow-id>/
      <date>-iter-1.md   # Iteration details from optimize runs
      <date>-iter-2.md
  learnings/
    <workflow-id>/
      failure-catalog.md # Accumulated failure patterns and fixes
```

## Failure Catalog Format

```markdown
# Failure Catalog: my-workflow

## TIMEOUT
- **Pattern:** Turn limit exceeded at 30 turns
- **Root cause:** Too many sequential commands, agent reasoning between each
- **Fix applied:** Chained commands with &&, reduced from 28 avg to 18 avg turns
- **Date:** 2026-02-26

## TIMING
- **Pattern:** Button not ready after page transition
- **Root cause:** React hydration delay
- **Fix applied:** Added sleep 2 before button click eval
- **Date:** 2026-02-26
```
