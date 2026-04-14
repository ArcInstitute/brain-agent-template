# The Brain Agent Pattern

## What It Is

A Brain Agent is an autonomous Claude Code agent that:
1. Wakes on a cron schedule (cloud-hosted, no laptop needed)
2. Clones a GitHub "brain" repo containing its persistent state
3. Reads its instructions from `prompts/role.md`
4. Does work (research, trading, monitoring, analysis, etc.)
5. Updates its logs, data, and optionally evolves its own instructions
6. Pushes everything back to GitHub
7. Communicates with its owner via GitHub Issues

## Why GitHub as Persistent Memory

Claude Code cloud scheduled tasks start fresh every session — no memory between runs. The brain repo solves this:
- Full git history = complete audit trail
- Any file is fair game for the agent to read, modify, or create
- Agent can evolve its own `prompts/role.md` over time
- Owner can review the agent's decisions via git log

## Self-Evolution

The agent is explicitly encouraged to edit `prompts/role.md` to improve its own instructions. This creates a feedback loop:

```
Session → Log results → Analyze what worked → Update prompts/role.md → Better next session
```

The Strategist pattern (weekly Opus session) is useful for deeper reflection: a more capable model reviews all logs and rewrites strategy.

## 2-Way Communication

**Owner → Agent (async):**
- Write `.md` files to `inbox/owner/` — standing directives the agent reads every session
- Create a GitHub Issue labeled `directive` for one-time instructions

**Agent → Owner (notifications):**
- Create GitHub Issues labeled `request` when it needs something
- GitHub sends owner an email automatically

## Dependency Management

All Python dependencies live in `pyproject.toml`. The bootstrap trigger runs `uv sync` before the agent starts, ensuring reproducible environments. The agent adds new packages with `uv add <package>`, which updates `pyproject.toml` so future sessions have them too.

## Multi-Role Agents

One brain repo can have multiple triggers pointing at different prompt files:
- `prompts/strategist.md` — weekly Opus session for deep analysis
- `prompts/trader.md` — hourly Sonnet session for execution
- `prompts/reporter.md` — daily Haiku session for summaries

All three share the same persistent state.
