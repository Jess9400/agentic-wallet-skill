# CLI Reference — Wallet Strategy Analyzer

All commands used by this skill. Parameters marked `*` are required.

---

## Portfolio PnL Commands (okx-dex-market)

### portfolio-supported-chains
```bash
onchainos market portfolio-supported-chains
```
Returns list of chains that support PnL analysis. Always run before portfolio queries if chain is uncertain.

---

### portfolio-overview
```bash
onchainos market portfolio-overview \
  --address <wallet>*  \
  --chain <chain>*     \
  --time-frame <n>     # 1=1D 2=3D 3=7D 4=30D 5=90D (default: 3)
```
**Returns:** `winRate`, `realizedPnl`, `unrealizedPnl`, `totalBuyCount`, `totalSellCount`, `top3Tokens[]`

---

### portfolio-dex-history
```bash
onchainos market portfolio-dex-history \
  --address <wallet>*  \
  --chain <chain>*     \
  --begin <ms>*        # start timestamp (milliseconds) — REQUIRED
  --end <ms>*          # end timestamp (milliseconds) — REQUIRED
  --cursor <cursor>    # pagination cursor from previous response
```
**Note:** `--begin` and `--end` are required. Compute as Unix ms:
- `end` = `Date.now()` (current time in ms)
- `begin` = end − (days × 86400 × 1000), e.g. 30 days = end − 2592000000
**Returns:** paginated list of DEX trades — `txHash`, `tokenSymbol`, `side` (buy/sell), `amountUsd`, `pnl`, `timestamp`, `cursor`

Paginate by passing last item's `cursor` as `--cursor` on the next call.

---

### portfolio-recent-pnl
```bash
onchainos market portfolio-recent-pnl \
  --address <wallet>*  \
  --chain <chain>*     \
  --limit <n>          # default: 100, max: 100
  --cursor <cursor>
```
**Returns:** per-token recent PnL entries — `tokenSymbol`, `realizedPnl`, `unrealizedPnl`, `holdDuration`, `entryPrice`, `exitPrice`

---

### portfolio-token-pnl
```bash
onchainos market portfolio-token-pnl \
  --address <wallet>*       \
  --chain <chain>*          \
  --token <tokenAddress>*
```
**Returns:** `realizedPnl`, `unrealizedPnl`, `avgCost`, `currentPrice`, `totalBought`, `totalSold`

---

## Balance Commands (okx-wallet-portfolio)

### portfolio-all-balances
```bash
onchainos portfolio all-balances \
  --address <wallet>*          \
  --chains <chain1,chain2>*
```
**Returns:** `tokens[]` with `symbol`, `balance`, `valueUsd`, `tokenContractAddress`

---

### portfolio-total-value
```bash
onchainos portfolio total-value \
  --address <wallet>*          \
  --chains <chain1,chain2>*
```
**Returns:** `totalValueUsd`, per-chain breakdown

---

## Signal / Tracker Commands (okx-dex-signal)

### tracker-activities (custom wallet tracking)
```bash
onchainos tracker activities \
  --tracker-type multi_address \
  --wallet-address <wallet>*   \
  --chain <chain>              \
  --trade-type <0|1|2>         # 0=all 1=buy 2=sell
```
**Returns:** transaction feed — `time`, `wallet`, `tokenSymbol`, `side`, `amountUsd`, `price`, `realizedPnl`

---

### leaderboard-list (for benchmarking)
```bash
onchainos leaderboard list \
  --chain <chain>*         \
  --time-frame <1|2|3|4|5> \  # 1=1D 2=3D 3=7D 4=30D 5=90D
  --sort-by <1|2|3|4|5>       # 1=PnL 2=winRate 3=txCount 4=volume 5=ROI
```
**Note:** No `--limit` flag — the CLI returns the default result set.
Use to benchmark analyzed wallet against top traders on the same chain.

---

## Chain Name Reference

| Chain | CLI name | chainIndex |
|---|---|---|
| Solana | `solana` | `501` |
| Ethereum | `ethereum` | `1` |
| X Layer | `xlayer` | `196` |
| Base | `base` | `8453` |
| BSC | `bsc` | `56` |
| Arbitrum | `arbitrum` | `42161` |
| Polygon | `polygon` | `137` |

---

## Time-frame Reference

| Value | Period |
|---|---|
| `1` | 1 day |
| `2` | 3 days |
| `3` | 7 days (default) |
| `4` | 30 days |
| `5` | 90 days |
