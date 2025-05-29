# SQL Server 正規化 vs 未正規化 查詢效能實驗流程

這份流程設計清楚分階段，並可直接在 SQL Server 上操作與觀察效能差異。

## 🧪 實驗目標

比較以下兩種資料設計在查詢效率上的表現：

1. **未正規化版本**：一張大表，資料重複
2. **正規化版本**：多張表格，使用外鍵關聯

## 🛠 實驗環境建議

- **SQL Server Developer 或 Express 版**
- 使用 **SSMS** 或 **Azure Data Studio**
- 建議記憶體至少 8GB（避免系統影響結果）

## 🧭 實驗流程共 5 步

### ✅ 第一步：建立資料表

### 🔸 A. 未正規化版本

```sql
CREATE TABLE UnnormalizedOrders (
    OrderID INT,
    CustomerName NVARCHAR(100),
    CustomerPhone NVARCHAR(20),
    ProductName NVARCHAR(100),
    UnitPrice DECIMAL(10,2),
    Quantity INT
);
```

### 🔸 B. 正規化版本

```sql
CREATE TABLE Customers (
    CustomerID INT PRIMARY KEY IDENTITY(1,1),
    CustomerName NVARCHAR(100),
    CustomerPhone NVARCHAR(20)
);

CREATE TABLE Orders (
    OrderID INT PRIMARY KEY,
    CustomerID INT,
    OrderDate DATE,
    FOREIGN KEY (CustomerID) REFERENCES Customers(CustomerID)
);

CREATE TABLE Products (
    ProductID INT PRIMARY KEY IDENTITY(1,1),
    ProductName NVARCHAR(100),
    UnitPrice DECIMAL(10,2)
);

CREATE TABLE OrderDetails (
    OrderID INT,
    ProductID INT,
    Quantity INT,
    PRIMARY KEY (OrderID, ProductID),
    FOREIGN KEY (OrderID) REFERENCES Orders(OrderID),
    FOREIGN KEY (ProductID) REFERENCES Products(ProductID)
);
```

---

### ✅ 第二步：插入模擬資料（約 10 萬筆）

### 🔸 A. 插入正規化資料

```sql
-- 插入 100 個顧客
INSERT INTO Customers (CustomerName, CustomerPhone)
SELECT 'Customer' + CAST(n AS NVARCHAR), '0912' + RIGHT('000000' + CAST(n AS VARCHAR), 6)
FROM (SELECT TOP 100 ROW_NUMBER() OVER (ORDER BY (SELECT NULL)) AS n FROM sys.objects) t;

-- 插入 20 個商品
INSERT INTO Products (ProductName, UnitPrice)
SELECT 'Product' + CAST(n AS NVARCHAR), 10 + n
FROM (SELECT TOP 20 ROW_NUMBER() OVER (ORDER BY (SELECT NULL)) AS n FROM sys.objects) t;

-- 插入 10 萬筆訂單與明細
DECLARE @i INT = 1;
WHILE @i <= 100000
BEGIN
    DECLARE @cust_id INT = ((@i - 1) % 100) + 1;
    DECLARE @order_id INT = @i;
    INSERT INTO Orders VALUES (@order_id, @cust_id, GETDATE());

    DECLARE @prod_id INT = ((@i - 1) % 20) + 1;
    INSERT INTO OrderDetails VALUES (@order_id, @prod_id, 1 + (@i % 5));

    SET @i += 1;
END;
```

### 🔸 B. 插入未正規化資料（從上面 JOIN 複製）

```sql
INSERT INTO UnnormalizedOrders
SELECT
    o.OrderID,
    c.CustomerName,
    c.CustomerPhone,
    p.ProductName,
    p.UnitPrice,
    od.Quantity
FROM Orders o
JOIN Customers c ON o.CustomerID = c.CustomerID
JOIN OrderDetails od ON o.OrderID = od.OrderID
JOIN Products p ON od.ProductID = p.ProductID;
```

---

### ✅ 第三步：執行查詢與觀察效能

### 開啟執行統計資訊

```sql
SET STATISTICS TIME ON;
SET STATISTICS IO ON;
```

### 查詢 A：未正規化表

```sql
SELECT
    OrderID,
    CustomerName,
    SUM(UnitPrice * Quantity) AS TotalAmount
FROM UnnormalizedOrders
GROUP BY OrderID, CustomerName;
```

### 查詢 B：正規化表

```sql
SELECT
    o.OrderID,
    c.CustomerName,
    SUM(p.UnitPrice * od.Quantity) AS TotalAmount
FROM Orders o
JOIN Customers c ON o.CustomerID = c.CustomerID
JOIN OrderDetails od ON o.OrderID = od.OrderID
JOIN Products p ON od.ProductID = p.ProductID
GROUP BY o.OrderID, c.CustomerName;
```

---

### ✅ 第四步：觀察結果

記錄以下資訊並比較：

| 項目 | 未正規化 | 正規化 |
| --- | --- | --- |
| 執行時間（ms） | 例如：50 ms | 例如：70 ms |
| IO 次數 | 例如：1000 次頁讀 | 例如：1500 次頁讀 |
| 是否使用索引 | ✅ / ❌ | ✅ / ❌ |
| 查詢語法複雜度 | 低 | 高（需 JOIN） |
| 結果數正確性 | ✅ | ✅ |

---

### ✅ 第五步：加上索引再測一次（進階比較）

```sql
-- 正規化表加索引
CREATE NONCLUSTERED INDEX idx_Orders_CustomerID ON Orders(CustomerID);
CREATE NONCLUSTERED INDEX idx_OrderDetails_OrderID ON OrderDetails(OrderID);
CREATE NONCLUSTERED INDEX idx_OrderDetails_ProductID ON OrderDetails(ProductID);
CREATE NONCLUSTERED INDEX idx_Products_ProductID ON Products(ProductID);

-- 再跑一次查詢 B 看差異
```

---

## 🎯 結論引導（可自行填寫）

- 在資料量不大的情況下，正規化 vs 未正規化查詢差距可能不大
- 隨著資料規模變大，「重複資料帶來的儲存浪費與更新困難」會放大
- **正規化需要 JOIN，確實有查詢成本，但可用索引、View、Materialized View 解決**

---

## 🧩 加碼建議

你可以試著再加入：

- 查詢訂單明細（含每項商品名稱與小計）
- 新增欄位：例如 `status`, `discount`，看正規化設計下怎麼加最清楚
- 用 `VIEW` 封裝正規化 JOIN 查詢，提升可讀性與維護性