# 結構調整與錯誤處理實驗

這組實驗會幫助你熟悉以下幾個重點：

- 如何使用 `ALTER TABLE` 動態新增與移除約束（例如 `CHECK`）
- 如何在資料操作違反約束時，透過 `TRY...CATCH` 結構捕捉錯誤訊息
- 實作整個流程的 SQL 範例，包含觀察點與預期行為

## 🧪 實驗目標

| 重點 | 說明 |
| --- | --- |
| `ALTER TABLE ADD CONSTRAINT` | 動態新增約束（如 CHECK） |
| `ALTER TABLE DROP CONSTRAINT` | 動態移除現有約束 |
| `TRY...CATCH` | 捕捉約束違反導致的錯誤，做錯誤處理 |

## 🧱 步驟一：建立資料表

```sql
IF OBJECT_ID('Inventory', 'U') IS NOT NULL DROP TABLE Inventory;

CREATE TABLE Inventory (
    ItemID INT PRIMARY KEY,
    ItemName NVARCHAR(50),
    Quantity INT
);
```

## ➕ 步驟二：動態加入 `CHECK` 約束

```sql
-- 限制 Quantity 不得為負數
ALTER TABLE Inventory
ADD CONSTRAINT CHK_Inventory_Quantity
CHECK (Quantity >= 0);
```

## ✅ 步驟三：插入合法資料

```sql
INSERT INTO Inventory VALUES (1, 'Pen', 10);
```

✔ 預期成功：數量為正數，符合 `CHECK` 條件。

## ❌ 步驟四：插入違反 CHECK 條件的資料 + TRY...CATCH

```sql
BEGIN TRY
    INSERT INTO Inventory VALUES (2, 'Notebook', -5);
END TRY
BEGIN CATCH
    PRINT 'ERROR： ' + ERROR_MESSAGE();
END CATCH;
```

❌ 預期：進入 `CATCH` 區塊並顯示錯誤訊息，例如：

> ⚠️ 錯誤：The INSERT statement conflicted with the CHECK constraint "CHK_Inventory_Quantity".
> 

## ➖ 步驟五：動態移除 `CHECK` 約束

```sql
ALTER TABLE Inventory
DROP CONSTRAINT CHK_Inventory_Quantity;
```

## ✅ 步驟六：再次插入負數資料（應該成功）

```sql
INSERT INTO Inventory VALUES (3, 'Pencil', -10);
```

✔ 預期成功：已移除 CHECK 約束，Quantity 不再被限制。

## 🎯 總結觀察點

| 步驟 | 行為 | 預期結果 |
| --- | --- | --- |
| 新增 CHECK 約束 | 限制 Quantity 必須 >= 0 | 插入 -5 時會報錯 |
| 用 TRY...CATCH 包住 INSERT | 避免錯誤中斷程序 | 能顯示錯誤訊息，程式不中斷 |
| 移除 CHECK 約束 | 不再限制 Quantity 數值 | 插入負值成功 |

## 🧠 延伸建議

- 你可以試著動態加入 `DEFAULT` 或 `UNIQUE` 約束，也能透過類似流程練習。
- 搭配 `sys.check_constraints` 檢查目前資料表中有哪些約束：

```sql
SELECT * FROM sys.check_constraints WHERE parent_object_id = OBJECT_ID('Inventory');
```