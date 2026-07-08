## Design for SDE1 agent
This is a design for an SDE1 agent that takes a Jira ticket, implementa the change in a Python codebase, and raises a production-quality Pull Request. It is bounded by automated quality and security guardrails.

## Models
The design presents 3 human-oversight models, listed in the order of supervision.

1. Gated (recommended default) - human approves the plan before implementation proceeds
2. Supervised - agent opens a draft PR, a human reviews and merges
3. Autonomous - agent auto-merges when the pipeline is green
