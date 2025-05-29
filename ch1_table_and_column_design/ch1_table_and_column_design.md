# Table and Column Design

## 🧱 資料表與欄位設計的核心原則

以下從設計角度切分為六大重點：

### 1. 表格設計的出發點：**一表一主題（One Table, One Entity）**

- 每張表格應專注於一種「實體」或「業務物件」，例如：
    - 客戶表 `Customers`
    - 訂單表 `Orders`
    - 商品表 `Products`
- 避免在同一張表混入多個主題的欄位（例如客戶資料混進訂單資訊）

---

### 2. 清晰、一致、語意明確的命名規則

- **表名**用複數（如：`Users`, `Orders`），因為它儲存多筆資料
- **欄位名**用單數（如：`user_id`, `email`）
- 命名規則要一致：建議選一種格式並貫徹
    - `snake_case`（如：`created_at`）：常見於 PostgreSQL、MySQL
    - `camelCase`（如：`createdAt`）：常見於程式物件命名中
- 避免使用模糊或縮寫（例如：`desc` 不如用 `description`）

---

### 3. 每張表一定要有**主鍵（Primary Key）**

- 用來唯一識別每一筆資料
- 常見的主鍵設計方式：
    - 整數流水號（如 `id INT AUTO_INCREMENT`）
    - UUID（適合分散式系統或需要全球唯一性時）
- 主鍵應該「不變且唯一」，不建議用 email、姓名當主鍵，因為它們可能變動

---

### 4. 建立適當的索引與外鍵

- **索引（Index）**：加快查詢速度（針對常查詢、JOIN、WHERE 條件欄位）
- **外鍵（Foreign Key）**：建立資料關聯，例如：
    
    ```sql
    FOREIGN KEY (customer_id) REFERENCES Customers(id)
    ```
    
- 外鍵可幫助資料一致性，例如刪除父項時可選擇是否連動刪除子項（ON DELETE CASCADE）

---

### 5. 欄位設計的考量

- **必要 vs 非必要欄位**
    - 哪些欄位是必填？（加上 `NOT NULL`）
    - 哪些欄位會有空值？（允許 `NULL`）
- **欄位的命名與用途應明確**
    - 不建議出現 `data1`, `data2`, `info`, `flag` 等模糊名稱
- **使用預設值（Default）提高資料完整性**
    
    ```sql
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
    ```
    

---

### 6. 實務上的其他建議

- **加入紀錄欄位：**
    - `created_at`, `updated_at`, `deleted_at`（便於日後追蹤與資料軌跡）
- **避免重複資料與欄位：**
    - 同樣的資料只存一份，例如「部門名稱」不該出現在每位員工資料中，而應設一張部門表
- **欄位順序建議：**
    - 建議將常查詢或識別欄位放前面，如 `id`, `name`, `status`, 最後才放時間戳記欄位

---

## 🛠 小範例：會員資料表設計

```sql
CREATE TABLE Members (
    member_id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(50) NOT NULL,
    email VARCHAR(100) NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    gender CHAR(1),
    birth_date DATE,
    status ENUM('active', 'inactive', 'banned') DEFAULT 'active',
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

---

以下是針對 **SQL Server** 常見的資料表與欄位設計錯誤情境，以及實務解法，幫助你更深入理解怎麼避免問題、提升資料庫品質與效能。

## 🎯 情境一：**設計不當導致資料重複與不一致**

### 🧨 問題情境：

一張 `Orders` 表中包含了訂單資訊 **和** 客戶資訊（如客戶姓名、地址）。結果同一個客戶出現多次，且地址還有拼錯的情況。

### 💡 解法：

- 把「客戶」資訊獨立出來成一張 `Customers` 表，使用外鍵關聯：

```sql
-- 客戶表
CREATE TABLE Customers (
    customer_id INT PRIMARY KEY IDENTITY(1,1),
    name NVARCHAR(100) NOT NULL,
    address NVARCHAR(200)
);

-- 訂單表
CREATE TABLE Orders (
    order_id INT PRIMARY KEY IDENTITY(1,1),
    customer_id INT NOT NULL,
    order_date DATETIME NOT NULL,
    total_amount DECIMAL(10,2),
    FOREIGN KEY (customer_id) REFERENCES Customers(customer_id)
);
```

> 🎯 目的：正規化資料、避免重複、易於維護。
> 

---

## 🎯 情境二：**欄位設計不明確，導致資料難以查詢與理解**

### 🧨 問題情境：

你看到一張表有欄位 `Flag1`, `Flag2`, `Data1`, `InfoA`，根本不知道每欄的意義，查詢與報表都很困難。

### 💡 解法：

- 替欄位命名要語意清楚：

```sql
-- 原來模糊的設計：
-- CREATE TABLE Products (Flag1 INT, InfoA NVARCHAR(100));

-- 改為語意清楚的設計：
CREATE TABLE Products (
    is_active BIT DEFAULT 1,
    description NVARCHAR(100)
);
```

> ✅ 建議：讓欄位名稱能反映用途，例如用 is_deleted, created_at, product_status 等。
> 

---

## 🎯 情境三：**資料型別過度寬鬆，造成空間浪費與效能低落**

### 🧨 問題情境：

你將所有文字欄位都設為 `NVARCHAR(MAX)`，結果資料表變超大，查詢慢、索引失效。

### 💡 解法：

- 為每個欄位估計合理長度，使用適當的型別與長度：

```sql
-- 不建議：
-- name NVARCHAR(MAX)

-- 改為：
name NVARCHAR(50)

```

- 如果內容是固定長度（如身份證字號），用 `CHAR(n)` 會更有效率：

```sql
national_id CHAR(10) NOT NULL
```

> 🚀 好處：節省儲存空間、索引更快。
> 

---

## 🎯 情境四：**沒有加上 created_at / updated_at，資料無法追蹤變動**

### 🧨 問題情境：

不知道某筆資料是何時建立的、何時修改的，無法做版本追蹤或時間篩選。

### 💡 解法：

- 加入系統欄位，並用 SQL Server 的 `DEFAULT` 或 `ON UPDATE` 自動處理：

```sql
created_at DATETIME DEFAULT GETDATE(),
updated_at DATETIME DEFAULT GETDATE()
```

或搭配觸發器（Trigger）實作每次更新時自動記錄 `updated_at`。

---

## 🎯 情境五：**查詢速度慢，但沒有設索引（Index）**

### 🧨 問題情境：

你常用 `email` 或 `order_date` 去查資料，但沒有索引，造成查詢掃全表（Table Scan）。

### 💡 解法：

- 為常用查詢條件建立索引：

```sql
CREATE NONCLUSTERED INDEX idx_customers_email ON Customers(email);
CREATE NONCLUSTERED INDEX idx_orders_order_date ON Orders(order_date);
```

> 🔍 補充：搭配查詢計劃（Execution Plan）觀察是否有效使用索引。
> 

---

## 🎯 情境六：**沒有設唯一鍵（UNIQUE），造成重複資料**

### 🧨 問題情境：

`email` 欄位沒有限制，導致使用者可以註冊多次相同 email。

### 💡 解法：

```sql
ALTER TABLE Users
ADD CONSTRAINT UQ_Users_Email UNIQUE (email);
```

> ✅ 建議：對於 email、帳號、身分證字號這種資料，務必加上 UNIQUE。
>