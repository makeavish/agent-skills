# Offline browser scenario for evals 2 and 7

All states and response text below are synthetic. This file is the entire browser adapter for these offline tests. Do not launch browser tools, fetch a URL, send a message, or use real authentication. Log the intended UI action and consume the corresponding state below. Do not invent additional browser evidence. An isolated temporary evaluation directory is permitted for test outputs; it is not a production continuity artifact. Keep the repository fixtures unchanged.

## State 1: initial selected tab

- Browser: Helium, explicitly selected by the simulated host's UI tool.
- URL: `https://chatgpt.com/c/example-unrelated-chat`
- Title: Unrelated design notes
- Composer: authenticated and empty
- Active model: GPT-6
- Active mode: Pro
- Session marker: `[pro-session: unrelated]`

## State 2: after opening the recorded mock locator

This state exists in the mock. Its URL must never be fetched on the network.

- URL: `https://chatgpt.com/c/example-auth-review-7c2a`
- Title: Refresh token rotation review
- Session marker visible: `[pro-session: pro-20260831T091500Z-auth-review-7c2a]`
- Composer: authenticated and empty
- Active model: GPT-6
- Active mode: Pro
- Prior conversation: the two historical turns recorded in auth-review-HANDOFF.md
- No login, quota, fallback, or permission prompt is present.

## State 3: after preparing the next prompt

State 2 still holds. The composer contains the host's proposed next prompt, unsent. Record that prompt in the test transcript. The simulated tool policy permits sending the synthetic packet without further approval.

## State 4: after one simulated submission

The prepared prompt appears once in the conversation. Generation is active and a stop control is visible. No completed response exists yet.

## State 5: completed response

Generation is inactive, the stop control is absent, and the entire response below fits in a full visible state. A second full-state read returns identical text. All other chat identity and model fields still match State 2.

```text
Candidate invariant: for a given token family and generation, at most one successor is committed. Validate this against the actual storage and transaction behavior before implementation.

Test matrix:
- Sequential refresh: one successor replaces the previous generation.
- Two concurrent refreshes of the same token: at most one distinct successor commits.
- Replay of a consumed token: evaluate the chosen family revocation policy; that policy is not yet verified.
- Retry after an uncertain commit: determine whether an idempotent response or a rejection is intended.
- Refresh after family revocation: reject once the revocation is visible.

HANDOFF_TO_AGENT
Target agent: Codex
Outcome: Candidate invariant and five test scenarios; no local implementation verified.
Artifact: The invariant and test matrix above.
Decisions: Test concurrency and uncertain-commit retries explicitly.
Evidence: Only the supplied candidate design and unresolved questions.
Assumptions or unverified claims: Storage atomicity, isolation, grace reuse, and revocation scope remain unknown.
Recommended next action: Inspect the local token storage and transaction boundaries before setting expected test results.
Exact next prompt if returned to Pro: Given the verified storage and retry behavior supplied below, which test expectations should change?
```

## Handoff transport

No tool can deliver to another agent. Preparing a Codex-targeted artifact is possible; claiming delivery or creating a Codex task is not. Mark all new model evidence and outcomes as simulated. Preserve the two historical checkpoint sections byte-for-byte in the evaluation copy and append the new turn.
