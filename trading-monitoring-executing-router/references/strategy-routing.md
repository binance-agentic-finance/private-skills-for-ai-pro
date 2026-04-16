# Strategy Routing

自然語言轉成策略規格後，不一定能直接走 `NumbaBacktestRunner`。OpenClaw 必須先判斷。

## 1. 既有 numba 策略

先嘗試映射到已存在的 numba 策略：

- `moving_average_cross`
- `price_moving_average`
- `rsi_reversion`
- `multi_timeframe_ma_spread`
- `donchian_breakout`

若可映射，直接使用：

`mvp_backtest --engine numba`

## 2. 若無法直接映射

1. 若已存在 `standard_bot` plugin，但尚未接進 numba：
   - 允許先使用 `mvp_backtest --engine python`
2. 若連 `standard_bot` plugin 都沒有：
   - 可讓其他 AI 在 `standard_bot` 內新增 plugin
   - 先做 `python engine` 驗證
   - 補 batch / step、PIT、smoke tests
   - 驗證通過後，再評估是否補 numba

## 3. 禁止事項

- 不可直接退回 legacy `cyqnt_trd/backtesting/*`
- 不可直接使用 `strategy_backtest.py`
- 不可先寫臨時 notebook 取代正式 plugin
- 新策略不得脫離 `standard_bot` 自成一套野生框架
