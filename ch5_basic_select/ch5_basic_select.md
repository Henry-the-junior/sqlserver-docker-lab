# SELECT, JOIN, GROUP BY, UNION, SUBQUERY

以下是針對 SQL Server 2022 中的 SELECT、JOIN、GROUP BY、UNION 和 SUBQUERY 等查詢語法的學習筆記，幫助你建立清晰的概念與實作技巧。

---

## 📌 SELECT：查詢資料的起點

- **基本語法**：
    
    ```sql
    SELECT 欄位1, 欄位2
    FROM 資料表
    WHERE 條件
    ORDER BY 欄位 ASC|DESC;
    ```
    
- **範例**：查詢 `employees` 資料表中所有員工的姓名與部門：
    
    ```sql
    SELECT name, department
    FROM employees;
    ```
    

---

## 🔗 JOIN：合併多個資料表

JOIN 用於將多個資料表依據相關欄位連接起來，常見的類型包括：

- **INNER JOIN**：僅返回兩個資料表中符合連接條件的記錄。
- **LEFT JOIN**：返回左表的所有記錄，即使右表中沒有匹配的記錄。
- **RIGHT JOIN**：返回右表的所有記錄，即使左表中沒有匹配的記錄。
- **FULL OUTER JOIN**：返回兩個資料表中所有的記錄，無論是否有匹配。

**範例**：查詢每位員工及其所屬部門的名稱

```sql
SELECT e.name, d.department_name
FROM employees e
INNER JOIN departments d ON e.department_id = d.id;
```

---

## 📊 GROUP BY：分組與聚合分析

GROUP BY 用於將查詢結果依據一個或多個欄位分組，通常與聚合函數（如 COUNT、SUM、AVG）一起使用。

- **範例**：統計每個部門的員工人數
    
    ```sql
    SELECT department, COUNT(*) AS employee_count
    FROM employees
    GROUP BY department;
    ```
    
- **HAVING 子句**：用於篩選分組後的結果，類似於 WHERE，但作用於聚合後的資料。
    
    ```sql
    SELECT department, COUNT(*) AS employee_count
    FROM employees
    GROUP BY department
    HAVING COUNT(*) > 10;
    ```
    

---

## 🔁 UNION：合併多個查詢結果

UNION 用於合併多個 SELECT 查詢的結果集，要求各查詢的欄位數量與資料型別相同。

- **UNION**：自動去除重複的記錄。
- **UNION ALL**：保留所有記錄，包括重複的。

**範例**：合併兩個資料表中的員工姓名：

```sql
SELECT name FROM employees_2021
UNION
SELECT name FROM employees_2022;
```

---

## 🔍 SUBQUERY：巢狀查詢

SUBQUERY（子查詢）是嵌套在其他查詢中的 SELECT 語句，常用於 WHERE、FROM 或 SELECT 子句中。

- **範例 1**：查詢薪資高於平均薪資的員工
    
    ```sql
    SELECT name, salary
    FROM employees
    WHERE salary > (SELECT AVG(salary) FROM employees);
    ```
    
- **範例 2**：使用子查詢作為虛擬資料表
    
    ```sql
    SELECT department, avg_salary
    FROM (
      SELECT department, AVG(salary) AS avg_salary
      FROM employees
      GROUP BY department
    ) AS dept_avg
    WHERE avg_salary > 50000;
    ```