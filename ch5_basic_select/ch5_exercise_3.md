# GROUP BY 中未列出所有非聚合欄位的錯誤與修正

## 🎯 實驗目的

1. 認識 SQL Server 對 GROUP BY 的語法要求。
2. 瞭解為什麼所有非聚合欄位都必須列在 GROUP BY 中。
3. 學習如何修正錯誤，確保聚合查詢語法正確。

---

## 🧰 實驗前置：建立測試資料表

```sql
CREATE TABLE Orders (
    Id INT PRIMARY KEY,
    CustomerName VARCHAR(50),
    Product VARCHAR(50),
    Quantity INT
);

INSERT INTO Orders (Id, CustomerName, Product, Quantity)
VALUES
(1, 'Alice', 'Apple', 5),
(2, 'Alice', 'Banana', 3),
(3, 'Bob', 'Apple', 4),
(4, 'Charlie', 'Apple', 2),
(5, 'Charlie', 'Banana', 1);
```

---

## ❌ 錯誤查詢示範：未將所有非聚合欄位列入 GROUP BY

```sql
SELECT CustomerName, Product, SUM(Quantity) AS TotalQty
FROM Orders
GROUP BY CustomerName;
```

### 🔴 預期錯誤訊息（SQL Server）：

```
Msg 8120, Level 16, State 1, Line 1
Column 'Orders.Product' is invalid in the select list because it is not contained in either an aggregate function or the GROUP BY clause.
```

---

## ✅ 正確寫法：將所有非聚合欄位加入 GROUP BY

```sql
SELECT CustomerName, Product, SUM(Quantity) AS TotalQty
FROM Orders
GROUP BY CustomerName, Product;
```

### ✅ 預期結果：

| CustomerName | Product | TotalQty |
| --- | --- | --- |
| Alice | Apple | 5 |
| Alice | Banana | 3 |
| Bob | Apple | 4 |
| Charlie | Apple | 2 |
| Charlie | Banana | 1 |

---

## 🧠 延伸說明

- SQL Server 要求：**任何未套用聚合函數的欄位，都必須明確列在 `GROUP BY` 中。**
- 聚合查詢實際上是將資料分組後，再對每組資料應用聚合運算。

---

## 💡 延伸觀察題

1. 如果我只想根據 CustomerName 分組，不管 Product 怎麼辦？
    
    ➜ 用 `STRING_AGG()` 或 `MIN(Product)` 等方式處理。
    
    ```sql
    SELECT CustomerName, MIN(Product) AS SampleProduct, SUM(Quantity) AS TotalQty
    FROM Orders
    GROUP BY CustomerName;
    ```
    ```sql
    SELECT CustomerName, STRING_AGG(Product, ',') AS SampleProduct, SUM(Quantity) AS TotalQty
    FROM Orders
    GROUP BY CustomerName;
    ```

2. 有沒有語法可以自動推導 GROUP BY 欄位？
    
    ➜ 沒有，開發者必須主動明確指定所有非聚合欄位。
    

---

## 📝 進階練習題（選做）

- 撰寫一個查詢，列出每位顧客購買的產品種類數量（使用 `COUNT(DISTINCT Product)`）。
    
    ```sql
    SELECT CustomerName, COUNT(DISTINCT Product) AS ProductCount
    FROM Orders
    GROUP BY CustomerName;
    ```