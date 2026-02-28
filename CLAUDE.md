# Muppet Framework

The Muppet Framework turns Claude Code into an autonomous engineering team. A "muppet" is a lead Claude agent that receives a high-level task, breaks it into subtasks, spawns teammate agents to do the work, reviews output, and decides what's next. The human approves the plan and reviews outcomes — the AI handles everything in between.

## Prerequisites

- **Claude Code** — installed and authenticated (`claude --version`)
- **beads** (`bd`) — git-backed issue tracker ([github.com/steveyegge/beads](https://github.com/steveyegge/beads)), v0.49+
- **tmux** — terminal multiplexer for session management

## Setup

Run the automated setup script:

```bash
./bin/muppet-setup
```

This will:
1. Verify prerequisites (Claude Code, beads, tmux)
2. Run `bd setup claude` to install Claude Code hooks
3. Enable Agent Teams in `~/.claude/settings.json`
4. Symlink `muppet-launch` and `muppet-attach` to `~/bin/`
5. Create a default `config/hosts.conf`
6. Print a verification summary

To check prerequisites without installing: `./bin/muppet-setup --check-only`

## Launching a Muppet

```bash
# Remote host
muppet-launch <host> <name> <prompt-file> [workdir]

# Local
muppet-launch local <name> <prompt-file> [workdir]
```

Examples:
```bash
muppet-launch krobot cli prompts/krobot-cli.txt ~/github.com/krobot
muppet-launch local test prompts/TEMPLATE.txt /tmp
```

## Attaching to a Running Muppet

```bash
muppet-attach          # scan all configured hosts
muppet-attach local    # only check local sessions
```

## Creating a New Prompt

1. Copy `prompts/TEMPLATE.txt`
2. Replace the `[REPLACE ...]` placeholders with your task, context, and constraints
3. Keep the governance rules block intact — it's the framework's safety architecture
4. See `prompts/krobot-cli.txt` for a complete example

## Governance Rules (Summary)

These rules are non-negotiable and must appear in every muppet prompt. See `docs/muppet-overview.md` for full text.

1. **Wait for "start working"** — no actions until user approves the plan
2. **Orchestrator only** — muppet never does implementation, delegates everything to teammates
3. **Beads for tracking** — every task gets a bead
4. **Git worktrees** — teammates work in worktrees, never main branch (skip if no git repo)
5. **No AskUserQuestion** — plain text questions only
6. **Minimize wall-clock time** — prefer fewer capable teammates over many
7. **Detach command** — typing "detach" immediately detaches tmux
8. **Orchestrator handles problems** — workers report failures, orchestrator decides resolution

## Key Files

| File | Purpose |
|------|---------|
| `bin/tmux-claude` | Start Claude Code in a tmux session |
| `bin/muppet-launch` | Launch a muppet session (local or remote) |
| `bin/muppet-attach` | List and connect to running muppets |
| `bin/muppet-setup` | Automated first-time setup |
| `prompts/TEMPLATE.txt` | Prompt template with governance rules and placeholders |
| `prompts/krobot-cli.txt` | Example: krobot CLI prompt |
| `config/hosts.conf` | Host list for muppet-attach (gitignored, user-specific) |
| `config/hosts.conf.example` | Template for hosts.conf |
| `docs/muppet-overview.md` | Full framework documentation |
| `docs/codex-vs-muppets.md` | Comparison with OpenAI Codex |
| `muppet-guide.md` | Iterative training guide (example) |
