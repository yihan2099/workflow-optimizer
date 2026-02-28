# Workflow Definition Spec

A workflow is anything repeatable that you want to optimize. Define it in a `workflow.md` file.

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
- JSON response matches schema

## Fixtures
Test inputs that exercise the workflow. Each fixture has:
- ID
- Input values (substituted into the run command)
- Expected success pattern for this specific input

## Constraints
- Timeout (ms)
- Max turns (for agent workflows)
- Max cost per run (optional)
```

## Example: Shell Script Workflow

```markdown
# Workflow: deploy-check

## Run Command
```bash
./deploy.sh ${environment} ${version}
```

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

System prompt: (optional, for agent SDK runners)

## Success Criteria
- Output contains URL matching `https://(x.com|twitter.com)/\S+/status/\d+`

## Fixtures
| ID | content | Success Pattern |
|----|---------|-----------------|
| f1 | "Test post ${marker}" | x.com.*status |
| f2 | "Another test ${marker}" | x.com.*status |

## Constraints
- Timeout: 300000ms
- Max turns: 30
```

## Example: HTTP API Workflow

```markdown
# Workflow: api-endpoint

## Run Command
POST ${base_url}/api/run
Body: { "input": "${input}" }
Poll: GET ${base_url}/api/run/:id

## Success Criteria
- Response status field is "done"
- Response result field is non-empty

## Fixtures
| ID | base_url | input | Success Pattern |
|----|----------|-------|-----------------|
| f1 | http://localhost:3000 | hello | "done" |

## Constraints
- Timeout: 120000ms
- Poll interval: 5000ms
```
