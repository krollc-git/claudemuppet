# krotrader Muppet — AI Autonomous Trading System Research

## Concept
Run a muppet (autonomous agent orchestrator) on krobot overnight to **invent a better trading system**, using krotrader as a starting point but not bound to it. Burn the nightly Claude Max token budget on autonomous research, experimentation, and backtesting. User kicks it off before bed, reviews results in the morning.

**This is NOT parameter tuning.** The AI has free reign to research and build entirely new approaches — new strategies, new indicators, new architectures. krotrader is the base to learn from, not a constraint.

## Hard Constraints (from user)
- Buys happen at end of day (market close)
- Sells are limit orders
- Sell lowest cost shares first (tax lot: LIFO by cost basis)
- Everything else is fair game

## krotrader Codebase (on krobot:~/github.com/krotrader/)
- **churninone.py** — Core backtester/simulator (~3500 lines). Churn trading: buy dips, sell small gains. Parameters: adaptive, cooldown, distance, sellpercent, stoploss, trailstop, regime, doubledown, buyonred, trytobuy, sharecap, etc.
- **prober.py** — Brute-force parameter optimizer. Runs `churninone --probe` for each ticker.
- **forecaster.py** — Runs optimized strategies forward 1 year, projects performance.
- **signals.py** — Reads `.avgroi` strategy files, emits buy signals → feeds automerrill/autoschwab.
- **sp500_probe.py** — Scans all S&P 500 stocks to find best candidates.
- **swingtrader.py** — Separate swing trading system with multiple strategy types.
- **supervisedstratgen.py** — Reverse-engineers strategy params from hindsight-optimal trades.
- **Config**: `~/.config/krotrader/prober.cfg` (or `~/.krobot/config/` on krobot)
- **User configs**: `~/.krobot/config/users/krowvebricks/prober.cfg` has the full ticker list

## Key Metrics
- Return on Average Capital (ROI)
- 3-Day Completion Rate
- Profit per week/year
- Sharpe ratio
- Max capital tied up

## What AI Should Explore
The AI is NOT limited to tweaking krotrader parameters. It should:
- **Research** — Web search for academic papers, quant strategies, proven approaches
- **Invent** — Design entirely new strategy architectures
- **Test everything** — Mean reversion, momentum, factor models, ML-based signals, volatility strategies, pairs trading, sector rotation, whatever shows promise
- **Build new backtesting code** if churninone is too limiting
- **Evaluate honestly** — Out-of-sample testing, walk-forward validation, no overfitting

## Git Worktree Approach
Use git worktrees so muppet research and production bot run independently:

```
~/github.com/krobot/              # main branch — production bot installs from here
~/github.com/krobot-muppet/       # research branch — muppet works here
```

**Setup:**
```bash
cd ~/github.com/krobot
git worktree add ../krobot-muppet -b research/muppet-YYYY-MM-DD
```

**How each thing runs:**
- **Running bot** — pip-installed from main into `~/.local/lib/python3.12/site-packages/krobot/`. Unaffected by repo changes until explicit reinstall.
- **Muppet** — works in `krobot-muppet/`, runs code directly with `PYTHONPATH=src python3 ...` (no pip install needed for backtesting/research)
- **Main branch updates** — make changes in `~/github.com/krobot/` on main, reinstall bot. Independent from muppet worktree.
- **Merging results** — `cd ~/github.com/krobot && git merge research/muppet-YYYY-MM-DD`

Single `.git` database shared between both directories — no push/pull needed, branches visible to both.

## CLI Prerequisite
Build a `krobot-cli` before starting muppet work. Gives the muppet a stable interface to:
- Run backtests: `krobot backtest NVDA --params ...`
- Fetch/inspect data: `krobot data NVDA`
- Run signals: `krobot signals`
- Test Schwab: `krobot schwab-test`

Without CLI, the agent wastes time writing throwaway scripts instead of inventing strategies.

## Design Plan
1. **Launcher**: `~/bin/krotrader-muppet` — SSH to krobot, tmux session, launch Claude Code with research prompt
2. **Research log**: Markdown file appended each iteration (what was tried, metrics, keep/revert)
3. **Baseline snapshot**: Run krotrader backtests before starting, record metrics to beat
4. **Git worktree**: `krobot-muppet/` on `research/muppet-YYYY-MM-DD` — main stays clean
5. **Backtest harness**: Standardized evaluation framework with out-of-sample validation
6. **Out-of-sample guard**: Train/test split to catch overfitting

## Muppet Architecture

### What is a Muppet?
A muppet is an autonomous agent orchestrator that replaces the human in the orchestration loop. Today Dave manually decomposes problems, opens tmux panes, gives each a focused prompt, checks output, and decides what's next. A muppet does exactly that — assess scope, decompose, delegate, review, iterate — without a human in the loop.

You're the puppeteer. The muppet is an imperfect version of you that runs the show while you're gone. Over time, through feedback, it gets better at being you.

### Guided Start
The first few minutes of a muppet run are the highest leverage for human input. The lead is reading the codebase, forming its decomposition plan, making the big architectural decisions that everything else flows from. A wrong turn there wastes the entire run. Catching it early is like finding a bug in requirements vs production — the earlier you find the problem the cheaper it is.

**The workflow:**
1. **Guided start** — give the muppet a prompt, it explores, asks 3-5 key questions ("should I preserve the existing API?" "React or vanilla JS?" "prioritize speed or correctness?"). You answer the ones that matter, ignore the ones you don't care about.
2. **Approve the plan** — the muppet shows its decomposition before spawning teammates. You glance at it, maybe say "split that task" or "don't bother with that one." Or just say "go."
3. **Auto mode** — from here it runs autonomously. No more questions, forced to decide on its own.

The guided start gets cheaper over time — as the muppet guide accumulates your preferences, there are fewer things it needs to ask about. Eventually it's just "here's my plan, looks good?" and you say go.

### Check-ins and Steering
The muppet isn't a black box you fire and forget — it's a team you can check in on at your discretion. Claude Code Agent Teams supports live interaction while the muppet is running:

- **Shift+Down** (in-process mode) cycles through teammates. Type to message any of them directly.
- **tmux split panes** — click into any pane to interact with that teammate's session.
- Other teammates keep working while you talk to one.

**What you can do mid-run:**
- Check in with the lead: "how's it going, what's the status?"
- Redirect a teammate: "wrong approach, try X instead"
- Give the lead new info: "don't touch the auth module, there's a deploy happening"
- Ask a teammate to explain what it's doing before it commits

Your level of involvement is fluid. Sometimes you let it run for hours untouched, sometimes you pop in every 20 minutes, sometimes you pair with the lead for the first 10 minutes then walk away. It depends on how much you trust the muppet and how critical the work is.

### The Forcing Function
Once in auto mode, the muppet is *forced* to make decisions. In an interactive session, Claude asks clarifying questions. An autonomous muppet can't ask — so it has to pick a direction, commit, produce something concrete. A bad concrete output is more useful than a perfect clarification loop. The forcing function of no human in the loop makes the AI actually decide instead of deferring. The system prompt explicitly says: never ask, always decide, document your reasoning.

### Implementation: Claude Code Agent Teams
Claude Code has built-in experimental agent teams (behind feature flag). This is the orchestration layer.

**Enable:**
```json
// ~/.claude/settings.json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

**How it works:**
- **Lead agent (the muppet)** — the main Claude Code session. Decomposes tasks, spawns teammates, reviews output. Does NOT do implementation itself.
- **Teammates** — separate Claude Code sessions, each with own context window. Claim tasks from shared task list, communicate directly with each other.
- **Shared task list (beads)** — tasks tracked as beads with dependencies. Blocked beads auto-unblock when predecessors close. `bd list` shows what was attempted, completed, or failed. Beads are the research log.
- **Hooks** — `TaskCompleted` can reject work that doesn't meet quality bar. `TeammateIdle` can redirect idle teammates.
- **tmux split panes** — each teammate gets its own pane (fits existing tmux workflow).

**Lead prompt guidance** — the muppet needs explicit instructions to be an orchestrator, not a player-coach:
> Decompose this into parallel workstreams. Spawn a teammate for each independent workstream. Don't do implementation work yourself — your job is to plan, delegate, and review.

Without this, the lead tends to do half the work itself and spawn one teammate for the rest.

### Alternatives Considered
- **Gas Town** (Steve Yegge) — full multi-agent orchestrator with 7 agent roles, built on beads. Manages 20-30 agents. Overkill but same philosophy.
- **OpenAI Codex** — "command center for agents" with built-in worktrees. Similar concept, different ecosystem.
- **claude-flow** — third-party multi-agent swarm framework for Claude Code.
- **Nightshift (Haplab)** — budget-aware overnight runner, but single-agent only (no orchestration).
- **Subagents (Task tool)** — lighter weight, but workers can only report back to parent, can't talk to each other, no shared task list.

Claude Code Agent Teams is the right fit: built-in, uses tmux, shared task list, inter-agent messaging, and the lead pattern matches what we want.

### Training the Muppet: Iterative Feedback Loop
The muppet starts fresh every session. It doesn't learn from previous runs automatically. "Training" the muppet means building up instruction files it reads on startup.

**Mechanism:**
1. **CLAUDE.md** — project-level rules, architecture, constraints. Read by muppet and all teammates.
2. **muppet-guide.md** — how the muppet should decompose, delegate, and evaluate. Preferences, quality bar, patterns.
3. **Memory files** — accumulated knowledge about what works and what doesn't.

**The feedback loop:**
1. Run the muppet overnight
2. Review results in the morning
3. Update the muppet guide ("you broke tasks too large," "you should have rejected that teammate's output," "always run tests first," "when overhauling UI start with layout before styling")
4. Next run reads updated guide, behaves better
5. Repeat

Over 10 runs the guide becomes a good encoding of your judgment. It's not ML training — it's accumulated written instructions, the same way you'd write better runbooks for a human team lead. The muppet gets better at being you.

**Example muppet-guide.md:**
```markdown
# How I want work decomposed
- Break UI work into: layout, components, styling, integration (in that order)
- Never have two teammates editing the same file
- Each task should take 10-15 minutes, not 45

# Quality bar
- Run tests before marking any task done
- Don't refactor code you weren't asked to touch
- If a teammate's output is bad, reject and respawn, don't try to fix

# My preferences
- Prefer simple CSS over framework abstractions
- Always preserve existing API contracts
- When in doubt, ship something ugly that works over something pretty that doesn't
```

### Lateral Muppet Communication (Experimental)
Multiple muppets running as separate sessions could communicate laterally through beads — sidestepping the "no grandchildren" limitation of Claude Code Agent Teams. Each muppet is a flat team (lead + teammates), and beads becomes the message bus between muppets:

- Research muppet builds a new strategy, writes results to a bead
- Review muppet picks up that bead, analyzes output, writes feedback
- Research muppet reads feedback, adjusts approach

This is more like a committee of managers with different perspectives negotiating through a shared work log, rather than a single hierarchy. No central authority — just muppets with different guide files coordinating through beads.

**Status: Speculative.** Will focus on single muppet architecture first. Lateral communication is a future extension once single-muppet runs are reliable.

### Token Considerations
Each teammate is a full Claude Code session. 5 teammates running in parallel burns 5x tokens. On a Max subscription burning the overnight budget this is fine, but worth monitoring on first few runs.

## Infrastructure
- krobot: 192.168.1.100, AMD 9600X 12-core, 30GB RAM, Ubuntu 24.04
- User has tmux launcher infrastructure already (tmuxclaude, tmuxheyclaude patterns)
- yfinance for historical data (already installed)
- numpy, pandas, scipy already available

## Status: INFRASTRUCTURE READY (2026-02-19)
- Claude Code installed on krobot, Agent Teams enabled and smoke-tested
- Next: build launcher script, muppet-guide.md, git worktree, then first run (CLI task)
