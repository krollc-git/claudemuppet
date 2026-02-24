# Muppet Framework — Overview

**The Muppet Framework**

So the idea is: right now I'm the orchestrator. I decompose a big task in my head, open multiple Claude sessions, give each one a focused prompt, check their output, and decide what's next. I'm the bottleneck.

A muppet replaces me in that loop. It's a lead Claude agent that receives a high-level prompt ("overhaul the trading strategy engine"), explores the codebase, breaks the work into tasks, spawns teammate agents to do the actual coding, reviews their output, rejects bad work, and spawns the next round. You give it work and let it run — check in when you want, or don't.

**Guided start, then auto mode:** You don't just throw a prompt at the muppet and walk away (though you can). The first few minutes are the highest leverage — the muppet explores the codebase, asks you 3-5 key questions, and shows its decomposition plan. You answer what matters, ignore what you don't care about, approve the plan, and it switches to auto mode. From there it's forced to make its own decisions — no more questions. Catching a bad architectural choice in minute 2 is a 30-second correction. Catching it after 4 hours means you throw everything away.

**Why it works better than just giving Claude a big prompt:** A single Claude session chokes on "overhaul the UI" — it asks 10 clarifying questions or tries to do everything at once and runs out of context. In auto mode, the muppet *can't* ask questions. It's forced to make decisions and produce something. A bad concrete output you can react to is worth more than a perfect clarification loop that produces nothing.

**Check in anytime:** The muppet isn't a black box. While it's running you can message the lead ("how's it going?"), redirect a teammate ("wrong approach, try X"), or give new info ("don't touch the auth module"). Other teammates keep working while you talk to one. Your involvement is fluid — let it run for hours untouched, or pop in every 20 minutes. It depends on how much you trust the muppet and how critical the work is.

**Infrastructure:**
- **Beads** (git-backed issue tracker) tracks every task the muppet creates. Run `bd list` and see what was attempted, completed, or failed. It's the work log.
- **Git worktrees** isolate the muppet's work from production. It works in a separate checkout, main branch is never touched. Good results get merged, bad results get deleted.
- **Claude Code Agent Teams** (experimental feature) is the engine — lead + teammates with shared task list and inter-agent messaging.

**The real unlock — iterative training:** After each run you review what the muppet decided and add rules to a `muppet-guide.md`: "break tasks smaller," "reject output without tests," "start with layout before styling." The muppet reads this on every startup. Over time it becomes a pretty good encoding of your judgment. It's not ML training — it's accumulated instructions, like onboarding a team lead who gets better with every performance review.

**You can also have multiple muppets** with different guide files. A conservative one for production work, an aggressive one for research, a review-only one that finds bugs but never writes code. Same framework, different personality. You could even chain them — aggressive muppet experiments first, conservative muppet reviews the output after.

It's a methodology, not software. The "code" is just configuration files and prompts. Anyone could build their own muppet — but everyone's would be different because the guide reflects *your* judgment and preferences.

## Infrastructure — Verified 2026-02-19

**Claude Code Agent Teams** is live and working. Smoke-tested on krobot.

**How to enable:**
```json
// ~/.claude/settings.json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

**How it works in practice:**
- Tell Claude in natural language to create a team — no special CLI flag
- `Shift+Down` to cycle between teammates, `Ctrl+T` for task list
- Teammates message each other directly (not just back to lead)
- For tmux split panes: `--teammate-mode tmux`
- For unattended runs: `--dangerously-skip-permissions` (teammates inherit)
- Team artifacts: `~/.claude/teams/` and `~/.claude/tasks/`

**Tools used by lead:**
- `TeamCreate` — create a named team
- `Task` — spawn a teammate with a prompt
- `TaskOutput` — wait for teammate result
- `SendMessage` — message a teammate (e.g. shutdown request)
- `TeamDelete` — clean up team

**Known limitations:**
- No session resumption (`/resume` doesn't restore teammates)
- No nested teams (teammates can't spawn their own teams)
- One team per session
- Higher token usage (each teammate = full Claude instance)

**Verified on:**
- krobot (192.168.1.100): Claude Code 2.1.47 installed at `~/.local/bin/claude`, Agent Teams enabled in settings.json, OAuth credentials copied from krodesktop
- krodesktop: Claude Code 2.1.47 at `~/.local/bin/claude`

## Governance Rules

These rules define how a muppet must behave. They are non-negotiable and must be included in every muppet's system prompt.

### Rule 1: Wait for "start working"
The muppet MUST NOT take any non-read-only action until the user explicitly says "start working." During guided start, the muppet explores the codebase, presents its plan, asks questions as plain text, and waits. No beads, no branches, no worktrees, no teammates, no file writes — NOTHING until "start working." This must be stated prominently at the top of every muppet prompt, before and inside the governance rules.

### Rule 2: Orchestrator Only — Teammates Do ALL Work
A muppet is an orchestrator. It MUST NOT do implementation work itself. ALL work — coding, testing, research, file editing, running commands, infrastructure tasks — MUST be delegated to a teammate. The muppet's job is ONLY to plan, decompose, delegate, review, and decide. The muppet doing work directly is a bug, not a judgment call.

**The standard**: ALL implementation goes to teammates. No exceptions. Not even "it's just one small change." The muppet never writes code, edits files, runs tests, or executes multi-step procedures.

**What the muppet may do directly (only when necessary to unblock teammates):**
- Create git worktrees and beads
- Check if a tool/package is available before spawning a teammate that needs it
- Verify a teammate's output after completion (read-only review)

If the muppet catches itself doing work instead of delegating, it must stop and spawn a teammate instead.

**Teammate model — capable teammates, not dumb workers:**
Teammates are full Claude instances. They can parallelize internally using Task tool sub-agents. The muppet knows this and should decompose at a high level — one teammate per logical unit of work, not one per sub-task.

- **Default to fewer teammates.** One teammate for "add 3 new enemy types" — the teammate spawns 3 Task sub-agents internally. This is lighter than 3 separate teammates.
- **Multiple teammates only when it saves wall-clock time.** Use multiple teammates when tasks are truly independent and unrelated (e.g., "overhaul sound engine" + "add pause menu"), or when one would block the other if done sequentially.
- **Teammates handle their own parallelism.** The muppet doesn't micromanage how a teammate structures its work internally.

**Communication model — Use Agent Teams:**
Teammates MUST be spawned using Claude Code Agent Teams (TeamCreate, SendMessage). This gives bidirectional communication — when a teammate hits a problem, it reports back, the muppet decides the resolution, and sends updated instructions. The `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` flag must be enabled (already set in `~/.claude/settings.json`).

### Rule 3: Beads for All Task Tracking
The muppet MUST use beads (`bd`) to track all work. Every task delegated to a teammate must have a corresponding bead. The muppet creates beads before spawning teammates, updates status as work progresses, and closes beads when work is reviewed and accepted. Beads are the work log — if it's not in beads, it didn't happen. Teammates must also be instructed to update their bead status (in_progress when starting, notes on approach/blockers).

### Rule 4: Git Worktrees for All Teammates
Every teammate MUST work in a git worktree, never on the main branch. The muppet creates worktrees before spawning teammates (e.g. `git worktree add ../krobot-muppet-taskname -b muppet/taskname`). This isolates teammate work from production and from each other. No two teammates should edit the same worktree simultaneously. The muppet merges accepted work back to the muppet's branch after review.

### Rule 5: No Interactive Widgets
Do NOT use the AskUserQuestion tool. Present all questions as plain text in the conversation. The user will type responses naturally. The guided start should feel like a chat, not a form.

### Rule 6: Minimize Wall-Clock Time
The goal is reduced wall-clock time, not token efficiency. Teammates are capable and can parallelize internally using Task sub-agents — the muppet doesn't need to spawn many teammates to get parallelism. Spawn multiple teammates only when the tasks are truly independent and concurrent teammates would finish faster than one teammate handling both. One capable teammate doing 3 things in parallel internally is better than 3 teammates with coordination overhead.

### Rule 7: Detach Command
If the user types "detach", the muppet MUST immediately run `tmux detach-client` via Bash. No confirmation, no response — just detach. This lets the user leave the muppet running in the background without needing tmux key combos.

### Rule 8: Orchestrator Handles Problems, Not Workers
Worker instructions stay simple: "do these steps, stop and report if something goes wrong." Workers do NOT need decision trees or contingency plans baked in — that's overfitting to the last problem. When a worker stops and reports a problem, the orchestrator reacts, thinks about it, decides the safe resolution, and re-dispatches the worker with updated instructions. Only escalate to the user for genuinely risky or destructive decisions.

## Tooling

### Scripts (~/claude/github.com/muppet/bin/, symlinked to ~/bin/)

**muppet-launch** — One-command muppet startup.
```
muppet-launch <name> <prompt-file> [workdir]
```
Example:
```
muppet-launch cli prompts/krobot-cli.txt ~/github.com/krobot
```
What it does: scp's the prompt to krobot, creates tmux session `muppet-<name>`, starts Claude Code with `--dangerously-skip-permissions`, waits for Claude to be ready, sends the prompt, and submits it. Prints connection instructions when done.

**muppet-attach** — Connect to a running muppet.
```
muppet-attach
```
Lists running muppet-* sessions on krobot, auto-attaches if one, prompts if multiple.

### Prompts (~/claude/github.com/muppet/prompts/)

Reusable prompt files for different muppet tasks. Each includes the governance rules and task-specific context.
- `krobot-cli.txt` — Build CLI for krobot

### Lessons from early launches (2026-02-19)

**What was clunky before muppet-launch:**
1. Manually SSH to krobot and create tmux session with correct PATH and cd
2. Wait for Claude to start, check with tmux capture-pane
3. Write prompt to temp file, scp to krobot
4. Send prompt via `tmux send-keys` — shows as "[Pasted text #1 +N lines]"
5. Send a separate Enter to submit (paste doesn't auto-submit in Claude Code TUI)
6. Check capture-pane again to verify it's working

**Fixed by muppet-launch:** All 6 steps are now one command.

**tmux nested prefix conflict:** krodesktop and krobot both used `Ctrl+B`. Fixed by setting krobot's tmux prefix to `Ctrl+A` (`~/.tmux.conf` on krobot). Now: `Ctrl+B` = local, `Ctrl+A` = remote.

**Mouse scrolling:** krobot had no `~/.tmux.conf`. Added `set -g mouse on`.

**Detach UX:** `Ctrl+A, d` works but user prefers typing "detach" — handled by governance rule 4 (muppet runs `tmux detach-client`).
