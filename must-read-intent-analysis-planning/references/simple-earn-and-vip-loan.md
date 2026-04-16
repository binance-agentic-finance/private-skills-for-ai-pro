# Simple Earn and VIP Loan

## Description

**Task summary**: Flexible/fixed subscribe & redeem, Simple Earn product browse/ops, VIP Loan Q&A and in-scope queries—distinct from plain **spot balance**; focuses on **yield and borrow** product paths.

**Typical user phrasing**: subscribe/redeem, APY, fixed terms, borrow rates, VIP loan limits and steps (may overlap account-asset intents—this task stresses **product type**).

**Hub**: **`simple-earn`**, **`vip-loan`**, **`assets`**, **`margin-trading`** (REST skills by **`name`**; no **`binance-cli`** overlap for this lane).

## Plan

### Step 1 — Account state (*MANDATORY*, always first)

Before **subscribe / borrow / redeem** planning, confirm **`assets`** shows enough **available** for the product line (Simple Earn, VIP Loan, or margin) and note earn locks / margin usage.

- **If available balance or collateral cannot support the action**: **Proactively tell the user** (shortfall, need transfer from Funding, wrong wallet) **before** preview/subscribe/borrow; do not assume the product flow will succeed. Use **[fuzzy-intent-and-account-onboarding.md](./fuzzy-intent-and-account-onboarding.md)** when they need to top up.

> Aligns with `task-upgrade-advice.md` §12: **pick product line** (Simple Earn / VIP Loan / margin) → list → positions → preview → subscribe/borrow → `getUserAsset` check.

### Status checks and when you cannot proceed

- **Before planning**: Product line clear; **available balance** for subscribe/stake/collateral; **existing positions/orders** (duplicate subscribe?, borrow limit?).
- **If preview fails or balance short**: (1) Product response and gap; (2) Transfer first?, other product/term?; (3) **No** `subscribe`/`borrow` until user confirms preview and amount.
- **Cross-task rules**: [task-upgrade-advice.md](./task-upgrade-advice.md).

### A. Structured pipeline (DAG)

| Step | Action |
|------|--------|
| **Product line** | Retail earn → `simple-earn`; VIP collateralized loan → `vip-loan`; spot margin borrow/repay → `margin-trading` (different product). |
| **Simple Earn** | `simple-earn/account` → `flexible/list` or `locked/list` → `subscriptionPreview` → confirm → `subscribe`; redeem: `position` → `redeem`. |
| **VIP Loan** | `loanable/data` + `collateral/data` + `interestRate` → confirm → `borrow`; ongoing `ongoing/orders`, repay `repay` / renew `renew`. |
| **Balance check** | Before/after `getUserAsset` (+ `get-funding-asset`) align with [account-and-asset-management.md](./account-and-asset-management.md). |

### B. Endpoint quick reference

**Base**: `https://api.binance.com` (signed USER_DATA unless noted).

1. **`simple-earn`**: `GET /sapi/v1/simple-earn/account`; flexible/locked list, position, preview, subscribe, redeem; BFUSD/RWUSD modules per **`simple-earn`** skill.
2. **`vip-loan`**: interest, loanable, collateral, `borrow`, `repay`, `renew`, `ongoing/orders` per **`vip-loan`** skill.
3. **`assets`**: `POST /sapi/v3/asset/getUserAsset`, `POST /sapi/v1/asset/get-funding-asset` around earn moves.
4. **`margin-trading`**: `POST /sapi/v1/margin/borrow-repay`—not VIP loan.
