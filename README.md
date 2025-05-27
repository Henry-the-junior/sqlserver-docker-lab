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
