## 📘 SQL Server 2022 資料型別總覽

SQL Server 提供多種內建資料型別，主要可分為以下幾類：

### 1. 整數與數值類型

- **整數類型**：`TINYINT`、`SMALLINT`、`INT`、`BIGINT`，根據儲存範圍選擇合適的類型。
- **精確數值類型**：`DECIMAL(p,s)`、`NUMERIC(p,s)`，適用於需要精確計算的場景，如財務數據。
- **近似數值類型**：`FLOAT`、`REAL`，用於科學計算，但可能有精度誤差。

### 2. 字元與字串類型

- **非 Unicode**：`CHAR(n)`、`VARCHAR(n)`，適用於英文等單字節語言。
- **Unicode**：`NCHAR(n)`、`NVARCHAR(n)`，支援多語言字符集，如中文，需注意其佔用空間為非 Unicode 的兩倍。

### 3. 日期與時間類型

- `DATE`、`TIME`、`DATETIME`、`SMALLDATETIME`、`DATETIME2`、`DATETIMEOFFSET`，根據所需的精度與時區資訊選擇合適的類型。

### 4. 二進位與大型物件類型

- `BINARY(n)`、`VARBINARY(n)`、`IMAGE`，用於儲存二進位數據，如圖片或檔案。需注意 `IMAGE` 已被標記為過時，建議使用 `VARBINARY(MAX)` 取代。

### 5. 其他特殊類型

- `UNIQUEIDENTIFIER`：儲存全域唯一識別碼（GUID）。
- `XML`：儲存 XML 格式的資料。
- `JSON`：雖無專屬資料型別，但可透過 `NVARCHAR` 搭配內建函數處理 JSON 資料。
- `HIERARCHYID`：表示階層結構的資料。
- `GEOGRAPHY`、`GEOMETRY`：用於地理與幾何資料的儲存與運算。

---

## ⚠️ 使用資料型別時的注意事項

### 1. 資料型別的選擇影響效能與儲存效率

選擇合適的資料型別不僅影響資料庫的儲存效率，還會影響查詢效能。例如，使用 `TINYINT` 儲存年齡比 `INT` 更節省空間，且在大量資料處理時效能更佳。

### 2. 隱含轉換可能導致效能下降

當查詢中涉及不同資料型別的欄位時，SQL Server 可能會進行隱含轉換，這可能導致索引無法被有效利用，影響查詢效能。因此，應盡量避免在查詢中混用不同的資料型別。

### 3. Unicode 與非 Unicode 的選擇

若應用需支援多語言，應使用 `NVARCHAR` 或 `NCHAR`；若僅處理英文資料，使用 `VARCHAR` 或 `CHAR` 可節省儲存空間。

### 4. 避免使用已過時的資料型別

如 `TEXT`、`NTEXT`、`IMAGE` 等資料型別已被標記為過時，建議改用 `VARCHAR(MAX)`、`NVARCHAR(MAX)`、`VARBINARY(MAX)` 等替代方案。

### 5. 精確控制數值的精度與範圍

使用 `DECIMAL(p,s)` 或 `NUMERIC(p,s)` 時，需根據實際需求設定適當的精度（precision）與小數位數（scale），以避免資料截斷或溢位。

---

## 🧠 實務建議

- **規劃資料表時，先確定每個欄位的資料型別**，以確保資料的正確性與一致性。
- **避免在查詢中進行不必要的資料型別轉換**，以提升查詢效能。
- **定期檢視與優化資料型別的使用**，特別是在資料量大或效能要求高的系統中。

---

如需更深入的學習，建議參考以下資源：

- [Microsoft Learn：資料類型 (Transact-SQL)](https://learn.microsoft.com/zh-tw/sql/t-sql/data-types/data-types-transact-sql?view=sql-server-ver17)
- [SQL Server 2022 資料型別介紹（YouTube）](https://www.youtube.com/watch?v=17L1H4GBqa0)

希望這份筆記能幫助你在學習 SQL Server 2022 的過程中，更加熟悉與掌握資料型別的使用。