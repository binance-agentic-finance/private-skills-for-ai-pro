# Trading Modes

這份 skill 的交易規則與過去版本不同：

- 不使用 testnet
- OpenClaw 的真實交易，必須以 `paper trade` 產生的信號為來源
- 真實交易不是由 repo 直接下單，而是由 OpenClaw 的交易能力執行
- 但 OpenClaw 下單前，必須先經過 paper signal、餘額確認、`CONFIRM`、時長確認這四個步驟

## 1. 模式 1：只回測

- 使用 `mvp_backtest`
- 不送單
- 只產出回測結果與 JSON

## 2. 模式 2：只產生 paper signal

- 使用 `mvp_paper`
- 或用 monitor / background loop 產生持續 paper signal
- 不對外下單

## 3. 模式 3：paper signal 驅動的真實交易

這是 OpenClaw 目前應採用的真實交易模式：

1. 先用 `standard_bot` 產生 paper signal
2. 根據最新 signal 決定是否提議交易
3. 先查詢並展示使用者餘額
4. 清楚告知：
   - 當前 paper signal
   - 準備交易的標的 / 方向 / 金額
   - 目前餘額
   - 風險管控方式
   - 預計執行時長
5. 等使用者明確輸入 `CONFIRM`
6. 收到 `CONFIRM` 後，才由 OpenClaw 的交易能力開始執行一段有時長限制的真實交易 session

## 4. 使用者交易前必須看到的資訊

OpenClaw 在任何真實交易前，必須展示：

- 最新 paper signal：`buy / sell / hold`
- signal 產生時間
- 監聽標的
- 目前價格
- 使用者餘額
- 預計使用的交易金額或數量
- 風險管控方式
- 預計執行時長
- 如何停止

## 5. `CONFIRM` 規則

若未收到 `CONFIRM`：

- 可以回測
- 可以生成 signal
- 可以啟動 paper 監聽
- 不可下真實單

若收到 `CONFIRM`，仍必須先滿足：

- 使用者已看到當前餘額
- 使用者已知道交易標的、方向、金額/數量
- 使用者已看到本次 session 的風險管控方式
- 使用者已給定執行時長

## 6. 時長規則

真實交易 session 必須是有時長限制的。

OpenClaw 不可在未經說明下啟動無限期交易。若使用者沒有給時長，必須先追問，例如：

- `10 分鐘`
- `1 小時`
- `到我手動停止為止`

若使用者選擇「到我手動停止為止」，仍需明確告知 `停止` 指令。
