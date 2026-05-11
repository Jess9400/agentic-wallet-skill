# Wallet Strategy Analyzer + Mutation Engine

An AI-powered on-chain skill for [OKX Agentic Wallet](https://web3.okx.com) that turns any trader's wallet history into a machine-readable trading strategy — then reverses, enhances, or hybridizes it.

Built for the **OKX Agentic Wallet Trading Competition — Skill Quality Prize** track.

---

## What It Does

Input one or more on-chain wallet addresses. The skill fetches up to 1,000 DEX trades, analyzes behavioral patterns, classifies the trader type, extracts IF-THEN strategy rules, and optionally mutates them.

### Five Modes

| Mode | Trigger | Description |
|---|---|---|
| `analyze` | "analyze this wallet" | Full behavioral profile: win rate, PnL, trader type, extracted rules |
| `reverse` | "fade this wallet" | Inverts a consistent loser's behavior into a contrarian strategy |
| `enhance` | "improve this strategy" | Keeps core edge, reduces leverage & drawdown, adds a volatility filter |
| `copy` | "copy-trade this wallet" | Generates entry/exit/sizing rules ready to deploy |
| `combine` | "merge these wallets" | Weights and merges 2–3 wallets by their individual strengths |

---

## How It Works

### Step 1 — Wallet Ingestion
Fetches 7–90 days of DEX history via OKX on-chain APIs:
- Win rate, realized/unrealized PnL, top tokens
- Full trade history (up to 1,000 records, paginated)
- Per-token PnL snapshots
- Current holdings

### Step 2 — Behavioral Profile
Classifies the trader into one of 8 types:

| Type | Detection Signals |
|---|---|
| Momentum Trader | Buys after breakout candles, short hold time |
| Mean Reversion Trader | Buys after drops, longer hold time |
| Scalper | >50 trades/week, <30 min avg hold |
| Swing Trader | 2–14 day holds, fewer larger positions |
| News/Event Trader | PnL clusters around volume spikes |
| Martingale / Gambler | Increasing size after losses, high drawdown |
| Liquidation Hunter | Short positions timed around cascade events |
| Accumulator | Repeated buys of same token over weeks |

Also computes **statistical confidence**: sample size quality, consistency across time frames, market regime dependency, and edge validity verdict.

### Step 3 — Strategy Extraction
Converts behavior into machine-readable rules:

```
Entry:  IF BTC breaks resistance AND volume > 1.5x average THEN long
Exit:   TAKE PROFIT at +8% | STOP LOSS at -3% | TIME EXIT after 6h
Size:   Max 15% of portfolio per position
Timing: US session only (9am–4pm EST)
Avoid:  Meme tokens <$500K market cap
```

### Step 4 — Strategy Mutation

**Reversal** — Fades consistently losing wallets by inverting their signals.  
**Enhancement** — Keeps the core edge, reduces leverage, adds a volatility filter.  
**Copy** — Generates actionable mirror-trading rules with lag window and risk controls.  
**Hybridization** — Merges 2–3 wallets weighted by their strongest dimension (entries from Wallet A, exits from Wallet B, sizing from Wallet C).

---

## OKX APIs Used

| Command | Purpose |
|---|---|
| `onchainos market portfolio-overview` | Win rate, PnL summary, top tokens |
| `onchainos market portfolio-dex-history` | Full DEX trade history |
| `onchainos market portfolio-recent-pnl` | Per-token recent PnL |
| `onchainos market portfolio-token-pnl` | Per-token PnL snapshot |
| `onchainos portfolio all-balances` | Current holdings |
| `onchainos leaderboard list` | Benchmark against top traders |
| `onchainos tracker activities` | Live wallet monitoring |

---

## Example Usage

```
User: analyze wallet 8sLMyyhSgBFz4hCff9EJjR6xd869VtW2nimEdqCz76ed on solana

→ Fetches 90-day DEX history
→ Win rate: 62% | Realized PnL: +$4,200 | 147 trades
→ Trader type: Momentum Trader
→ Extracts entry/exit/sizing rules
→ Offers: reverse / enhance / copy / combine options
```

```
User: combine wallets 0xAbc... and 0xDef... on ethereum

→ Analyzes both wallets independently
→ Wallet A: strong entries (momentum), weak exits
→ Wallet B: poor entries, excellent exit timing
→ Composite strategy: A's entries + B's exits + averaged sizing
```

---

## Installation

This skill is built for the [OKX Onchain OS](https://web3.okx.com/onchain-os) skill format.

```bash
# Install via skills CLI
npx skills add jessicanascimento/agentic-wallet-skill
```

Requires `onchainos` CLI — installed automatically on first use via the pre-flight check in `skill.md`.

---

## Competition Submission

Submitted to the **OKX Agentic Wallet Trading Competition** — Skill Quality Prize track.  
Competition page: https://web3.okx.com/boost/trading-competition/agentic-trading

**Author:** jessicanascimento2394@gmail.com  
**License:** MIT
