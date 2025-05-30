# FOREIGN KEY 強化資料一致性(Referential Integrity) 實驗

此實驗觀察 **`FOREIGN KEY` 如何強化資料一致性（Referential Integrity）**。這個實驗會示範：

- 插入合法 / 不合法外鍵資料
- 刪除主表中仍被參照的資料
- 調整外鍵行為以觀察不同反應

## 🧪 實驗目標

- 建立一對主從表格（Parent / Child）
- 加入 `FOREIGN KEY` 約束
- 觀察插入與刪除資料時的參照一致性行為
- 理解 SQL Server 拒絕違反參照完整性的操作

## 🧱 步驟一：建立資料表

```sql
-- 主表：Customers（客戶）
CREATE TABLE Customers (
    CustomerID INT PRIMARY KEY,
    Name NVARCHAR(50)
);

-- 從表：Orders（訂單），參考 Customers 的 CustomerID
CREATE TABLE Orders (
    OrderID INT PRIMARY KEY,
    CustomerID INT,
    OrderDate DATE,
    CONSTRAINT FK_Orders_Customers FOREIGN KEY (CustomerID)
        REFERENCES Customers(CustomerID)
);
```

## 🔄 步驟二：插入資料

```sql
-- 插入兩筆客戶資料
INSERT INTO Customers VALUES (1, 'Alice'), (2, 'Bob');

-- 插入訂單資料（合法的 CustomerID）
INSERT INTO Orders VALUES (100, 1, '2025-01-01'); -- Alice
```

✔ 預期：成功，因為 CustomerID = 1 存在於主表。

## ❌ 步驟三：插入不合法的外鍵值

```sql
-- 插入不存在的 CustomerID
INSERT INTO Orders VALUES (101, 999, '2025-01-02');
```

❌ 預期結果：失敗

錯誤訊息類似：

> The INSERT statement conflicted with the FOREIGN KEY constraint "FK_Orders_Customers". The conflict occurred in database ...
> 

## ❌ 步驟四：嘗試刪除仍被參照的主表資料

```sql
-- 試著刪除 Alice 的資料
DELETE FROM Customers WHERE CustomerID = 1;
```

❌ 預期結果：失敗

因為 Alice 的訂單（OrderID = 100）仍存在，會違反外鍵約束。

## 🛠️ 步驟五：重新建立外鍵，加上級聯操作觀察不同行為（延伸選項）

若你想讓刪除主表資料時能一併刪除從表資料，可嘗試以下操作：

```sql
-- 先移除原本的外鍵約束
ALTER TABLE Orders DROP CONSTRAINT FK_Orders_Customers;

-- 改成 ON DELETE CASCADE
ALTER TABLE Orders
ADD CONSTRAINT FK_Orders_Customers FOREIGN KEY (CustomerID)
    REFERENCES Customers(CustomerID)
    ON DELETE CASCADE;
```

然後再試：

```sql
-- 再插入資料
INSERT INTO Customers VALUES (1, 'Alice');
INSERT INTO Orders VALUES (100, 1, '2025-01-01');

-- 再刪除主表資料
DELETE FROM Customers WHERE CustomerID = 1;
```

✔ 預期結果：成功，`Orders` 中的相關訂單也會自動被刪除。

## ✅ 實驗重點觀察

| 操作 | 預期結果 | 原因 |
| --- | --- | --- |
| 插入合法 CustomerID 的訂單 | 成功 | 外鍵約束條件成立 |
| 插入不存在的 CustomerID | 失敗 | 違反外鍵參照 |
| 刪除仍被訂單參照的客戶 | 失敗 | 外鍵阻擋破壞參照完整性 |
| 加上 ON DELETE CASCADE 後刪除客戶 | 成功，並刪除訂單 | 級聯操作允許從表資料同步刪除 |