# SQL Server Docker Lab

這個專案提供一個可重現的 SQL Server 練習環境，使用 Docker 建立，適合用於資料庫學習與測試用途。

## ✅ 環境需求

- 已安裝 Docker 與 Docker Compose（建議使用 Docker Desktop）
- 支援 `.env` 檔案功能的 Docker 版本

## 📦 專案結構

- `docker-compose.yml`：定義 SQL Server 容器的設定
- `init-scripts/`：放置初始化的 SQL 腳本（可選）
- `.env.example`：提供環境變數設定的範本
- `.gitignore`：排除不需要版本控制的資料與密碼檔
- `README.md`：使用說明

## 🚀 如何使用

1. 複製 `.env.example` 成 `.env`，並設定密碼：

   ```
   cp .env.example .env
   ```

2. 啟動容器：

   ```
   docker compose up -d
   ```

3. 使用 DBeaver、SSMS 或 `sqlcmd` 連線至 SQL Server：

   - Host：localhost
   - Port：1433
   - Username：sa
   - Password：你設定的密碼（`.env` 中）

## 📂 資料掛載與初始化（可選）

- `data/` 目錄會儲存 SQL Server 資料檔，可保留練習結果。
- `init-scripts/` 可放置 SQL 腳本，供你手動或透過工具執行。

## 🛠 常見指令

- 停止容器：

  ```
  docker compose down
  ```

- 查看容器日誌：

  ```
  docker compose logs -f
  ```

## 🔐 安全提醒

請勿在 `.env` 中使用弱密碼或將含密碼的 `.env` 檔案上傳至公開 GitHub repository。
太好了！這樣可以讓你的 `README.md` 更完整，幫助其他人快速解決在 WSL 中用 SQL Server Docker 容器時的常見問題。
以下是你可以直接加進去的段落內容，格式已整理好，可以複製貼上到你 `README.md` 裡：


## 🧯 常見問題排除（WSL 環境）

因為我是在 **Windows + WSL** 的環境下啟動本專案，以下是幾個常見的問題與解法。

### 🛰️ 查詢 WSL 的 IP 位址

SQL Server Docker 容器預設不會對 Windows 主機自動開放連線，所以你需要查出 WSL 的 IP，供 Windows 上的工具（如 SSMS）連線：

```bash
ip addr show eth0
```

找出 `inet` 那一行，例如：

```
inet 172.21.90.85/20 ...
```

### 🔁 將 WSL SQL Server 轉發到 Windows `localhost:1433`（portproxy）

在 Windows PowerShell (以系統管理員執行) 中設定 port forwarding：

```powershell
netsh interface portproxy delete v4tov4 listenport=1433 listenaddress=0.0.0.0
netsh interface portproxy add v4tov4 listenport=1433 listenaddress=0.0.0.0 connectport=1433 connectaddress=WSL_IP
```

請將 `WSL_IP` 替換成你剛剛查到的 IP，例如：

```powershell
netsh interface portproxy add v4tov4 listenport=1433 listenaddress=0.0.0.0 connectport=1433 connectaddress=172.21.90.85
```

> 💡 然後在 SSMS 使用 `localhost,1433` 作為 Server Name 連線即可！

### 🛠️ 修正 `./data` 權限問題（避免容器啟動錯誤）

若你使用了 volume 掛載本機 `./data` 資料夾，請確保它有正確的權限，否則容器會出錯並停止。

在 **WSL 終端機中執行**：

```bash
sudo chown -R 10001:0 ./data
```

這將把資料夾的擁有者設成與 SQL Server 容器相容的身份（UID 10001 是 SQL Server 預設使用者）。

### 🔗 SSMS 連線加密方式

選擇 `Optional` 才能連線
