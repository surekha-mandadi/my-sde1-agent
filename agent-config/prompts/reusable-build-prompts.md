# Build Prompts - Instructing Claude to Build the SDE1 Agent

Ready-to-use prompts for generating the SDE1 agent implementation, given the **current repository structure**. These prompts point the Claude/Codex at the existing spec files as the authoritative source, so it implements the specified design rather than spinning its own.

**Files the prompt reference:**
- [`sde1-agent-design.md`](..\docs\sde1-agent-design.md) - architeture + phases and workflow
- [`definition-of-ready.md`](definition-of-ready.md) - DoR
- [`definition-of-done.md`](definition-of-done.md) - DoD guardrails
- [`decision-trace.schema.json`](decision-trace.schema.json) + [`decision-trace.example.jsonl`](decision-trace.example.jsonl) - audit entry format
- [`pull_request_template.example.md`](pull_request_template.example.md) - PR body content

## Prompts for each phase

Give these one at a time reviewing each phase built before the next.

```text
1) Read docs\sde1-agent-design.md and agent-config\definition-of-done.md, then create pyproject.toml (Python 3.11, src/ layout) configuring ruff, mypy strict on src/, pytest with pytest-cov --cov-fail-under=80, and a [dev] extra with bandit + pip-audit. Don't add anything the DoD doesn't require.
```

```text
2) i. Create the TicketClient interface as detailed in the section 1 of [`requirements-client-for-sde1-agent.md`](requirements-client-for-sde1-agent.md). Copy the prompt after verifying the correctness.
ii. Based on the requirement tool, prompt to create the Backend-specific implementation as detailed in the section 2 of  [`requirements-client-for-sde1-agent.md`](requirements-client-for-sde1-agent.md). Include the section 3 content for either backend.
```

```text
3) Read docs\sde1-agent-design.md (Section 10), then create src/agent/checkpoints.py implementing the decision-checkpoint schema from 10.2 — a `Checkpoint` dataclass (run_id, jira_id, phase, timestamp, oversight_model, decision, rationale, alternatives_considered, assumptions, inputs_referenced, outcome) — and an `emit_checkpoint()` function that writes each checkpoint as a structured JSON log line to stdout (for the Actions run log) and appends it to a `decision-trace.jsonl` file per Section 10.3. Match decision-trace.example.jsonl. Add tests. Don't add Jira/GitHub posting, PR-description rendering, or a trace viewer — those belong to other components.
```

```
4) Create a .github/workflows/ci.yml running every DoD guardrail from agent-config/definition-of-done.md on pull_request, one step per check.
```

```
5) Read docs\sde1-agent-design.md (Sections 7 Phase 1-3, 8.1) and agent-config\definition-of-ready.md, then create src/agent/phases/understand.py, readiness.py, and plan.py implementing Phase 1 (fetch ticket via the TicketClient, normalize into the Ticket model), Phase 2 (evaluate against the DoR checklist; on failure call post_comment + transition_status and stop), and Phase 3 (produce a task breakdown — files to change, tests to add, edge cases — and emit a checkpoint via emit_checkpoint from Section 10). Gate Phase 3's output behind Gated mode only: after emitting the plan checkpoint, post the plan via post_comment and pause the run pending human approval — do not proceed to implementation. Don't implement Supervised/Autonomous branching, Phase 4+, or the approval-polling mechanism itself; stub the pause as a single function the runtime can call back into.
```

```
6) Create src/sde1_agent/run.py implementing the 6 phases from section 7 of the design doc as a CLI with --ticket and --oversight, using the ticket client, checkpoints, and `gh` for the PR. Enforce the DoR gate (reject+comment on ticket, exit) before writing code. Add tests mocking the ticket client and gh.
```

```
7) Create .github/workflows/agent-run.yml (workflow_dispatch: ticket_id, target_branch, oversight_model) that runs run.py with the required secrets and uploads decision-trace.jsonl as an artifact. No secrets in code. Also copy agent-config/pull_request_template.example.md to .github/pull_request_template.md.
```

