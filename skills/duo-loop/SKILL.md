---
name: duo-loop
description: Review a change in a loop with two cheap reviewers (DeepSeek + Muse Spark), escalating to Grok + Claude. Use when the user says 'duo loop', 'review loop', or 'get it reviewed'.
---

# duo-loop

T-Stack `sol-loop` adapted: no Sol. Two cheap OpenCode reviewers by default, escalating only when needed.

Use for any change before filing a PR. Minimal fixes only. Push back on scope creep.

## Reviewers (OpenCode model IDs)

Tier 1 (default, cheap):
- `opencode-go/deepseek-v4.1-flash#high` (~$0.15/$0.60 per M) — reasoner, catches logic/state bugs.
- `opencode-go/muse-spark-1.3-contributor#medium` (~$0.10/$0.20 per M) — strict checker, catches breakage + scope creep. `#high` if diff >300 lines.

Tier 2 (escalation — disputed blocker after 2 replies, still-open should after round 2, high-risk paths, or user asks):
- `opencode-go/grok-4.7#high` (~$2/$6) + `opencode/claude-sonnet-5-5` (Opus `opencode/claude-opus-5-5` for auth, secrets, billing, CI workflows, `.env`, migrations, perms, deps, servers, >1000 lines)

High-risk = auth, secrets, billing, CI workflows, `.env`, migrations, perms, deps, servers/deploys, >1000 lines.
- Reviewers are read-only. They never edit, commit, push, comment, or spawn subagents.

In T3 Code use `delegate_task` (one per reviewer, parallel). In OpenCode/Claude Code use the `subagent` tool with `agent: reviewer` and `model:` set to the IDs above.

## Budget

- Default: 3 rounds. Round 1 is full. Later rounds check only changed code + open findings.
- Extend to 5 only for security, perms, auth, data loss, state/concurrency with an open blocker/should + evidence, or when the user asks.
- Stop early if a round returns only nits. Declined nits and comment/doc/format-only fixes need no new round.
- Keep a ledger: id, severity, outcome. Never guess usage.

## 1. Scope

Run `git status --short`. Do not stash, discard, or include unrelated changes. Limit review to the change paths if they separate cleanly. Otherwise stop and ask.

Find the base after `git fetch origin`:

- PR: `gh pr view <n> --json baseRefName,headRefOid`. Base: `git merge-base HEAD origin/<baseRefName>`.
- Local: `git merge-base HEAD origin/<default-branch>`. No remote: base the user names, or `HEAD` for uncommitted only.

The change is base to working tree in those paths, with untracked files. Use one diff command all rounds. State intent (1-2 sentences) + out-of-scope. Run covering checks. Note missing or pre-failing checks in the brief.

## 2. Start both reviewers (parallel)

Same brief to both. T3 children get no history.

```
Review only. Do not edit, commit, push, comment on PRs, or start subagents.
Repo: <abs path> Base: <sha> Intent: <intent> Out of scope: <scope> Checks: <results>
See: git diff <base> [-- <paths>]; git status --short (read untracked).
Read surrounding code. Find bugs, broken callers, mismatch with intent.
Skip style unless it hides a bug. Hold to the smallest change. Flag code,
tests, or guards the intent does not need. Other paths, rare cases, small
perf, extra compat: nits.
blocker = intended case wrong. should = real bug callers will hit, in intent. nit = rest.
Reply:
VERDICT: APPROVE or CHANGES
S1 [blocker|should|nit] <file>:<line> <claim>. Fix: <suggestion> (stable ids)
```

Do not edit scoped files while review runs. Timeout/error/no-reply is not approval. On failure, cancel, verify stop, restart with the ledger.

## 3. Answer findings

Check each finding yourself. Another path, rare case, or small cost is a nit, whatever label it came with.

- Agree: smallest fix. Prefer delete/narrow over new lock, cache, helper, option. Fix the case that happened. Add one focused test that fails without the fix, if intent needs it and the repo has a test setup.
- Disagree: say why, with evidence (file:line, test, output). Two reviewers disagreeing = keep the stricter one unless you can prove it wrong.
- Out of scope or nit: decline in one line. List as follow-up.

Update the ledger. Re-run checks + `git diff --shortstat <base>`. If review grew the diff, cut first. Do not agree just to end the loop.

## 4. Next round

New round only if budget remains AND (a blocker/should is open, behavior code changed, or approval does not cover the current diff). Fresh subagents each round. Send the round-1 brief + ledger (`fixed:` / `disagree:` / `declined:` + evidence) + same base diff + change since last round. Open ids and changed code only. Mark each old id `resolved` or `still open: <reason>`.

## 5. Stop and ship

Agreed: both APPROVE on the current diff, no blocker/should open, no concern of your own. Stuck: item disputed after two replies, or budget ends. Show both sides in 2-3 lines each. User decides. Budget end is not approval.

Do not push during the loop. Push after agreement only if asked. PR line (above model blurb):

`Reviewed with duo-loop: <N> rounds (deepseek-v4.1-flash + muse-spark-1.3). Verdict: approved.`
Escalated runs append `+ escalated to grok-4.7 + <claude-model> round <M>`.

Report: rounds, models, what changed, diff size before/after, fixed + declined findings with reasons, pushed or not.
