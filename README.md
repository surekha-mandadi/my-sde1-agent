## Design for SDE1 agent
This is a design for an SDE1 agent that takes a ticket, implements the change in a Python codebase, and raises a production-quality Pull Request. It is bounded by automated quality and security guardrails.
The repository currently holds the design deliverable only. No agent runtime code is implemented yet. 
The design, templates, and prompts to build the agent are available for reference.

## Models
The design presents 3 human-oversight models, listed in the order of supervision.

1. Gated (recommended default) - human approves the plan before implementation proceeds
2. Supervised - agent opens a draft PR, a human reviews and merges
3. Autonomous - agent auto-merges when the pipeline is green
