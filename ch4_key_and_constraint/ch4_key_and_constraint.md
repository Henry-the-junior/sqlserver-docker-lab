# 主鍵、外鍵、約束 (pk, fk, constraint)

以下是關於 SQL Server 2022 中「主鍵（Primary Key, PK）」、「外鍵（Foreign Key, FK）」以及「約束（Constraint）」的學習筆記，適合初學者理解並實作。

---

## 🧩 主鍵（Primary Key, PK）

### 定義與特性

- 主鍵是表格中用來**唯一識別每一筆資料**的欄位或欄位組合。
- 每個表格只能有一個主鍵，但主鍵可以由多個欄位組成（稱為複合主鍵）。
- 主鍵欄位的值**不能為 NULL**，且必須**唯一**。([Microsoft Learn](https://learn.microsoft.com/en-us/sql/relational-databases/tables/primary-and-foreign-key-constraints?view=sql-server-ver17&utm_source=chatgpt.com), [DataFlair](https://data-flair.training/blogs/primary-and-foreign-key-in-sql/?utm_source=chatgpt.com))

### 建立主鍵的語法

在建立表格時指定主鍵：

```sql
CREATE TABLE Students (
    StudentID INT NOT NULL,
    Name NVARCHAR(50),
    PRIMARY KEY (StudentID)
);
```

([W3Schools](https://www.w3schools.com/sql/sql_primarykey.ASP?utm_source=chatgpt.com))

在已存在的表格中新增主鍵：

```sql
ALTER TABLE Students
ADD CONSTRAINT PK_Students PRIMARY KEY (StudentID);
```

---

## 🔗 外鍵（Foreign Key, FK）

### 定義與特性

- 外鍵是用來**建立兩個表格之間的關聯性**的欄位，通常參考另一個表格的主鍵。
- 外鍵可以為 NULL，表示該筆資料暫時沒有對應的關聯。
- 透過外鍵，可以**維護資料的一致性與完整性**，避免出現無效的參考。([DataFlair](https://data-flair.training/blogs/primary-and-foreign-key-in-sql/?utm_source=chatgpt.com))

### 建立外鍵的語法

在建立表格時指定外鍵：

```sql
CREATE TABLE Orders (
    OrderID INT NOT NULL PRIMARY KEY,
    StudentID INT,
    FOREIGN KEY (StudentID) REFERENCES Students(StudentID)
);
```

在已存在的表格中新增外鍵：

```sql
ALTER TABLE Orders
ADD CONSTRAINT FK_Orders_Students FOREIGN KEY (StudentID)
REFERENCES Students(StudentID);
```

---

## 🛡️ 約束（Constraint）

### 常見的約束類型

- **NOT NULL**：指定欄位不能為 NULL。
- **UNIQUE**：確保欄位中的值唯一。
- **PRIMARY KEY**：唯一識別每一筆資料，結合 NOT NULL 和 UNIQUE 的特性。
- **FOREIGN KEY**：建立表格之間的參照關係，維護參照完整性。
- **CHECK**：限制欄位的值必須符合特定條件。
- **DEFAULT**：為欄位指定預設值。([W3Schools](https://www.w3schools.com/sql/sql_constraints.asp?utm_source=chatgpt.com))

### 建立約束的語法範例

建立具有多種約束的表格：

```sql
CREATE TABLE Students (
    StudentID INT NOT NULL,
    Name NVARCHAR(50) NOT NULL,
    Age INT CHECK (Age >= 0),
    EnrollmentDate DATE DEFAULT GETDATE(),
    PRIMARY KEY (StudentID)
);
```

---

## 🔄 參照完整性與級聯操作 (Referential Integrity and Cascading Actions)

在使用外鍵時，可以定義當參考的主表資料被更新或刪除時，從表的行為：([Microsoft Learn](https://learn.microsoft.com/en-us/sql/relational-databases/tables/primary-and-foreign-key-constraints?view=sql-server-ver17&utm_source=chatgpt.com))

| 指令 | 說明 |
| --- | --- |
| `ON DELETE CASCADE` | 刪除主表資料時，自動刪除從表中對應資料。 |
| `ON UPDATE CASCADE` | 更新主表主鍵時，自動更新從表中對應資料。 |
| `ON DELETE SET NULL` | 刪除主表資料時，從表對應欄位設為 NULL。 |
| `ON DELETE SET DEFAULT` | 刪除主表資料時，從表對應欄位設為預設值。 |
| `ON DELETE NO ACTION` | 若從表中存在對應資料時，禁止刪除主表資料。 |

例如，定義當主表資料被刪除時，從表資料也一併刪除：

```sql
ALTER TABLE Orders
ADD CONSTRAINT FK_Orders_Students FOREIGN KEY (StudentID)
REFERENCES Students(StudentID)
ON DELETE CASCADE;
```

---

## 🎓 補充學習資源

以下是一些推薦的學習資源，幫助您更深入了解主鍵、外鍵與約束的概念與實作：

- [微軟SQL Server 2022基礎資料庫設計和開發- 主鍵(PK)和外鍵(FK)](https://www.youtube.com/watch?v=-rPSbwGSU8k&utm_source=chatgpt.com)
- [索引、FOREIGN KEY 外鍵碼、REFERENCES 關係教學(中文字幕)](https://www.youtube.com/watch?v=Aj-CXeWH9BI&utm_source=chatgpt.com)

這些資源提供了詳細的教學與實作範例，適合進一步學習與練習。
