# 聚合函數誤用在 WHERE 子句的錯誤與修正

## 🎯 實驗目的

1. 了解 `WHERE` 子句無法使用聚合函數（如 COUNT、SUM、AVG 等）。
2. 學會正確使用 `HAVING` 子句來篩選分組後的資料。
3. 理解 SQL 查詢的語句結構與執行順序。

---

## 🧰 實驗前置：建立測試資料表與資料

```sql
CREATE TABLE Sales (
    Id INT IDENTITY PRIMARY KEY,
    Product VARCHAR(50),
    Quantity INT
);

INSERT INTO Sales (Product, Quantity)
VALUES
('Apple', 10),
('Apple', 12),
('Apple', 8),
('Banana', 15),
('Banana', 18),
('Cherry', 5);
```

---

## ❌ 錯誤查詢示範：聚合函數誤用在 WHERE 子句

```sql
SELECT Product, SUM(Quantity) AS TotalQty
FROM Sales
WHERE SUM(Quantity) > 20
GROUP BY Product;
```

### 🔴 預期錯誤訊息（SQL Server）：

```
Msg 147, Level 15, State 1, Line 3
An aggregate may not appear in the WHERE clause unless it is in a subquery contained in a HAVING clause or a select list, and the column being aggregated is an outer reference.
```

---

## ✅ 正確寫法：使用 HAVING 子句

```sql
SELECT Product, SUM(Quantity) AS TotalQty
FROM Sales
GROUP BY Product
HAVING SUM(Quantity) > 20;
```

### ✅ 執行結果：

| Product | TotalQty |
| --- | --- |
| Apple | 30 |
| Banana | 33 |

---

## 🧠 延伸說明

- `WHERE` 是用來篩選 **原始資料（未分組）**，無法使用聚合函數。
- `HAVING` 是用來篩選 **分組後的結果**，可使用聚合條件。
- 查詢執行順序如下（簡化版）：
    
    ```
    FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY
    ```
    

---

## 💡 觀察與思考題

1. 為什麼 `WHERE` 不支援聚合函數？
2. 如果我想先過濾掉單筆資料的條件，再分組，要怎麼做？
    
    > ➜ 先用 WHERE 篩選資料 → 再用 GROUP BY → 最後用 HAVING 過濾分組結果。
    > 

---

## 📝 可選進階練習（加分題）

- 改寫查詢，找出 `SUM(Quantity)` 介於 20 到 35 的產品名稱：
    
    ```sql
    SELECT Product, SUM(Quantity) AS TotalQty
    FROM Sales
    GROUP BY Product
    HAVING SUM(Quantity) BETWEEN 20 AND 35;
    ```