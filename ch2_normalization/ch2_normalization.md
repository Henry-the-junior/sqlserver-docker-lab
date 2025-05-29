# 正規化 (Normalization)
這是資料庫設計中非常核心的一環，可以幫助你：

- 避免資料重複與異常 (redundancy & anomalies)
- 提高資料一致性與維護性
- 優化儲存與更新流程

---

## 📘 正規化是什麼？

正規化是一系列的規則，用來把「混亂或重複」的資料表結構，**整理成更合理的關聯表設計**。這些規則會一步步拆解資料表，把不同主題分開，並建立清楚的關聯。

---

## 🧱 常見的正規化階段（通常做到 3NF 就夠用了）

我們來一層一層看這三個最常用的正規化階段：

---

### 🔹 1NF：第一正規化（First Normal Form）

**目標：** 移除重複欄位與多值欄位 → 每個欄位只能放一筆原子資料（Atomic Value）

**錯誤範例：**

```
CustomerName | Phones
-------------|---------------
Alice        | 0912-000000, 0922-111111

```

**改善後（拆成多筆紀錄）：**

```
CustomerName | Phone
-------------|---------------
Alice        | 0912-000000
Alice        | 0922-111111

```

📌 **重點：**

- 不可在同一欄位存多個值（用逗號隔開是錯的）
- 若有多筆資料，應轉為多筆記錄（通常會拆成子表）

---

### 🔹 2NF：第二正規化（Second Normal Form）

**前提：** 資料已符合 1NF

**目標：** 移除與「主鍵部分」無關的欄位（Partial Dependency）

**錯誤範例（複合主鍵）：**

```
OrderID | ProductID | ProductName | UnitPrice
--------|-----------|-------------|----------
1001    | P001      | Apple       | 20
1001    | P002      | Banana      | 15

```

`OrderID` + `ProductID` 是複合主鍵，但 `ProductName`, `UnitPrice` 是只跟 `ProductID` 有關，和整個主鍵無關。

**改善後（拆出 Product 表）：**

- `OrderDetails(OrderID, ProductID, Quantity)`
- `Products(ProductID, ProductName, UnitPrice)`

📌 **重點：**

- 複合主鍵下，要檢查欄位是否只跟其中一部分有關 → 拆表！

---

### 🔹 3NF：第三正規化（Third Normal Form）

**前提：** 資料已符合 2NF

**目標：** 移除非主鍵間的傳遞依賴（Transitive Dependency）

**錯誤範例：**

```
EmployeeID | Name  | DepartmentID | DepartmentName
-----------|-------|--------------|----------------
E001       | John  | D01          | Sales
E002       | Alice | D02          | HR

```

這裡 `DepartmentName` 是依賴 `DepartmentID`，不是依賴主鍵 `EmployeeID`，違反 3NF。

**改善後：**

- `Employees(EmployeeID, Name, DepartmentID)`
- `Departments(DepartmentID, DepartmentName)`

📌 **重點：**

- 非主鍵欄位之間的依賴也要拆開 → 用 ID 建立關聯表

---

## 🧰 正規化的常見效益與缺點

| 優點 ✅ | 缺點 ⚠️ |
| --- | --- |
| 提高資料一致性與準確性 | JOIN 操作增加，查詢變複雜 |
| 減少儲存空間 | 執行效能可能下降（過度正規化） |
| 易於維護與更新 | 對於查詢頻繁的系統可能要反正規化 |

---

## 🧪 小測驗：這張表應該正規化嗎？

```
CustomerID | CustomerName | OrderID | ProductName | ProductPrice
-----------|--------------|---------|-------------|--------------
C001       | Alice        | O1001   | Apple       | 20
C001       | Alice        | O1001   | Banana      | 15

```

這張表包含：

- 客戶資料
- 訂單資料
- 商品資料

👉 明顯混合了不同主題！應拆成至少三張表：

1. Customers
2. Orders
3. Products + OrderDetails

---

進行一個「**從非正規化表格出發 → 正規化設計 → SQL Server 實作**」的完整流程。

## 🧾 非正規化資料表：CustomerOrders（目前很混亂）

```
OrderID | CustomerName | CustomerPhone | ProductName | UnitPrice | Quantity
--------|--------------|---------------|-------------|-----------|---------
1001    | Alice        | 0912000000    | Apple       | 20        | 2
1001    | Alice        | 0912000000    | Banana      | 15        | 1
1002    | Bob          | 0922111222    | Apple       | 20        | 3

```

### ❗ 問題：

- 客戶資訊重複（Alice 出現兩次）
- 商品資訊重複（Apple 出現兩次）
- 無法有效維護價格（若 Apple 價格改變，得改很多筆）
- 結構混合了客戶、訂單與商品資訊 → 不符合正規化原則

---

## ✅ 正規化設計（做到第三正規化）

### 1. **Customers**

| CustomerID | CustomerName | CustomerPhone |
| --- | --- | --- |
| 1 | Alice | 0912000000 |
| 2 | Bob | 0922111222 |

### 2. **Orders**

| OrderID | CustomerID | OrderDate |
| --- | --- | --- |
| 1001 | 1 | 2024-01-01 |
| 1002 | 2 | 2024-01-02 |

### 3. **Products**

| ProductID | ProductName | UnitPrice |
| --- | --- | --- |
| 1 | Apple | 20 |
| 2 | Banana | 15 |

### 4. **OrderDetails**

| OrderID | ProductID | Quantity |
| --- | --- | --- |
| 1001 | 1 | 2 |
| 1001 | 2 | 1 |
| 1002 | 1 | 3 |

---

## 🛠 SQL Server 建表語法練習

### 1️⃣ Customers

```sql
CREATE TABLE Customers (
    CustomerID INT PRIMARY KEY IDENTITY(1,1),
    CustomerName NVARCHAR(100) NOT NULL,
    CustomerPhone CHAR(10)
);
```

### 2️⃣ Orders

```sql
CREATE TABLE Orders (
    OrderID INT PRIMARY KEY,
    CustomerID INT NOT NULL,
    OrderDate DATE DEFAULT GETDATE(),
    FOREIGN KEY (CustomerID) REFERENCES Customers(CustomerID)
);
```

### 3️⃣ Products

```sql
CREATE TABLE Products (
    ProductID INT PRIMARY KEY IDENTITY(1,1),
    ProductName NVARCHAR(100) NOT NULL,
    UnitPrice DECIMAL(10,2) NOT NULL
);
```

### 4️⃣ OrderDetails

```sql
CREATE TABLE OrderDetails (
    OrderID INT,
    ProductID INT,
    Quantity INT NOT NULL,
    PRIMARY KEY (OrderID, ProductID),
    FOREIGN KEY (OrderID) REFERENCES Orders(OrderID),
    FOREIGN KEY (ProductID) REFERENCES Products(ProductID)
);
```

---

## 🔄 接下來你可以練習的內容

- 使用 `INSERT INTO` 新增上述資料
- 嘗試用 `JOIN` 查出完整訂單內容（訂單編號、客戶名稱、產品名稱、數量、價格）
- 模擬修改 Apple 的價格，看看只有一筆要更新（比未正規化省事）
