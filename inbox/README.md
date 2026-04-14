# Inbox

Two-way communication between owner and agent.

## Owner → Agent: `inbox/owner/`

Write `.md` files here to send standing instructions to the agent. The agent reads all files in this directory at the start of every session.

Examples:
- `pause.md`: "Skip this week — markets are closed for the holiday"
- `focus.md`: "Prioritize X for the next month"
- `override.md`: "Do not touch NVDA this week"

Move files to `inbox/resolved/` once they've been acted on.

## Agent → Owner: GitHub Issues

The agent creates GitHub Issues labeled `request` when it needs something from you (API keys, permissions, data sources). You'll receive an email notification. Close the issue once resolved.

To send a directive proactively, create a GitHub Issue labeled `directive`. The agent will read it, act, comment to acknowledge, and close it.
