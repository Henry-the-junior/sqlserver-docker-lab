# NOT NULL、DEFAULT、CHECK 觀察實驗

以下是針對 **`NOT NULL`、`DEFAULT`、`CHECK`** 這三種常見 SQL Server 資料約束所設計的實驗流程，包含建表、插入資料、觸發錯誤、觀察行為，讓你清楚掌握它們在**資料輸入限制**上的作用。

## 🧪 實驗目標

- 了解 `NOT NULL` 如何防止空值
- 觀察 `DEFAULT` 如何在未指定時補上預設值
- 測試 `CHECK` 如何限制欄位值的合法範圍

## 🧱 建立資料表：Products 商品表

```sql
CREATE TABLE Products (
    ProductID INT PRIMARY KEY,                        -- 主鍵
    ProductName NVARCHAR(100) NOT NULL,               -- 不允許為 NULL
    Category NVARCHAR(50) DEFAULT 'General',          -- 預設分類為 General
    Price DECIMAL(10, 2) CHECK (Price >= 0 AND Price <= 10000)  -- 價格範圍限制
);
```

## 🔄 資料操作實驗

### ✅ 1. 插入符合所有約束的資料

```sql
INSERT INTO Products (ProductID, ProductName, Category, Price)
VALUES (1, 'Pen', 'Stationery', 30.5);
```

✔ 預期：成功。

### 🚫 2. 插入 NULL 到 NOT NULL 欄位

```sql
INSERT INTO Products (ProductID, ProductName, Price)
VALUES (2, NULL, 50);
```

❌ 預期錯誤：

> Cannot insert the value NULL into column 'ProductName', table 'Products'; column does not allow nulls.
> 

### ✅ 3. 測試 DEFAULT 約束

```sql
INSERT INTO Products (ProductID, ProductName, Price)
VALUES (3, 'Notebook', 100);
```

✔ 預期結果：Category 欄位自動填入 `General`

### 🚫 4. 測試 CHECK 約束：負值價格

```sql
INSERT INTO Products (ProductID, ProductName, Category, Price)
VALUES (4, 'Eraser', 'Stationery', -10);
```

❌ 預期錯誤：

> The INSERT statement conflicted with the CHECK constraint "CK_Products_Price".
> 

### 🚫 5. 測試 CHECK 約束：價格超過上限

```sql
INSERT INTO Products (ProductID, ProductName, Category, Price)
VALUES (5, 'Laptop', 'Electronics', 20000);
```

❌ 預期錯誤：

> 同樣違反 CHECK 約束，價格不能超過 10000。
> 

### ✅ 6. 測試 NULL 在 CHECK 約束中的行為

```sql
INSERT INTO Products (ProductID, ProductName, Category, Price)
VALUES (6, 'Chair', 'Furniture', NULL);
```

✔ 預期：成功。因為 `CHECK` 不會限制 NULL（除非你額外加條件）。

## 🧠 總結對照表

| 約束類型 | 行為限制 | 遇到違規時表現 |
| --- | --- | --- |
| `NOT NULL` | 欄位值不可為 NULL | 拒絕插入 NULL，報錯 |
| `DEFAULT` | 未提供值時自動填入預設值 | 插入成功，填入預設內容 |
| `CHECK` | 限制欄位值必須滿足條件 | 不符條件則插入失敗，報錯 |
| `CHECK + NULL` | 若欄位值為 NULL 則不觸發條件 | 插入成功（除非使用 IS NOT NULL） |

## 🛠 延伸建議（可選）

- 將 `CHECK` 改寫為 `CHECK (Price IS NOT NULL AND Price >= 0)` 來防止 NULL。
- 加上 `DEFAULT GETDATE()` 觀察日期欄位的預設值行為。
- 嘗試用 `ALTER TABLE` 動態加入約束。