# Token Deployment and On-chain Pay

## Description

**Task summary**: Create/launch/deploy tokens on BNB Chain etc. and social link integration. Binance skills focus on **trading, assets, market data, audit**—**no full contract IDE or one-click deploy**; use skills for **pay flows**, token info, and security checks.

**Typical intents**: BNB Chain token create/launch; delegated deploy and naming; integrate social links.

**Hub**: on-chain pay API → skill **`name`** **`onchain-pay`**; other steps use Web3 **`name`s** (`query-token-info`, `query-token-audit`, `query-address-info`, …).

## Plan

### Step 1 — Account state (*MANDATORY*, always first)

Before **on-chain pay** or any step that spends exchange balance, confirm **`assets`** (and funding/spot availability) **can cover** the intended fiat/crypto amount and fees.

- **If balance cannot support pay or subscribe flows**: **Proactively tell the user** (shortfall, wrong wallet, need transfer) **before** `estimated-quote` / `pre-order` / `order`; do not assume success. Use **[fuzzy-intent-and-account-onboarding.md](./fuzzy-intent-and-account-onboarding.md)** for top-up / transfer guidance.

> Aligns with `task-upgrade-advice.md` §8: no one-click deploy REST; pay **pairs → payment → quote → address/network → pre-order → poll**; post-deploy **search + audit + address holdings**.

### Status checks and when you cannot proceed

- **Before planning**: Pay path **fiat/pair, amount, recipient address and network**; before `estimated-quote`, enough funds (optional `assets`).
- **If on-chain pay fails or limit hit**: (1) Error and limits; (2) Change method/network or amount?; (3) User must confirm address/network before final `pre-order`/`order`.
- **Cross-task rules**: [task-upgrade-advice.md](./task-upgrade-advice.md).

### A. Structured pipeline (DAG)

| Step | Action |
|------|--------|
| **Need** | Fiat buy to address vs send existing crypto → `pre-order` `customization` (e.g. `SEND_PRIMARY`). |
| **Capability** | `trading-pairs` → `payment-method-list` → `estimated-quote` (consistent fiat/crypto/amount). |
| **Pre-order** | **`address` + `network`** required; `crypto-network` for fees/limits. |
| **Order** | `pre-order` → `order` poll to terminal state. |
| **Post-deploy** | If contract: `search` + `audit` + `query-address-info` (deployer)—**not** Remix/Hardhat replacement. |

### B. Endpoint quick reference

1. **Onchain Pay** (`onchain-pay-open-api`, RSA signing—handled in **`onchain-pay`** skill material): `trading-pairs`, `payment-method-list`, `estimated-quote`, `pre-order`, `order`, `crypto-network` per **`onchain-pay`** (`papi/v1` or `v2` paths).
2. **Post-deploy token**: `query-token-info` `search/ai` by contract address.
3. **Audit**: `POST .../security/token/audit`.
4. **Deployer holdings**: `GET .../address/pnl/active-position-list/ai`.
5. **Contract author/deploy**: no dedicated Binance REST; Remix/Hardhat off-platform.
