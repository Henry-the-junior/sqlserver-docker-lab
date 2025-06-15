# 子查詢 vs JOIN：效能比較與執行計劃觀察

## 🎯 實驗目的

1. 示範在 `SELECT` 子句中使用子查詢（含聚合與 HAVING 條件）可能造成的效能差異。
2. 比較子查詢與 JOIN 在含有 `HAVING SUM(...) > ?` 條件下的執行策略。
3. 強化「先 JOIN，再 GROUP BY 聚合與篩選」的查詢效能與可維護性概念。

---

## 🧰 實驗前置：建立測試資料表與資料

```sql
-- 建立表格
DROP TABLE IF EXISTS Orders;
DROP TABLE IF EXISTS Customers;

CREATE TABLE Customers (
    Id INT PRIMARY KEY,
    Name VARCHAR(50)
);

CREATE TABLE Orders (
    Id INT PRIMARY KEY,
    CustomerId INT,
    Amount INT
);

-- 插入 10 萬個顧客
INSERT INTO Customers (Id, Name)
SELECT TOP 100000
    ROW_NUMBER() OVER (ORDER BY (SELECT NULL)),
    CONCAT('Customer_', ROW_NUMBER() OVER (ORDER BY (SELECT NULL)))
FROM sys.all_objects a CROSS JOIN sys.all_objects b;

-- 插入 1000 萬筆訂單，分散給顧客
INSERT INTO Orders (Id, CustomerId, Amount)
SELECT TOP 10000000
    ROW_NUMBER() OVER (ORDER BY (SELECT NULL)),
    ABS(CHECKSUM(NEWID())) % 100000 + 1,
    ABS(CHECKSUM(NEWID())) % 1000 + 1
FROM sys.all_objects a CROSS JOIN sys.all_objects b;

-- 建立索引
CREATE INDEX IX_Orders_CustomerId ON Orders(CustomerId);

```

---

## 📊 實驗查詢與觀察

### ❌ Query A：子查詢 + HAVING 條件

```sql
PRINT '== Query A: Subquery ==';
SELECT
    Name,
    (SELECT SUM(Amount)
     FROM Orders o
     WHERE o.CustomerId = c.Id
     HAVING SUM(Amount) > 40000) AS TotalSpent
FROM Customers c
OPTION (MAXDOP 1);

```

🔍 **說明與推論：**

- 每位客戶都會執行一次 `SUM(...) HAVING ...` 聚合子查詢。
- `HAVING` 是聚合後的條件，因此仍需完整聚合才能進行比較。
- 雖然 SQL Server 會嘗試最佳化，但這樣的寫法仍造成**重複計算與內部過濾**，對大數據量較不利。

---

### ✅ Query B：JOIN + GROUP BY + HAVING

```sql
PRINT '== Query B: JOIN + GROUP BY ==';
SELECT
    c.Name,
    SUM(o.Amount) AS TotalSpent
FROM Customers c
JOIN Orders o ON c.Id = o.CustomerId
GROUP BY c.Name
HAVING SUM(o.Amount) > 40000
OPTION (MAXDOP 1);

```

🔍 **說明與優勢：**

- `JOIN` 先將 Orders 與 Customers 合併，一次聚合所有資料。
- `HAVING` 適當使用在 `GROUP BY` 後，SQL Server 執行計劃可選擇 Hash Aggregate 或 Stream Aggregate。
- 對大型資料來說，這種查詢方式更具擴展性與效率。

---

## 📈 執行計劃觀察建議（使用 SSMS）

| 比較項目 | 子查詢版本 | JOIN 版本 |
| --- | --- | --- |
| 查詢次數 | 每位顧客各執行一次子查詢 | 一次查詢處理所有資料 |
| 聚合方式 | 每次子查詢聚合 | 單次聚合全部 |
| 執行計劃常見節點 | Nested Loop、Compute Scalar、Stream Aggregate | Hash Match、Stream Aggregate |
| 資源成本 | 較高，難最佳化 | 較低，容易併行處理 |
| 結果過濾方式 | 子查詢內 HAVING 條件 | GROUP BY 後 HAVING 篩選 |

---

## 📚 實務建議與結論

| 查詢策略 | 效能 | 可維護性 | 推薦使用情境 |
| --- | --- | --- | --- |
| 子查詢 + HAVING | ❌ 較差 | ❌ 難讀難優化 | 小資料量、一次性查詢 |
| JOIN + GROUP BY + HAVING | ✅ 優秀 | ✅ 清楚且擴充性強 | 大型資料集、日常分析查詢 |

---

## 📝 小結回顧

> 「能先 JOIN 就先 JOIN，聚合與篩選交給 GROUP BY + HAVING。」
> 

這樣的查詢架構：

- 更容易調整條件或接續處理。
- 在執行計劃與效能上幾乎永遠優於逐列子查詢。