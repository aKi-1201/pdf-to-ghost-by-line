# 階段三：Make.com 工作流設計 (Data Workflow)

## 1. 準備 Google Drive
* 建立一個專屬的「待發布文章」資料夾。
* 確保文章標題為預期的網頁標題。

## 2. 建立 Make.com Scenario (劇本)
在 Make.com 建立一個新的 Scenario，包含以下模組流程：

### 模組 A：Webhook (觸發器)
* 選擇 `Webhooks -> Custom Webhook`。
* 產生一個專屬的 Webhook URL (這個網址將在階段四交給 LINE Bot 使用)。

### 模組 B：Google Drive (檔案搜尋)
* 選擇 `Google Drive -> Search for Files/Folders`。
* **Query**: 設定搜尋條件，抓取「待發布資料夾」中 `modifiedTime` 最新的一份文件。

### 模組 C：Google Drive (下載轉換)
* 選擇 `Google Drive -> Download a File`。
* **File ID**: 帶入模組 B 取得的 ID。
* **Convert to**: 選擇 `HTML` 格式。

### 模組 D：Ghost (查重與更新機制) 🔄
為了避免同一份 PDF 被重複上架，需要先查詢 Ghost 是否已存在相同文章：

1. **Ghost (Search Posts) 模組**
   * 加入 `Ghost -> Search Posts` 模組。
   * **Field**: 設定為 `title` (標題)。
   * **Operator (比對條件)**: 設定為 **`contains` (包含)**，而非完全相符，以解決中文標題與空格造成的誤判問題。
   * **Value**: 帶入從檔名處理後的標題變數。

2. **檔名變數過濾 (大小寫 .pdf 處理)**
   * 在標題變數中，需要同時替換大寫 `.PDF` 與小寫 `.pdf`，以確保任何大小寫組合皆能正確去除副檔名。
   * 替換時，目標字串（置換後的值）請直接**留空**，不要輸入任何引號字元，避免字串陷阱。

3. **Router (分流器)**
   * 在 Search Posts 模組後，加入 **`Router`** (分流器) 模組。
   * **Route 1（建立新文章）**：Filter 條件設為 `Total number of bundles` **`= 0`**（表示搜尋結果為空，文章尚未存在）。
   * **Route 2（更新舊文章）**：Filter 條件設為 `Total number of bundles` **`> 0`**（表示已有相符文章）。在此 Route 的 Ghost 模組中，將 **Post ID** 對應至 Search Posts 回傳的文章 ID，以實現覆蓋更新。

### 模組 E：Ghost (封面大圖自動化) 🖼️
* 在 Ghost 的 Create / Update Post 模組中，找到 **`Feature Image`** 欄位。
* 將 AI 生成的圖片 URL 變數填入此欄位。
* 設定後，文章在 Ghost 首頁列表、LINE 連結展開時，都會自動顯示此大圖。

### 模組 F：LINE (多重訊息連發) 📩
* 在 Make.com Scenario 的最後端，加入 **`LINE -> Send a Reply Message`** 模組。
* 在 **`messages`** 陣列設定中，點擊 **`+ Add item`** 新增**兩個**訊息物件：
  * **第一則訊息**：傳送提示文字，例如：`✅ 老闆，最新文章已成功上架 Ghost！`
  * **第二則訊息**：**僅**傳送 Ghost 產生的文章 **`URL`** 變數（不加任何其他文字）。
* 只傳送純網址的第二則訊息，可完美觸發 LINE 的網址大圖（OGP）預覽功能，讓連結展開時顯示文章封面圖。

---

## 3. Scenario 系統排程與進階設定 ⚙️

### 啟動排程
* 完成模組設定後，點擊 Scenario 左下角的排程開關（切換至 **ON**）。
* 開啟後即可安全關閉瀏覽器，由 Make.com 雲端 24 小時接手監聽。

### 進階設定建議
| 設定項目 | 建議值 | 說明 |
|---|---|---|
| Sequential processing | **No** | 允許平行處理，避免單一任務卡死整個佇列 |
| Store incomplete executions | **Yes** | 儲存執行失敗的任務資料，方便事後查看錯誤原因與重試 |

### ⚠️ Free Plan 60 天閒置暫停限制
Make.com 免費方案規定：若一個 Scenario 超過 **60 天**未被觸發執行，系統將自動暫停該 Scenario。
請確保在兩個月內至少透過 LINE 觸發一次流程，以維持 Scenario 的啟用狀態。