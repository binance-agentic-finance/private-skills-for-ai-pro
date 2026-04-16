# Binance Square Posting

## Description

**Task summary**: Draft posts, multilingual posting, scheduling, news scrape-then-publish to Binance Square; **`square-post`** is the publish core; images, fetch, translation are general agent capabilities.

**Typical intents**: Write and publish Square articles; multilingual scheduled posts; news optimized for Square.

**Hub**: publish path uses skill **`name`** **`square-post`**.

## Plan

### Step 1 — Account state (*MANDATORY*, always first)

If the post references **portfolio, PnL, “my holdings”, or trade sizing**, check **`assets`** (and relevant positions) **before** drafting claims—**do not** invent balances.

- **If they imply actions their account cannot support** (no funds, no position): **Proactively correct** or ask them to confirm numbers; avoid misleading followers. For funding gaps, point to **[fuzzy-intent-and-account-onboarding.md](./fuzzy-intent-and-account-onboarding.md)** when relevant.

> Aligns with `task-upgrade-advice.md` §10: **draft finalize → self-check length/sensitive words → optional market facts → single `content/add`**; scheduled = external scheduler repeating flow.

### Status checks and when you cannot proceed

- **Before planning**: Account/Square permissions per **`square-post`** / API requirements; if citing prices, pull citeable facts per [market-data-and-analysis.md](./market-data-and-analysis.md).
- **If `content/add` fails** (sensitive word/length): (1) Error type; (2) User accepts edit/shorter text?; (3) **Do not** republish without confirmed draft.
- **Cross-task rules**: [task-upgrade-advice.md](./task-upgrade-advice.md).

### A. Structured pipeline (DAG)

| Step | Action |
|------|--------|
| **Draft** | Body, disclaimers (if market) in agent; check `square-post` error table. |
| **Data** | Cite prices/ranks: [market-data-and-analysis.md](./market-data-and-analysis.md) first, embed in `bodyTextOnly`. |
| **Publish** | One `content/add`; success → URL from `data.id`. |
| **Schedule** | No official schedule API → cron/external fires `content/add` at time. |

### B. Endpoint quick reference

1. **Post**: `POST https://www.binance.com/bapi/composite/v1/public/pgc/openApi/content/add` — headers `X-Square-OpenAPI-Key`, `Content-Type: application/json`, `clienttype: binanceSkill`; body `bodyTextOnly` (plain text). Success `code`=`000000`; link `https://www.binance.com/square/post/{id}`.
2. **Errors**: `10005` KYC; `20002`/`20022` sensitive words; `20013` length—see **`square-post`** skill error table.
3. **Schedule / news scrape**: no Binance REST in this skill—external or manual.
4. **Market cites**: [market-data-and-analysis.md](./market-data-and-analysis.md) then `content/add`.

### C. Scheduled posting (Python / Shell)

Default: user **cron** or **Python** at time T builds body and calls `content/add` (same as §A). Store `X_SQUARE_OPENAPI_KEY` in env; use your own shell or Python script templates with the same endpoint and headers.
