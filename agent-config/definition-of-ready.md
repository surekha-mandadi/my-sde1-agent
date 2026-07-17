# Definition of Ready (DoR)

The agent picks up a Jira ticket **only if every item below is satisfied**. If any item fails, the agent posts a clarification comment listing the gaps, transitions the ticket to "Needs Info", and stops ("reject and comment" behaviour).

# Checklist

- [ ] **Clear summary** - the title states the change in one line.
- [ ] **Unambiguous description** - the problem/desired behaviour is described without contradictions or open questions.
- [ ] **Testable acceptance criteria** - explicit, verifiable criteria are present (Given/when/then or an equivalent checklist) .
- [ ] **SDE1-sized scope** - a single logical concern; bounded number of files; no cross-service or architectural changes.
- [ ] **No blocking dependencies** - no unresolved upstream tickets, decisions, or access requirements.
- [ ] **Affected area identified** - the relevant module/package/service is known or inferable from the ticket.

# Scope metrics (reject if exceeded)

| Signal | Threshold (guideline) |
|-----------|--------------------|
| Files likely touched | <= ~10 |
| New public interfaces | <= 1 |
| Cross=service coordination | None |
| Data-model / migratoin changes | None (unless trivial additions) |
| Ambiguity in acceptance criteria | None |

## Rejection comment template

```
This ticket is not ready for automated implementation yet. Missing:
<item 1>
<item 2>

Please update the ticket and move it back to <Ready state> to re-trigger the agent.
```
