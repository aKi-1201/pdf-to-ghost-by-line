## 2. 取得 Admin API 授權
* 前往你的 Ghost 後台 ( `https://yourname.duckdns.org/ghost` )。
* 導覽至 **Settings (設定) -> Integrations (整合)**。
* 點擊 **Add Custom Integration**，命名為「Make.com Automation」。
* **獲取金鑰**：
  * 複製 **Admin API Key** (供 Make.com 認證使用)。
  * 複製 **API URL**。

---

## 3. Ghost 隱私與 SEO 阻擋設定 🔒

### 防止搜尋引擎爬蟲收錄 (noindex)
如果你希望網站只讓知道網址的人瀏覽，而不被 Google / Bing 等搜尋引擎收錄，請利用 Ghost 的 Code Injection 功能注入 `<meta>` 標籤：

1. 進入 Ghost 後台，導覽至 **Settings -> Code Injection**。
2. 在 **Site Header (網站頂部)** 的黑色程式碼框中貼入以下語法：

```html
<meta name="robots" content="noindex, nofollow">
```

3. 點擊 **Save** 儲存。

**說明**：
* `noindex`：告知搜尋引擎機器人不要把任何一頁收錄進搜尋結果。
* `nofollow`：告知機器人不要順著網站內的連結繼續爬行。
* 此設定會自動套用至網站的**每一個頁面**的 `<head>` 區塊。

> ⚠️ 若網站在此設定前已被 Google 收錄，需等 Google 下次拜訪時才會正式移除，通常需要數天至數週。