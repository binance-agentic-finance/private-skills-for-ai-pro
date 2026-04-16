---
name: must-read-intent-analysis-planning
description: |
  ALWAYS load this skill first for any Binance AI Pro / Clawbot user request that might touch accounts, API keys, trading, balances, transfers, earn, VIP loan, market or on-chain data, token research, product help, Square posts, campaigns, education, or vague onboarding.

  This planning skill is the single routing layer: it turns messy natural language into a clear intent (via intent_category.json), aligns every case with shared pre-planning rules (task-upgrade-advice.md), and hands you a ready-made execution story—§A DAG for narrowing and ordering, §B endpoint quick reference, §C binance-cli where applicable—so you invoke the right hub skill by name (CEX or Web3) with confidence and consistency.

  Use it whenever the user is not purely chit-chat: it saves iteration, keeps spot, futures, earn, and Web3 lanes separate, and makes your plan auditable and repeatable for Binance AI Pro / Clawbot workflows.
metadata:
  author: binance
  version: "1.0"
---

# Must-read: user intent analysis & planning

Route **user intent → skills → ordered plan**. In the hub, pick executable skills by **`name`** (CEX family: e.g. `assets`, `spot`, `convert`, `binance` for `binance-cli`; Web3 family: e.g. `query-token-info`, `crypto-market-rank`, `query-token-audit`). This skill holds **routing and plans** only—no per-skill file paths here.

> **PREREQUISITE:** Read [`task-upgrade-advice.md`](./references/task-upgrade-advice.md) for cross-task pre-planning (balances, open orders, earn/loan locks) and the default pipeline index. In **every** intent `references/*.md` **Plan**, **Step 1** is **account state**: confirm **available funds** support the user’s action (**trade**, **earn**, etc.); if not, **prompt the user proactively** before deeper steps.

## Intent plans (`references/`)

| Intent (summary) | Plan & endpoints |
|------------------|------------------|
| [Cross-task DAG overview](./references/task-upgrade-advice.md) | Pre-planning + §1–§12 pipeline table |
| [Account & assets](./references/account-and-asset-management.md) | Balances, ledgers, sub-account, fiat |
| [API auth & debugging](./references/api-authorization-and-debugging.md) | Keys, signatures, read paths, sub-account IP |
| [Trading execution](./references/trading-execution.md) | Spot / convert / algo / futures / options / margin |
| [Market data & analysis](./references/market-data-and-analysis.md) | Web3 ranks, token info, klines |
| [On-chain signals & security](./references/onchain-signals-and-security.md) | Audit, smart money, address, scheduling notes |
| [Token research & opportunities](./references/token-research-and-opportunities.md) | Research stack; not investment advice |
| [Product help & order lookup](./references/product-help-and-order-lookup.md) | FAQ-first; read-only order lookup |
| [Token deployment & on-chain pay](./references/token-deployment-and-onchain-pay.md) | Onchain Pay + post-deploy checks |
| [Campaigns & rewards](./references/campaigns-and-rewards.md) | Rules from site; optional market context |
| [Binance Square posting](./references/binance-square-posting.md) | `content/add`, scheduling pattern |
| [Education & learning](./references/education-and-learning.md) | Concepts first; optional API demos |
| [Simple Earn & VIP Loan](./references/simple-earn-and-vip-loan.md) | Earn / VIP loan / margin distinction |
| [Fuzzy intent & onboarding](./references/fuzzy-intent-and-account-onboarding.md) | Cold start vs funded routing |

## Taxonomy

- **[intent_category.json](./intent_category.json)** — categories and example queries (Chinese labels).

## Related hub entrypoints (by `name` only)

| Area | Typical `name` values |
|------|------------------------|
| Spot / Convert / USDS-M via CLI | `binance` (`binance-cli` maps to `spot`, `convert`, `derivatives-trading-usds-futures` semantics) |
| Other CEX REST / SAPI | e.g. `assets`, `spot`, `convert`, `algo`, `simple-earn`, `vip-loan`, `sub-account`, `fiat`, `margin-trading`, `p2p`, and `derivatives-trading-*` / `derivatives-trading-options` as needed |
| Web3 | e.g. `query-token-info`, `crypto-market-rank`, `trading-signal`, `query-token-audit`, `query-address-info`, `meme-rush`, `binance-tokenized-securities-info` |

## Notes

- Tasks describe **routing and capability**, not investment advice; campaigns and product terms follow **official Binance** pages.
- If the user message matches a **contract address**, **audit first** (`query-token-audit`)—see [on-chain signals](./references/onchain-signals-and-security.md) §A.
