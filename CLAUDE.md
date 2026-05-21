# Congress Trader — Build Agent Instructions

This file is a kickoff prompt for **Claude Code** to autonomously build the Phase 1
Python scripts described in `README.md`. If you'd rather build manually, follow the
README and the archived skill prompts.

To use with Claude Code: open this directory in VS Code with the Claude Code extension,
then say "Read CLAUDE.md and build Phase 1." The agent will work through each phase,
flipping `[ ]` to `[V]` on each task as it completes.

---

## Mission

You are the build agent for the Congress Trader system. Your job is to build the
Phase 1 Python scripts (paper trading) end-to-end. Use the skill prompts in
`skills/_archived/` as the Claude system prompts for each script's API calls.

**Important — what you do NOT do:**
- Do NOT register API accounts or generate API keys. Tell the user which accounts to
  create (links below) and have them paste keys into `.env`.
- Do NOT enable live trading. Phase 1 is paper trading only. Live trading requires
  60+ days of paper validation and a manual switch in `.env`.
- Do NOT execute trades without the `state/HALT` kill-switch wired in. If `state/HALT`
  exists, every trade-execution script must exit immediately.

---

## Phase 1 — Build the autonomous runtime

### 1.1 — Bootstrap

- [ ] Create a Python venv at `./venv` and `pip install -r requirements.txt`
- [ ] Verify `.env.example` exists with placeholders for every required key
- [ ] Verify `state/HALT` kill-switch logic is wired into every trade-executing script

### 1.2 — Phase 1 scripts

Build each of these Python scripts. The system prompt for each script's Claude API
call lives in `skills/_archived/<name>/SKILL.md`.

| Script | Schedule | Model | System prompt source |
|---|---|---|---|
| `congress_monitor.py` | Every 4h weekdays | Haiku | `skills/_archived/congress-monitor/SKILL.md` |
| `signal_generator.py` | On-demand (called by monitor) | Sonnet | `skills/_archived/signal-generator/SKILL.md` |
| `risk_manager.py` | On-demand (called by signal) | Haiku | `skills/_archived/risk-manager/SKILL.md` |
| `trade_executor.py` | On-demand (called by risk) | Haiku | `skills/_archived/trade-executor/SKILL.md` |
| `portfolio_reviewer.py` | 4:15 PM weekdays | Sonnet | `skills/_archived/portfolio-reviewer/SKILL.md` |
| `member_researcher.py` | First Sunday monthly | Sonnet | `skills/_archived/member-researcher/SKILL.md` |

### 1.3 — Scheduling

- [ ] Create the Windows Task Scheduler jobs (one per schedule above) on Windows, or
  the equivalent cron entries on Linux.

### 1.4 — Validation

- [ ] Run end-to-end on paper for at least one week and confirm: filings poll, signals
  generate, risk gate fires, paper trades execute, portfolio reviewer reports at close.
- [ ] Verify `state/HALT` actually halts all trading within one polling cycle.

---

## State files (the contract between Layer 1 and Layer 2)

| File | Written by | Contents |
|---|---|---|
| `state/portfolio_state.json` | trade_executor + portfolio_reviewer | Open positions, cash, total value |
| `state/trades.json` | trade_executor | Complete trade history with reasoning |
| `state/signals.json` | signal_generator | All GO/WATCH/NO-GO signals + score breakdown |
| `state/member_state.json` | congress_monitor | Last-seen filing date per member |
| `state/member_performance.json` | member_researcher | Win rates, avg return per member |
| `state/seen_trades.json` | congress_monitor | Dedup hash set |
| `state/system_health.json` | every script | Heartbeat, last run, errors |
| `state/HALT` | manual | Kill switch (any content) |

The companion Claude Code sub-agents in
`claude-code-subagent-cookbook/agents/congress/` read these files.

---

## API accounts the user must create

| Service | What for | Where |
|---|---|---|
| Alpaca | Paper + live brokerage | alpaca.markets |
| Finnhub | Congressional filings + quotes | finnhub.io |
| FMP | House/Senate filing backup | financialmodelingprep.com |
| MarketAux | News sentiment (5K req/mo free) | marketaux.com |
| Reddit | PRAW sentiment | reddit.com/prefs/apps |
| Telegram | Notifications | @BotFather |
| Anthropic | Claude API for reasoning | console.anthropic.com |
