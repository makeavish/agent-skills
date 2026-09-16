---
contract: "pro-browser-handoff/v2"
session_id: "<session-id>"
originating_agent: "<host-agent-name>"
current_agent: "<host-agent-name>"
handoff_target: "<target-agent-name>"
handoff_delivery: "not-ready"
escalation_reason: >-
  <explicit-request-or-specific-reason-for-proactive-consultation>
parent_session_id: ""
parent_artifact: ""
status: initializing
revision: 0
turn: 0
workspace: "<absolute-workspace-path>"
repository: ""
branch: ""
commit: ""
dirty_state: >-
  <clean-or-brief-scoped-note>
browser_app: "<Helium-or-explicit-host-managed-browser>"
browser_control: "<available-host-tool>"
chat_url: ""
chat_title: ""
session_marker: "<non-sensitive-session-marker>"
requested_model: "GPT-6"
requested_mode: "Pro"
observed_model: ""
observed_mode: ""
model_verification: "unverified"
model_verified_at: ""
model_evidence: >-
  <verbatim-visible-model-and-mode-evidence>
created_at: "<ISO-8601-UTC>"
updated_at: "<ISO-8601-UTC>"
last_confirmed_step: "artifact_created"
sensitivity: "local-private"
blocker: >-
  <empty-or-exact-blocker-and-recovery-step>
artifact_paths: []
---

# Session goal

<One sentence describing the user-visible outcome.>

## Resume in one minute

- Current state: Artifact initialized; browser preflight has not completed.
- Last accepted result: None yet.
- Next action: Verify the authenticated ChatGPT chat and the visible GPT-6 + Pro controls.

## Goal and done criteria

- Goal:
- Done when:

## Constraints and supplied context

- User constraints:
- Authority boundary:
- Local investigation and failed attempts:
- Context supplied to Pro:
- Context deliberately withheld:

## Current practical artifact

<Put a short usable result here, or link an exact file under outputs/.>

## Decisions

| Revision | Decision | Rationale or evidence | Status |
| --- | --- | --- | --- |

## Checkpoints

### Turn 000 - initialization

- Prompt or delta sent: None.
- Context sources transmitted: None.
- Response capture: none
- Response outcome: None.
- Accepted: None.
- Rejected: None.
- Unverified: None.
- Output paths: None.

## Evidence and deliverables

- Local evidence the host agent verified:
- Pro-provided evidence not yet verified:
- Deliverables:

## Open questions and risks

- None recorded yet.

## Resume packet for Pro

```markdown
[pro-session: <session-id>]

Goal and done criteria:

Verified current state:

Constraints and decisions already made:

Practical artifact so far:

Unresolved questions:

Task for this turn:

End with HANDOFF_TO_AGENT, naming the target agent and including Outcome, Artifact, Decisions, Evidence, Assumptions or unverified claims, Recommended next action, and Exact next prompt if returned to Pro.
```

## Handoff to agent

- Target agent:
- Delivery status and evidence:
- Workspace access and path mappings:
- User objective:
- What Pro produced:
- Practical artifact path:
- Accepted decisions:
- Suggestions still requiring verification:
- Local state Pro could not inspect:
- Open risks:
- Next local action:
- Exact next Pro prompt:
- Resume route: `<canonical-chat-url>` with visible GPT-6 + Pro re-verification required.
