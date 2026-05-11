---
name: wallet-strategy-analyzer
description: "AI-powered on-chain wallet strategy analysis, mutation, and autonomous execution engine. Input one or more wallet addresses to: (1) analyze trading history, win rate, PnL, leverage patterns, and behavioral profile; (2) extract machine-readable IF-THEN strategy rules from any trader's behavior; (3) mutate strategies via reversal (fade losers), enhancement (lower risk, same edge), or hybridization (merge multiple wallets into a synthetic strategy); (4) auto-discover top traders from the leaderboard and rank them by edge quality; (5) autonomously execute copy-trades with risk-gated daily limits, pre-trade security scans, and automatic stop-loss enforcement. Triggers: 'analyze wallet', 'what strategy does this wallet use', 'extract trading rules from address', 'copy this trader', 'auto-copy', 'start copying', 'reverse this loser wallet', 'enhance strategy', 'combine wallets', 'build strategy from wallet', 'what type of trader is this', 'show win rate', 'trader profile', 'strategy mutation', 'fade this wallet', 'find best trader to copy', 'discover top wallets', 'auto-execute', 'set daily limit'. Do NOT use for: simple balance checks (use okx-wallet-portfolio); generic PnL queries without strategy intent (use okx-dex-market); signal alerts (use okx-dex-signal)."
license: MIT
metadata:
  author: jessicanascimento2394@gmail.com
  version: "1.1.0"
  homepage: "https://web3.okx.com/boost/trading-competition/agentic-trading"
---

# Wallet Strategy Analyzer + Mutation Engine

Turn any on-chain wallet into a machine-readable trading strategy — then reverse, enhance, or hybridize it.

**Seven modes:**

| Mode | Trigger phrases | What it does |
|---|---|---|
| `analyze` | "analyze wallet", "what strategy", "trader profile" | Full behavioral analysis of a single wallet |
| `reverse` | "reverse", "fade", "invert this wallet" | Inverts a consistently losing wallet into a contrarian strategy |
| `enhance` | "enhance", "improve risk", "same edge lower risk" | Keeps core edge, reduces leverage and drawdown |
| `copy` | "copy", "copy-trade rules", "replicate" | Generates actionable copy-trading entry/exit rules |
| `combine` | "combine", "merge wallets", "hybridize" | Merges 2–3 wallets into a synthetic composite strategy |
| `discover` | "find best trader", "top wallets", "who should I copy" | Auto-ranks leaderboard traders by edge quality and recommends the best |
| `auto-copy` | "start copying", "auto-execute", "mirror automatically" | Fully autonomous: monitor → security scan → execute → report |

---

## Pre-flight Checks

> Read `../okx-agentic-wallet/_shared/preflight.md`. If that file does not exist, read `_shared/preflight.md` instead.

---

## Instruction Priority

1. **`<NEVER>`** — Absolute prohibition. Never bypass.
2. **`<MUST>`** — Mandatory step. Skipping breaks the skill.
3. **`<SHOULD>`** — Best practice. Deviation acceptable with reason.

---

## Step 0 — Mode Detection

Before any command, identify the user's intent:

| If user says... | Mode |
|---|---|
| "analyze", "profile", "what strategy", "trader type" | → **analyze** |
| "reverse", "fade", "invert", "short this loser" | → **reverse** |
| "enhance", "reduce risk", "same edge", "improve" | → **enhance** |
| "copy", "copy-trade", "replicate", "mirror" | → **copy** |
| "combine", "merge", "hybridize", 2+ wallets given | → **combine** |
| "find best", "discover", "top traders", "who should I copy", "leaderboard" | → **discover** |
| "start copying", "auto-copy", "auto-execute", "mirror automatically", "set daily limit" | → **auto-copy** |

If mode is ambiguous with a single wallet given, default to **analyze** and ask if they also want mutation options after the report.

---

## Step 1 — Wallet Ingestion

### 1a. Resolve chain

<MUST>
Ask the user which chain(s) to analyze if not specified. Supported chains for PnL analysis: call `onchainos market portfolio-supported-chains` and present the list. For balance queries: call `onchainos portfolio chains`.
</MUST>

Default chain priority when not specified: `solana` → `ethereum` → `base`.

### 1b. Fetch data (run in parallel where possible)

For each wallet address provided, fetch all of the following:

```bash
# PnL overview — win rate, total realized PnL, top tokens, buy/sell stats
onchainos market portfolio-overview --address <wallet> --chain <chain> --time-frame 3

# DEX trade history — up to 1000 records (paginate with --cursor if needed)
onchainos market portfolio-dex-history --address <wallet> --chain <chain>

# Recent PnL by token — last 100 positions
onchainos market portfolio-recent-pnl --address <wallet> --chain <chain>

# Current holdings — what the wallet holds right now
onchainos portfolio all-balances --address <wallet> --chains <chain>
```

`--time-frame` mapping:
- `1` = 1 day, `2` = 3 days, `3` = 7 days, `4` = 30 days, `5` = 90 days

<MUST>
Always fetch `--time-frame 3` (7D) as the baseline. If the user requests a longer window, also fetch `--time-frame 4` (30D) and `--time-frame 5` (90D) for trend comparison.
</MUST>

<NEVER>
Never fabricate or estimate trade counts, PnL, or win rates. Only use values returned by the CLI. If a chain is unsupported, say so and suggest alternatives.
</NEVER>

---

## Step 2 — Behavioral Profile (analyze mode)

After ingestion, compute and display the **Trader Profile** using this fixed template:

```
## Trader Profile — {walletShort}

**Chain:** {chain}  |  **Period:** {timeframe}

### Performance Summary
| Metric | Value |
|--------|-------|
| Win Rate | {winRate}% |
| Realized PnL | {realizedPnl} USD |
| Unrealized PnL | {unrealizedPnl} USD |
| Total Trades | {totalTrades} |
| Avg Hold Time | {avgHoldTime} |
| Best Token | {topToken} (+{topPnl} USD) |

### Trader Type
**Primary:** {traderType}
**Secondary traits:** {secondaryTraits}

### Behavioral Fingerprint
{behaviorBullets}

### Statistical Confidence
| Signal | Assessment |
|--------|------------|
| Sample size | {sampleQuality} |
| Consistency | {consistency} |
| Market regime | {regimeDependency} |
| Edge validity | {edgeValidity} |
```

#### Field-mapping rules

**`{walletShort}`** — First 6 + last 4 chars of address (e.g. `0x0fcc...7cb8`). Never show full address unless user asks.

**`{traderType}`** — Classify from the table below using `portfolio-overview` + `portfolio-dex-history` signals:

| Type | Detection signals |
|---|---|
| Momentum Trader | Buys after breakout candles, short hold time (<4h avg), high volume, positive PnL on trending tokens |
| Mean Reversion Trader | Buys after drops, longer hold time (>24h), negative correlation between entry timing and recent candle direction |
| Scalper | >50 trades/week, very short hold (<30 min avg), small per-trade PnL, high trade frequency |
| Swing Trader | 2–14 day hold times, fewer but larger positions, trades macro setups |
| News/Event Trader | Token PnL clusters around launch dates or high-volume spike events |
| Martingale / Gambler | Increasing position sizes after losses, high max drawdown, inconsistent sizing |
| Liquidation Hunter | Short positions timed around leveraged longs, profits during cascades |
| Accumulator | Repeated buys of same token over weeks, low sell frequency |

**`{secondaryTraits}`** — Comma-separated secondary patterns detected (e.g., "tends to FOMO near highs, cuts losses quickly").

**`{behaviorBullets}`** — 3–5 bullet points synthesized from trade history. Examples:
- "Enters after 15%+ green candles (momentum trigger)"
- "Exits within 6 hours regardless of profit"
- "Concentrates 60% of volume in top 3 tokens"
- "Wins more often during US market hours (9am–4pm EST)"

**`{sampleQuality}`** — Based on trade count:
- <20 trades → "Low (statistically inconclusive — treat as directional only)"
- 20–100 trades → "Moderate (patterns emerging, validate over time)"
- >100 trades → "High (statistically meaningful)"

**`{consistency}`** — Compare 7D vs 30D win rates. If delta >15%: "Inconsistent (regime-sensitive)". If delta <5%: "Consistent".

**`{regimeDependency}`** — If top profitable tokens are all meme/high-beta: "Bull market dependent". If profitable in mixed conditions: "Regime-neutral".

**`{edgeValidity}`** — Verdict: "Statistically valid", "Lucky streak — insufficient evidence", or "Negative edge (consistent loser — reversal candidate)".

---

## Step 3 — Strategy Extraction

After the profile, always generate machine-readable strategy rules:

```
## Extracted Strategy Rules — {walletShort}

### Entry Conditions
{entryRules}

### Exit Conditions
{exitRules}

### Position Sizing
{sizingRules}

### Timing
{timingRules}

### Avoid
{avoidRules}
```

#### Rule generation logic

**Entry rules** — derive from `portfolio-dex-history`:
- Most common token types bought (meme, L1, DeFi blue chip?)
- Typical entry timing relative to volume spikes
- Format: `IF <condition> AND <condition> THEN enter`

**Exit rules** — derive from average hold time and realized PnL distribution:
- Does wallet cut losses or hold? (compare avg loss hold vs avg win hold)
- Format: `TAKE PROFIT at +{n}% | STOP LOSS at -{n}% | TIME EXIT after {h}h`

**Sizing rules** — derive from position value distribution:
- Avg trade size in USD
- Max single-position concentration

**Timing rules** — derive from trade timestamps:
- Active hours / days (if pattern detected)

**Avoid rules** — invert worst-performing patterns:
- Token types with negative PnL, or chains where wallet consistently loses

---

## Step 4 — Strategy Mutation

### Mode: REVERSE

<MUST>
Only offer reversal when `{edgeValidity}` = "Negative edge (consistent loser — reversal candidate)" OR the user explicitly requests reversal. Never suggest reversing a profitable wallet without a disclaimer.
</MUST>

```
## Reversal Strategy — Fading {walletShort}

**Thesis:** This wallet has a consistent negative edge. Inverting its behavior exploits its predictability.

### Reversal Rules
{reversalEntryRules}
{reversalExitRules}
{reversalSizingRules}

### Confidence
{reversalConfidence}

⚠️ Disclaimer: Reversal strategies assume the loser remains consistent. If behavior changes, the edge disappears. Monitor weekly.
```

**Reversal logic:**
- Original entry trigger → reversed signal (e.g., wallet buys breakouts → you SHORT breakouts or wait for exhaustion)
- Original leverage → halved (loser-reversal is higher-conviction but uncertainty is still high)
- Original exit timing → tightened (exit sooner; don't mirror the holder behavior)

### Mode: ENHANCE

```
## Enhanced Strategy — {walletShort}

**Thesis:** The core edge is valid, but risk parameters are suboptimal.

### Original vs Enhanced
| Parameter | Original | Enhanced |
|-----------|----------|---------|
| Leverage | {origLev}x | {enhLev}x |
| Stop Loss | {origSL}% | {enhSL}% |
| Position size | {origSize} | {enhSize} |
| Hold time | {origHold} | {enhHold} |
| Volatility filter | None | Added |

### Enhancement Rules
{enhancedRules}

### Expected Impact
- Same entry signals as original
- Reduced max drawdown by ~{ddReduction}%
- Lower peak return, but higher Sharpe
```

**Enhancement logic:**
- If original leverage implied >30% drawdown → reduce leverage to target max 15% drawdown
- Add volatility filter: skip entry if recent 24h token volatility >50%
- Tighten stop loss to half the average losing trade loss
- If wallet trades concentrated in 1–2 tokens, suggest diversifying to top 3–5

### Mode: COPY

```
## Copy-Trading Rules — {walletShort}

Ready-to-deploy entry and exit rules for mirroring this wallet.

### Trigger Wallet
Monitor: {walletFull}
Chain: {chain}
Alert on: Buy transactions > ${minTradeSize} USD

### Entry Rule
When {walletShort} buys token X:
- Confirm token is not a honeypot (run okx-security scan)
- Check liquidity > $50,000
- Enter within {lagWindow} minutes of detected buy
- Max position: {maxPosition}% of portfolio

### Exit Rule
- Mirror wallet's sell if detected within {exitLag} minutes
- OR: TAKE PROFIT at +{tp}% from your entry
- STOP LOSS at -{sl}% from your entry (do NOT mirror if wallet holds through loss)

### Risk Controls
- Max daily copy-trades: {maxDailyTrades}
- Skip if wallet buy size < ${minMirrorSize} (likely a test tx)
- Skip tokens with market cap < $500K
```

**Copy logic:**
- `{lagWindow}` — based on wallet's avg hold time: if <1h, use 5-min lag; if >4h, use 15-min lag
- `{tp}` — 70% of wallet's avg winning trade return
- `{sl}` — 50% of wallet's avg losing trade loss (exit sooner than original)
- `{maxDailyTrades}` — cap at wallet's median daily trade count

### Mode: COMBINE

<MUST>
Require at least 2 wallet addresses for combine mode. Run full analyze (Step 2–3) on each before synthesizing.
</MUST>

```
## Composite Strategy — Synthetic Wallet

Merging {n} wallets into one AI-generated strategy.

### Wallet Contributions
| Wallet | Strength Used | Weight |
|--------|---------------|--------|
| {wallet1Short} | {strength1} | {weight1}% |
| {wallet2Short} | {strength2} | {weight2}% |
| {wallet3Short} | {strength3} | {weight3}% |

### Composite Entry Rules
{compositeEntry}

### Composite Exit Rules
{compositeExit}

### Composite Sizing
{compositeSize}

### Expected Edge
{compositeEdge}
```

**Combination logic:**
- **Wallet weighting**: proportional to each wallet's win rate (higher win rate = more weight on its signal)
- **Strength extraction**: use each wallet's strongest dimension (e.g., Wallet A has best entries → use its entry rules; Wallet B has best exits → use its exit timing)
- **Composite entry**: require signal agreement from the wallet owning the entry role; use others as confirmation filters
- **Composite exit**: use exit rules from the wallet with the best realized PnL exit timing

---

## Step 5 — Next Steps Prompt

After every mode's output, always append:

```
---
**What would you like to do next?**
- 🔄 Run a different mutation (reverse / enhance / copy / combine)
- 📊 Analyze another wallet for comparison
- 🔁 Combine this wallet with another for a composite strategy
- 🏆 Discover the best traders on the leaderboard to copy
- 🤖 Start autonomous copy-trading with a daily risk limit
- 💡 Explain any part of the strategy in more detail
```

---

## Step 6 — Leaderboard-to-Strategy Pipeline (discover mode)

Automatically finds, ranks, and analyzes the best traders on-chain — zero manual input required.

### 6a. Fetch leaderboard candidates

<MUST>
Run all three leaderboard queries in parallel to get a complete picture across ranking dimensions.
</MUST>

```bash
# Confirm chain is supported
onchainos leaderboard supported-chains

# Fetch top 20 by win rate (7D)
onchainos leaderboard list --chain <chain> --time-frame 3 --sort-by 2 --limit 20

# Fetch top 20 by realized PnL (7D)
onchainos leaderboard list --chain <chain> --time-frame 3 --sort-by 1 --limit 20

# Fetch top 20 by ROI (7D)
onchainos leaderboard list --chain <chain> --time-frame 3 --sort-by 5 --limit 20
```

`--sort-by` values: `1`=PnL, `2`=win rate, `3`=tx count, `4`=volume, `5`=ROI
`--time-frame` values: `1`=1D, `2`=3D, `3`=7D, `4`=30D, `5`=90D

### 6b. Score and deduplicate

For each unique wallet address returned across all three queries, compute a **composite edge score**:

```
edgeScore = (winRate × 0.4) + (roiNormalized × 0.35) + (consistencyBonus × 0.25)
```

- `winRate` — from leaderboard response (0–100)
- `roiNormalized` — ROI scaled to 0–100 relative to the highest ROI in the result set
- `consistencyBonus` — +25 if wallet appears in ≥2 leaderboards; +0 if only in one

Rank all wallets by `edgeScore` descending. Take the top 5.

### 6c. Auto-analyze top 3

Run full **Steps 1–3** (analyze mode) on the top 3 wallets automatically. Display each profile in compact form:

```
## Leaderboard Discovery Results — {chain} ({timeframe})

Scanned {n} unique wallets across PnL, Win Rate, and ROI leaderboards.

### Top Traders by Edge Score

| Rank | Wallet | Edge Score | Win Rate | ROI | Trader Type | Recommendation |
|------|--------|------------|----------|-----|-------------|----------------|
| 1 | {w1Short} | {s1} | {wr1}% | {roi1}% | {type1} | {rec1} |
| 2 | {w2Short} | {s2} | {wr2}% | {roi2}% | {type2} | {rec2} |
| 3 | {w3Short} | {s3} | {wr3}% | {roi3}% | {type3} | {rec3} |

### Best Copy Candidate: {w1Short}
{extractedRulesCompact}

Would you like me to start autonomous copy-trading for {w1Short}? I'll need your budget and daily loss limit.
```

`{rec}` values: "Strong copy candidate", "Copy with caution (regime-sensitive)", "Enhance before copying", "Reversal candidate only"

<NEVER>
Never auto-execute any trade in discover mode. Discovery is read-only. Only proceed to execution after explicit user confirmation in auto-copy mode.
</NEVER>

---

## Step 7 — Autonomous Copy-Trade Execution (auto-copy mode)

Fully autonomous loop: monitor target wallet → detect new buy → security gate → size position → execute → report.

### 7a. Setup confirmation

<MUST>
Before starting any autonomous execution, collect and confirm ALL of the following parameters with the user:
</MUST>

```
## Auto-Copy Setup — Confirm Parameters

**Target wallet:** {walletShort}
**Chain:** {chain}
**Budget:** ${budget} total allocated
**Max per trade:** ${maxPerTrade} ({maxPct}% of budget)
**Daily loss limit:** ${dailyLossLimit} (stop all trades if hit)
**Min trade to mirror:** ${minMirrorSize} (ignore smaller buys — likely test txs)
**Security scan:** Required before every trade ✓
**Liquidity floor:** $50,000 minimum ✓

Type **confirm** to start, or adjust any parameter above.
```

<NEVER>
Never begin autonomous execution without receiving the word "confirm" or explicit affirmation from the user on the setup parameters above.
</NEVER>

### 7b. Monitor loop

```bash
# Watch target wallet for new transactions
onchainos tracker activities \
  --tracker-type multi_address \
  --wallet-address <targetWallet> \
  --chain <chain> \
  --trade-type 1
```

Poll on each new buy transaction detected. For each detected buy:

### 7c. Pre-trade security gate

<MUST>
Run ALL of the following checks before executing. Fail any one → skip this trade, log reason, continue monitoring.
</MUST>

```bash
# 1. Security scan on the token being bought
onchainos security token-scan --address <tokenAddress> --chain <chain>

# 2. Check your current balance is sufficient
onchainos portfolio all-balances --address <yourWallet> --chains <chain>

# 3. Get current token price for position sizing
onchainos market price --address <tokenAddress>
```

**Gate conditions — skip trade if ANY of these are true:**

| Check | Skip condition |
|---|---|
| Security scan | `riskLevel` = high or `isHoneypot` = true |
| Daily loss limit | Today's realized losses ≥ `{dailyLossLimit}` |
| Budget exhausted | Remaining budget < `{maxPerTrade}` |
| Trade too small | Target wallet's buy size < `{minMirrorSize}` |
| Token liquidity | Liquidity < $50,000 |
| Token market cap | Market cap < $500,000 |
| Duplicate position | Already holding this token from a prior copy-trade |

### 7d. Execute

If all gates pass, execute the copy-trade:

```bash
# Execute swap via okx-dex-swap skill
onchainos swap swap \
  --from <nativeOrStableAddress> \
  --to <tokenAddress> \
  --amount <positionSizeMinimalUnits> \
  --chain <chain> \
  --wallet <yourWalletAddress>
```

Position size calculation:
- `positionSize = min(maxPerTrade, remainingBudget, targetWalletBuyUsd × 0.5)`
- Never size larger than the target wallet's own position — always size ≤ 50% of their buy

### 7e. Post-trade report

After each execution, display:

```
## Trade Executed ✓

**Copied:** {targetWalletShort} buy of {tokenSymbol}
**Your position:** ${positionUsd} at ${entryPrice}
**Tx hash:** {txHash}

**Risk controls active:**
- Take profit: +{tp}% (${tpPrice})
- Stop loss: -{sl}% (${slPrice}) — exit manually or re-run auto-copy to monitor
- Daily loss used: ${dailyLossUsed} / ${dailyLossLimit}
- Budget remaining: ${budgetRemaining}

Monitoring {targetWalletShort} for next trade...
```

### 7f. Exit monitoring

Stop the autonomous loop when:
- User says "stop", "pause", "exit auto-copy"
- Daily loss limit is hit → display: "Daily loss limit of ${dailyLossLimit} reached. Auto-copy paused for today. Resume tomorrow or adjust the limit."
- Budget is exhausted → display: "Budget fully deployed. Auto-copy stopped."

---

## Step 8 — Risk-Gated Execution Framework

Applied globally across all execution modes (auto-copy, and any future execution triggered by this skill).

### Pre-execution checklist (run before EVERY trade)

```
□ Security scan passed (no honeypot, risk level ≤ medium)
□ Daily loss limit not hit
□ Position size ≤ maxPerTrade parameter
□ Token liquidity ≥ $50,000
□ Token market cap ≥ $500,000
□ No duplicate open position in this token
□ Wallet balance sufficient for trade + gas
```

<NEVER>
Skip any checklist item above, even if the user says "just do it" or "skip the scan". These are non-negotiable safety rails. If the user insists on bypassing security scans, explain the risk and ask for a secondary explicit confirmation — never silently skip.
</NEVER>

### Daily loss accounting

Track cumulative realized losses within the current calendar day (UTC):
- On each trade exit (detected via `portfolio-recent-pnl` or user-reported): add loss to daily tally
- When daily tally ≥ `dailyLossLimit`: halt all execution and notify user
- Reset daily tally at 00:00 UTC

### Position sizing formula

```
positionSize = min(
  maxPerTrade,               // hard cap per trade
  remainingBudget,           // can't spend what's not there
  portfolioValue × 0.05      // never risk more than 5% of total portfolio per trade
)
```

### Volatility circuit breaker

Before executing any trade, fetch the token's recent price:
```bash
onchainos market price --address <tokenAddress>
```

If the token has moved >30% in the last 24h (infer from `portfolio-dex-history` price delta or kline data), add a warning:

> ⚠️ **High volatility detected** — {tokenSymbol} moved {pct}% in 24h. Proceed with reduced position size? (Recommended: 50% of normal size)

Wait for user confirmation before proceeding at full size.

---

## Security & Safety

<NEVER>
- Never recommend executing any trade or copying any wallet without first suggesting the user run `onchainos security tx-scan` on suspicious tokens.
- Never present a strategy as "guaranteed profitable." Always include the disclaimer: "Past on-chain behavior does not guarantee future results. All strategies carry risk."
- Never expose full wallet addresses in user-facing output unless the user explicitly asks. Use truncated form `{first6}...{last4}`.
- Never recommend unlimited token approvals.
- Never fabricate PnL figures, trade counts, or win rates. All numbers must come from CLI output.
</NEVER>

<MUST>
When the `edgeValidity` is "Lucky streak — insufficient evidence", add a prominent warning:
> ⚠️ **Statistical Warning**: This wallet's performance may reflect luck rather than skill. The sample size is insufficient for reliable strategy extraction. Treat any derived rules as hypotheses, not validated strategies.
</MUST>

---

## Error Handling

| Error | Action |
|---|---|
| Chain not supported by portfolio PnL | Run `onchainos market portfolio-supported-chains`, present supported list, ask user to choose |
| Wallet has <5 trades | "Insufficient history for pattern recognition. Need at least 20 trades for a reliable profile." |
| Network error | Retry once. If still failing, report error and suggest checking connection. |
| Address format invalid | Validate: EVM = `0x` prefix, 42 chars. Solana = Base58, 32–44 chars. Ask user to re-enter if invalid. |
| Rate limit hit | Suggest creating personal API key at https://web3.okx.com/onchain-os/dev-portal |

---

## CLI Quick Reference

See `references/cli-reference.md` for full parameter tables.
