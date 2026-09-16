---
contract: "pro-browser-handoff/v2"
session_id: "pro-20260831T091500Z-auth-review-7c2a"
originating_agent: "Codex"
current_agent: "Codex"
handoff_target: "Codex"
handoff_delivery: "returned-to-host"
status: "handed-off"
revision: 2
turn: 2
workspace: "/workspace/auth-service"
browser_app: "Helium"
chat_url: "https://chatgpt.com/c/example-auth-review-7c2a"
chat_title: "Refresh token rotation review"
session_marker: "[pro-session: pro-20260831T091500Z-auth-review-7c2a]"
requested_model: "GPT-6"
requested_mode: "Pro"
model_verification: "verified"
---

# Session goal

Review refresh-token rotation for replay and concurrent refresh races.

This is a synthetic evaluation fixture. Its chat URL is an opaque mock identifier, not a live conversation. Use the attached browser mock for offline resume tests.

## Resume in one minute

- Current state: The first review proposed token-family tracking; local implementation is still unverified.
- Last accepted result: Family-level replay detection is worth testing.
- Next action: Ask Pro for one invariant and a compact concurrency/replay test matrix.

## Current practical artifact

Candidate design only: rotate refresh tokens atomically and revoke a family on confirmed replay.

## Checkpoints

### Turn 001 - candidate rotation design

- Revision: 1
- Prompt or delta sent: Propose a candidate refresh-token rotation design; local storage and transaction behavior are not yet supplied.
- Context sources transmitted: Synthetic goal only; no local code.
- Response capture: exact (synthetic historical response)
- Response outcome: Rotate atomically and track a token family; treat replay-triggered family revocation as a candidate policy.
- Accepted: Family-level replay detection is worth testing.
- Rejected: None.
- Unverified: Storage atomicity, transaction isolation, and revocation scope.
- Output paths: Current practical artifact in this file.

### Turn 002 - unresolved concurrency behavior

- Revision: 2
- Prompt or delta sent: Identify what remains unknown before implementing that candidate design.
- Context sources transmitted: The candidate design from Turn 001; no new local evidence.
- Response capture: exact (synthetic historical response)
- Response outcome: Concurrent refreshes and uncertain-commit retries require explicit expected behavior; grace reuse remains a policy decision.
- Accepted: Request an invariant and concurrency/replay test matrix next.
- Rejected: None.
- Unverified: Current storage, transaction isolation, grace reuse, and revocation scope.
- Output paths: Current practical artifact in this file.

## Resume packet for Pro

```markdown
[pro-session: pro-20260831T091500Z-auth-review-7c2a]

Goal: Review refresh-token rotation for replay and concurrent refresh races.
Accepted: Family-level replay detection is worth testing.
Unverified: Current storage, transaction isolation, grace reuse, and revocation scope.
Task: Give one invariant and a compact concurrency/replay test matrix. Do not assume access to local code.
```

## Handoff to agent

- Target agent: Codex
- Exact next Pro prompt: Give one invariant and a compact concurrency/replay test matrix. Do not assume access to local code.
- Resume route: `https://chatgpt.com/c/example-auth-review-7c2a`; verify the title, marker, authentication, and visible GPT-6 + Pro controls before sending.
