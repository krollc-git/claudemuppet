## The Concept: Autonomous Agent Orchestration (Working Title: "Muppet Framework")

AI coding agents (Claude Code, GitHub Copilot, OpenAI Codex) can now write code, run tests, and edit files autonomously. But they choke on large tasks — they ask endless clarifying questions, lose context, or try to do everything at once. The current workflow is: give the AI a prompt, babysit it, intervene when it goes sideways, repeat. The human is the bottleneck.

The muppet framework replaces the human in that orchestration loop. It's a methodology for autonomous AI agent orchestration where a lead AI agent (the "muppet") receives a high-level objective, decomposes it into parallel workstreams, spawns separate AI worker agents to do the actual implementation, reviews their output, rejects bad work, and iterates — all without a human in the loop.

Three design decisions make this different from existing multi-agent approaches:

**1. Guided Start, Then Forced Autonomy.** The first few minutes of a run are the highest-leverage moment for human input. The lead agent explores the problem, presents its decomposition plan, and asks 3-5 key questions. The human answers what matters, ignores what they don't care about, approves the plan, and walks away. From that point forward, the agent operates in "auto mode" — it cannot ask questions and is forced to make its own decisions. This is a deliberate design constraint, not a limitation. A bad concrete output you can react to is more valuable than a perfect clarification loop that produces nothing.

**2. Iterative Guide-Based Training.** The lead agent reads a "muppet guide" file on every startup — a set of accumulated natural language instructions about how to decompose tasks, what quality bar to enforce, and what decisions to make in ambiguous situations. After each autonomous run, the human reviews the results and updates the guide ("you broke tasks too large," "always run tests before accepting work," "when in doubt, ship something ugly that works"). Over 10-15 runs, this guide becomes a meaningful encoding of the human's judgment and preferences. It's not machine learning — it's accumulated written instructions, the same way you'd write better runbooks for a human team lead.

**3. Governance Rules.** The lead agent operates under non-negotiable constraints that have been refined through iterative testing:

- **Wait for approval**: The muppet cannot take any action beyond reading code until the human explicitly says "start working." This prevents premature execution before the plan is reviewed.
- **Orchestrator only**: The muppet never writes code itself — all implementation is delegated to worker agents.
- **Tracked delegation**: Every task is logged in a structured work tracker (beads). If it's not tracked, it didn't happen.
- **Isolated execution**: All worker agents operate in git worktrees, never on the production branch. Failures are contained.
- **No interactive widgets**: The muppet communicates in plain text, not GUI dialogs. The guided start should feel like a conversation.
- **Maximize parallelism**: The optimization target is wall-clock time, not token cost. When tasks can run concurrently, the muppet spawns multiple workers in parallel rather than serializing them.
- **Detach command**: The human can type "detach" to leave the muppet running in the background and reconnect later.

These rules prevent common failure modes: the AI doing half the work itself and losing the big picture, premature execution before alignment, serializing work that could be parallelized, and opaque progress tracking.

The framework is a methodology, not software. The "code" is configuration files and prompts. The novelty is in the workflow design — the specific combination of guided start, forced autonomy, guide-based iterative training, and governance constraints.

The concept was inspired in part by Steve Yegge's "Gas Town" (https://steve-yegge.medium.com/welcome-to-gas-town-4f25ee16dd04) — a multi-agent orchestrator framework he built on top of his beads issue tracker. Gas Town uses up to 20-30 agents with 7 specialized roles. The muppet framework takes a different approach: simpler architecture, fewer agents, but with the guided start, forced autonomy, and iterative guide-based training as the core innovations.

I have a working prototype running on my home infrastructure using Claude Code's experimental Agent Teams feature. The lead agent successfully decomposes tasks, spawns worker agents, and coordinates their output. I've been iterating on the governance rules and workflow design for the past few days, and the results are promising enough that I wanted to bring it to your attention early.

---

## Potential Use Cases

### 1. Intellectual Property / Patent

The individual components of this framework exist in prior art (multi-agent AI systems, task decomposition, human-in-the-loop AI). However, the specific combination of workflow innovations may be patentable:

- The guided start to forced autonomy transition as a deliberate design pattern
- Iterative guide-based training as a form of AI behavior shaping without fine-tuning
- The governance constraint architecture (orchestrator-only, tracked delegation, isolated execution)

I'd like to explore this with the university's tech transfer office. A prior art search would clarify whether there's enough novelty to file.

### 2. Research Publications

Several testable hypotheses are embedded in this framework, each of which could be an independent publication:

- **The forcing function hypothesis**: Does removing an AI agent's ability to ask clarifying questions (forced autonomy) produce better task completion outcomes than interactive clarification? Under what conditions? This is empirically testable.
- **Guide convergence**: How does the quality of the muppet guide improve over iterations? How many autonomous runs until the guide stabilizes? What's the learning curve shape?
- **Decomposition strategies**: What task granularity, parallelism patterns, and delegation strategies produce the best outcomes for AI worker agents?
- **Guided start ROI**: Quantifying the value of 2 minutes of human input at the start of an autonomous run vs. fully autonomous from the beginning.
- **Alignment through onboarding**: The muppet guide represents a novel form of AI alignment — shaping agent behavior through accumulated natural language performance feedback rather than fine-tuning or reinforcement learning. This concept extends beyond software engineering to any domain requiring autonomous AI agents that reflect human judgment.

### 3. Grant Funding

NSF, DARPA, and industry sponsors are actively funding AI agent research. This framework provides a concrete basis for a research proposal around "human-guided autonomous agent orchestration." The working prototype demonstrates feasibility, the research questions above provide a multi-year research agenda, and the interdisciplinary nature (software engineering, AI, human-computer interaction) broadens the potential funding sources. A strong publication would establish credibility for a grant application.

### 4. Pedagogy / Curriculum Integration

Writing a good muppet guide requires the same skills as effective software project management: clear task decomposition, explicit quality criteria, unambiguous delegation, and constructive performance feedback. "If you can't explain it to a muppet, you don't understand the problem well enough."

This could be integrated into software engineering or project management courses. Students learn orchestration and delegation by training AI agents rather than managing human teams — a novel pedagogical approach with a fast feedback loop (the AI runs the tasks immediately, students see results and iterate on their guide). This is also a publishable pedagogy paper.

### 5. Business Development / Licensing

Every company with a software engineering team is trying to figure out how to extract autonomous value from AI coding tools. The current state of the art is ad-hoc prompting with heavy human supervision. A structured methodology for autonomous agent orchestration — especially one with an iterative training component — has commercial value. Potential paths include licensing the methodology, consulting engagements, or a startup through the university's commercialization pipeline.

### 6. Proof of Concept: Weather Lookup Smoke Test

On February 19, 2026, I ran an end-to-end smoke test of the framework. The task was deliberately simple — have the muppet find the current weather for zip code 26501 — to validate the orchestration workflow without the complexity of a real software engineering task.

**What happened:**

1. I launched a muppet session on a remote server via SSH in a tmux session.
2. The muppet received its initialization prompt (the task objective plus all four governance rules).
3. It entered the **guided start** phase: presented its decomposition plan ("spawn one research teammate to look up the weather, review the result, report back") and waited for approval.
4. I connected to the session remotely using a custom `muppet-attach` script, reviewed the plan, and said "start working."
5. The muppet spawned a research teammate, the teammate performed a web search, and the muppet reported the result back: 59°F, overcast, in Morgantown, WV.
6. I tested the **detach** command (governance rule 4) to leave the muppet running in the background while disconnecting my terminal. Reattaching confirmed the session was still alive and responsive.

**What it validated:**

- The guided start → approval → autonomous execution workflow functions as designed.
- Governance rules are followed: the muppet orchestrated without doing the research itself.
- The detach/reattach workflow allows fluid human involvement — check in when you want, walk away when you don't.
- The infrastructure (Claude Code Agent Teams, tmux sessions, remote SSH access) is stable enough for longer autonomous runs.

### 7. Real-World Test: CSSP UI Redesign

Following the smoke test, I ran the framework against a real software engineering task: redesigning the UI of CSSP (Cyber Sandbox Software Portal), a Ruby on Rails web application used by WVU's LCSEE department for managing AWS cloud lab environments.

The muppet explored the codebase (11 view directories, 20+ ERB templates, Bootstrap 5.2 styling), presented a decomposition plan, and — after my approval — spawned parallel worker agents to handle layout, component styling, and page-specific redesigns concurrently. The entire process from launch to completed UI redesign took under 30 minutes of wall-clock time, including the guided start conversation.

This would have been a multi-day task for a single developer working manually. The parallelism governance rule — optimizing for wall-clock time over token efficiency — was the key factor. Multiple workers redesigning different view directories simultaneously compressed the timeline dramatically.
