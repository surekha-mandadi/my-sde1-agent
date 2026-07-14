```mermaid
flowchart TD
    START(["User command<br/>workflow_dispatch"]) --> P1["Phase 1: Understand<br/>fetch & normalize ticket"]
    P1 --> P2{"Phase 2: Readiness<br/>DoR gate"}

    P2 -->|Not ready| REJECT["Reject & comment on Jira<br/>transition: Needs Info"]
    REJECT --> END1(["Stop"])

    P2 -->|Ready| P3["Phase 3: Plan<br/>task breakdown +<br/>decision checkpoint"]

    P3 --> GATE1{"Gated mode?<br/>plan approval gate"}
    GATE1 -->|Yes - pause for approval| APPROVE1{"Approved?"}
    APPROVE1 -->|No, with feedback| REVISE1["Incorporate feedback,<br/>re-plan"]
    REVISE1 --> GATE1
    APPROVE1 -->|Declined| END2(["Stop"])
    APPROVE1 -->|Yes| P4
    GATE1 -->|No - Supervised/Autonomous| P4

    P4["Phase 4: Implement<br/>branch, code, tests +<br/>decision checkpoint"]
    P4 --> GATE2{"Gated mode?<br/>code approval gate (optional)"}
    GATE2 -->|Yes - pause for approval| APPROVE2{"Approved?"}
    APPROVE2 -->|No, with feedback| REVISE2["Revise diff"]
    REVISE2 --> GATE2
    APPROVE2 -->|Declined| END3(["Stop"])
    APPROVE2 -->|Yes| P5
    GATE2 -->|No| P5

    P5["Phase 5: Self-verify<br/>run DoD guardrails locally"]
    P5 --> CHECK{"All guardrails pass?"}
    CHECK -->|No| CORRECT["Bounded self-correction<br/>(max N iterations)"]
    CORRECT --> P5
    CHECK -->|Still failing| MODE1{"Oversight model"}
    MODE1 -->|Gated / Supervised| DRAFTPR["Open draft PR<br/>with failure summary"]
    MODE1 -->|Autonomous| ABORT["Abort & comment on Jira<br/>(no PR opened)"]
    ABORT --> END4(["Stop"])

    CHECK -->|Yes| P6["Phase 6: Raise PR<br/>push branch, open PR,<br/>link ticket, attach decision trace"]
    P6 --> MODE2{"Oversight model"}
    MODE2 -->|Gated| HUMANMERGE["Proceed after gate(s) passed,<br/>human merges"]
    MODE2 -->|Supervised| DRAFTREVIEW["Draft PR,<br/>human reviews & merges"]
    MODE2 -->|Autonomous| AUTOMERGE["Auto-merge on green CI"]

    HUMANMERGE --> DONE(["Jira transitioned:<br/>In Review / Done"])
    DRAFTREVIEW --> DONE
    AUTOMERGE --> DONE
    DRAFTPR --> DONE
    ```