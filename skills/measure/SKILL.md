---
name: measure
description: Run a workflow N times and compute aggregate metrics (success rate, duration, failures). Standalone — does not invoke other skills.
argument-hint: "<workflow-path> [--runs 5]"
allowed-tools: Bash, Read, Write, Glob, Agent
---

# Measure — Run N Times, Collect Metrics

## Input Parsing

1. **workflow-path** — path to the workflow definition (workflow.md)
2. **--runs N** (default: 5) — number of executions

## Phase 1: Load Workflow

Read the workflow.md file. Extract:
- **Run Command** — shell command or agent prompt template
- **Success Criteria** — exit code, output pattern, URL pattern
- **Fixtures** — test inputs (cycled round-robin)
- **Constraints** — timeout (ms)

If the workflow has a setup step, run it first.

## Phase 2: Execute Runs

For each run (1..N):
1. Pick fixture: `fixtures[i % fixtures.length]`
2. Substitute fixture values into the run command
3. Start timer
4. Execute via Bash tool (shell commands) or describe the agent invocation
5. Stop timer, record duration
6. Check success criteria against output
7. Record result

### Run Result

```
Run {i}:
  success: PASS / FAIL
  fixture: {fixture_id}
  duration_ms: {wall clock time}
  error: {error message or "none"}
  output: {first 500 chars of stdout}
```

If a run exceeds the timeout, mark it as FAIL with error "TIMEOUT".

### Parallel Runs

For workflows with no shared state between runs, use the `Agent` tool to run subsets concurrently (up to 3 agents). Each agent writes results to `.workflow-optimizer/{workflow-id}/runs/run-{i}.md`.

## Phase 3: Aggregate

Compute across all N runs:

| Metric | Formula |
|--------|---------|
| Success rate | pass_count / N |
| Avg duration | mean(duration_ms) |
| Failure distribution | count per error type |

## Phase 4: Report

```
============================================================
  {workflow-name} — {N} runs
============================================================
  Success Rate : XX.X%
  Avg Duration : XX.Xs
  Failures:
    TIMEOUT        2
    TIMING         1
============================================================
```

## Output

Write results to `.workflow-optimizer/{workflow-id}/baseline.md` with the aggregate metrics and per-run details.
