# Workflow Definition Spec

A workflow is anything repeatable that you want to optimize. Define it in a `workflow.md` file within its own directory.

## Directory Structure

```
my-workflow/
  workflow.md       # Required — defines the workflow
  fixer.md          # Optional — domain-specific fix instructions per failure category
```

## Required Fields

```markdown
# Workflow: <name>

## Run Command
How to execute one run. Can be a shell command, API call, or agent prompt.

## Success Criteria
How to determine if a run succeeded. One or more of:
- Exit code 0
- Output contains pattern (regex or string)
- URL returned matching pattern

## Fixtures
Test inputs substituted into the run command. Each fixture has:
- ID
- Input values (variable substitution via ${variable})
- Expected success pattern

## Constraints
- Timeout (ms) — kill the run if exceeded
```

## Optional Fields

```markdown
## Setup Command
One-time setup before the first run (e.g., start a server).

## Rebuild Command
Run after applying fixes (e.g., rebuild Docker image).
```

## Example: Shell Script Workflow

```markdown
# Workflow: deploy-check

## Run Command
./deploy.sh ${environment} ${version}

## Success Criteria
- Output contains "deployment successful"
- Exit code 0

## Fixtures
| ID | environment | version | Success Pattern |
|----|------------|---------|-----------------|
| f1 | staging | latest | deployment successful |
| f2 | staging | v2.1.0 | deployment successful |

## Constraints
- Timeout: 60000ms
```

## Example: Agent Workflow

```markdown
# Workflow: post-to-twitter

## Run Command
Agent prompt template:
> Post the following to Twitter: ${content}
> Return the post URL when done.

## Success Criteria
- Output contains URL matching `https://(x.com|twitter.com)/\S+/status/\d+`

## Fixtures
| ID | content | Success Pattern |
|----|---------|-----------------|
| f1 | "Test post ${marker}" | x.com.*status |
| f2 | "Another test ${marker}" | x.com.*status |

## Constraints
- Timeout: 300000ms
```

## Example: HTTP API Workflow

```markdown
# Workflow: api-endpoint

## Run Command
POST ${base_url}/api/run
Body: { "input": "${input}" }

## Success Criteria
- Exit code 0 (curl returns 0)
- Output contains "done"

## Fixtures
| ID | base_url | input | Success Pattern |
|----|----------|-------|-----------------|
| f1 | http://localhost:3000 | hello | "done" |

## Constraints
- Timeout: 120000ms
```

## Fixer File (Optional)

The `fixer.md` file maps failure categories to workflow-specific fix instructions. Without it, the optimize skill applies generic fixes.

```markdown
# Fixer: my-workflow

## TIMEOUT
- Chain sequential commands with &&
- Remove unnecessary intermediate steps

## TIMING
- Add sleep 2 before element interactions
- Add retry loop for NOT_READY elements

## UI_CHANGE
- Inspect current page structure
- Update selectors in the run command/prompt

## CONTENT
- Switch input method
- Add input validation
```
