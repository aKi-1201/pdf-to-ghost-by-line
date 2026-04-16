# Oracle Cloud 部署 Ghost 部落格完整紀錄

這份文件記錄了在 Oracle Cloud 永久免費方案 (VM.Standard.E2.1.Micro) 的 Ubuntu 24.04 環境下，從零開始架設 Ghost (v6.30.0+) 的完整步驟。

## 1. 虛擬機規格與前置準備
* **主機規格**：VM.Standard.E2.1.Micro (1/8 OCPU, 1GB RAM)
* **作業系統**：Canonical Ubuntu 24.04 LTS
* **網域準備**：使用 DuckDNS 申請免費網域（例如 `laiwei.duckdns.org`），並**務必確認 IP 已正確指向 Oracle 主機的 Public IP**。

---

## 2. 第一道防火牆：Oracle Cloud 網頁控制台設定
在 SSH 連線進主機前，必須先在 Oracle 網頁後台打通外網連線：
1. 進入 `Compute` > `Instances` > 點擊主機名稱。
2. 點擊 `Primary VNIC` 旁邊的 `Subnet` 連結。
3. 點擊 `Security Lists` (預設安全清單)。
4. 新增 `Ingress Rules` (傳入規則)：
   * Source CIDR: `0.0.0.0/0`
   * IP Protocol: `TCP`
   * Destination Port Range: `80,443`
5. 儲存設定。

---

## 3. 連線主機與最佳化 (Swap 虛擬記憶體) 🌟 必做！
由於 E2.1.Micro 只有 1GB RAM，在執行 `npm install` 或運行 MySQL 時極易崩潰，必須建立 2GB 的 Swap。

連線進主機後，依序執行：
```bash
# 建立 2GB 虛擬記憶體
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# 設定開機自動掛載
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

---

## 4. 第二道防火牆：解除 Ubuntu 內部 iptables 封鎖
Oracle 的 Ubuntu 映像檔預設有極嚴格的 iptables 規則，會導致 Let's Encrypt 申請 SSL 憑證時發生 `Timeout during connect` 錯誤。
最直接有效的解決方式是清空預設阻擋規則，並預設允許連線（外層已有 Oracle Security List 保護，此舉是安全的）：

```bash
# 允許所有的連線通過
sudo iptables -P INPUT ACCEPT
sudo iptables -P FORWARD ACCEPT
sudo iptables -P OUTPUT ACCEPT

# 清空所有現有的複雜規則
sudo iptables -F

# 將狀態存檔 (需先安裝 iptables-persistent，若無則可跳過或先 apt install iptables-persistent)
sudo netfilter-persistent save
```

---

## 5. 安裝基礎環境 (Node.js 22, Nginx, MySQL, pnpm)
Ghost 6.30.0 以上版本強制要求 Node.js 22，並依賴 pnpm 進行套件管理。

**1. 安裝 Nginx 與 MySQL**
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y nginx mysql-server
```

**2. 安裝 Node.js 22**
```bash
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt-get install -y nodejs
```

**3. 安裝 pnpm 與 Ghost-CLI**
```bash
# Node 22 已內建 npm，直接用來安裝 pnpm 與 ghost-cli
sudo npm install -g pnpm
sudo npm install -g ghost-cli@latest
```

---

## 6. 設定 MySQL 資料庫
進入 MySQL 控制台：
```bash
sudo mysql
```
在 MySQL 提示字元中輸入以下指令（自訂密碼請替換 `YourPassword`）：
```sql
CREATE DATABASE ghost_db;
CREATE USER 'ghost_user'@'localhost' IDENTIFIED BY 'YourPassword';
GRANT ALL PRIVILEGES ON ghost_db.* TO 'ghost_user'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

---

## 7. 安裝與設定 Ghost
Ghost 不能安裝在 root 目錄，必須建立專屬資料夾並賦予權限。

**1. 準備目錄**
```bash
sudo mkdir -p /var/www/ghost
sudo chown ubuntu:ubuntu /var/www/ghost
sudo chmod 775 /var/www/ghost
cd /var/www/ghost
```

**2. 執行一鍵安裝**
```bash
ghost install
```

**3. 安裝過程中的互動問答紀錄**
* **Blog URL**: `https://laiwei.duckdns.org` (必須加 https)
* **MySQL hostname**: `127.0.0.1` (直接按 Enter)
* **MySQL username**: `ghost_user`
* **MySQL password**: (輸入步驟 6 設定的密碼)
* **Ghost database name**: `ghost_db`
* **Set up "ghost" mysql user?**: `No` (我們已經自己建了)
* **Set up Nginx?**: `Yes`
* **Set up SSL?**: `Yes` (需輸入 Email 以便憑證過期通知)
* **Set up Systemd?**: `Yes`
* **Start Ghost?**: `Yes`

---

## 8. 疑難排解紀錄 (Troubleshooting)

* **問題**：`ghost install` 跑到一半跳出 `Error getting validation data` 或 `Timeout during connect`。
* **原因**：DNS 的 IP 指向錯誤，或是 iptables 擋住了 Let's Encrypt 的 Port 80 連線。
* **解法**：
  1. 確定 DuckDNS IP 正確。
  2. 執行步驟 4 的 `iptables -F`。
  3. 用瀏覽器打開 `http://您的網域`，確認不再轉圈圈（出現 502 或預設畫面即可）。
  4. 進入 `/var/www/ghost` 目錄，手動補跑憑證申請：`ghost setup ssl`。

---
**完工！**
* 網站前台：`https://laiwei.duckdns.org`
* 管理後台：`https://laiwei.duckdns.org/ghost`