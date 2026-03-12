---
name: baseline
description: Persist and load metric snapshots for tracking workflow optimization progress over time. Standalone — does not invoke other skills.
argument-hint: "<workflow-id> [save|load|list]"
allowed-tools: Read, Write, Glob
---

# Baseline — Metric Persistence

## Commands

### save

Save current measurement results as a baseline snapshot.

```
/baseline my-workflow save
```

Reads the latest results from `.skill-optimizer/{workflow-id}/baseline.md` and copies to `.skill-optimizer/{workflow-id}/snapshots/{date}.md`.

### load

Load the most recent snapshot.

```
/baseline my-workflow load
```

Reads the latest file from `.skill-optimizer/{workflow-id}/snapshots/`.

### list

List all saved snapshots for a workflow.

```
/baseline my-workflow list
```

Output:
```
Snapshots for my-workflow:
  2026-02-25.md  (60.0% success, 5 runs)
  2026-02-26.md  (70.0% success, 5 runs)
  2026-02-28.md  (80.0% success, 5 runs)
```

## Storage Layout

```
.skill-optimizer/
  {workflow-id}/
    baseline.md              # Current metrics (written by measure/optimize)
    snapshots/
      {date}.md              # Historical snapshots
    iterations/
      iter-1.md              # Optimization iteration details
      iter-2.md
    failure-catalog.md       # Accumulated failure patterns + fixes
```

## Snapshot Format

```markdown
# Snapshot: {workflow-id} — {date}

| Metric | Value |
|--------|-------|
| Runs | 5 |
| Success Rate | 80.0% |
| Avg Duration | 120.0s |

## Failure Distribution
| Category | Count |
|----------|-------|
| TIMEOUT | 1 |

## Run Details
| # | Success | Duration | Error |
|---|---------|----------|-------|
| 1 | PASS | 95s | |
| 2 | PASS | 130s | |
| 3 | FAIL | 300s | TIMEOUT |
| 4 | PASS | 110s | |
| 5 | PASS | 100s | |
```
