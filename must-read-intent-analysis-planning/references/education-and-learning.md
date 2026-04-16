# Education and Learning

## Description

**Task summary**: Study plans, reading lists, term definitions, investing literacy, language learning—**explain and structure** first; optional market skills as **teaching examples**, not mandatory.

**Typical intents**: Equity/research study plans; trading glossary; language learning and expression analysis.

**Hub**: demos can use Web3 skills by **`name`** (e.g. **`query-token-info`**, **`crypto-market-rank`**); **`skill-creator`** is not in the hub (see this package [SKILL.md](../SKILL.md) for routing scope).

## Plan

### Step 1 — Account state (*MANDATORY*, always first)

If the learner asks **“with my money / how much can I buy / real account sizing”**, run **`assets`** (and execution checks per [trading-execution.md](./trading-execution.md)) **before** worked examples with numbers.

- **If their balance cannot support the example size**: **Proactively say so** and switch to **hypothetical** numbers or ask them to fund first—**do not** teach as if they had that cash. Use **[fuzzy-intent-and-account-onboarding.md](./fuzzy-intent-and-account-onboarding.md)** when they need funding basics.

> Aligns with `task-upgrade-advice.md` §11: **concepts first** → if user wants examples: **search → dynamic → klines**; ranking lesson one `unified/rank`; reusable tutorial → `skill-creator`.

### Status checks and when you cannot proceed

- **Before planning**: Pure concepts/terms/plans → usually no account; if “size for my money” or “how much to buy”, confirm balance and goals ([trading-execution.md](./trading-execution.md)).
- **If info thin**: Ask example vs real account; until clear, **only** formulas and conditional examples.
- **Cross-task rules**: [task-upgrade-advice.md](./task-upgrade-advice.md).

### A. Structured pipeline (DAG)

| Step | Action |
|------|--------|
| **Default** | Terms, study plan, structure → **no API**. |
| **Demo chain** | Pick mainstream: `search` → `dynamic` explain fields → **then** kline OHLCV. |
| **Ranking lesson** | One `unified/rank`—explain `rankType` / `period` / `sortBy`. |
| **Boundary** | State **teaching ≠ buy/sell advice**; `skill-creator` for reusable tutorials. |

### B. Endpoint quick reference

1. **Terms**: zero API.
2. **Optional single token**: `search/ai`, `dynamic/info` per **`query-token-info`** skill.
3. **Optional rank**: `POST .../unified/rank/list/ai`.
4. **Optional klines**: `GET https://dquery.sintral.io/u-kline/v1/k-line/candles` per **`query-token-info`** skill.
5. **Skill packaging**: Cursor `skill-creator`—not Binance HTTP.
