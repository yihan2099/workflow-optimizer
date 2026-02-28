# skill-optimizer

A methodology for iteratively measuring and optimizing agent workflows. Defined as markdown skills that any LLM agent can pick up and execute.

## What This Is

A set of composable skills that implement a closed-loop optimization cycle:

```
measure → categorize failures → fix → remeasure → compare → repeat
```

Each skill is a standalone `.md` file with clear inputs, steps, and outputs. An agent reads the skill, follows the steps, and produces structured results.

## Skills

| Skill | Description |
|-------|-------------|
| [measure](skills/measure.md) | Run a workflow N times, collect metrics |
| [categorize](skills/categorize.md) | Bucket failures into actionable categories |
| [compare](skills/compare.md) | Diff two baselines, compute deltas |
| [optimize](skills/optimize.md) | Full loop: measure → fix → remeasure until target met |
| [baseline](skills/baseline.md) | Persist and load metric snapshots |

## Quick Start

Point any agent at the `optimize` skill with your workflow definition:

```
/optimize my-workflow --runs 5 --target-rate 0.8
```

The agent will:
1. Run your workflow 5 times
2. Compute success rate, duration, cost
3. Categorize failures (AUTH, TIMEOUT, UI_CHANGE, etc.)
4. Apply fixes (using your provided fixer instructions)
5. Re-run and compare
6. Repeat until 80% success rate or max iterations

## Workflow Definition

Each workflow needs a `workflow.md` in its directory:

```
.skill-optimizer/
  workflows/
    my-workflow/
      workflow.md       # How to run it, what success looks like
      fixtures.md       # Test inputs
      fixer.md          # How to fix each failure category
  baselines/
    my-workflow/
      2026-02-28.md     # Metric snapshots
  learnings/
    my-workflow/
      failure-catalog.md
```

See [workflow-definition.md](workflow-definition.md) for the full spec.

## Principles

- **Methodology over code** — Skills describe *what to do*, not *how to implement it*
- **Agent-native** — Written for LLM agents to follow, not humans to run
- **Composable** — Each skill works standalone or as part of the optimize loop
- **Universal** — Works with any runtime: shell commands, agent SDKs, HTTP APIs
