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

### 模組 D：Ghost (發布文章)
* 選擇 `Ghost -> Make an API Call`。
* **Method**: `POST`
* **URL**: `/admin/posts/`
* **Body**: 將模組 C 產生的 HTML 內容帶入 `html` 欄位，標題帶入 `title` 欄位，並設定 status 為 `published`。
*(註：若 Make 有原生的 Create a Post 模組，可直接填寫對應欄位)*