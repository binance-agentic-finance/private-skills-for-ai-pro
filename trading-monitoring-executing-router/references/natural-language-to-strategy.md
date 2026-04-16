# Natural Language To Strategy

自然語言不能直接拿去回測，必須先補成正式規格。

## 1. 示例

使用者輸入：

`5分鐘 MA 上穿 1小時 MA 做多，下穿做空`

轉換後規格：

- `plugin_id = multi_timeframe_ma_spread`
- `instrument_id = BTCUSDT`
- `market_type = futures`
- `primary_timeframe = 5m`
- `reference_timeframe = 1h`
- `primary_ma_period = 20`
- `reference_ma_period = 20`
- `spread_threshold_bps = 0`
- `decision_time = 5m bar close`
- `execution_model = next 5m bar open`
- `position_mode = long_short`

## 2. 一定要補的欄位

- 標的
- 市場
- 主決策週期
- 參考週期
- 指標與參數
- 做多 / 做空 / 平倉語意
- execution model
- fee / slippage
- 回測時間範圍

## 3. 保守預設

若使用者沒有講清楚，OpenClaw 應優先採保守預設：

- execution model：`next_open`
- fee：`taker fee`
- spread threshold：`0`
- 若語意明顯包含下穿做空，預設為 `long_short`

## 4. 規格模板

```text
策略名稱：
標的：
市場：
主決策週期：
參考週期：
指標與參數：
進場規則：
出場規則：
部位模式：
decision time：
execution model：
fee/slippage：
liquidity cap：
回測期間：
```
