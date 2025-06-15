# UNION 與 UNION ALL 的差異與重複資料處理行為

## 🎯 實驗目的

1. 觀察 `UNION` 自動去除重複資料的行為。
2. 理解 `UNION ALL` 如何保留所有結果。
3. 學會根據需求選擇適合的用法。

---

## 🧰 實驗前置：建立兩個測試資料表

```sql
CREATE TABLE Customers2023 (
    Name VARCHAR(50)
);

CREATE TABLE Customers2024 (
    Name VARCHAR(50)
);

INSERT INTO Customers2023 (Name)
VALUES ('Alice'), ('Bob'), ('Charlie');

INSERT INTO Customers2024 (Name)
VALUES ('Bob'), ('Diana'), ('Alice');
```

🔍 資料總覽：

- `Alice` 和 `Bob` 出現在兩年中。
- `Charlie` 只在 2023。
- `Diana` 只在 2024。

---

## ✅ 實驗一：使用 UNION（自動去重）

```sql
SELECT Name FROM Customers2023
UNION
SELECT Name FROM Customers2024;
```

### ✅ 預期結果：

| Name |
| --- |
| Alice |
| Bob |
| Charlie |
| Diana |

🔍 **行為說明**：

- `UNION` 自動進行 **DISTINCT 去重**。
- 不會顯示重複的 `Alice` 和 `Bob`。

---

## ✅ 實驗二：使用 UNION ALL（保留所有資料）

```sql
SELECT Name FROM Customers2023
UNION ALL
SELECT Name FROM Customers2024;
```

### ✅ 預期結果：

| Name |
| --- |
| Alice |
| Bob |
| Charlie |
| Bob |
| Diana |
| Alice |

🔍 **行為說明**：

- `UNION ALL` **不去重**，直接合併兩個結果集。
- 出現 2 次 `Alice`、2 次 `Bob`。

---

## 📊 比較分析：UNION vs UNION ALL

| 特性 | UNION | UNION ALL |
| --- | --- | --- |
| 重複資料處理 | 自動去除重複 | 保留所有資料 |
| 效能 | 較慢（需進行 DISTINCT） | 較快（直接合併） |
| 使用場景 | 需要唯一值 | 需要完整資料（如報表統計） |

---

## 💡 使用情境建議

| 情境描述 | 建議用法 |
| --- | --- |
| 合併會員名單（不允許重複） | `UNION` |
| 整合訂單紀錄（允許重複，如統計數量） | `UNION ALL` |
| 原始資料分析，需保留來源重複紀錄 | `UNION ALL` |
| 合併查詢報告，僅顯示不同來源的唯一資料 | `UNION` |

---

## 🧠 延伸挑戰

- 加入欄位來源識別（例如加上來源年份）觀察行為：
    
    ```sql
    SELECT Name, '2023' AS Year FROM Customers2023
    UNION
    SELECT Name, '2024' FROM Customers2024;
    ```

---

## 📝 小結

- 用 `UNION` 時請確認是否**真的需要去重**，以免不必要地耗費資源。
- 若資料重複是預期的，請選擇 `UNION ALL`，它的效能會更好。