# skill-optimizer

A methodology for iteratively measuring and optimizing agent workflows. Defined as markdown skills that any LLM agent can pick up and execute.

## Architecture

The primary skill is **optimize** — a single, self-contained closed-loop that runs the full cycle:

```
measure → categorize → fix → remeasure → compare → repeat
```

Individual skills (measure, categorize, compare, baseline) work standalone for one-off tasks but do NOT call each other. The optimize skill has all logic inline — no skill-to-skill invocation required.

## Skills

| Skill | Purpose | Standalone? |
|-------|---------|-------------|
| [optimize](skills/optimize.md) | Full loop: measure → fix → remeasure until target met | Yes — primary entry point |
| [measure](skills/measure.md) | Run a workflow N times, collect metrics | Yes |
| [categorize](skills/categorize.md) | Bucket failures into actionable categories | Yes |
| [compare](skills/compare.md) | Diff two metric snapshots | Yes |
| [baseline](skills/baseline.md) | Persist and load metric snapshots | Yes |

## Quick Start

### Full optimization loop
```
/optimize path/to/workflow.md --runs 5 --target-rate 0.8
```

### Measure only (no fixes)
```
/optimize path/to/workflow.md --runs 5 --baseline-only
```

### One-off measurement
```
/measure path/to/workflow.md --runs 10
```

## Workflow Definition

Each workflow needs a directory with at minimum a `workflow.md`:

```
my-workflow/
  workflow.md       # Run command, success criteria, fixtures, constraints
  fixer.md          # (Optional) How to fix each failure category
```

See [workflow-definition.md](workflow-definition.md) for the full spec.

## Storage

All optimization data is stored in `.skill-optimizer/` relative to where the skill is invoked:

```
.skill-optimizer/
  {workflow-id}/
    baseline.md              # Latest aggregate metrics
    snapshots/
      {date}.md              # Historical snapshots
    iterations/
      iter-1.md              # Per-iteration changes + results
    failure-catalog.md       # Accumulated failure patterns + fixes
```

## Principles

- **Self-contained skills** — Each skill has all logic inline, no cross-skill invocation
- **Agent-native** — Written for LLM agents to follow, not humans to run
- **File-based data flow** — Results persist to `.skill-optimizer/` as markdown files
- **Parallel-capable** — Measurement runs can use Agent tool for concurrency
- **Universal** — Works with shell commands, agent prompts, or HTTP APIs
