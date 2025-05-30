# Referential Integrity and Cascading Actions 實驗流程

以下是針對 SQL Server 2022 中「參照完整性與級聯操作（Referential Integrity and Cascading Actions）」的進階學習筆記，並設計了具體的 SQL 實驗流程，幫助你透過實作深入理解各種級聯行為的效果。

---

## 📘 參照完整性與級聯操作：概念總覽

在 SQL Server 中，透過外鍵（Foreign Key）約束，可以定義當主表（Parent Table）中的資料被更新或刪除時，從表（Child Table）中的對應資料應採取的行動，以維護資料的一致性。常見的級聯操作包括：

- `ON DELETE CASCADE`：當主表的資料被刪除時，自動刪除從表中對應的資料。
- `ON UPDATE CASCADE`：當主表的主鍵被更新時，自動更新從表中對應的外鍵值。
- `ON DELETE SET NULL` / `ON UPDATE SET NULL`：將從表中的外鍵值設為 `NULL`。
- `ON DELETE SET DEFAULT` / `ON UPDATE SET DEFAULT`：將從表中的外鍵值設為預設值。
- `NO ACTION` / `RESTRICT`：若從表中存在對應資料，則拒絕對主表的刪除或更新操作。

---

## 🧪 SQL 實驗設計：實作級聯操作

以下是一系列的 SQL 實驗，透過建立資料表、設定外鍵約束，並進行資料的更新與刪除操作，觀察各種級聯行為的效果。

### 🧱 步驟 1：建立資料表與基本資料

```sql
-- 建立主表：Departments
CREATE TABLE Departments (
    DeptID INT PRIMARY KEY,
    DeptName NVARCHAR(50)
);

-- 建立從表：Employees
CREATE TABLE Employees (
    EmpID INT PRIMARY KEY,
    EmpName NVARCHAR(50),
    DeptID INT,
    CONSTRAINT FK_Employees_Departments FOREIGN KEY (DeptID)
        REFERENCES Departments(DeptID)
        ON DELETE CASCADE
        ON UPDATE CASCADE
);

-- 插入範例資料
INSERT INTO Departments VALUES (1, 'HR'), (2, 'IT'), (3, 'Finance');
INSERT INTO Employees VALUES (101, 'Alice', 1), (102, 'Bob', 2), (103, 'Charlie', 3);
```

### 🔄 實驗 1：`ON DELETE CASCADE` 的效果

**操作：** 刪除 Departments 表中 DeptID 為 2 的部門。([Interface Technical Training](https://www.interfacett.com/class-content/referential-integrity-options-cascade-set-null-and-set-default/?utm_source=chatgpt.com))

```sql
DELETE FROM Departments WHERE DeptID = 2;
```

**預期結果：** Employees 表中 DeptID 為 2 的員工（Bob）也將被自動刪除。

### 🔁 實驗 2：`ON UPDATE CASCADE` 的效果

**操作：** 將 Departments 表中 DeptID 為 3 的部門 ID 更新為 4。

```sql
UPDATE Departments SET DeptID = 4 WHERE DeptID = 3;
```

**預期結果：** Employees 表中原本 DeptID 為 3 的員工（Charlie）的 DeptID 將自動更新為 4。

### ⚙️ 實驗 3：`ON DELETE SET NULL` 的效果

**操作：** 修改外鍵約束，設定 `ON DELETE SET NULL`，然後刪除 DeptID 為 1 的部門。

```sql
-- 先刪除原有的外鍵約束
ALTER TABLE Employees DROP CONSTRAINT FK_Employees_Departments;

-- 重新建立外鍵約束，設定 ON DELETE SET NULL
ALTER TABLE Employees
ADD CONSTRAINT FK_Employees_Departments FOREIGN KEY (DeptID)
    REFERENCES Departments(DeptID)
    ON DELETE SET NULL;

-- 刪除 DeptID 為 1 的部門
DELETE FROM Departments WHERE DeptID = 1;
```

**預期結果：** Employees 表中原本 DeptID 為 1 的員工（Alice）的 DeptID 將被設為 NULL。

### 🛠️ 實驗 4：`ON DELETE SET DEFAULT` 的效果

**操作：** 修改 Employees 表，為 DeptID 欄位設定預設值，並將外鍵約束設為 `ON DELETE SET DEFAULT`，然後刪除 DeptID 為 4 的部門。

```sql
-- 修改 DeptID 欄位，設定預設值為 99
ALTER TABLE Employees
ADD CONSTRAINT DF_Employees_DeptID DEFAULT 99 FOR DeptID;

-- 需要給一個預設的部門，否則在重設員工部門時會報錯
INSERT INTO Departments (DeptID, DeptName) VALUES (99, 'Default Dept');

-- 先刪除原有的外鍵約束
ALTER TABLE Employees DROP CONSTRAINT FK_Employees_Departments;

-- 重新建立外鍵約束，設定 ON DELETE SET DEFAULT
ALTER TABLE Employees
ADD CONSTRAINT FK_Employees_Departments FOREIGN KEY (DeptID)
    REFERENCES Departments(DeptID)
    ON DELETE SET DEFAULT;

-- 刪除 DeptID 為 4 的部門
DELETE FROM Departments WHERE DeptID = 4;
```

**預期結果：** Employees 表中原本 DeptID 為 4 的員工（Charlie）的 DeptID 將被設為預設值 99。

---

## 📌 注意事項與限制

- **`SET NULL` 與 `SET DEFAULT` 的限制：** 使用這些操作時，外鍵欄位必須允許 NULL 或已設定預設值，否則操作將失敗。
- **`ON UPDATE CASCADE` 的使用情境：** 當主鍵值可能會變更時，`ON UPDATE CASCADE` 可確保從表中的外鍵值自動更新，維持資料一致性。
- **避免多重級聯路徑：** SQL Server 不允許在同一資料表中存在多條級聯路徑，否則會導致錯誤。

---

## 🎥 延伸學習資源

- **影片教學：** [SQL Server part-5 Cascading referential integrity - YouTube](https://www.youtube.com/watch?v=qhz7CpSepZ0)
- **文章參考：** [DELETE CASCADE and UPDATE CASCADE in SQL Server foreign key](https://www.sqlshack.com/delete-cascade-and-update-cascade-in-sql-server-foreign-key/)

