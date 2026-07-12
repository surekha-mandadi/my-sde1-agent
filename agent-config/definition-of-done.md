# Definition of Done (DoD)

A change is "done" only when **every** guardrail below passes in CI. All are **required status checks** and block the PR. The agent runs the same suite locally (Phase 5) before openeing the PR.

# Checklist

- [ ] **Lint** - "<your lint tool>" clean.
- [ ] **Formatting** - "<your format tool>" clean.
- [ ] **Type check** - `mypy` (or `pyright`) clean on chnaged code.
- [ ] **unit tests** - pytest all passing.
- [ ] **Coverage** - changed-line coverage >= 80%; no dilution of overall coverage.
- [ ] **SAST** - "<your  tool>" reports no high-severity findings.
- [ ] **Secret scan** - gitleaks or detect-secrets finds no committed secrets.
- [ ] **Dependency scan** - pip-audit reports no unresolved high/critical CVEs.
- [ ] **PR hygience** - links the Jira ticket, follows the PR template, focused diff.

## Tooling reference

Insert your project tools here

| Guardrail | Command (guideline) | Blocks PR? |
|-----------|---------------------|:----------:|
| Lint | `ruff check .` | Yes |
|Format|`ruff format --check . `|Yes|
|Types|`mypy .`|Yes|
|Unit tests|`pytest`|Yes|
|Coverage|`pytest --cov --cov-fail-under=80`|Yes|
|SAST|`bandit - r <pkg> - ll`|Yes (high sev)|
|Dependency CVEs|`pip-audit`|Yes (high/critical)|
|Secret scan | `gitleaks detect --no-banner` | Yes |
|PR hygiene| template + ticket-link check | Yes |

## Notes

- Thresholds (like coverage %) are configurable per repo via `pyproject.toml` / CI env.
- The suite is intentionally identical between agent;s local Phase 5 dry run and the PR pipeline, so a local-green change = green in CI