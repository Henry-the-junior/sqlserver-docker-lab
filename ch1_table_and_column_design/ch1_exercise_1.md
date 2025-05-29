### ✅ **SQL Server 資料表操作練習**

---

### 🧱 1. 建立資料表（CREATE TABLE）

```sql
CREATE TABLE Employees (
    EmployeeID INT PRIMARY KEY IDENTITY(1,1),
    FullName NVARCHAR(100),
    Position NVARCHAR(50),
    HireDate DATE,
    Salary DECIMAL(10, 2)
);
```

📌**說明**：

* `IDENTITY(1,1)` 會自動產生遞增的主鍵值。
* 使用 `NVARCHAR` 支援多語言文字。
* `DECIMAL(10,2)` 表示最多 10 位數，其中 2 位是小數。

---

### ✍️ 2. 插入資料（INSERT INTO）

```sql
INSERT INTO Employees (FullName, Position, HireDate, Salary)
VALUES
(N'王小明', N'工程師', '2022-03-01', 65000.00),
(N'陳美麗', N'行銷專員', '2021-07-15', 55000.00),
(N'李志強', N'業務代表', '2020-12-10', 48000.00);
```

---

### 🔍 3. 查詢資料（SELECT）

```sql
-- 查詢所有資料
SELECT * FROM Employees;

-- 查詢薪水高於 50000 的員工
SELECT FullName, Salary FROM Employees
WHERE Salary > 50000;
```

---

### 🛠️ 4. 更新資料（UPDATE）

```sql
-- 調整薪水
UPDATE Employees
SET Salary = Salary + 2000
WHERE Position = N'工程師';
```

---

### ❌ 5. 刪除資料（DELETE）

```sql
-- 刪除薪水低於 50000 的員工
DELETE FROM Employees
WHERE Salary < 50000;
```

---

### 🗑️ 6. 刪除資料表（DROP TABLE）

```sql
DROP TABLE Employees;
```

⚠️**注意**：刪除表格會連同裡面的所有資料一起刪除，無法復原，請謹慎操作。

