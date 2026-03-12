---
name: categorize
description: Bucket failures from a measurement run into actionable categories (TIMEOUT, AUTH, UI_CHANGE, etc). Standalone — does not invoke other skills.
argument-hint: "<baseline-path>"
allowed-tools: Read, Write
---

# Categorize — Failure Analysis

## Input

Path to a baseline or measurement file containing run results with error messages.

## Failure Categories

For each failed run, match error text against these patterns (first match wins):

| Category | Patterns | Fixable? |
|----------|----------|----------|
| AUTH | logged out, sign in, expired session, unauthorized | No — manual re-login |
| RATE_LIMIT | 429, rate limit, too many requests | No — wait required |
| TIMEOUT | timeout, timed out, deadline, turn limit exceeded | Yes |
| UI_CHANGE | selector not found, element not found, no such element | Yes |
| TIMING | not ready, loading, spinner, pending | Yes |
| CONTENT | text empty, failed to type, paste fail | Yes |
| BROWSER | browser crash, chromium, target closed, page closed | Partial |
| UNKNOWN | (no pattern matched) | Review needed |

## Process

For each failed run:
1. Combine `error` + `output` into a single search text
2. Test against each category pattern in order
3. Record: `{ category, error, fixture_id }`

## Output

### Failure Summary

```
Failures (3/5 runs):
  TIMEOUT    2  (fixable)
  AUTH       1  (not fixable)
Fixable: 2/3 (67%)
```

### Grouped Failures

```
TIMEOUT (2):
  1. [f1] Timed out after 300s
  2. [f2] Turn limit exceeded at 30 turns

AUTH (1):
  1. [f3] Session expired, login required
```

### Decision

| Condition | Recommendation |
|-----------|---------------|
| All AUTH or RATE_LIMIT | SKIP — not fixable by code changes |
| Majority fixable categories | FIX — proceed to optimization |
| All UNKNOWN | REVIEW — examine raw output manually |
| Mix of fixable + unfixable | FIX — address fixable, note unfixable |
