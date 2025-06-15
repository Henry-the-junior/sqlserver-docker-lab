# JOIN 條件錯誤導致大量重複列的問題與修正

## 🎯 實驗目的

1. 示範錯誤 JOIN 條件導致的「多對多配對」或「笛卡爾積」問題。
2. 比較錯誤與正確的 INNER JOIN 寫法。
3. 建立對 JOIN 邏輯與結果控制的正確認知。

---

## 🧰 實驗前置：建立兩張測試資料表

```sql
-- 建立員工表
CREATE TABLE Employees (
    Id INT PRIMARY KEY,
    Name VARCHAR(50),
    DepartmentId INT
);

-- 建立部門表
CREATE TABLE Departments (
    Id INT PRIMARY KEY,
    DepartmentName VARCHAR(50)
);

-- 插入員工資料
INSERT INTO Employees (Id, Name, DepartmentId)
VALUES
(1, 'Alice', 1),
(2, 'Bob', 2),
(3, 'Charlie', 1);

-- 插入部門資料
INSERT INTO Departments (Id, DepartmentName)
VALUES
(1, 'Sales'),
(2, 'IT'),
(3, 'HR');
```

---

## ❌ 錯誤 JOIN 示範：沒有寫對應條件

```sql
-- 故意省略 ON 條件
SELECT e.Name, d.DepartmentName
FROM Employees e
JOIN Departments d;
```

### 🔴 預期結果：

```sql
Msg 102, Level 15, State 1, Line 3
Incorrect syntax near 'd'.
```

---

## ✅ 正確 INNER JOIN 寫法

```sql
-- 使用正確的 INNER JOIN 條件
SELECT e.Name, d.DepartmentName
FROM Employees e
INNER JOIN Departments d ON e.DepartmentId = d.Id;
```

### ✅ 預期結果：

| Name | DepartmentName |
| --- | --- |
| Alice | Sales |
| Bob | IT |
| Charlie | Sales |

🔍 **說明**：

- 每位員工會被正確對應到其所屬的部門。
- 避免了多餘配對造成資料重複或失真。

---

## 🧠 延伸說明與常見錯誤

- **錯誤類型**：
    - 忘記寫 `ON` 子句。
    - 寫錯 JOIN 條件，例如 `=` 寫成 `!=`。
    - 欄位名稱拼錯或使用錯誤欄位比對。
- **常見結果問題**：
    - 筆數暴增。
    - 一對一變成一對多或多對多。
    - 結果看似合理但實際錯誤（邏輯錯誤比語法錯誤更危險）。

---

## 🔍 執行計劃觀察（可選）

使用 `Include Actual Execution Plan` 功能查看 JOIN 行為：

- 錯誤寫法會顯示 Nested Loop without filter。
- 正確寫法會顯示使用 `Seek` 或適當 JOIN Operator。

---

## 💡 思考與變化題

1. 如果某些員工的 `DepartmentId` 為 NULL，INNER JOIN 結果會怎樣？
    
    > ➜ 不會出現在結果中，因為 INNER JOIN 只顯示匹配成功的記錄。
    > 
2. 改成 `LEFT JOIN` 會有什麼差異？
    
    > ➜ 可顯示所有員工，即使他們沒有部門（顯示 NULL）。
    > 

---

## 📝 加分練習題

```sql
-- 改寫成 LEFT JOIN，顯示所有員工（包含無部門者）
SELECT e.Name, d.DepartmentName
FROM Employees e
LEFT JOIN Departments d ON e.DepartmentId = d.Id;
```