# On-chain Signals and Security

## Description

**Task summary**: BSC-style transfer patterns, smart-money signals, token security audit, address holdings; and **data/signal parts** of monitoring/alert intents (long-running scripts still depend on user env and cron/serverless).

**Typical intents**: BSC transfer lookup; smart-money monitoring/alerts; scheduled monitoring/push; buy-signal notifications; token audit; smart-money + token analysis.

**Hub**: Web3 skills by **`name`** (e.g. `query-token-audit`, `trading-signal`, `query-address-info`, `query-token-info`, `meme-rush`); exchange order placement is not this task.

## Plan

### Step 1 — Account state (*MANDATORY*, always first)

Whenever the user might **trade on the exchange**, **size a position**, or tie signals to **“what I can do with my account”**, check **`assets`** (available balances) and relevant **`spot`** / **`derivatives-trading-usds-futures`** **before** audit/signal depth.

- **If the account cannot support trading they imply** (no funds, wrong wallet): **Proactively tell them** before signal shopping; point to [trading-execution.md](./trading-execution.md) / **[fuzzy-intent-and-account-onboarding.md](./fuzzy-intent-and-account-onboarding.md)**. Pure on-chain read with **no** CEX execution angle → Step 1 can be a **quick** `assets` sanity check or skip if clearly irrelevant—still **prompt** if they later ask to trade.

> Aligns with `task-upgrade-advice.md` §5: **contract → audit first**; then signals; `dynamic` liquidity on signal tokens; address paging; meme may align narrative.

### Status checks and when you cannot proceed

- **Before planning**: If **contract address**, **audit** first; if user will **trade on exchange**, also confirm account state ([trading-execution.md](./trading-execution.md)).
- **If audit high-risk or liquidity very low**: (1) State risk; (2) Ask if they still want signal/monitor view; (3) **Do not** imply safe to size up; ask for addresses to monitor if missing.
- **Cross-task rules**: [task-upgrade-advice.md](./task-upgrade-advice.md).

### A. Structured pipeline (DAG)

| Step | Action |
|------|--------|
| **Audit first** | If `contractAddress`: **`query-token-audit` first**; honeypot/high risk → **risk first**, then decide signals. |
| **Smart money** | `trading-signal` → sort/filter by `direction`, `maxGain`, `exitRate`, `alertPrice` vs `currentPrice`. |
| **Deepen** | For picks: `dynamic/info` liquidity; very low → soften “actionable” wording. |
| **Address** | `query-address-info` paging; continuous monitor = user **poll** same API (cron)—skill does not host daemons. |
| **Meme** | New launch talk: `meme-rush` stage vs signal list same narrative? |

### B. Endpoint quick reference

1. **`query-token-audit`**: `POST https://web3.binance.com/bapi/defi/v1/public/wallet-direct/security/token/audit` — `binanceChainId`, `contractAddress`, UUID `requestId`.
2. **`trading-signal`**: `POST .../smart-money/ai` — `chainId`, `page`, `pageSize`.
3. **`query-address-info`**: `GET .../address/pnl/active-position-list/ai` — `address`, `chainId`, `offset`.
4. **`query-token-info`**: search/dynamic per [market-data-and-analysis.md](./market-data-and-analysis.md) for signal token context.
5. **`meme-rush`**: rank endpoints per **`meme-rush`** skill.
6. **Monitoring**: all **pull** REST; scheduled push = user cron/serverless polling—no WebSocket hosting in skill.

### C. Default scheduling (Python / Shell)

“Scheduled monitor / conditional alert / timed push” defaults to **user-owned** scripts; agent/skill does **single** pulls, **no** hosted service.

| Method | Use | Notes |
|--------|-----|--------|
| **Shell + cron** | Fixed interval | `crontab` → `.sh` → `curl` + `jq`; webhook/log per user. |
| **Python** | Poll/threshold | `requests`/`httpx` loop or APScheduler; alerts (DingTalk/TG/mail) user-defined. |

Do **not** hardcode API keys; use env/secrets. Use the same endpoint URLs in your own script templates.
