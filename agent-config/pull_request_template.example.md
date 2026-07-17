<!--
Example PR template for the SDE1 agent. Copy to `.github/pull_request_template.md` in the target repo during the build phase. The agent fills each section.
-->

## Ticket

<!-- Prefix the PR title with the ticket key, e.g. PROJ-123:...-->
Implements PROJ-XYZ

## Summary

<!-- What changed and why, in plain terms.-->

## Plan

<!-- The approved task breakdown (files changed, tests added, edge cases). -->

## Defintion of Done

- [ ] Lint (ruff) clean
- [ ] Formatting clean
- [ ] Types (mypy) clean
- [ ] Unit tests passing
- [ ] Coverage >= threshold, no regression
- [ ] SAST clean
- [ ] Dependency scan (pip-audit) clean
- [ ] Secret scan (gitleaks) clean

## Assumptions

<!-- Explicit assumptions made during implementation (from decison trace)-->

## Decision trace

<!-- Link to the decision-trace.jsonl artifact for this run.-->

co-authored-by: <!--your project trailer to identify ai assisted work-->