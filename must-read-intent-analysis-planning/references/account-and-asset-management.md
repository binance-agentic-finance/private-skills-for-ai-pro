# Account and Asset Management

## Description

**Task summary**: Help users query main- and sub-account assets, reconcile balances and distribution, track deposit/withdrawal/transfer status, and support sub-account–related questions and operational guidance.

**Typical intents** (from intent taxonomy): Binance account assets and trading status; main-account balance confirmation; delegated-operation authorization and fund checks; balance and asset distribution; fund flow tracing; account opening and trading questions; account feature closure and refund inquiries (informational).

**Skill boundaries**: Assets and transfers → **`assets`**; sub-accounts → **`sub-account`**; fiat rails → **`fiat`** as secondary. If the user also asks about contracts or unified margin, bridge to derivative skills in **[trading-execution.md](./trading-execution.md)**.

## Plan

### Step 1 — Account state (*MANDATORY*, always first)

Before any later step, confirm whether **funds, wallets, and permissions** support what the user wants (e.g. **trading**, **earn**, **transfers**, **fiat**, **sub-account moves**). Start with **`assets.getUserAssets`** and **available** vs locked balances per wallet; add **`spot`** / **`derivatives-trading-usds-futures`** when the issue ties to orders or positions.

- **If state does not support the next action** (empty or wrong wallet, insufficient **available**, margin/earn locks): **Proactively tell the user** what is missing and what to do (deposit, internal transfer, cancel conflicting orders, check app). **Do not** continue as if balances were sufficient. For cold-start / funding paths, use **[fuzzy-intent-and-account-onboarding.md](./fuzzy-intent-and-account-onboarding.md)**.

1. **Account assets**: Call `assets.getUserAssets`; review available and total value per wallet (especially spot `SPOT` and funding `FUNDING`).
2. **Open positions**: Call `derivatives-trading-usds-futures.getPositions`; check USDS-M positions (side, size, unrealized PnL).
3. **Recent trades and open orders**: Call `spot.getOrders`; note trading habits (preferred pairs) and any open orders.

> **After Step 1**: If funds are insufficient for the user’s stated goal, **stop and prompt** before deep ledger work; otherwise continue. If underfunded for general onboarding, see **[fuzzy-intent-and-account-onboarding.md](./fuzzy-intent-and-account-onboarding.md)**.

> Aligns with `task-upgrade-advice.md` §1: scope → snapshot → ledger alignment → sub-account/fiat branches → consolidated output.

### Status checks and when you cannot proceed

- **Before planning**: Confirm scope (main vs funding vs sub-account vs fiat) and whether the **time window** for the issue is clear.
- **If ledgers disagree with snapshot or APIs lack permission**: (1) State what was checked and what is missing; (2) Ask whether other wallets/sub-accounts exist, or whether the user can provide a txId or in-app order time; (3) Do **not** invent “where funds went”; for policy/account-closure topics, point to the website and support.
- **Cross-task rules**: See the opening of [task-upgrade-advice.md](./task-upgrade-advice.md) for global flow and “ask the user” guidance.

### A. Structured pipeline (DAG)

| Step | Action |
|------|--------|
| **Scope** | Separate: main spot only / includes Funding / includes sub-account / includes fiat orders / includes contracts or PM (latter → [trading-execution.md](./trading-execution.md)). |
| **Balance snapshot** | `getUserAsset` or `wallet/balance` → answering “how much X left” may stop here; “where did money go” → ledgers. |
| **Ledger alignment** | In the time window: `deposit/hisrec`, `withdraw/history`, `asset/transfer` (GET) **cross-check** with snapshot (large moves should match records). |
| **Sub-account** | Only when sub-account email or master–sub transfer is explicit: `sub-account/list` → `assets` → `universalTransfer` history if needed. |
| **Fiat** | Channels/limits: public `get-capabilities` / `get-price`; **my orders**: SAPI `fiat/orders`, `fiat/payments` aligned with asset timeline. |
| **Close out** | Per wallet type: **balance + 3–5 key recent ledger lines**; refunds/closures—state facts only and cite the website. |

### B. Endpoint quick reference

HTTP paths and parameters follow the **`assets`**, **`sub-account`**, and **`fiat`** skills’ own §B / quick references. Base: `https://api.binance.com` (REST needs `X-MBX-APIKEY` plus signed `timestamp`/`signature` unless marked public).

**Spot snapshot supplement**: If **`binance-cli`** is enabled, the **`binance`** skill can run `binance-cli spot get-account` to **cross-check** REST **`assets`** / **`spot`** account views; deposits/withdrawals/transfers remain on **`assets`**.

1. **Main-account spot / funding balances**
   - `POST /sapi/v3/asset/getUserAsset`: per-asset balances (optional `asset` filter).
   - `GET /sapi/v1/asset/wallet/balance`: wallet summary by `quoteAsset`.
   - `GET /api/v3/account` (`spot` skill): spot balances and order locks (if tied to spot orders).

2. **Deposit / withdrawal / transfer tracing**
   - `GET /sapi/v1/capital/deposit/hisrec`: deposit history (`coin`, `status`, `startTime`/`endTime`, etc.).
   - `GET /sapi/v1/capital/withdraw/history`: withdrawal history.
   - `GET /sapi/v1/asset/transfer` (GET): universal transfer history; `POST /sapi/v1/asset/transfer`: transfer (`type`, `asset`, `amount`, `fromSymbol`/`toSymbol` per docs).

3. **Funding wallet / snapshots**
   - `POST /sapi/v1/asset/get-funding-asset`: funding wallet assets.
   - `GET /sapi/v1/accountSnapshot`: daily snapshots (`type`: SPOT/MARGIN, etc.).

4. **Sub-accounts (master view)**
   - `GET /sapi/v1/sub-account/list`: list (`email`, `page`, `limit`).
   - `GET /sapi/v4/sub-account/assets` or `GET /sapi/v3/sub-account/assets`: assets for a given sub-account `email`.
   - `GET /sapi/v1/sub-account/universalTransfer` (GET): master–sub history; `POST .../universalTransfer`: universal transfer (`fromAccountType`/`toAccountType`: `SPOT`, `USDT_FUTURE`, `COIN_FUTURE`, `MARGIN`, `ISOLATED_MARGIN`, etc.).

5. **Fiat orders and payments (API key required)**
   - `GET /sapi/v1/fiat/orders`: fiat order history (`transactionType`, etc.).
   - `GET /sapi/v1/fiat/payments`: fiat payment history.
   - **Public (no key)**: `https://www.binance.com/bapi/fiat/v1/public/fiatpayment/agent/get-capabilities?country={CC}` and same-prefix `get-buy-and-sell-payment-methods`, `get-deposit-and-withdraw-payment-methods`, `get-price` (capabilities/channels/reference price only—not account ledgers).
