# OpenAI Codex App vs Muppet Framework Comparison

## What Codex App Is
OpenAI's desktop app (macOS, Feb 2026) for multi-agent coding. A "command center" where you dispatch parallel coding agents to work on tasks in isolated cloud containers/worktrees. Each agent gets its own sandbox with restricted filesystem and no network. Results come back as diffs/PRs for human review.

Key features:
- Cloud sandbox per task (isolated containers, no internet by default)
- Git worktrees for parallel work without conflicts
- Project threads (each thread = agent session with persistent history)
- Automations (scheduled jobs: bug triage, CI failure summaries, vuln scanning)
- App Server architecture: JSON-RPC over JSONL, Item/Turn/Thread primitives
- Powers CLI, VS Code, web, macOS app, JetBrains through single API
- codex-1 model optimized for coding tasks

## Where Muppets Are More Evolved

### 1. AI Orchestrator, Not Human Orchestrator
**Codex**: The developer IS the orchestrator. You sit in the control room dispatching tasks, reviewing diffs, manually deciding what to do when something fails. The agents are workers, but you're the manager.
**Muppets**: The orchestrator is an AI agent. It reads the task, breaks it into subtasks, spawns teammates, assigns work, monitors progress, handles failures, and re-routes — all autonomously. The human says "start working" and reviews outcomes. This is a fundamentally different level of autonomy: Codex automates coding, Muppets automate *engineering management*.

### 2. Governance as Architecture
**Codex**: Permission modes (never/ask/on-failure/always) for sandbox operations. Configuration is environment setup.
**Muppets**: Seven non-negotiable governance rules baked into every muppet prompt — not as optional settings, but as architectural invariants:
- Orchestrator NEVER does implementation (doing work directly is a bug, not a judgment call)
- ALL work delegated via Agent Teams (teammates with team_name/name params)
- Wait-for-start protocol (no actions until user says "start working")
- Beads for all task tracking
- Git worktrees mandatory (workers never touch main branch)
- Workers report failures upward; orchestrator decides resolution
- Minimize wall-clock time (parallelize internally, don't over-spawn)

These aren't toggles. They're the rules of engagement that make the system predictable and safe at scale.

### 3. Distributed Issue Tracking with Dependency Graphs
**Codex**: Tasks are threads with persistent history, but there's no dependency graph, no blocking relationships, no way to say "task B can't start until task A finishes."
**Muppets**: Beads (`bd`) is a git-backed issue tracker integrated into every muppet workflow. Every task gets a bead. Dependencies are explicit (`bd dep add`). `bd ready` shows what's unblocked. `bd blocked` shows bottlenecks. Issues are JSONL files in `.beads/` committed to the repo — they travel with the code across machines, sessions, and agents. Any agent on any machine can pick up where another left off.

### 4. Formal Escalation Protocol
**Codex**: Agent fails → developer investigates. There's no structured escalation path.
**Muppets**: Explicit three-tier model: (1) Workers keep it simple — do steps, stop and report on failure. (2) Orchestrator receives the report, diagnoses the problem, decides resolution, re-dispatches. (3) Only risky or destructive decisions escalate to the human. This means most failures are handled without human intervention, but the human retains veto power over anything dangerous.

### 5. Prompt Engineering as System Design
**Codex**: A generic agent runs in a configured environment. The "architecture" is the sandbox setup script.
**Muppets**: Every muppet is instantiated from a carefully crafted prompt template that includes governance rules, task context, project-specific knowledge, and behavioral constraints. The `claudemuppet` repo contains reusable templates, governance docs, and launch/attach scripts. The prompt isn't configuration — it IS the system architecture. Changing the prompt changes the system's behavior, capabilities, and safety properties.

### 6. Persistent State That Lives in the Repo
**Codex**: Threads support resumption and archival, but state is per-thread, per-project, inside OpenAI's infrastructure.
**Muppets**: All state lives in git. Beads issues are JSONL files in `.beads/`. CLAUDE.md files carry project context. Memory files persist across sessions. When you push the repo, you push the project's entire management state with it. No vendor lock-in, no cloud dependency for state persistence.

### 7. Multi-Machine Orchestration
**Codex**: Cloud containers on OpenAI's infrastructure. One machine (your Mac) dispatches to their cloud.
**Muppets**: Orchestrate across your own physical machines via SSH. krodesktop launches muppets on krobot, cssp, 123p, etc. Each remote machine has its own Claude Code instance, its own beads database, its own CLAUDE.md. The muppet framework isn't limited to one vendor's cloud — it works across any machines you can SSH into.

### 8. tmux-Based Session Management
**Codex**: macOS desktop app. Cloud tasks continue if you close the app, but you need the app to interact.
**Muppets**: `muppet-launch` starts an orchestrator in a tmux session. `muppet-attach` reconnects. Detach anytime, reattach from anywhere — over SSH, from a phone, from a different machine. Works headless on any Linux box. No GUI dependency, no macOS requirement. The "detach" command is even a governance rule.

## Where Codex App Is Better

### 1. Polished UI
Desktop app with diff views, thread management, project organization, visual review workflow. Muppets are terminal-only — powerful but not pretty.

### 2. GitHub PR Integration
Codex agents create PRs directly from their work. Muppets work with local git; creating PRs is a separate step.

### 3. Scheduled Automations
Codex supports recurring tasks — daily bug triage, CI failure summaries, vulnerability scanning. Muppets are on-demand only; there's no cron-like scheduler built in.

### 4. Cloud Scale
Cloud containers mean effectively unlimited parallel agents without provisioning hardware. Muppets are bounded by the machines you own and Claude API rate limits.

### 5. Lower Barrier to Entry
Any ChatGPT subscriber clicks "New Task" and goes. Muppets require understanding tmux, SSH, beads, governance rules, and prompt templates — it's a power-user framework.

## The Bottom Line

Codex App is a **polished multi-agent IDE with cloud sandbox** — great for dispatching independent coding tasks in parallel and reviewing diffs. OpenAI's own framing says it best: *"your repo got a control room."*

Muppets are an **AI-orchestrated autonomous workforce with governance** — the orchestrator is itself an AI that plans, delegates, handles failures, and tracks everything in a distributed issue tracker. It's closer to having an autonomous engineering team than a tool you supervise.

The key philosophical difference: **Codex puts the human as orchestrator supervising AI workers. Muppets put an AI as orchestrator with the human as the executive who says "go" and reviews outcomes.**
