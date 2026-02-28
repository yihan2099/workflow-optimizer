---
name: categorize
description: Bucket failures from a measurement run into actionable categories. Determines what's fixable and what requires manual intervention.
argument-hint: "<results> [--custom-rules <path>]"
---

# Categorize — Failure Analysis

## Input

Takes results from a `measure` run (either in-memory or from a JSON file).

## Default Failure Categories

| Category | Pattern (in error or raw output) | Fixable by Agent? |
|----------|----------------------------------|-------------------|
| AUTH | logged out, not logged in, login, sign in, expired session, auth | No — requires manual re-login |
| RATE_LIMIT | 429, rate limit, too many requests | No — wait required |
| TIMEOUT | timeout, timed out, abort, deadline | Yes — optimize prompts, reduce turns |
| UI_CHANGE | selector not found, element not found, cannot find, no such element | Yes — update selectors |
| TIMING | not ready, loading, spinner, pending | Yes — add sleeps, fallbacks |
| CONTENT | content, text empty, failed to type, paste fail | Yes — fix input methods |
| BROWSER | browser crash, chromium, playwright, cdp, target closed, page closed | Partial — restart session, add recovery |
| UNKNOWN | (no pattern matched) | Depends on error text |

## Process

For each failed run result:

1. Combine `error` + `raw_output` into a single search text
2. Test against each category pattern in order (first match wins)
3. Record: `{ category, error, raw_output, fixture_id }`

## Custom Rules

If `--custom-rules` is provided, load additional patterns from the file. Custom rules are checked **before** defaults, allowing overrides.

Custom rules format:
```markdown
| Category | Pattern |
|----------|---------|
| MY_CATEGORY | /my custom pattern/i |
| AUTH | /my specific auth error/i |
```

## Output

### Failure Summary

```
Failure Distribution:
  TIMEOUT        3  (fixable)
  TIMING         2  (fixable)
  AUTH           1  (not fixable)
  UNKNOWN        1  (review needed)

Fixable: 5/7 (71%)
```

### Grouped Failures

For each category, list the specific errors:

```
TIMEOUT (3):
  1. [f1] Timed out after 300s
  2. [f2] Turn limit exceeded at 30 turns
  3. [f1] Deadline exceeded

TIMING (2):
  1. [f3] Element not ready after 3 retries
  2. [f1] Page still loading after 10s wait
```

### Decision

Based on the distribution:

| Condition | Recommendation |
|-----------|---------------|
| All AUTH or RATE_LIMIT | SKIP — not fixable by code changes |
| Majority TIMEOUT/TIMING/UI_CHANGE/CONTENT | FIX — proceed to optimization |
| All UNKNOWN | REVIEW — examine raw output manually |
| Mix of fixable + unfixable | FIX — address fixable categories, note unfixable |
