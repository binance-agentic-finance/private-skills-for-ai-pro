# Watcher Session

Jarvis 介面不能依賴主動跳通知，因此背景監聽規範不能建立在推播一定會成功上。

正確做法是：一旦啟動背景監聽，必須同時啟動一個 watcher subagent，持續把每輪輪詢結果寫進管理 bot 的 workspace session。之後使用者只要輸入 `現在狀態`，主 agent 就必須從 session 中讀取最新狀態並回報；使用者輸入 `停止`，主 agent 就必須使用 session 中保存的 `run_id` 停掉 paper signal session 與後續交易 session。

## 1. 核心原則

- 不可等到任務結束才回報
- 中間狀態必須持續寫入 workspace session
- session 中必須始終保有可供查詢與停止的最新 `run_id`

## 2. watcher subagent 的職責

若平台支援 subagent，則背景監聽啟動成功後，主 agent 必須立即啟動一個專用 watcher subagent。

watcher subagent 的唯一職責是：

- 定期讀取 paper signal / monitor 狀態
- 追蹤 log 更新
- 維護遞增的 `poll_index`
- 將每輪結果寫入 workspace session

watcher subagent 不負責：

- 修改策略
- 改變交易金額
- 直接跳過 `CONFIRM`
- 直接自行下單

## 3. session 內必須保存的欄位

至少應保存：

- `run_id`
- `poll_index`
- 輪詢時間
- 監聽標的
- 當前價格
- 最新 signal：`buy / sell / hold`
- 目前持倉
- 損益
- 是否已進入真實交易 session
- 使用者餘額快照
- 風控基準與是否已觸發風控
- 預計結束時間或剩餘時長
- `status_command`
- `stop_command`
- `kill_command`

## 4. 使用者輸入 `現在狀態`

當使用者輸入：

- `現在狀態`
- `status`
- `目前狀況`

主 agent 必須直接讀取 workspace session 中 watcher subagent 寫入的最新狀態，並回報：

- 目前 `run_id`
- 最新 `poll_index`
- 最新輪詢時間
- 當前價格
- 最新 signal
- 目前持倉與損益
- 風控是否已觸發
- 是否已開始真實交易
- 若有時長限制，還剩多久

## 5. 使用者輸入 `停止`

當使用者輸入：

- `停止`
- `stop`
- `停止監聽`

主 agent 必須從 workspace session 取得目前活躍的 `run_id`，並依序：

1. 停掉 paper signal session
2. 停掉 watcher subagent
3. 若已有真實交易 session，同步停止該 session
4. 將停止結果回寫到 workspace session
5. 回報最終狀態

## 6. 更新頻率

watcher subagent 應遵守：

- 第一則狀態更新：背景流程啟動後幾秒內
- 固定心跳：預設每 `30` 秒至少更新一次
- 事件更新：以下事件發生時應立即更新
  - 新 signal
  - 交易提議形成
  - 餘額查詢完成
  - 收到 `CONFIRM`
  - 真實交易開始
  - 真實交易停止
  - watcher 異常

## 7. watcher 不可靜默卡住

若 watcher subagent 發生以下情況：

- 已啟動但沒有寫入第一則狀態
- 固定心跳中斷
- 背景流程仍在跑，但 session 久久沒有更新

主 agent 必須：

1. 立即通知使用者 watcher 異常
2. 自己接管一次最新狀態讀取與回報
3. 視情況停止舊 watcher
4. 重啟新的 watcher

若重啟後仍不穩定，主 agent 應退回「自己輪詢、自己更新 session」模式。
