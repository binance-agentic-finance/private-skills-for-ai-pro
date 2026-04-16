---
name: trading-monitoring-executing-router
description: |
  ALWAYS load this skill first for any strategy, backtest, monitoring, or trading request.

  This skill is the single routing layer for OpenClaw crypto trading: it enforces the mandatory pipeline—clone crypto_trading repo, standard_bot backtest via NumbaBacktestRunner, paper-generated signals, balance/risk display, explicit CONFIRM, and watcher subagent—so every execution path is auditable and repeatable.

  Use it whenever the user mentions strategy, backtest, signal, monitoring, or trading. If local historical parquet data is missing or stale, the repo's Binance API download flow must run first. Live trading must follow paper-generated signals only. Never use legacy backtesting, ad-hoc scripts, or alternative execution paths. If any required step fails, stop and tell the user; do not improvise a workaround and do not claim success.
metadata:
  author: binance crypto trading
  version: "1.0"
---

# OpenClaw Trading Skill

Route **user request -> formal strategy spec -> standard_bot -> backtest / paper signal -> balance & risk check -> CONFIRM -> watcher subagent -> live trading**. This skill holds **routing and hard rules** only—detailed operation steps are split into `references/`.

> **PREREQUISITE:** The `crypto_trading` repo (`https://github.com/nthu-chung/crypto_trading`) must be cloned and its environment bootstrapped per [`repo-bootstrap.md`](./references/repo-bootstrap.md) before any backtest, signal generation, or trading. Local historical parquet must be present and fresh; if not, use the repo's Binance K-bar download flow first. Higher-timeframe data should be resampled from local `1m` parquet—never use raw Binance API results as formal backtest input.

## Reference map (`references/`)

| Topic | Plan & details |
|-------|----------------|
| [Repo bootstrap](./references/repo-bootstrap.md) | Clone / pull repo, create venv, install deps, historical data refresh & parquet update rules |
| [Natural language to strategy](./references/natural-language-to-strategy.md) | Convert user intent to formal strategy spec; mandatory fields, conservative defaults, blank template |
| [Strategy routing](./references/strategy-routing.md) | Existing Numba strategies, `--engine python` fallback, new plugin creation rules |
| [Backtest workflow](./references/backtest-workflow.md) | `historical parquet -> resample -> standard_bot signal -> NumbaBacktestRunner`; PIT discipline, CLI template, output metrics |
| [Trading modes](./references/trading-modes.md) | Backtest-only / paper-signal-only / paper-signal-driven live trading; pre-CONFIRM display requirements |
| [Risk controls](./references/risk-controls.md) | Mandatory pre-trade display, `x%` max-loss rule, hard risk trigger flow, required session fields |
| [Watcher session](./references/watcher-session.md) | Watcher subagent polling, workspace session fields, status / stop / kill commands, heartbeat rules |

## Critical rules

1. Any strategy / backtest / signal / trading request must first clone or pull `crypto_trading`.
2. Backtesting and signal generation use **`standard_bot`** only—never legacy `cyqnt_trd/backtesting/*`, `strategy_backtest.py`, ad-hoc notebooks, or alternative scripts.
3. Backtest mainline: **`standard_bot -> mvp_backtest.py -> NumbaBacktestRunner`** (`--engine numba`).
4. Live trades follow **paper-generated signals only**; OpenClaw must not recompute a separate signal set.
5. Before any live trade, display: account & balance snapshot, latest paper signal, trade plan, risk controls, session duration.
6. **No `CONFIRM` = no real trade.** Backtesting, signal generation, and paper monitoring are allowed without confirmation.
7. After `CONFIRM`, automatically start a **watcher subagent** and keep writing status into the workspace session.
8. On failure: stop immediately, inform the user, continue debugging the correct mainline. Never switch to an alternative trading path or disguise a workaround as a completed flow.
9. Unless the user explicitly agrees, do not alter the mainline design.
10. This skill does not use testnet.

## Workflow

### 1. Bootstrap & data

- Follow [repo-bootstrap.md](./references/repo-bootstrap.md): clone repo, create venv, refresh local parquet.
- Raw Binance API results must not be used as formal backtest input.

### 2. Backtest

- Convert natural language to formal spec per [natural-language-to-strategy.md](./references/natural-language-to-strategy.md).
- Route to engine per [strategy-routing.md](./references/strategy-routing.md).
- Run backtest per [backtest-workflow.md](./references/backtest-workflow.md); read results from `docs/backtests/*.json`.

### 3. Live trading

- Follow [trading-modes.md](./references/trading-modes.md): paper signal -> propose trade -> display balance/signal/risk/duration -> wait for `CONFIRM`.
- Risk controls per [risk-controls.md](./references/risk-controls.md).

### 4. Monitoring & stop

- After `CONFIRM`, spawn watcher subagent per [watcher-session.md](./references/watcher-session.md).
- User commands: `status` reads latest watcher state; `stop` shuts down paper signal session, watcher, and live session by `run_id`.

## Notes

- This skill describes **execution routing and safety rules**, not investment advice.
- If a required flow step is blocked by a technical issue, the user must be informed before any alternative path is attempted.
