# Prompt template: Requirements (ticket) client for the SDE1 agent

Use this as-is, or swap the "Backend-specific" block to target Jira or GitHub
Projects. It's written to be handed to a coding agent (e.g. Claude Code) to
generate the client described in Section 6 / Phase 1-2 of the SDE1 agent
design doc.

---

## Prompt

You are implementing the **requirements client** for an autonomous SDE1
agent (see attached design doc, Sections 4, 6, 7 Phase 1-2, 8.1). This client
is the agent's only way of reading a ticket, reporting status back, and
moving the ticket through its workflow. It must be a thin, well-tested
Python module - no business logic, no DoR/DoD evaluation, no planning. Those
live elsewhere; this client only talks to the ticket system.

### 1. Interface (backend-agnostic)

Define an abstract base class `TicketClient` with:

- `fetch_ticket(ticket_id: str) -> Ticket` - retrieve a ticket and normalize
  it into the shared `Ticket` model below. Must raise a typed exception
  (`TicketNotFoundError`) if the ticket doesn't exist, and
  `TicketClientAuthError` on auth/permission failure.
- `post_comment(ticket_id: str, body: str) -> None` - add a comment (used
  for DoR rejection, plan approval requests, status updates, decision-trace
  links).
- `transition_status(ticket_id: str, target_status: str) -> None` - move the
  ticket to a named status (e.g. `Needs Info`, `In Review`). Must raise
  `InvalidTransitionError` if the target status isn't reachable from the
  current one, rather than silently no-op-ing.
- `link_pull_request(ticket_id: str, pr_url: str) -> None` - attach/record
  the PR link on the ticket (comment or native link field, whichever the
  backend supports).

Define the shared `Ticket` dataclass (or pydantic model) with at minimum:
`id`, `title`, `description`, `acceptance_criteria: list[str]`,
`labels: list[str]`, `linked_issues: list[str]`, `attachments: list[Attachment]`,
`status`, `url`. Field mapping/normalization from the backend's native shape
into this model happens inside each concrete client, not in the agent.

### 2. Backend-specific implementation

> Replace this section with the backend you're targeting.

**Jira variant:**
- Implement `JiraTicketClient(TicketClient)` using the Jira Cloud REST API
  v3 (no SDK dependency beyond `requests`/`httpx`).
- Auth via `JIRA_BASE_URL`, `JIRA_EMAIL`, `JIRA_API_TOKEN` env vars (basic
  auth with API token). Fail fast with a clear error at construction time if
  any are missing.
- Map Jira's ADF (Atlassian Document Format) description/comment bodies to
  and from plain text/markdown.
- Acceptance criteria may live in a custom field, a description section
  under a heading, or a linked "AC" checklist - support at least the
  description-heading convention, and make the custom-field-ID configurable.

**GitHub Projects variant:**
- Implement `GitHubProjectsTicketClient(TicketClient)` using the GitHub
  GraphQL API v2 (Projects v2) via `httpx`, authenticated with
  `GITHUB_TOKEN` (reuse the Action's built-in token; no extra secret).
  `owner`, `repo`, and `project_number` are constructor args, not env vars. No secrets in code.
- Treat a Project item's linked Issue as the ticket; `fetch_ticket` reads
  the issue body/labels and any custom fields on the Project item (e.g. a
  single-select "Status" field) that the agent needs.
- `transition_status` updates the Project item's Status field via
  `updateProjectV2ItemFieldValue`, not the underlying issue's open/closed
  state, unless the target status maps to closing the issue.

### 3. Cross-cutting requirements (apply to either backend)

- **Least privilege**: document the exact token scopes required (Jira:
  read + comment + transition on the relevant project; GitHub: `repo` +
  `project` read/write scoped to the target repo/project) in the module
  docstring.
- **Retries**: transient network/5xx errors retry with exponential backoff
  (max 3 attempts); 4xx errors (auth, not-found, invalid transition) fail
  immediately without retry.
- **No silent failures**: every method either returns a typed result or
  raises one of the typed exceptions above - never returns `None` or an
  empty value to signal failure.
- **Idempotency**: `post_comment` and `transition_status` should be safe to
  retry from the agent's side without double-posting or erroring on a
  target state that's already current (transitioning to a status the
  ticket is already in is a no-op, not an error).
- **Testability**: no real network calls in unit tests - use recorded
  fixtures or a mocked HTTP layer (`respx` for `httpx`, or `responses` for
  `requests`). Cover: happy path for each method, not-found, auth failure,
  invalid transition, and malformed/partial API responses (missing
  optional fields shouldn't crash normalization).
- Prefer a small number of property-based tests (`hypothesis`) for the
  normalization logic specifically (`Ticket` construction from varied raw
  payload shapes), per the design doc's testing guidance.
- Type-annotate everything; the module must pass `mypy` and `ruff` clean
  (Section 8.2 of the design doc).

### 4. Deliverables

- `<backend>_client.py` implementing `TicketClient`.
- `models.py` with the shared `Ticket`/`Attachment` dataclasses and the
  typed exceptions.
- `tests/test_<backend>_client.py` with mocked-HTTP unit tests as above.
- A short module docstring documenting required env vars/constructor args
  and token scopes.

Do not implement DoR evaluation, planning, branch/PR creation, or CI
integration here - those are separate components per the design doc's
Section 4.1 component table.
