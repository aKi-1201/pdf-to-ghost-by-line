# 階段四：LINE Bot 遠端控制介面 (Remote Control)

## 1. 申請 LINE Messaging API
* 前往 [LINE Developers](https://developers.line.biz/) 控制台。
* 建立一個新的 Provider，並創建一個 **Messaging API Channel**。
* 在 Channel 設定中，取得 **Channel Access Token (long-lived)**。

## 2. 綁定 Webhook
* 在 Messaging API 設定頁面，找到 **Webhook settings**。
* 將【階段三】Make.com 產生的 `Custom Webhook URL` 填入。
* 開啟 **Use webhook** 選項並點擊 Verify 測試連線。
* (建議關閉 Auto-reply messages 以免干擾)。

## 3. 建立圖文選單 (Rich Menu)
* 進入 LINE Official Account Manager 後台。
* 導覽至 **圖文選單** 設定。
* 設計一個包含「發布最新文章」按鈕的選單介面。
* 將該按鈕的動作設定為發送特定文字 (例如 `CMD_SYNC_POST`)。

## 4. 完善 Make.com 回報機制
* 回到 Make.com 的 Scenario。
* 在最後端加上一個 **LINE -> Send a Push Message / Reply Message** 模組。
* 設定接收到執行成功後，回傳：「✅ 老闆，最新文章已成功上架 Ghost！」並附上文章連結。