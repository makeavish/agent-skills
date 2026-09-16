---
contract: "codex-chatgpt-pro-handoff/v1"
session_id: "pro-20260831T091500Z-auth-review-7c2a"
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

## Resume in one minute

- Current state: The first review proposed token-family tracking; local implementation is still unverified.
- Last accepted result: Family-level replay detection is worth testing.
- Next action: Ask Pro for one invariant and a compact concurrency/replay test matrix.

## Current practical artifact

Candidate design only: rotate refresh tokens atomically and revoke a family on confirmed replay.

## Resume packet for Pro

```markdown
[pro-session: pro-20260831T091500Z-auth-review-7c2a]

Goal: Review refresh-token rotation for replay and concurrent refresh races.
Accepted: Family-level replay detection is worth testing.
Unverified: Current storage, transaction isolation, grace reuse, and revocation scope.
Task: Give one invariant and a compact concurrency/replay test matrix. Do not assume access to local code.
```

## Handoff to Codex

- Exact next Pro prompt: Give one invariant and a compact concurrency/replay test matrix. Do not assume access to local code.
- Resume route: `https://chatgpt.com/c/example-auth-review-7c2a`; verify the title, marker, authentication, and visible GPT-6 + Pro controls before sending.
