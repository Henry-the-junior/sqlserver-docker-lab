# Cascading Actions 觀察實驗

以下是專為學習 **SQL Server 中外鍵的級聯操作（Cascading Actions）** 所設計的一套完整實驗流程，包含：

- `ON DELETE CASCADE`
- `ON UPDATE SET NULL`
- `ON DELETE SET DEFAULT`

每一個情境都有明確的建表、操作、觀察點與預期結果，適合你逐步執行。

## 🧪 實驗目標

- 觀察不同外鍵約束下的「資料更新或刪除」行為
- 學會透過 `ON DELETE`、`ON UPDATE` 的行為來維持參照完整性

## 🧱 實驗環境前置步驟：DROP + CREATE

```sql
-- 先清除舊表（可重複執行）
IF OBJECT_ID('Children', 'U') IS NOT NULL DROP TABLE Children;
IF OBJECT_ID('Parents', 'U') IS NOT NULL DROP TABLE Parents;

-- 建立主表
CREATE TABLE Parents (
    ParentID INT PRIMARY KEY,
    Name NVARCHAR(50)
);

-- 插入主表資料
INSERT INTO Parents VALUES (1, 'Alpha'), (2, 'Beta'), (3, 'Gamma');
```

## ✅ 實驗 1：ON DELETE CASCADE

### 建立從表與外鍵

```sql
CREATE TABLE Children (
    ChildID INT PRIMARY KEY,
    ParentID INT,
    Info NVARCHAR(50),
    CONSTRAINT FK_CASCADE FOREIGN KEY (ParentID)
        REFERENCES Parents(ParentID)
        ON DELETE CASCADE
);
```

### 插入資料並刪除主表資料

```sql
-- 插入從表資料
INSERT INTO Children VALUES (101, 1, 'C1-A'), (102, 1, 'C1-B');

-- 刪除主表中 ID = 1 的資料
DELETE FROM Parents WHERE ParentID = 1;
```

### 預期結果：

- Parents 表中 ID = 1 被刪除
- Children 表中所有 ParentID = 1 的資料也**一併被刪除**

## ✅ 實驗 2：ON UPDATE SET NULL

### 清除從表並重新建立

```sql
DROP TABLE Children;

-- 建立允許 NULL 的外鍵欄位，且設定 ON UPDATE SET NULL
CREATE TABLE Children (
    ChildID INT PRIMARY KEY,
    ParentID INT NULL,
    Info NVARCHAR(50),
    CONSTRAINT FK_UPDATE_NULL FOREIGN KEY (ParentID)
        REFERENCES Parents(ParentID)
        ON UPDATE SET NULL
);
```

### 插入資料並更新主表主鍵

```sql
-- 插入資料
INSERT INTO Children VALUES (201, 2, 'C2-A');

-- 更新 Parent 主鍵（會觸發外鍵 ON UPDATE）
UPDATE Parents SET ParentID = 20 WHERE ParentID = 2;
```

### 預期結果：

- Parents 表中原本 ID = 2 的資料改為 20
- Children 表中原本為 ParentID = 2 的欄位變為 `NULL`

## ✅ 實驗 3：ON DELETE SET DEFAULT

### 清除從表並建立預設值設定

```sql
DROP TABLE Children;

-- 建立欄位預設值為 99
CREATE TABLE Children (
    ChildID INT PRIMARY KEY,
    ParentID INT DEFAULT 99,
    Info NVARCHAR(50),
    CONSTRAINT FK_DELETE_DEFAULT FOREIGN KEY (ParentID)
        REFERENCES Parents(ParentID)
        ON DELETE SET DEFAULT
);

-- 插入資料
INSERT INTO Parents VALUES (99, 'Default');
INSERT INTO Children VALUES (301, 3, 'C3-A');
```

### 刪除主表資料

```sql
-- 刪除 Parent 中 ID = 3 的資料
DELETE FROM Parents WHERE ParentID = 3;
```

### 預期結果：

- Parents 表中 ID = 3 的資料被刪除
- Children 表中原本 ParentID = 3 的欄位變為預設值 99

## 🎯 總結觀察點

| 級聯方式 | 說明 | 子表資料的變化 |
| --- | --- | --- |
| `ON DELETE CASCADE` | 主表刪除 → 子表自動刪除 | 子表資料也被刪除 |
| `ON UPDATE SET NULL` | 主表更新主鍵 → 子表欄位設為 NULL | 外鍵欄位變 NULL |
| `ON DELETE SET DEFAULT` | 主表刪除 → 子表欄位設為預設值 | 外鍵欄位變 99（或你指定的 DEFAULT 值） |

當然可以！這段語法是 SQL Server 中常用的\*\*「安全地刪除資料表」\*\*語法，讓我們逐一解釋：

## 補充：OBJECT_ID 語法

### 📌 常見物件類型參數對照（第二個參數）

| 類型碼    | 說明                           |
| ------ | ---------------------------- |
| `'U'`  | User table（使用者資料表）           |
| `'V'`  | View（檢視表）                    |
| `'P'`  | Stored Procedure（預存程序）       |
| `'FN'` | Scalar Function（純量函數）        |
| `'IF'` | Inline Table-Valued Function |
| `'TF'` | Table-Valued Function        |

### 🔁 延伸寫法（也很常見）

你也可能看到這種簡寫（少了類型參數，SQL Server 會自行判斷）：

```sql
IF OBJECT_ID('Parent') IS NOT NULL DROP TABLE Parent;
```

但為了更保險，**建議加上類型參數 `'U'`**，以避免名稱重複時刪錯物件（例如刪到同名的預存程序）。
