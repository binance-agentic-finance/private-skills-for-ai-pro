# API Authorization and Debugging

## Description

**Task summary**: For API key setup, delegated trading auth, signature failures, sub-account API and futures order issues—**connectivity and permission** problems—help determine whether the issue is account permissions, key configuration, or client-side calls; within automation limits, pair with balance/position checks.

**Typical intents**: Authorize AI to trade with profit targets; API key setup guidance; API config and call failures; sub-account BNB futures with API issues; account auth and signature debugging; trade command failures and signatures; query positions and troubleshoot API auth.

**Skill boundaries**: No single skill “generates or fixes” user-local code; `healthcheck` is environment/system hardening; `assets` / derivative skills **verify** whether auth works (read balances/positions/order error semantics). `skill-vetter` reviews third-party skill install risk—different from exchange API debugging.

## Plan

### Step 1 — Account state (*MANDATORY*, always first)

Even for “auth / signature” issues, **first** confirm whether the account **could** support the user’s intended action once keys work (**trade**, **earn**, **futures**, etc.): **`assets.getUserAssets`** (available balances); add **`spot`** / **`derivatives-trading-usds-futures`** when they care about orders or positions.

- **If balances clearly cannot support trading or the stated goal** while debugging keys: **say so up front** (e.g. “even with a fixed key, available balance is zero for this market”) and point to deposit / transfer / **[fuzzy-intent-and-account-onboarding.md](./fuzzy-intent-and-account-onboarding.md)**—**do not** imply fixing the signature alone will make the strategy executable.

1. **Account assets**: `assets.getUserAssets` across wallets (especially `SPOT`, `FUNDING`).
2. **Open positions**: `derivatives-trading-usds-futures.getPositions` for USDS-M.
3. **Recent activity**: `spot.getOrders` for habits and open orders.

> **After Step 1**: If underfunded for the user’s goal, **prompt before** deep error classification; else continue.

> Aligns with `task-upgrade-advice.md` §2: classify error → permission truth → per-market read path → sub-account/IP → signing environment → supply-chain tools.

### Status checks and when you cannot proceed

- **Before planning**: Error code / HTTP status; which account owns the key (main/sub); whether “enable trading” is on; whether IP allowlist matches the caller environment.
- **If read path still fails** (permissions vs business mismatch): (1) State verified layers and failure layer; (2) Ask whether API rights were checked in the app, whether a sub-account key is needed, whether local time is synced; (3) Separate “exchange-side limits” vs “local code/signature”—do not claim “fixed” when unclear.
- **Cross-task rules**: See [task-upgrade-advice.md](./task-upgrade-advice.md) opening.

### A. Structured pipeline (DAG)

| Step | Action |
|------|--------|
| **Classify** | Map errors to: `-1022` signature, `-2015` key, timestamp, IP, sub-account key mismatch, business codes—do not conflate. |
| **Permission truth** | Read-only: `apiRestrictions`, `apiTradingStatus`, `account/status`. |
| **Read path** | Spot `GET /api/v3/account` → USDS-M `fapi/v2/account` or `positionRisk` → COIN-M `dapi/v1/account`; **failure layer = problem layer**. |
| **Sub-account** | `sub-account/list` + `ipRestriction` (GET) to align email ↔ key ↔ IP. |
| **Signature / env** | Time sync, `recvWindow`, secret; **P2P SAPI params are not sorted**. |
| **Supply chain** | `skill-vetter` / `healthcheck` vs exchange verification—**separate conclusions**. |

### B. Endpoint quick reference

1. **Verify API key permissions and trading switch (`assets`)**
   - `GET /sapi/v1/account/apiTradingStatus`: whether trading is allowed.
   - `GET /sapi/v1/account/apiRestrictions`: key permission scope.
   - `GET /sapi/v1/account/status`: account status.

2. **Verify spot readability (`spot`)**
   - `GET /api/v3/account`: if balances return, spot USER_DATA is readable.

3. **Verify USDS-M (`derivatives-trading-usds-futures`)**
   - `GET /fapi/v2/account` or `GET /fapi/v3/account`; `GET /fapi/v2/positionRisk`; on order failure compare `GET /fapi/v1/apiTradingStatus`.

4. **Verify COIN-M (`derivatives-trading-coin-futures`)**
   - `GET /dapi/v1/account`, `GET /dapi/v1/balance`.

5. **Sub-account + IP (`sub-account`)**
   - `GET /sapi/v1/sub-account/list`; `GET /sapi/v2/sub-account/subAccountApi/ipRestriction` with `email` + `subAccountApiKey`; POST/DELETE IP list per **`sub-account`** skill docs.

6. **`healthcheck` / `skill-vetter`**: follow each skill’s own notes (not expanded as REST here); complements HTTP error troubleshooting.

### C. `binance-cli` auth and environment (`binance` skill)

If the user uses **`binance-cli`**, the **`binance`** skill documents auth and profiles (keys, profiles, testnet/demo flags—see that skill’s material, not repeated here):

- **Env**: `BINANCE_API_KEY`, `BINANCE_API_SECRET`; optional `BINANCE_API_ENV` (`prod` | `testnet` | `demo`).
- **Profile**: `binance-cli profile create`, `change`, `view`; `--profile <name>` on commands.
- **Security**: Do not log secrets; prefer read-only first (e.g. `binance-cli spot get-account`) before writes.
