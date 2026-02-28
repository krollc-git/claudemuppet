# Muppet Framework

An autonomous AI engineering team, not just AI coding assistants.

A **muppet** is a Claude Code agent that acts as an engineering lead. You give it a task ("overhaul the trading engine"), it explores the codebase, breaks the work into subtasks, spawns teammate agents to write code, reviews their output, handles failures, and delivers results. You approve the plan upfront, then check in when you want.

The key difference from other multi-agent frameworks: **the orchestrator is an AI**, not you. You're the executive who says "go" and reviews outcomes. The muppet handles decomposition, delegation, failure recovery, and progress tracking autonomously.

## Install

See [INSTALL.md](INSTALL.md) for full setup instructions.

Quick version: install prerequisites (Claude Code, beads, tmux), then `./bin/muppet-setup`.

## Quick Start

```bash
# Start Claude Code in tmux
tmux-claude

# Launch a muppet
muppet-launch local my-task prompts/my-task.txt ~/my-project

# Attach to a running muppet
muppet-attach
```

## How It Works

1. **Write a prompt** — Copy `prompts/TEMPLATE.txt`, fill in your task context
2. **Launch** — `muppet-launch` starts a Claude Code session in tmux with your prompt
3. **Guided start** — The muppet explores the codebase, asks you key questions, shows its plan
4. **Say "start working"** — The muppet creates beads, spawns teammates, and begins
5. **Check in anytime** — `muppet-attach` reconnects. Redirect teammates, add info, or just watch
6. **Review outcomes** — Accepted work gets merged; bad work gets rejected and re-done

## Governance Rules

Every muppet follows these non-negotiable rules (included in every prompt):

1. **Wait for "start working"** — Guided start is read-only. No code, no branches, no beads until you approve.
2. **Orchestrator only** — The muppet never writes code. All implementation is delegated to teammates via Agent Teams.
3. **Beads for all tracking** — Every task gets a bead (`bd create`). If it's not in beads, it didn't happen.
4. **Git worktrees** — Teammates work in isolated worktrees, never on main.
5. **No interactive widgets** — Questions are plain text, not form fields.
6. **Minimize wall-clock time** — Fewer capable teammates over many simple ones.
7. **Detach command** — Type "detach" to leave the muppet running in the background.
8. **Orchestrator handles problems** — Workers report failures upward; the orchestrator decides what to do.

## Creating Your First Muppet

```bash
cp prompts/TEMPLATE.txt prompts/my-feature.txt
```

Edit `prompts/my-feature.txt`:
- Replace `[TASK DESCRIPTION]` with what you want built
- Replace `[PROJECT CONTEXT]` with codebase details
- Replace `[CONSTRAINTS]` with boundaries and requirements
- Keep the governance rules block unchanged

Then launch:
```bash
muppet-launch local my-feature prompts/my-feature.txt ~/my-project
```

## Iterative Training

After each run, review what the muppet decided and add lessons to a `muppet-guide.md` in your project. The muppet reads this on startup and improves over time. See the included `muppet-guide.md` for an example template.

This isn't ML training — it's accumulated instructions, like onboarding a team lead who gets better with every performance review.

## Infrastructure

- **[Beads](https://github.com/steveyegge/beads)** — Git-backed issue tracker. Every task tracked as JSONL in `.beads/`.
- **[Claude Code Agent Teams](https://docs.anthropic.com/)** — Lead + teammates with shared task list and messaging.
- **tmux** — Session management. Launch, detach, reattach from anywhere.
- **Git worktrees** — Isolated checkouts so teammates never conflict.

## Documentation

- `docs/muppet-overview.md` — Full framework documentation
- `docs/codex-vs-muppets.md` — Comparison with OpenAI Codex
- `docs/examples/` — Example muppet plans for specific projects
- `muppet-guide.md` — Example iterative training guide

## Philosophy

> A bad concrete output you can react to is worth more than a perfect clarification loop that produces nothing.

Muppets are forced to make decisions in auto mode. They can't ask infinite clarifying questions. This produces imperfect but concrete output that you can redirect — which is faster than getting nothing while an agent deliberates.
