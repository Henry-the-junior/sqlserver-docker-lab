## ✅ **SQL Server 練習流程：TRANSACTION / ROLLBACK / COMMIT**

---

### 💡【一、情境說明】

你會在以下情況使用 `TRANSACTION`：

| 使用情境               | 原因                       |
| ------------------ | ------------------------ |
| 👥 批次新增多筆訂單或會員     | 確保資料要一起成功，避免只成功一部分造成不一致  |
| 💵 多步驟帳務轉帳操作       | 轉出與轉入資料要同時完成             |
| 🧾 批次修改或刪除資料       | 操作前後要保持資料完整性，若失敗要能還原     |
| 🛠️ 搭配 TRY...CATCH | 當異常發生時使用 `ROLLBACK` 回復資料 |

---

### 🧱【二、建立測試資料表】

```sql
CREATE TABLE BankAccounts (
    AccountID INT PRIMARY KEY IDENTITY(1,1),
    AccountName NVARCHAR(100),
    Balance DECIMAL(10, 2)
);

-- 初始資料
INSERT INTO BankAccounts (AccountName, Balance)
VALUES (N'王小明', 10000.00), (N'李美麗', 15000.00);
```

---

### 🔁【三、模擬轉帳操作】

#### ✅ 成功轉帳：使用 `BEGIN TRANSACTION + COMMIT`

```sql
BEGIN TRANSACTION;

-- 王小明扣款
UPDATE BankAccounts
SET Balance = Balance - 3000
WHERE AccountName = N'王小明';

-- 李美麗加款
UPDATE BankAccounts
SET Balance = Balance + 3000
WHERE AccountName = N'李美麗';

COMMIT;
```

🔎 查詢結果：

```sql
SELECT * FROM BankAccounts;
```

---

#### ✅ 更貼近實際情景

```sql
BEGIN TRY
    BEGIN TRANSACTION;

    -- 確認王小明帳戶存在且餘額足夠
    IF EXISTS (
        SELECT 1 FROM BankAccounts
        WHERE AccountName = N'王小明' AND Balance >= 3000
    )
    AND EXISTS (
        SELECT 1 FROM BankAccounts
        WHERE AccountName = N'李美麗'
    )
    BEGIN
        -- 扣款
        UPDATE BankAccounts
        SET Balance = Balance - 3000
        WHERE AccountName = N'王小明';

        -- 加款
        UPDATE BankAccounts
        SET Balance = Balance + 3000
        WHERE AccountName = N'李美麗';

        COMMIT;
    END
    ELSE
    BEGIN
        RAISERROR(N'帳戶不存在或餘額不足', 16, 1);
        ROLLBACK;
    END
END TRY
BEGIN CATCH
    ROLLBACK;
    PRINT ERROR_MESSAGE();
END CATCH;
```

🔎 查詢結果：

```sql
SELECT * FROM BankAccounts;
```

---

### 🧹【五、刪除資料表（結束練習）】

```sql
DROP TABLE BankAccounts;
```

---

### 📌【學習重點整理】

| 指令                  | 功能                 |
| ------------------- | ------------------ |
| `BEGIN TRANSACTION` | 開始一個交易區段           |
| `COMMIT`            | 提交變更               |
| `ROLLBACK`          | 還原到交易開始前狀態         |
| `TRY...CATCH`       | 捕捉錯誤並執行 `ROLLBACK` |

