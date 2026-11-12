---
name: council
description: Get review from Grok + Claude in parallel, loop until they agree. Use when the user says 'council'.
---

# Council (cheap by default, Grok + Claude on escalation)

T-Stack `council` adapted to what you have in OpenCode. Cheap pair first; strong pair only when it matters. Before an audit, fix known issues and run relevant checks first. Do not defer known fixes until reviews return.

## Members

Tier 1 (default): `opencode-go/deepseek-v4.1-flash#high` + `opencode-go/muse-spark-1.3-contributor#medium` (`#high` over 300 lines).
Tier 2 (escalation): `opencode-go/grok-4.7#high` + `opencode/claude-sonnet-5-5` (`opencode/claude-opus-5-5` for high-risk).

Escalate when Tier 1 splits for two rounds, a blocker/should stays open with evidence, the change is high-risk, or the user asks.

If the user names members, use those. If a member is unavailable, report it instead of substituting a third provider.

In T3 Code: `orchestrator_capabilities` to resolve, then one `delegate_task` per member in parallel. In OpenCode/Claude Code: `subagent` tool with `agent: reviewer` + `model:` per member. Read-only work for review/feedback.

## Process

Give each delegate the request + relevant context. Do not wait idle: when a result arrives, check its findings against the code at once and decide what to fix while the other runs. Do not edit files while a delegate still reviews the same checkout. Non-blocking member: do not wait; treat late result as feedback, no score.

For code review ask each for a score out of 10. Each full review pass = one round. Fresh review after fixes = new round. Default 3 rounds max (5 for security/data-loss/state with open blocker + evidence). Stop early on nits-only.

Keep scope limited. Fight review-driven scope creep: fix real bugs in the intended case, decline the rest with a written reason + follow-up list.

## Report (PR description, before model line; or final reply if no PR)

```markdown
## Council review

Rounds: 2

| Delegate | Round 1 | Round 2 |
| --- | --- | --- |
| DeepSeek V4.1 Flash | 7/10 | 9/10 |
| Muse Spark 1.3 | 8/10 | 10/10 |
```

Escalated runs add rows for `Grok 4.7` / `Claude Sonnet 5.5` (or Opus) from the escalation round on.

`no score` if a delegate gives none. Do not invent one. If PR open, edit the description. Never post as a comment for this.
