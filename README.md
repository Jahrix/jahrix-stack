# jahrix-stack

Agent skills for PR work, adapted from [t3dotgg/tstack](https://github.com/t3dotgg/tstack) (MIT). No Sol. Reviewers are Grok + Claude via OpenCode, with T3 Code / Claude Code fallbacks.

## Skills

| Skill | Say | What it does | Changed from t-stack |
|---|---|---|---|
| `full-send` | "full send" | Change to merged PR, nobody watching: prove, review, file, babysit, merge | `duo-loop` instead of `sol-loop`; rest unchanged |
| `full-stop` | "full stop" | Same, stops ready-to-merge, never merges | same as above |
| `duo-loop` | "duo loop" | Reviews change in loop until both agree | **replaces `sol-loop`**: DeepSeek Flash + Muse Spark cheap default (~$0.1-0.15/M in), escalates to Grok 4.7 + Claude |
| `council` | "council" | Parallel review from two providers, scored rounds | **2 members, cheap-first** (was 3: Opus + Sol + Gemini) |
| `file-pr` | "file a PR" | Short problem-first PR, model attribution | unchanged |
| `babysit-pr` | "babysit" | Watches PR via `watch_pull_request` (T3) or `pr-wait` script, fixes findings | unchanged (`Sol round` wording fixed) |
| `takeover` | "take over" | Finishes someone else's PR/branch/thread, keeps credit | unchanged |
| `modernize` | "modernize" | Rebases old PR, verdict MODERNIZE/REWORK/CLOSE | unchanged |
| `leaving-pr-comments` | (automatic) | `> [!NOTE] 🤖 <model> on behalf of <user>` prefix | unchanged |

Deps, same as t-stack:

- `full-send` uses `duo-loop`, `file-pr`, `babysit-pr`.
- `full-stop` uses `full-send`.
- `takeover` uses `modernize`, `file-pr`, `babysit-pr`, `leaving-pr-comments`.
- `file-pr`, `babysit-pr`, `modernize` use `leaving-pr-comments`.

## Install

```bash
git clone <this-repo> jahrix-stack
cp -R jahrix-stack/skills/* ~/.claude/skills/   # Claude Code
cp -R jahrix-stack/skills/* ~/.codex/skills/    # Codex
# OpenCode: point `subagent` model param at the IDs in duo-loop/council
```

## Requirements

- Authenticated `gh` CLI.
- `python3` for `babysit-pr`'s `pr-wait`.
- OpenCode with `opencode-go/deepseek-v4.1-flash` + `opencode-go/muse-spark-1.3-contributor` (cheap default). Escalation uses `opencode-go/grok-4.7` + Claude (`opencode/claude-sonnet-5-5`, `opencode/claude-opus-5-5` high-risk). Override the defaults in your `CLAUDE.md`/`AGENTS.md`.
- T3 Code: `delegate_task` for parallel reviewers; elsewhere use the harness subagent tools.

## Make them yours

Starting points. Put personal settings (which Claude variant sits on council, merge method, title conventions) in `CLAUDE.md`/`AGENTS.md` so skills stay updateable. Example: keep `--merge` never `--squash`, `type(scope): what now works` titles, `no-slop` wording rules.

## Credits

Fork-adapted from `t3dotgg/tstack` (MIT, see LICENSE). `pr-wait` script unchanged from t-stack.
