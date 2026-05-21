# Congress Trader

A reference architecture for an autonomous trading system that mirrors high-performing
Congressional trade disclosures. Built around the STOCK Act's public-record filings —
no inside information, no special access.

> ⚠️ **This is an architecture and scaffold, not a production trading system.** It does
> not place real trades out of the box. Code that actually calls broker APIs lives in
> Phase 1 and onward — implement it carefully, test it on paper for at least 60 days,
> and never run it with money you can't afford to lose. The author is not your financial
> advisor.

## Why this is legal

The [STOCK Act of 2012](https://en.wikipedia.org/wiki/STOCK_Act) requires members of
Congress to publicly disclose their personal stock transactions within 45 days. Once
filed, these PTRs (Periodic Transaction Reports) are public record. Commercial
products like **Capitol Trades**, **Quiver Quant**, **Unusual Whales**, and the
**NANC** / **KRUZ** ETFs all operate on this exact data. The strategy this repo
implements — wait for disclosure, evaluate, optionally mirror — is documented and
widely available.

## Architecture

Three layers:

```
Layer 1 — Autonomous runtime (Python + Task Scheduler/cron)
    Each script handles one trading function on its own schedule.
    Scripts call:
      - Anthropic API for Claude reasoning (Haiku for monitoring, Sonnet for signals)
      - Broker API (Alpaca paper-first, then live)
      - Telegram for human notifications
    Writes state to JSON files.

Layer 2 — Claude Code sub-agents (.claude/agents/)
    Human-in-the-loop oversight from your IDE.
    Read the same state files and config that Layer 1 writes.
    Can update watchlist, risk params, or research individual members.

Layer 3 — State files (state/)
    JSON bridge between Layer 1 (autonomous) and Layer 2 (interactive).
    Manual kill switch: create state/HALT to pause all trading.
```

## Repository layout

```
congress-trader/
├── README.md                  ← this file
├── CLAUDE.md                  ← agent kickoff prompt (build the Layer 1 scripts)
├── config/
│   ├── watchlist.json         ← curated members + tier weights (public performance data)
│   └── risk_params.json       ← position sizing, stop-loss, daily caps
└── skills/_archived/          ← SKILL.md prompts for each Python script's Claude call
    ├── congress-monitor/      ← polls filings every 4h, dedup, signal trigger
    ├── signal-generator/      ← combines momentum + sentiment + member weight → GO/WATCH/NO-GO
    ├── market-analyst/        ← on-demand quote + chart context
    ├── risk-manager/          ← gates trade against position size, daily loss, cash
    ├── trade-executor/        ← places order via broker API
    ├── sentiment-analyzer/    ← Reddit + news headline polarity
    ├── portfolio-reviewer/    ← daily P&L + position review at market close
    └── member-researcher/     ← monthly performance refresh + watchlist tuning
```

The companion **Claude Code sub-agents** (interactive IDE side) live separately —
see the [claude-code-subagent-cookbook](https://github.com/rngauf/claude-code-subagent-cookbook)
repo's `agents/congress/` folder.

## Watchlist

`config/watchlist.json` is curated from publicly reported performance data. Examples
of priority-tier members at time of writing:

| Member | Chamber | Reason tracked |
|---|---|---|
| Ro Khanna (D-CA) | House | 112% excess return vs S&P Jan 2024–Apr 2026; AI/semis focus |
| Nancy Pelosi (D-CA) | House | Consistent 3-year outperformer (65%, 70.9%, 20.1%) |
| Warren Davidson (R-OH) | House | 2025 #1 performer (+78.8% vs S&P 16.6%) |
| Terri Sewell (D-AL) | House | 2025 #3 performer (+67.9% vs S&P 16.6%) |

Refresh sources (built into the `monthly_refresh` block of `watchlist.json`):
[unusualwhales.com/politics](https://unusualwhales.com/politics),
[quiverquant.com/congresstrading](https://quiverquant.com/congresstrading),
[capitoltrades.com](https://capitoltrades.com).

## Phases

| Phase | Status | Trigger |
|---|---|---|
| 1 — Paper trading foundation | Build first | Manual |
| 2 — AI intelligence + sentiment | After Phase 1 | Phase 1 passes end-to-end test |
| 3 — Live trading | After 60+ days paper, win rate >50% | Manual switch in `.env` |
| 4 — Options trading | Portfolio ≥ $2,300 | Telegram alert, manual enable |

## What you need

- Python 3.10+
- An Anthropic API key
- A broker account (Alpaca recommended — free paper trading, free live API)
- Free API keys: Finnhub (filings), FMP (filings backup), MarketAux (news), Reddit (sentiment)
- A Telegram bot for notifications

Set all keys in `.env` (gitignored, never commit).

## Disclaimers

- **Not financial advice.** This is a coding project.
- **STOCK Act filings can lag 45 days.** The disclosure-mirror strategy is structurally
  late by design. Modeling assumes you accept that.
- **Past performance does not guarantee future results.** Members on the priority tier
  in 2024 may not be on it in 2027.
- **Test on paper for at least 60 days** before connecting real money.

## License

MIT
