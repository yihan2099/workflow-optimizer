---
name: measure
description: Run a workflow N times and compute aggregate metrics. The foundation of all optimization — you can't improve what you don't measure.
argument-hint: "<workflow-path> [--runs 5] [--parallel] [--json <path>]"
---

# Measure — Run N Times, Collect Metrics

## Input Parsing

1. **workflow-path** — path to the workflow definition (workflow.md)
2. **--runs N** (default: 5) — number of executions
3. **--parallel** — run fixtures concurrently (default: sequential)
4. **--json path** — export results to JSON

## Phase 1: Load Workflow

Read the workflow.md file. Extract:
- Run command template
- Success criteria (patterns, exit codes)
- Fixtures list
- Constraints (timeout, max turns)

If the workflow has a setup step, run it first.

## Phase 2: Execute Runs

For each run (1..N):
1. Pick fixture: cycle through fixtures round-robin (run 1 → fixture 1, run 2 → fixture 2, ...)
2. Substitute fixture inputs into the run command template
3. Start timer
4. Execute the run command
5. Stop timer, record duration
6. Check success criteria against output
7. Record result:

```
Run Result:
  - success: boolean
  - artifact: captured output (URL, file path, etc.)
  - error: error message if failed
  - duration_ms: wall clock time
  - raw_output: full output text
  - turns: agent turn count (if applicable)
  - tokens: { input, output } (if applicable)
  - cost: dollar cost (if applicable)
```

If a run exceeds the timeout, mark it as failed with error "TIMEOUT".

## Phase 3: Parse Turn Metrics (Agent Workflows Only)

If the workflow is an agent workflow (raw_output contains turn markers), parse efficiency:

### Turn Classification

**Efficient turns** — contain actual work:
- Shell commands (bash, curl, python, etc.)
- Browser interactions (click, type, navigate, eval)
- Tool calls (tool_use, tool_result)

**Wasted turns** — no useful work:
- Text-only responses ("I'll now...", "Let me...", "Looking at...")
- Redundant observations without action

### Metrics Extracted

| Metric | Formula |
|--------|---------|
| Efficient turns | Count of turns with useful actions |
| Wasted turns | Count of text-only turns |
| Efficiency ratio | efficient / total |
| Fallback count | Occurrences of fallback patterns (e.g., NOT_FOUND) |
| Retry count | Occurrences of retry patterns (e.g., NOT_READY) |
| Recovery count | Successful recoveries after fallbacks |
| Recovery rate | recoveries / fallbacks |

## Phase 4: Compute Cost (If Token Data Available)

If the runner reports token usage:

```
cost = (input_tokens / 1,000,000) * input_price + (output_tokens / 1,000,000) * output_price
```

Common pricing:
| Model | Input $/MTok | Output $/MTok |
|-------|-------------|--------------|
| Sonnet 4.6 | $3 | $15 |
| Opus 4.6 | $15 | $75 |
| Haiku 4.5 | $0.80 | $4 |

## Phase 5: Aggregate

Compute across all N runs:

| Metric | Formula |
|--------|---------|
| Success rate | pass_count / N |
| Avg turns | mean(turns) for runs with turn data |
| Avg duration | mean(duration_ms) |
| Avg efficiency | mean(efficiency_ratio) for runs with turn data |
| Avg fallbacks | mean(fallback_count) |
| Avg cost | mean(cost) for runs with cost data |
| Failure distribution | count per failure category |

## Phase 6: Report

Print a summary table:

```
============================================================
  <workflow-name> — N runs
============================================================
  Success Rate : XX.X%
  Avg Turns    : XX.X
  Avg Duration : XX.Xs
  Avg Efficiency: XX%
  Avg Cost     : $X.XXXX
  Failures:
    TIMEOUT        2
    TIMING         1
============================================================
```

## Output

Return the aggregate metrics as structured data for use by other skills (compare, optimize).

If `--json` specified, write the full results (all run results + aggregates) to the given path.
