### ✅ **SQL Server 練習：DEFAULT + NOT NULL**

---

### 🧱 1. 建立資料表（加入 DEFAULT 與 NOT NULL）

```sql
CREATE TABLE Products (
    ProductID INT PRIMARY KEY IDENTITY(1,1),
    ProductName NVARCHAR(100) NOT NULL,
    Category NVARCHAR(50) DEFAULT N'未分類',
    Price DECIMAL(10, 2) NOT NULL,
    CreatedDate DATETIME DEFAULT GETDATE()
);
```

📌**說明**：

* `ProductName` 和 `Price` 被設為 `NOT NULL`，不可為空。
* `Category` 和 `CreatedDate` 有預設值，未指定會自動補上。
* `GETDATE()` 是內建函式，會填入當下時間。

---

### ✍️ 2. 插入完整資料（全部欄位都指定）

```sql
INSERT INTO Products (ProductName, Category, Price, CreatedDate)
VALUES (N'滑鼠', N'電腦周邊', 399.00, '2024-01-01');
```

✅ 預期成功。

---

### ✍️ 3. 插入時省略 DEFAULT 欄位

```sql
INSERT INTO Products (ProductName, Price)
VALUES (N'鍵盤', 799.00);
```

✅ 預期 `Category` 為「未分類」，`CreatedDate` 為目前時間。

---

### 🧪 4. 嘗試插入 NULL 到 NOT NULL 欄位

```sql
-- 會出錯
INSERT INTO Products (ProductName, Price)
VALUES (NULL, 299.00);
```

⚠️ 預期錯誤：`ProductName` 不允許 NULL。

---

### 🧪 5. 嘗試省略 NOT NULL 欄位（不給值）

```sql
-- 會出錯
INSERT INTO Products (ProductName)
VALUES (N'耳機');
```

⚠️ 預期錯誤：`Price` 是 NOT NULL，但沒有提供值，也沒有預設值。

---

### 🧹 6. 查詢觀察結果

```sql
SELECT * FROM Products;
```

你應該可以看到：

* 有預設值的欄位會自動填上資料
* `NOT NULL` 欄位沒有給值會報錯，所以沒有資料
* 嘗試插入 `NULL` 到 `NOT NULL` 欄位也會報錯，所以也沒有資料

---

### 🗑️ 7. 刪除資料表（結束練習）

```sql
DROP TABLE Products;
```

