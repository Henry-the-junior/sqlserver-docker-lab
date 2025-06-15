# 未使用 GROUP BY 就使用 HAVING 的錯誤與修正方式

## 🎯 實驗目的

1. 理解 `HAVING` 的正確用途與條件。
2. 觀察未使用 `GROUP BY` 時，直接使用 `HAVING` 的錯誤訊息。
3. 學會如何搭配 `GROUP BY` 正確使用 `HAVING` 來過濾聚合結果。

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

## ❌ 錯誤查詢示範：未使用 GROUP BY 直接用 HAVING

```sql
SELECT CustomerName, SUM(Quantity) AS TotalQty
FROM Orders
HAVING SUM(Quantity) > 5;
```

### 🔴 預期錯誤訊息（SQL Server）：

```
Msg 8121, Level 16, State 1, Line 1
Column 'Orders.CustomerName' is invalid in the select list because it is not contained in either an aggregate function or the GROUP BY clause.
```

🔍 **解析**：

- SQL Server 無法知道如何對 `CustomerName` 做分組運算。
- `HAVING` 子句只能套用在已經分組後的結果上。
- 如果沒有 `GROUP BY`，整張表會視為一組，且不可單獨使用非聚合欄位。

---

## ✅ 正確寫法：加上 GROUP BY

```sql
SELECT CustomerName, SUM(Quantity) AS TotalQty
FROM Orders
GROUP BY CustomerName
HAVING SUM(Quantity) > 3;
```

### ✅ 預期結果：

| CustomerName | TotalQty |
| --- | --- |
| Alice | 8 |
| Bob | 4 |

---

## 🧠 補充說明：WHERE vs. HAVING

| 功能 | WHERE 子句 | HAVING 子句 |
| --- | --- | --- |
| 使用時機 | 分組之前過濾原始資料 | 分組之後過濾聚合資料 |
| 可用函數 | 不可使用聚合函數（錯誤） | 可使用聚合函數（如 `SUM`、`AVG`） |
| 是否需搭配 GROUP BY | 否 | 是（大多數情況） |

---

## 💡 延伸觀察題

1. 如果只要總和超過 10 的總量，而不需要列出客戶名稱，可以怎麼做？
    
    ➜ 就不需要 `GROUP BY`，整張表當成一組使用：
    
    ```sql
    SELECT SUM(Quantity) AS TotalQty
    FROM Orders
    HAVING SUM(Quantity) > 10;
    ```
    
    ✅ 這是**唯一不需 GROUP BY 也能用 HAVING 的情況**，因為沒有非聚合欄位。
    

---

## 📝 加分練習

- 查詢購買總數超過 2 的客戶與產品組合：
    
    ```sql
    SELECT CustomerName, Product, SUM(Quantity) AS TotalQty
    FROM Orders
    GROUP BY CustomerName, Product
    HAVING SUM(Quantity) > 2;
    ```