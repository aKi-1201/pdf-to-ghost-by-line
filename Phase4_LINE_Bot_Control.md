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

## 4. 完善 Make.com 回報機制 (多重訊息連發)
* 回到 Make.com 的 Scenario。
* 在最後端加上一個 **LINE -> Send a Reply Message** 模組。
* 在 **`messages`** 陣列設定中，點擊 **`+ Add item`** 依序新增**兩個**訊息物件：
  * **第一則訊息（Type: text）**：傳送提示文字，例如：`✅ 老闆，最新文章已成功上架 Ghost！`
  * **第二則訊息（Type: text）**：**僅**填入 Ghost 模組產生的文章 **`URL`** 變數，不要加入任何其他文字。

> 💡 **為什麼要分兩則訊息？**
> LINE 的 OGP（Open Graph Protocol）大圖預覽功能，只有在訊息內容**完全只有網址**時才會被觸發。若網址與其他文字混在同一則訊息中，則只會顯示純文字，不會展開大圖預覽。將網址單獨放在第二則訊息，可確保文章封面圖在 LINE 聊天室中被完整展示。