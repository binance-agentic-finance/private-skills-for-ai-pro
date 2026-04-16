# Trading Execution

## Description

**Task summary**: Spot / convert / algo / margin / futures / options: place, amend/cancel, query orders, take-profit/stop-loss, grids, automated trading; **data-side** support for strategy advice (market data, order state) and skill selection on the execution path.

**Typical intents**: Quant/short-term strategies; auto-trading and clearing orders; BNB futures entries and strategy; TP/SL; grid trading and order monitoring; DOGE/FDUSD grids; order status; long BTC execution, etc.

**Skill boundaries**: “Investment advice” must stay compliant; real orders must use the skill for the **actual product** (spot ≠ futures ≠ options). **Execution path**: same market via **REST** (per-product skill §B / quick reference for that **`name`**) or, when `binance-cli` is installed, **`binance`** (`spot` / `convert` / `futures-usds` CLI surface, cross-checked with **`spot`**, **`convert`**, **`derivatives-trading-usds-futures`** REST).

## Plan

### Step 1 — Account state (*MANDATORY*, always first)

Before **any** order, convert, borrow, or sizing talk, confirm **available balance**, **margin**, and **open orders / positions** support the action. Use **`assets.getUserAssets`**, **`derivatives-trading-usds-futures`** (positions / margin), and **`spot`** (open orders) as needed for the locked market.

- **If the account cannot support the trade** (insufficient available, wrong wallet, margin too low, conflicting open orders): **Proactively tell the user** and **do not** place or confirm orders until they fix funding, transfer, cancel, or resize—offer concrete options. Use **[fuzzy-intent-and-account-onboarding.md](./fuzzy-intent-and-account-onboarding.md)** when they need deposit / P2P / transfer guidance.

1. **Assets**: `assets.getUserAssets` (`SPOT`, `FUNDING`).
2. **Positions**: `derivatives-trading-usds-futures.getPositions`.
3. **Orders**: `spot.getOrders` for habits and open orders.

> **After Step 1**: If underfunded or blocked, **stop and prompt**; otherwise continue to market lock and reads.

> Aligns with `task-upgrade-advice.md` §3: **single market lock** → read-only orders/positions → precision/rules before writes → algo sub-order tracking; prices from [market-data-and-analysis.md](./market-data-and-analysis.md)—do not use order APIs as price feeds.

### Status checks and when you cannot proceed

- **Before writes**: **Single market** locked; **available balance** and **margin**; **open orders and positions** (incl. algo, grids); avoid in-flight conflicts.
- **If under-margined or missing params**: (1) State gap and ceiling; (2) Ask about deposit/transfer, cancel/reduce, smaller size or leverage; (3) **Do not** place orders or confirm convert for the user until confirmed.
- **Cross-task rules**: [task-upgrade-advice.md](./task-upgrade-advice.md).

### A. Structured pipeline (DAG)

| Step | Action |
|------|--------|
| **Market lock** | One skill: `spot` / `convert` / `algo` / `fapi` / `dapi` / `eapi` / `papi` / `margin` / `p2p` (quotes only); **never** mix `fapi` and `dapi` in one flow. CLI: `binance-cli spot` \| `convert` \| `futures-usds` (see §C). |
| **Read-only** | `openOrders` → `order` (by id) → futures `positionRisk`; CLI e.g. `get-open-orders`, `query-order`, `position-information-v-3`; cancel/amend after confirm. |
| **Writes** | Spot: `exchangeInfo`/`myFilters` then `order`; futures: `leverage`/`marginType` if needed; convert: `exchangeInfo` → `getQuote` → confirm → `acceptQuote`. CLI: §C; **prod requires user `CONFIRM`**. |
| **Algo** | After place: `algo/*/openOrders` + `subOrders`. |
| **Market data** | From [market-data-and-analysis.md](./market-data-and-analysis.md). |

### B. REST quick reference

**Bases**: Spot/Convert/Algo/Margin/SAPI → `https://api.binance.com`; USDS-M → `https://fapi.binance.com`; COIN-M → `https://dapi.binance.com`; Options → `https://eapi.binance.com`.

1. **Spot (`spot`)**: `POST /api/v3/order`; `DELETE /api/v3/order`; `DELETE /api/v3/openOrders`; `GET /api/v3/order`, `openOrders`, `allOrders`; conditional/OCO per **`spot`** skill.
2. **Convert (`convert`)**: `GET /sapi/v1/convert/exchangeInfo`; `POST .../getQuote`; `POST .../acceptQuote`; limit orders per **`convert`** skill.
3. **Algo TWAP/VP (`algo`)**: futures/spot TWAP and VP endpoints per **`algo`** skill.
4. **USDS-M**: `POST /fapi/v1/order`; `DELETE /fapi/v1/order`; `POST /fapi/v1/batchOrders`; `GET /fapi/v2/positionRisk`; `GET /fapi/v2/balance`; algo orders per **`derivatives-trading-usds-futures`** skill.
5. **COIN-M**: `POST /dapi/v1/order`, `GET /dapi/v1/openOrders`, `GET /dapi/v1/positionRisk`, etc.
6. **Options**: `POST /eapi/v1/order`, `GET /eapi/v1/openOrders`, `GET /eapi/v1/position`, `DELETE /eapi/v1/allOpenOrders`.
7. **PM**: `GET /papi/v1/account`, UM/CM order endpoints per **`derivatives-trading-portfolio-margin`** / **`derivatives-trading-portfolio-margin-pro`** skills.
8. **Margin spot**: `POST /sapi/v1/margin/order`; `GET /sapi/v1/margin/openOrders`.
9. **P2P**: public `quote-price`, `ad-list`, `trade-methods`; private order history—**do not sort** SAPI params.

### C. `binance-cli` (`binance` skill, vs §B)

| Subcommand | REST skill (`name`) | Notes |
|------------|----------------------|--------|
| `binance-cli spot <endpoint>` | `spot` | CLI ↔ REST alignment is defined inside **`binance`** + **`spot`** |
| `binance-cli convert <endpoint>` | `convert` | same |
| `binance-cli futures-usds <endpoint>` | `derivatives-trading-usds-futures` | same |

Install via `@binance/binance-cli`; **`CONFIRM`** for prod; `algo`, `p2p`, margin, COIN-M, options, PM → REST only.
