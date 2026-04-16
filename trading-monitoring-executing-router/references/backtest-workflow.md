# Backtest Workflow

## 1. 正式回測主線

OpenClaw 做回測時，必須使用這條資料路徑：

`historical parquet -> local resample -> standard_bot signal -> NumbaBacktestRunner`

## 2. 具體規則

- 歷史資料優先本地化，不要每次直接重打 API
- 若本地資料不足，先透過 repo 內的 Binance API 下載流程補抓資料，再回到本地 parquet 路徑
- 不可把 Binance API 即時回傳結果直接當成正式回測資料來源
- 儲存粒度優先為 `1m`
- `5m / 15m / 1h` 這類高週期優先由本地 `1m` 重採樣得到
- 回測引擎優先使用 `--engine numba`
- 不要先建立大量 Python snapshot 物件再做批次回測

## 3. 時序規則

- 只能使用 `decision time` 當下可見資料
- 若策略在 `bar close` 形成信號，最早只能在下一根 `bar open` 成交
- 不可使用尚未收盤的高週期 bar
- feature 不可使用未來資料
- 回測與 paper signal 應共用同一套策略邏輯

## 4. 回測命令模板

```bash
REPO_DIR="${REPO_DIR:-$PWD/crypto_trading}"
cd "$REPO_DIR"
source .venv-standard-bot/bin/activate

python -m cyqnt_trd.standard_bot.entrypoints.mvp_backtest \
  --engine numba \
  --market-type futures \
  --strategy multi_timeframe_ma_spread \
  --symbol BTCUSDT \
  --interval 5m \
  --secondary-interval 1h \
  --primary-ma-period 20 \
  --reference-ma-period 20 \
  --spread-threshold-bps 0 \
  --historical-dir data/mtf_90d \
  --start-ts 1768003200000 \
  --end-ts 1775779200000 \
  --download-missing \
  --output-json docs/backtests/btc_mtf_ma_cross_5m_1h_20_20_90d.json
```

## 5. 回測輸出規範

OpenClaw 在 demo、摘要、簡報或回覆時，應優先從 `docs/backtests/*.json` 讀正式結果，而不是只抄 terminal 數字。

至少應回報：

- 收益率 `total_return`
- 最大回撤 `metrics.max_drawdown`
- `Sharpe ratio`
- 交易次數 `metrics.trade_count`
- signal 次數 `metrics.signal_count`
- 最終資金 `metrics.final_equity`
- JSON 輸出檔路徑

若需要 `win_rate`，且 JSON 沒有現成欄位，才允許由 `trades` 推導，且必須明說是推導值。
