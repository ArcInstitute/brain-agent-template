---
name: new-brain-agent
description: |
  Scaffold a new autonomous Brain Agent — a self-evolving Claude Code scheduled task
  with a GitHub brain repo, self-modifiable prompts, and GitHub Issues for 2-way
  owner communication. Creates the repo from template, fills in CLAUDE.md and
  prompts/role.md, and creates the CCR scheduled trigger automatically.
  Triggers on: brain agent, new agent, scheduled agent, autonomous agent, create agent, new brain.
---

# New Brain Agent

Scaffold a complete autonomous brain agent: GitHub repo + cloud scheduled trigger, wired up and ready to run.

## What Gets Created

1. **GitHub repo** from `sullivanj91/brain-agent-template` — pre-structured with `prompts/`, `inbox/`, `logs/`, `data/`, `scripts/`
2. **`prompts/role.md`** — agent's instructions, filled in with your purpose and schedule; agent evolves this over time
3. **`CLAUDE.md`** — project context auto-loaded by every Claude Code session
4. **CCR scheduled trigger** — bootstraps with `uv sync` then reads `prompts/role.md`

## Workflow

### Step 1: Gather inputs

Ask the user (use AskUserQuestion):
- Agent name (will become `<name>-brain` repo)
- One-line purpose ("what does this agent do each session?")
- Schedule (human-readable, e.g. "every weekday at 9am ET") — convert to UTC cron
- Model: default `claude-sonnet-4-6`; suggest Opus for complex weekly reasoning, Haiku for lightweight reporting
- GitHub visibility: public or private
- CCR environment to use (list available environments via RemoteTrigger list, or ask user)
- Environment variables needed (names only — user adds values in the environment settings)

### Step 2: Create and populate the repo

```bash
# Create from template
gh repo create <name>-brain --template sullivanj91/brain-agent-template --public  # or --private

# Clone locally
git clone https://github.com/<owner>/<name>-brain /tmp/<name>-brain
cd /tmp/<name>-brain
```

Fill in `CLAUDE.md` — replace placeholders:
- `{{AGENT_NAME}}` → agent name
- `{{AGENT_PURPOSE}}` → one-line purpose
- `{{TRIGGER_NAME}}` → agent name + "Agent"
- `{{MODEL}}` → chosen model
- `{{SCHEDULE}}` → human-readable schedule

Fill in `prompts/role.md` — replace placeholders:
- `{{AGENT_NAME}}` → agent name
- `{{MODEL}}` → chosen model
- `{{SCHEDULE}}` → human-readable schedule
- `{{AGENT_TASK_DESCRIPTION}}` → detailed description of what the agent should do each session, written in second person ("You are a..."). Include specific steps, data sources, outputs expected.

```bash
cd /tmp/<name>-brain
git add -A
git commit -m "init: scaffold from brain-agent-template"
git push origin main
```

### Step 3: Create the CCR trigger

Use the `RemoteTrigger` tool (load with ToolSearch first):

```json
{
  "action": "create",
  "body": {
    "name": "<AgentName> Agent",
    "cron_expression": "<UTC cron>",
    "enabled": true,
    "job_config": {
      "ccr": {
        "environment_id": "<environment_id>",
        "session_context": {
          "model": "<model>",
          "sources": [
            {"git_repository": {"url": "https://github.com/<owner>/<name>-brain"}}
          ],
          "allowed_tools": ["Bash", "Read", "Write", "Edit", "Glob", "Grep", "WebSearch", "WebFetch"]
        },
        "events": [
          {
            "data": {
              "uuid": "<generate lowercase v4 uuid>",
              "session_id": "",
              "type": "user",
              "parent_tool_use_id": null,
              "message": {
                "role": "user",
                "content": "## Setup\n```bash\ncd /home/user && pip install uv -q && cd <name>-brain && uv sync\n```\n\nYour full instructions are in `prompts/role.md` — read that file and follow it exactly."
              }
            }
          }
        ]
      }
    }
  }
}
```

After creating the trigger, update `prompts/role.md` with the trigger ID and push.

### Step 4: Output

Report back:
- GitHub repo URL
- Trigger URL: `https://claude.ai/code/scheduled/<trigger_id>`
- Recommended next step: "Run the trigger manually ('Run now') to seed the initial state before the first scheduled run."

## Key Rules

- Always use `uv add <package>` in the agent prompt, never bare pip install
- Always use `uv run python` in the agent prompt, never bare python3
- Cron expressions are in UTC — convert from user's local time and confirm
- Generate a fresh lowercase UUID v4 for the trigger event
- If the agent needs multiple roles/schedules (like Trader + Reporter), create separate triggers pointing at the same repo with different prompt files (e.g. `prompts/trader.md`, `prompts/reporter.md`)

## Reference Files

- [Brain Pattern overview](references/pattern.md)
- [Example prompts/role.md](references/prompt-template.md)
