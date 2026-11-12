---
name: merge-ready
description: Use when the user says "merge-ready" as an instruction, asks to get or take a change or PR to merge-ready. Do not use for questions about whether something is ready to merge, or when the user asks you to merge.
---

# Merge Ready

The user is not watching this thread. They read your updates and your final
message long after you stop. Own the work from its current state to an open
pull request that they can merge without checking anything again. Never merge
it.

Apply this skill only when the user clearly asked for it. Otherwise do not
assume this autonomy.

## What the request grants

- Commit, push, rebase, resolve conflicts, and use `--force-with-lease` on
  this task's branch.
- Use the test methods that proof needs without asking first. This replaces
  ask-first or leave-it-to-the-user rules in project files for this task,
  including rules for browser automation, computer use, simulators, running
  the app or a dev server, and screenshots. Rules that name the correct
  command, fixture, or check still apply. A harness or system approval prompt
  still applies.

It does not grant:

- Merging, enabling auto-merge, entering a merge queue, pushing to the default
  branch, bypassing branch protection, or delegating any of these.
- Deleting data, touching production, changing things outside the repo,
  closing a PR, or force-pushing any other branch.

## Steps

1. **Block automatic merging.** If a PR already exists, disable auto-merge
   (`gh pr merge <pr> --disable-auto`) and remove it from any merge queue.
   Check again before each push. If you cannot prevent an automatic merge,
   ask before you continue.
2. **Finish the change.** Reach the user's goal with the smallest change that
   works. Fix the case that happened, not every case that could. If you are
   on the default branch, create a branch first.
3. **Prove it.** See "Proof".
4. **Get an independent review.** See "Independent review".
5. **File the PR.** See "Filing the PR".
6. **Babysit the PR.** See "Babysitting".
7. **Hand off** when the readiness gate passes. See "Readiness gate" and
   "Handoff".

Stay active from step 1 to step 7. If you end your turn while checks or bots
are pending and nothing is set to wake you, the work stops, because nobody
will prompt you again.

## Proof

Proof is evidence you observed yourself in this session: command output, a
reproduction, a screenshot. Use the most direct proof the change allows:

- Reproduce the original failure, apply the fix, then watch the same
  reproduction pass.
- Run the checks that the repo's CI runs (typecheck, lint, tests).
- Add one focused test that fails without the fix, when the behavior has none
  and the repo has a test setup.
- For UI and runtime behavior, run the app and look at the change. Green
  mocked tests do not prove that the user experience works.

Get proof again for each later fix before you push it, and after behavior
changes or meaningful conflict fixes. If you cannot find a way to prove a
change, ask the user.

## Independent review

Have a model other than the one that wrote the change review it before you
file the PR. Use the first route that works:

1. In T3 Code, `delegate_task` with a review role on another provider's
   model.
2. A CLI skill for another provider: `codex` when you are Claude,
   `claude-cli` when you are Codex, or `cursor-cli` or `antigravity-cli`.
3. A subagent with fresh context on your own model. Say in the handoff that
   the review was not cross-model.

Find the base with `git fetch origin` and
`git merge-base HEAD origin/<default-branch>`. Do not edit the reviewed files
while a round runs. Send this brief, because the reviewer has no history:

```
Review only. Do not edit, commit, push, comment on PRs, or start subagents.
Repo: <abs path> Base: <sha> Intent: <intent> Out of scope: <scope> Checks: <results>
See: git diff <base>; git status --short (read untracked files).
Read the surrounding code. Find bugs, broken callers, and mismatch with intent.
Skip style unless it hides a bug. Hold to the smallest change.
blocker = intended case wrong. should = real bug callers will hit. nit = rest.
Reply:
VERDICT: APPROVE or CHANGES
R1 [blocker|should|nit] <file>:<line> <claim>. Fix: <suggestion>
```

Check each finding and its severity yourself.

- **Agree:** make the smallest fix. Prefer deleting or narrowing code over a
  new helper, option, or guard.
- **Disagree:** say why with evidence (file and line, a test, output).
- **Nit or out of scope:** decline in one line and list it as a follow-up.

Run at most 3 rounds. Later rounds cover only changed code and open findings.
Start another round only while a blocker or should is open, or behavior code
changed after the last approval. Stop early when a round returns only nits. A
timeout, an error, or no reply is not approval. If an item stays disputed
after two replies, or the rounds run out, the review did not approve. Report
both sides in the handoff.

## Filing the PR

- Check whether a PR for this branch already exists.
- Rebase onto the latest default branch, then review the diff against it.
- Follow the repo's title conventions; look at recently merged PRs. Say why
  the change matters, not which mechanism changed.
- Open the description with the problem in plain words, then the solution.
  Do not lead with an inventory of changes. Add one short line that says how
  you proved the change.
- Check the title, description, and attachments for secrets and private
  details.
- Open a real PR, not a draft, so review bots run.

## Babysitting

Use one watcher per PR. Never write your own polling or `sleep` loop.

- **In T3 Code:** handle the existing checks and comments, call
  `watch_pull_request`, and end your turn. A wake is news, not a verdict;
  check the readiness gate yourself. Call `unwatch_pull_request` before the
  handoff or a question.
- **Elsewhere:** run `gh pr checks <pr> --watch` as one blocking command. Add
  `--required` when the repo marks required checks. Start it again after
  each push.

Act only on checks and comments newer than the latest push. Verify every bot
finding against the source before you change code. Fix real findings and CI
failures. Tell repository failures apart from infrastructure flakes. When you
dismiss a finding, reply with a written reason, then resolve the thread after
the reply is posted.

Keep the PR at the size the goal needs. Decline unneeded tests, guards for
edge cases that do not matter, and small performance costs, and list them as
follow-ups. If another PR makes this one obsolete, stop and ask the user.

Start each comment you post on the user's behalf with this callout, then the
comment itself:

```md
> [!NOTE]
> 🤖 **<model> responding on behalf of <user>**
```

Use the model name the harness reports, or the harness name when it reports
none. Lead with the conclusion, cite the code or check, and skip filler.

## Readiness gate

Fetch the target branch and check the live PR: head, base, checks, reviews,
and mergeability. All of these must be true on the latest commit:

- Your proof covers every change in the final diff.
- Required checks are green.
- Review bots have finished, and each finding is fixed or dismissed with a
  written reason.
- The independent review approved the current behavior diff.
- No human review still requests changes.
- The branch is up to date with the default branch, and GitHub reports it as
  mergeable.
- The PR is open, not a draft, and not set to merge without the user.

You can ignore an automated review that only asks for human review when you
have high confidence in the change.

## Handoff

Give one verdict:

- **Ready to merge:** the gate passes. Leave the PR open and unmerged.
- **Waiting for required approval:** only human approval remains. Confirm
  that this is the actual blocker.
- **Blocked:** say what prevents readiness, what you tried, and what is
  needed.

If someone else merged the PR, report that instead.

Report the PR link, the final head SHA, the verdict, your proof, the review
outcome (model, rounds, declined findings), each decision you made for the
user, and remaining risks. State that the PR is still open and unmerged.

## Asking the user

A question in a normal message goes unseen, and the thread looks done. Use
the harness's question tool so the thread shows that it is waiting:
`AskUserQuestion` in Claude Code, `request_user_input` in Codex. Put all the
context in the question: the PR link, what blocks you, and the options with
your recommendation first.

Ask only when you cannot continue alone:

- You cannot prove the change.
- The readiness gate cannot pass, and you cannot fix that yourself.
- The next step is outside the grant above.
- The goal is ambiguous in a way that changes the result.
- Another PR makes this one obsolete.
- You cannot prevent an automatic merge.

Decide everything else yourself with sensible defaults, and list those
decisions in the handoff. If no question tool is available, or it returns no
answer, leave the PR open and unmerged, and start your final message with the
question.
