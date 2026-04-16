# Token Research and Opportunities

## Description

**Task summary**: Alts and sectors (AI, DeFi, Meme), new launches and Alpha, risk/reward and entry talk—**research** intents; skills supply **data, signals, audit, rankings**; agent supplies framework and risk wording, **not** definitive investment advice.

**Typical intents**: Alt “investment ideas”; new tokens and trends; AI-theme tokens; DeFi inflows; Meme/new launches; specific tickers, entries, position sizing, etc.

**Hub**: Web3 skills by **`name`** (`query-token-info`, `crypto-market-rank`, `trading-signal`, `query-token-audit`, `meme-rush`, `binance-tokenized-securities-info`, …); CEX **`alpha`**; execution → [trading-execution.md](./trading-execution.md) (incl. **`binance`** / **`binance-cli`**).

## Plan

### Step 1 — Account state (*MANDATORY*, always first)

Before **entry / sizing / “how much should I buy”** research, confirm **`assets`** (available balance) and, if relevant, **`spot`** / **`derivatives-trading-usds-futures`** for positions and open orders.

- **If funds cannot support the position they describe**: **Proactively say so** and stop short of executable sizing; point to [trading-execution.md](./trading-execution.md) / **[fuzzy-intent-and-account-onboarding.md](./fuzzy-intent-and-account-onboarding.md)**. Pure narrative research with **no** account tie → still run Step 1 **when** they ask about real money.

> Aligns with `task-upgrade-advice.md` §6: **locate → audit → market position → behavior/narrative**; RWA separate; close with **facts + assumptions**; orders → [trading-execution.md](./trading-execution.md).

### Status checks and when you cannot proceed

- **Before planning**: Separate research from “place orders”; if user wants **executable plan** (size, leverage), confirm **balance, orders, risk** ([trading-execution.md](./trading-execution.md)).
- **If audit fails or data thin**: (1) Risk disclosure first; (2) Ask if they still want narrative/price layer; (3) **No** definitive buy plan without funding state.
- **Cross-task rules**: [task-upgrade-advice.md](./task-upgrade-advice.md).

### A. Structured pipeline (DAG)

| Step | Action |
|------|--------|
| **Locate** | `search/ai` or keywords → `chainId`+`contractAddress`; Alpha: `alpha` `token/list` + `ticker`/`klines`. |
| **Risk** | `query-token-audit`; **fail** → risk-led answer, less “opportunity”. |
| **Market position** | `unified/rank` + `inflow/rank` (`tagType=2`) → percentile, relative strength. |
| **Behavior/narrative** | `trading-signal`; Meme: `meme-rush` or exclusive rank; optional `social/hype`. |
| **RWA** | Only `binance-tokenized-securities-info` pipeline—**do not** mix with Meme copy. |
| **Close** | **Facts (API) + assumptions (risk, sizing bands)**; execution → [trading-execution.md](./trading-execution.md). |

### B. Endpoint quick reference

1. **Locate**: `query-token-info` `search/ai`; Alpha: `alpha-trade` exchange-info, ticker, klines, CEX alpha token list per **`alpha`** skill.
2. **Market/sector**: `crypto-market-rank` `unified/rank/list/ai`; inflow `inflow/rank/query/ai` (`tagType: 2`).
3. **Signals/risk**: `query-token-audit`; `trading-signal`; Meme ranks per **`meme-rush`** (or other Web3 skills by **`name`** as needed).
4. **RWA**: `binance-tokenized-securities-info` workflow (API 1→6 per **`binance-tokenized-securities-info`** skill).
5. **Orders**: [trading-execution.md](./trading-execution.md) only—no trade REST in this task.
