# krogambler Muppet — AI Autonomous NHL Betting Model Research

## Concept
Run a muppet (autonomous agent orchestrator) on krobot overnight to **invent a better NHL betting model**, using the current Elo system as a starting point but not bound to it. Free reign to research and build entirely new approaches.

## Work Location
**krobot repo** (`krobot:~/github.com/krobot/`) in a research branch — NOT krogambler.
- krobot is the production Discord bot framework with NHL module at `src/krobot/nhl/`
- krogambler is a standalone reference implementation (not used at runtime)
- krobot rule: "Never import from or reference krotrader/krogambler at runtime"
- Branch: `research/nhl-muppet-YYYY-MM-DD` off main

## Hard Constraints (from user)
- Must produce moneyline picks with edges (model prob vs market prob)
- Everything else is fair game

## Reference: krogambler standalone (krobot:~/github.com/krogambler/)

### Architecture
- **Elo-based** moneyline value betting system
- Compares model win probability to market implied probability to find edges
- Data from ESPN API (scores, schedules, odds) + MoneyPuck (advanced stats)

### Key Files
- **nhl/predictor.py** — Main prediction engine. MoneylinePredictor class. Analyzes games, applies adjustments (goalie, B2B, form, roster/injury via GAR)
- **nhl/elo.py** — SimpleEloSystem. K-factor=6, HIA=14 Elo pts, margin-of-victory multiplier, mean regression 1/3 per offseason
- **nhl/backtest.py** — Backtester + grid search prober. Tests param combos across historical seasons
- **nhl/data.py** — Data fetching/storage (ESPN API, SQLite)
- **nhl/scraper.py** — Game roster scraping from ESPN
- **nhl/config.py** — Config management, constants
- **nhl/roi.py** — ROI calculations
- **nhl/cli.py** — CLI interface
- **nhlprober.cfg** — Parameter ranges for grid search

### Data Available
- **nhl_data.db** — Game results, schedules (SQLite)
- **nhl_odds_history.db** — Historical odds for ROI backtesting
- **moneypuck_cache/** — MoneyPuck skater/goalie advanced stats (xGoals, GAR, GSAE)
- **current_nhl_elo_ratings.json** — Current live Elo ratings
- Seasons: 2022-2025

### Current Parameters (from nhlprober.cfg)
- HIA: 8-24 (step 4)
- K_FACTOR: 4-10 (step 2)
- B2B_PENALTY: 10-30 (step 5)
- FORM_WEIGHT: 0-3.0 (step 1.0)
- BACKUP_PENALTY: 0-16 (step 4)
- GOALIE_SV_WEIGHT: 50-200 (step 50)
- GAR_SCALE: 100-250 (step 50)

### Known Gaps (from backtestplan.md)
- B2B_PENALTY and FORM_WEIGHT exist in config but are NOT applied during backtest
- Backtest starts all teams at flat 1500 Elo (no roster-based initialization)
- No injury/roster adjustments during backtest
- Historical game roster data partially collected via ESPN API
- Odds are from ESPN's single provider feed (stale/consensus, not sharp lines)

### Baseline Performance
- ~57% accuracy (games where model favorite wins)
- Target: 58-60% with full adjustments
- ROI backtesting available but limited by odds data

## What AI Should Explore
Not limited to Elo. Should research and try:
- **Beyond Elo** — Glicko-2, TrueSkill, Bradley-Terry variants, Bayesian rating systems
- **Feature-rich models** — ML/statistical models using MoneyPuck advanced stats directly (xGoals, Corsi, high-danger chances, GSAE)
- **Situational factors** — Rest days, travel distance, altitude, timezone, schedule density, playoff implications
- **Player-level modeling** — Individual player impact models vs team-level aggregation
- **Market-based signals** — Line movement analysis, closing line value, reverse line movement
- **Ensemble methods** — Combine multiple model types for better calibration
- **Academic literature** — Sports analytics papers, Pinnacle/sharp market analysis
- **Calibration optimization** — Not just accuracy but proper probability calibration (Brier score)
- **Kelly criterion / bankroll management** — Optimal bet sizing given edge estimates

## Why This Is a Great Muppet Candidate
- Fast feedback loop (backtest runs in seconds vs minutes for stock strategies)
- Clear baseline to beat (57% accuracy, ROI)
- Simpler model = more room for novel approaches
- Huge academic/community literature on sports betting models
- Well-structured historical data already in SQLite

## Muppet Architecture
Same architecture as krotrader muppet — see `krotrader-muppet.md` for full details.

- **Claude Code Agent Teams** (experimental flag) for lead + teammate pattern
- **Git worktree**: `krobot-muppet/` on `research/nhl-muppet-YYYY-MM-DD`
- **Lead agent (muppet)** decomposes research into parallel workstreams (e.g., one teammate researches Glicko-2, another builds ML model on MoneyPuck stats, another backtests situational factors)
- **Beads** track all tasks — the research log
- **muppet-guide.md** for iterative feedback on the muppet's decomposition and quality bar
- Fast backtests (seconds, not minutes) make this ideal for agent teams — teammates can test many approaches in parallel

## Infrastructure
- krobot: remote host, AMD 9600X 12-core, 30GB RAM, Ubuntu 24.04
- Python deps: requests, numpy, scipy (already installed)
- Data: ESPN API, MoneyPuck CSVs, SQLite databases

## Status: NOT STARTED — revisit later
