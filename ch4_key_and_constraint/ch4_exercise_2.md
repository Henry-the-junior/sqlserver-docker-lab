# PRIMARY KEY 和 UNIQUE 的差異觀察實驗

以下是針對「比較 `PRIMARY KEY` 和 `UNIQUE` 的差異」所設計的 SQL Server 實驗流程，涵蓋**建表、資料操作、錯誤觸發**與**觀察重點**，讓你清楚了解它們在**唯一性與 NULL 值處理**上的不同。

## 🧪 實驗目標

- 理解 `PRIMARY KEY` 和 `UNIQUE` 的**相同點**與**差異**
- 比較它們在處理 **重複值** 和 **NULL 值** 時的行為

## 🧱 建立資料表

```sql
-- 建立用來比較的資料表
CREATE TABLE ConstraintTest (
    ID INT PRIMARY KEY,                 -- 主鍵，不能重複、不能為 NULL
    Email NVARCHAR(100) UNIQUE,         -- 唯一鍵，不能重複，但可以有一個 NULL
    PhoneNumber NVARCHAR(20) UNIQUE
);
```

## 🔄 資料操作實驗

### 🧩 1. 插入合法資料

```sql
INSERT INTO ConstraintTest VALUES (1, 'a@example.com', '12345678');
INSERT INTO ConstraintTest VALUES (2, 'b@example.com', '87654321');
```

✔ 預期：成功，無違反任何約束。

---

### 🚫 2. 嘗試重複主鍵

```sql
INSERT INTO ConstraintTest VALUES (1, 'c@example.com', '99999999');
```

❌ 預期：失敗。`ID` 欄位為 `PRIMARY KEY`，不允許重複。

---

### 🚫 3. 嘗試插入 NULL 到主鍵

```sql
INSERT INTO ConstraintTest (ID, Email, PhoneNumber) VALUES (NULL, 'd@example.com', '11112222');
```

❌ 預期：失敗。`PRIMARY KEY` 欄位不允許為 NULL。

---

### 🚫 4. 嘗試重複 UNIQUE 值

```sql
INSERT INTO ConstraintTest VALUES (3, 'a@example.com', '77777777');
```

❌ 預期：失敗。`Email` 欄位設為 `UNIQUE`，不能重複。

---

### ✅ 5. 插入多個 NULL 到 UNIQUE 欄位

```sql
INSERT INTO ConstraintTest VALUES (4, NULL, NULL);
INSERT INTO ConstraintTest VALUES (5, NULL, NULL);
```

❌ 預期：第一筆會新增。`UNIQUE` 約束允許 `NULL`，但不允許多個 `NULL` 值。

## ✅ 總結對比表

| 特性 | PRIMARY KEY | UNIQUE |
| --- | --- | --- |
| 是否允許 NULL 值 | ❌ 不允許 | ✅ 允許一個 NULL |
| 是否允許重複 | ❌ 不允許 | ❌ 不允許 |
| 每張表能有幾個？ | 僅一個 | 僅一個 |
| 常用用途 | 資料列唯一識別 | 限制特定欄位不可重複 |

## 📌 補充建議

如果你想延伸這個實驗，可以試：

- 加入複合 `UNIQUE` 約束（例如 `(Email, PhoneNumber)`）
- 嘗試在 `UNIQUE` 欄位建立索引，觀察與 `PRIMARY KEY` 索引的差異