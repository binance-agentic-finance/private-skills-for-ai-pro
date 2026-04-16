# Risk Controls

真實交易 session 必須帶有硬風控，而且 OpenClaw 必須在使用者輸入 `CONFIRM` 前先把風控方式展示清楚。

## 1. 交易前必須先告知的風控方式

在使用者輸入 `CONFIRM` 前，OpenClaw 至少要清楚展示：

- 本次使用哪個帳戶 / 餘額快照
- 本次 session 的 `session_start_equity`
- 最大允許虧損比例：`x%`（預設 `10%`，可由使用者指定）
- 風控觸發線：`stop_equity = session_start_equity * (1 - x%)`
- 觸發後會執行什麼：
  - 停止新交易
  - 取消未成交訂單（若工具支援）
  - 嘗試平掉現有持倉
  - 強制結束交易 session
  - 將結果寫入 workspace session

## 2. 權益基準

風控應以 session 啟動時的帳戶總權益為基準，而不是只看可用餘額。

- `session_start_equity`
- `max_loss_ratio = x%`（預設 `10%`）
- `stop_equity = session_start_equity * (1 - max_loss_ratio)`

OpenClaw 在交易 session 執行期間，必須持續檢查：

- `current_equity`
- 現有持倉
- 未成交訂單

## 3. 風控觸發條件

一旦：

- `current_equity <= stop_equity`

就必須立刻觸發風控。

## 4. 風控觸發後的強制動作

風控觸發後，OpenClaw 必須依序執行：

1. 停止接受新的交易信號去開新倉
2. 取消未成交訂單（若工具支援）
3. 嘗試平掉現有持倉
4. 停止真實交易 session
5. 停止 watcher subagent 或改由它只回報停止狀態
6. 將結果寫入 workspace session
7. 回報使用者：
   - 已觸發風控
   - 目前權益
   - 損失比例
   - 是否已平倉
   - session 是否已終止

## 5. 風控觸發後不得自動重啟

若 session 因風控停止：

- 不得自動重啟
- 不得自動重新進場
- 必須等使用者重新確認後，才可開始新的交易 session

## 6. 風控資訊也必須寫入 workspace session

workspace session 至少應保存：

- `session_start_equity`
- `current_equity`
- `stop_equity`
- `loss_ratio`
- `risk_triggered`
- `risk_reason`
- `risk_triggered_at`
- `positions_closed`
- `session_stopped`
