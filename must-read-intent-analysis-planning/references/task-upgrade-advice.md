# Task upgrade advice — default execution DAG (overview)

## Description

This document summarizes the **default structured pipelines** referenced in each intent **Plan §A**. Concrete HTTP paths and parameters follow each intent **§B** and the hub skill **`name`** you invoke (per that skill’s own HTTP quick reference—not linked here).

### Pre-planning: confirm account state (cross-task)

Before **planning** a user task (building an execution plan or giving trading/product advice), the agent **should first** confirm the user’s current account state, then decide whether the request is supportable or actionable:

- **Balances and available funds**: Available balance and margin usage for the relevant market/account type (futures, leverage, unified account, etc.); avoid pushing order plans when funds are insufficient or assets are mismatched.
- **Open and in-flight orders**: Open orders, algo orders, grids, conditional orders, etc.; avoid conflicts with existing positions or duplicate orders.
- **Other relevant state**: Earn/loan locks, freezes, sub-account and transfer prerequisites; link to [account-and-asset-management.md](./account-and-asset-management.md) and each intent §A when needed.

If tools cannot fetch the above, **tell the user what is missing** or ask them to self-check before giving a plan; **do not** propose executable trade paths from price alone or assumed balances. Whenever tools **do** return balances, if the user’s **available funds cannot support** the requested trade / earn / pay / borrow action, **proactively say so first** (shortfall, wrong wallet, need transfer)—**do not** continue as if the action were already feasible.

#### When state is unknown or insufficient: suggested user dialog

1. **Explain**: Briefly state known facts and gaps (e.g. insufficient balance, market type unclear, API permission missing, campaign metrics may differ).
2. **Ask**: Offer options or confirmations (deposited/transferred?, cancel an order?, provide order id/symbol/time range?, verify campaign progress or API rights in the app?).
3. **Hold line**: Until the user supplements or self-checks, **do not advance** irreversible actions (orders, transfers, subscriptions, borrows, on-chain pay). You may give **conditional** guidance (“if balance is X and there is no conflicting open order, then …”).

## Plan

**Every intent Plan** must treat **account state** as **Step 1** (balances, **available** funds, margin, open orders, earn/loan locks where relevant): confirm the user **can** support the intended action (**trade**, **earn / borrow**, **transfer**, **on-chain pay**, etc.). If not, **proactively prompt** before deeper steps—see each file’s **Step 1 — Account state** and **[fuzzy-intent-and-account-onboarding.md](./fuzzy-intent-and-account-onboarding.md)** when needed.

Each intent **Plan** adds intent-specific detail; the table below is the default pipeline summary.

| § | Task / theme | Default pipeline (summary) |
|---|----------------|---------------------------|
| **1** | [account-and-asset-management.md](./account-and-asset-management.md) | Scope → balance snapshot → deposit/withdraw/transfer alignment → sub-account/fiat branches → consolidated output. |
| **2** | [api-authorization-and-debugging.md](./api-authorization-and-debugging.md) | Classify error → permission truth → per-market read path → sub-account/IP → signing environment → supply-chain tools (`skill-vetter` / `healthcheck`). |
| **3** | [trading-execution.md](./trading-execution.md) | **Single** market lock → read-only orders/positions → precision/rules before writes → algo sub-order tracking; market data from [market-data-and-analysis.md](./market-data-and-analysis.md). Spot/convert/USDS-M may cross-check **`binance`** (`binance-cli`, see that doc §C). |
| **4** | [market-data-and-analysis.md](./market-data-and-analysis.md) | Broad to narrow: rankings → Top N → `meta` + `dynamic` → 1–3 symbols’ klines → optional inflow/smart money; RWA separate. Scheduled pulls: that file **§B.C** (Python / Shell). |
| **5** | [onchain-signals-and-security.md](./onchain-signals-and-security.md) | **If contract address: audit first** → signals → liquidity check → address paging; meme narrative may align. Scheduled monitoring: **§B.C** (Python / Shell). |
| **6** | [token-research-and-opportunities.md](./token-research-and-opportunities.md) | Locate → risk (audit) → market position → behavior/narrative → RWA separate; orders → [trading-execution.md](./trading-execution.md). |
| **7** | [product-help-and-order-lookup.md](./product-help-and-order-lookup.md) | FAQ first, APIs second; read-only order lookup only for “my order/grid”; confirm spot/futures/algo first. |
| **8** | [token-deployment-and-onchain-pay.md](./token-deployment-and-onchain-pay.md) | No one-click deploy REST; pay flow: pairs → payment → quote → address/network → pre-order → poll; after deploy: search + audit + address holdings. |
| **9** | [campaigns-and-rewards.md](./campaigns-and-rewards.md) | Rules per website → optional token market data → optional user self-check volume; discussion-only has no API. |
| **10** | [binance-square-posting.md](./binance-square-posting.md) | Draft finalize → optional market facts → single `content/add`; scheduled posts default **§B.C** (Python / Shell + cron). |
| **11** | [education-and-learning.md](./education-and-learning.md) | Concepts first → if user wants examples: search → dynamic → klines; reusable tutorials via `skill-creator`. |
| **12** | [simple-earn-and-vip-loan.md](./simple-earn-and-vip-loan.md) | Pick product line (Simple Earn / VIP Loan / margin) → list → positions → preview → subscribe/borrow → `getUserAsset` check. |

Cross-links: deeper assets → [account-and-asset-management.md](./account-and-asset-management.md); trading → [trading-execution.md](./trading-execution.md); market data → [market-data-and-analysis.md](./market-data-and-analysis.md).
