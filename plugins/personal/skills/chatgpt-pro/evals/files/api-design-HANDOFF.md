---
contract: "codex-chatgpt-pro-handoff/v1"
session_id: "pro-20260829T174500Z-api-design-9f11"
status: "handed-off"
revision: 4
turn: 4
workspace: "/workspace/orders-api"
browser_app: "Helium"
chat_url: "https://chatgpt.com/c/example-missing-api-design-9f11"
chat_title: "Orders API compatibility review"
session_marker: "[pro-session: pro-20260829T174500Z-api-design-9f11]"
requested_model: "GPT-6"
requested_mode: "Pro"
model_verification: "verified"
---

# Session goal

Review an additive orders API design without breaking current clients.

## Resume in one minute

- Current state: The original chat URL is reported missing.
- Last accepted result: Preserve current response fields and add idempotency explicitly.
- Next action: Recover in a linked child chat from the bounded packet below.

## Current practical artifact

Candidate contract: additive fields only; explicit idempotency key; stable error envelope. All compatibility claims remain unverified locally.

## Resume packet for Pro

```markdown
[pro-session: pro-20260829T174500Z-api-design-9f11]

Goal: Review an additive orders API design without breaking current clients.
Accepted: Preserve current response fields and make idempotency explicit.
Unverified: Existing endpoints, response shapes, error schema, persistence, and client behavior.
Task: Produce a compact compatibility matrix and identify the three highest-risk assumptions. Do not assume access to local code.
```

## Handoff to Codex

- Exact next Pro prompt: Produce a compact compatibility matrix and identify the three highest-risk assumptions.
- Resume route: The old URL is missing; create a linked child session and preserve this artifact unchanged.
