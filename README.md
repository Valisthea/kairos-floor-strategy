# KAIROS Floor Strategy

**Autonomous AI trading strategy for Hyperliquid perpetual futures**
*OKX Onchain OS Plugin — Season 1 Challenge Submission*

> They sell tools. We run your fund.

---

## What It Does

KAIROS Floor runs a 4-phase autonomous trading pipeline:

1. **SENTINEL** — Detects the market regime (trending/ranging/volatile) using ADX, EMA alignment, and Fear & Greed
2. **SCANNER** — Scores trade setups on 9 technical factors (EMA, ADX, RSI, MACD, volume, multi-TF, ATR breakout, BB compression)
3. **GUARDIAN** — Validates risk: max positions, daily loss limits, funding rates, direction lock
4. **EXECUTOR** — Sizes positions by conviction tier (SCOUT → APOCALYPSE), places orders via Hyperliquid Plugin

## Conviction Tiers

| Tier | Score | Leverage | Risk/Trade |
|------|-------|----------|------------|
| SCOUT | 0-30 | 5x | 0.5% |
| STANDARD | 30-50 | 10x | 1.0% |
| CONFIDENT | 50-70 | 15x | 1.5% |
| CONVICTION | 70-85 | 20x | 2.0% |
| APOCALYPSE | 85+ | 25x | 2.0% |

## Quick Start

```bash
# Check status
node scripts/config.mjs status

# Verify connectivity
node scripts/market-data.mjs ping

# Run one cycle (paper trading by default)
node scripts/kairos-engine.mjs cycle --symbols BTC,ETH,SOL

# Run autonomously for up to 8 hours (1-480 min)
node scripts/kairos-engine.mjs auto --duration 120

# Manage open positions (run every 30s during trading)
node scripts/risk-manager.mjs check-positions

# Switch to live trading (requires explicit confirmation)
node scripts/config.mjs set-mode live --confirm

# Switch back to paper trading
node scripts/config.mjs set-mode dry-run

# Emergency stop (halts cycle + auto at next boundary)
touch .kairos-data/HALT     # create
rm .kairos-data/HALT        # resume
```

## Safety First

- **Dry-run mode is the default.** No real orders are placed until you explicitly run `config.mjs set-mode live --confirm`.
- **Mode is read from persisted state only** — there is no `--mode` CLI override. This prevents accidental live trading.
- **Kill switch:** create `.kairos-data/HALT` to halt trading at the next cycle boundary without killing the process.
- Every position has a stop-loss. If SL placement fails, the engine emergency-closes the position.
- Circuit breakers: daily loss cap (5%), consecutive loss cap (5), max drawdown (15%).
- Step-lock protection locks in every $2 of profit.
- Time stop: positions auto-close after 120 minutes.
- All state writes are atomic (tmp + rename) so a crash mid-write cannot corrupt state.
- All CLI arguments to the Hyperliquid binary are passed as discrete argv entries — no shell, no split-on-whitespace.

## Risk Disclaimer

⚠️ **ADVANCED RISK LEVEL.** This strategy trades perpetual futures with leverage. You can lose all your invested capital. This is not financial advice. Never trade with funds you cannot afford to lose.

## Requirements

- Node.js 18+
- Hyperliquid Plugin installed in OKX Onchain OS
- USDC balance on Hyperliquid

## Architecture

```
kairos-floor-strategy/
├── SKILL.md                    # AI agent brain — orchestrates everything
├── plugin.yaml                 # Plugin manifest
├── .claude-plugin/plugin.json  # Claude skill metadata
├── scripts/
│   ├── kairos-engine.mjs       # Core 4-phase pipeline engine
│   ├── risk-manager.mjs        # Position management (trailing stops, step-lock)
│   ├── market-data.mjs         # API wrappers (Hyperliquid, CoinGecko, F&G)
│   └── config.mjs              # Configuration management
├── references/
│   └── strategy-guide.md       # Detailed strategy documentation
└── LICENSE                     # MIT
```

## License

MIT © 2026 Kairos Lab
