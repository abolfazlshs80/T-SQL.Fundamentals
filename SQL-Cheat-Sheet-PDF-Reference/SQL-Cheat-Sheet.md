# SQL Cheat Sheet

**SQL Server / T-SQL edition** · A practical quick reference based on the supplied 9-page SQL cheat sheet.

The source covers DML, DDL, DCL, query clauses, joins, subqueries, aggregates, string and date functions, conditional expressions, set operators, and transaction control. Its examples mix SQL dialects. This edition uses executable T-SQL where possible and labels SQL Server-specific corrections. **Additional SQL Server note** callouts are clarifications added for safety, behavior, or version compatibility; they are not claims about the source PDF.

## Contents

1. [DML: read and change rows](#1-dml-read-and-change-rows)
2. [DDL: define and change objects](#2-ddl-define-and-change-objects)
3. [DCL: grant and revoke permissions](#3-dcl-grant-and-revoke-permissions)
4. [Querying: filters, grouping, and sorting](#4-querying-filters-grouping-and-sorting)
5. [Joins](#5-joins)
6. [Subqueries](#6-subqueries)
7. [Aggregate functions](#7-aggregate-functions)
8. [String functions](#8-string-functions)
9. [Date and time](#9-date-and-time)
10. [Conditional and NULL-handling expressions](#10-conditional-and-null-handling-expressions)
11. [Set operators](#11-set-operators)
12. [Transactions and savepoints](#12-transactions-and-savepoints)
13. [Source-example corrections at a glance](#13-source-example-corrections-at-a-glance)

## 1. DML: read and change rows

**Purpose:** Retrieve, add, modify, or remove table rows.

```sql
-- Read selected columns
SELECT EmployeeId, FirstName, LastName
FROM dbo.Employees;

-- Add a row
INSERT INTO dbo.Employees (EmployeeId, FirstName, LastName, DepartmentId)
VALUES (101, N'Mary', N'Doe', 4);

-- Modify matching rows
UPDATE dbo.Employees
SET DepartmentId = 5
WHERE EmployeeId = 101;

-- Remove matching rows
DELETE FROM dbo.Employees
WHERE EmployeeId = 101;
```

| Command | Effect | Important distinction |
|---|---|---|
| `SELECT` | Returns rows | Name the columns you need instead of relying on `*`. |
| `INSERT` | Adds rows | Specify the target columns to make the value mapping clear. |
| `UPDATE` | Changes existing rows | Without `WHERE`, every row is updated. |
| `DELETE` | Removes rows | Without `WHERE`, every row is deleted; the table remains. |

**Warning:** Check the `WHERE` condition before running `UPDATE` or `DELETE`. Use `N'...'` for Unicode text literals.

## 2. DDL: define and change objects

**Purpose:** Create, alter, or remove database objects and table structure.

```sql
CREATE TABLE dbo.Employees
(
    EmployeeId   int           NOT NULL,
    FirstName    nvarchar(50)  NOT NULL,
    LastName     nvarchar(50)  NOT NULL,
    DepartmentId int           NULL,
    CONSTRAINT PK_Employees PRIMARY KEY (EmployeeId)
);

ALTER TABLE dbo.Employees
ADD Email nvarchar(254) NULL;

-- Removes rows but retains the table definition
TRUNCATE TABLE dbo.Employees;

-- Removes the table definition and its data
DROP TABLE dbo.Employees;
```

| Command | What it does | Use with care |
|---|---|---|
| `CREATE` | Creates a database object, such as a table, index, view, or stored procedure. | A table definition must use valid SQL Server data types and constraints. |
| `ALTER` | Changes an existing object's definition. | Existing data can prevent some schema changes. |
| `TRUNCATE TABLE` | Removes all rows while keeping the table. | No `WHERE`; SQL Server has restrictions involving foreign keys and other table features. |
| `DROP TABLE` | Removes the table and its definition. | Destructive; dependent objects may be affected. |

**Additional SQL Server note:** `TRUNCATE TABLE` is not a row-by-row substitute for `DELETE`; it cannot filter rows. In SQL Server, it can participate in a transaction and be rolled back before that transaction is committed.

## 3. DCL: grant and revoke permissions

**Purpose:** Give or remove database permissions for a user or role.

```sql
GRANT SELECT, INSERT
ON OBJECT::dbo.Employees
TO [EmployeeAppRole];

REVOKE INSERT
ON OBJECT::dbo.Employees
FROM [EmployeeAppRole];
```

**Additional SQL Server note:** Grant permissions to database roles where practical; the principal (for example, `[EmployeeAppRole]`) must exist in the database. `REVOKE` removes a grant or deny from that principal, but other grants or role memberships can still provide effective access.

## 4. Querying: filters, grouping, and sorting

**Purpose:** Filter rows, summarize them into groups, filter summaries, and order the returned rows.

```sql
SELECT DepartmentId, COUNT(*) AS EmployeeCount
FROM dbo.Employees
WHERE EmployeeId >= 100
GROUP BY DepartmentId
HAVING COUNT(*) > 5
ORDER BY EmployeeCount DESC, DepartmentId;
```

```sql
-- Filter and sort detail rows
SELECT ProductId, ProductName, Price
FROM dbo.Products
WHERE Price >= 10 AND Price < 100
ORDER BY Price DESC;
```

| Clause | Purpose | Typical use |
|---|---|---|
| `WHERE` | Filters individual rows before grouping. | `WHERE Price >= 10` |
| `GROUP BY` | Forms groups for aggregate calculations. | `GROUP BY DepartmentId` |
| `HAVING` | Filters groups after aggregation. | `HAVING COUNT(*) > 5` |
| `ORDER BY` | Sorts the final result. | `ORDER BY Price DESC` |

**Additional SQL Server note:** In a grouped query, each selected expression must either be grouped or aggregated. Put row-level conditions in `WHERE` and aggregate conditions in `HAVING`. `ASC` is ascending (the default); `DESC` is descending. Only `ORDER BY` guarantees presentation order.

## 5. Joins

**Purpose:** Combine rows from related tables or from a table and itself.

```sql
SELECT e.EmployeeId, e.FirstName, d.DepartmentName
FROM dbo.Employees AS e
INNER JOIN dbo.Departments AS d
    ON d.DepartmentId = e.DepartmentId;

-- Keep every employee, including employees without a matching department
SELECT e.EmployeeId, e.FirstName, d.DepartmentName
FROM dbo.Employees AS e
LEFT JOIN dbo.Departments AS d
    ON d.DepartmentId = e.DepartmentId;
```

| Join | Rows returned |
|---|---|
| `INNER JOIN` | Matching rows from both inputs. |
| `LEFT [OUTER] JOIN` | Every left row and any matching right row; unmatched right-side columns are `NULL`. |
| `RIGHT [OUTER] JOIN` | Every right row and any matching left row. |
| `FULL [OUTER] JOIN` | Every row from both inputs; unmatched columns on the other side are `NULL`. |
| `CROSS JOIN` | Every left/right combination (Cartesian product). |
| Self join | A table joined to itself using separate aliases and an explicit relationship. |

```sql
-- A self join: pair each employee with their manager
SELECT e.EmployeeId, e.FirstName,
       m.EmployeeId AS ManagerId, m.FirstName AS ManagerFirstName
FROM dbo.Employees AS e
LEFT JOIN dbo.Employees AS m
    ON m.EmployeeId = e.ManagerId;

-- SQL Server supports FULL OUTER JOIN directly
SELECT e.EmployeeId, d.DepartmentId
FROM dbo.Employees AS e
FULL OUTER JOIN dbo.Departments AS d
    ON d.DepartmentId = e.DepartmentId;
```

**SQL Server compatibility:** `NATURAL JOIN` is not supported in T-SQL. Write the intended `JOIN ... ON` condition explicitly.

**Additional SQL Server note:** A `CROSS JOIN` can grow rapidly: its row count is the product of the input row counts.

## 6. Subqueries

**Purpose:** Use a query result as input to a condition in another query.

```sql
-- IN: match a value against the rows returned by a subquery
SELECT c.CustomerId, c.City
FROM dbo.Customers AS c
WHERE c.City IN
(
    SELECT s.City
    FROM dbo.Suppliers AS s
);

-- ANY: true if the comparison is true for at least one returned value
SELECT p.ProductId, p.Price
FROM dbo.Products AS p
WHERE p.Price < ANY
(
    SELECT sp.UnitPrice
    FROM dbo.SupplierProducts AS sp
);

-- ALL: true if the comparison is true for every returned value
SELECT o.OrderId, o.TotalAmount
FROM dbo.Orders AS o
WHERE o.TotalAmount > ALL
(
    SELECT po.TotalAmount
    FROM dbo.PreviousOrders AS po
);
```

- `IN` tests membership in the subquery result.
- `ANY` (also called `SOME` in SQL Server) compares against at least one value; `ALL` compares against every value.
- The comparison operator matters: `> ANY` means greater than at least one value; `> ALL` means greater than every value.
**Additional SQL Server note:** `NULL` values in a subquery can make `IN`/`NOT IN` comparisons evaluate to `UNKNOWN`. For anti-matching logic, consider `NOT EXISTS` and an explicit correlation instead.

## 7. Aggregate functions

**Purpose:** Calculate a value over a set of rows, or over each `GROUP BY` group.

```sql
SELECT
    COUNT(*)            AS RowCount,
    COUNT(Price)        AS NonNullPriceCount,
    SUM(Price)          AS TotalPrice,
    AVG(Price)          AS AveragePrice,
    MIN(Price)          AS LowestPrice,
    MAX(Price)          AS HighestPrice
FROM dbo.Products;
```

| Function | Result |
|---|---|
| `COUNT(*)` | Number of rows. |
| `COUNT(expression)` | Number of non-`NULL` expression values. |
| `SUM(expression)` | Sum of non-`NULL` values. |
| `AVG(expression)` | Average of non-`NULL` values. |
| `MIN(expression)` / `MAX(expression)` | Lowest / highest value. |

Aggregate functions can be combined with `GROUP BY` to return one summary per group. `WHERE` filters input rows; `HAVING` filters the resulting groups.

## 8. String functions

**Purpose:** Combine, extract, measure, trim, change case, or replace text.

```sql
SELECT
    CONCAT(FirstName, N' ', LastName) AS FullName,
    SUBSTRING(ProductName, 1, 5)      AS FirstFiveCharacters,
    LEN(ProductName)                  AS CharacterCount,
    UPPER(ProductName)                AS UpperName,
    LOWER(ProductName)                AS LowerName,
    TRIM(ProductName)                 AS TrimmedName,
    LEFT(ProductName, 3)              AS Prefix,
    RIGHT(ProductName, 4)             AS Suffix,
    REPLACE(ProductName, N'Old', N'New') AS ReplacedName
FROM dbo.Products;
```

| Function | T-SQL behavior |
|---|---|
| `CONCAT(a, b, ...)` | Concatenates values; `NULL` arguments are treated as empty strings. |
| `SUBSTRING(text, start, length)` | Extracts `length` characters starting at the 1-based position `start`. |
| `LEN(text)` | Counts characters, excluding trailing spaces. |
| `UPPER` / `LOWER` | Converts text to upper / lower case, subject to collation. |
| `TRIM(text)` | Removes leading and trailing spaces by default. |
| `LEFT` / `RIGHT` | Returns characters from the start / end of a string. |
| `REPLACE(text, old, new)` | Replaces occurrences of `old` with `new`. |

**T-SQL compatibility:** The source lists `SUBSTR`, `CHAR_LENGTH`, and `LENGTH` as alternatives. Use `SUBSTRING` and `LEN` in SQL Server.

**Additional SQL Server note:** `LEN` excludes trailing spaces; `DATALENGTH` measures bytes, not characters. Basic `TRIM` is available starting with SQL Server 2017. Positional forms such as `TRIM(LEADING ... FROM ...)` require SQL Server 2022 and database compatibility level 160; use `LTRIM`/`RTRIM` when targeting older versions.

## 9. Date and time

**Purpose:** Get the current date/time, extract a date part, shift a date, or calculate a date difference.

```sql
SELECT
    CAST(GETDATE() AS date)                  AS CurrentDate,
    CAST(SYSDATETIME() AS time)              AS CurrentTime,
    CURRENT_TIMESTAMP                       AS CurrentTimestamp,
    DATEPART(year, OrderDate)                AS OrderYear,
    DATEADD(day, 7, OrderDate)               AS OneWeekLater,
    DATEDIFF(day, OrderDate, ShipDate)        AS DayBoundariesToShip,
    CONVERT(char(10), OrderDate, 23)          AS ISODateText
FROM dbo.Orders;
```

| Source concept | T-SQL equivalent |
|---|---|
| `CURRENT_DATE()` | `CAST(GETDATE() AS date)` |
| `CURRENT_TIME()` | `CAST(SYSDATETIME() AS time)` |
| `CURRENT_TIMESTAMP()` | `CURRENT_TIMESTAMP` (no parentheses); `SYSDATETIME()` provides higher precision. |
| `DATE_PART` / `EXTRACT` | `DATEPART(datepart, date_expression)` |
| `DATE_ADD` / `DATE_SUB` | `DATEADD(datepart, signed_number, date_expression)`; use a negative number to subtract. |
| `TIMESTAMPDIFF` / `DATEDIFF` | `DATEDIFF(datepart, startdate, enddate)` |
| `TO_CHAR` | `CONVERT` with a style for common formats; `FORMAT` for culture-aware formatting. |

**Additional SQL Server note:** `DATEDIFF` counts date-part boundaries crossed, not exact elapsed duration; its argument order is start date, then end date. `FORMAT` is convenient for display but can be slower than `CONVERT` for large result sets.

## 10. Conditional and NULL-handling expressions

**Purpose:** Return values conditionally and handle missing or equal values.

```sql
SELECT
    OrderId,
    CASE
        WHEN TotalAmount > 1000 THEN N'High'
        WHEN TotalAmount > 500  THEN N'Medium'
        ELSE N'Low'
    END AS OrderCategory,
    COALESCE(DiscountedAmount, TotalAmount, 0) AS AmountToShow,
    NULLIF(TotalAmount, DiscountedAmount) AS DifferenceOrNull
FROM dbo.Orders;
```

| Expression | Behavior |
|---|---|
| `CASE WHEN ... THEN ... ELSE ... END` | Returns the result for the first true condition; supports multiple conditions. |
| `COALESCE(a, b, ...)` | Returns the first non-`NULL` expression. |
| `NULLIF(a, b)` | Returns `NULL` when `a = b`; otherwise returns `a`. |
| `IF(condition, a, b)` | **Not T-SQL syntax.** Use `CASE`; `IIF(condition, a, b)` is a T-SQL alternative. |

**Additional SQL Server note:** `IIF` is a shorthand for `CASE`; `CASE` is more portable and handles multiple branches clearly. Use `IS NULL` / `IS NOT NULL` to test for `NULL`—not `= NULL`.

## 11. Set operators

**Purpose:** Combine or compare the result sets of compatible `SELECT` statements.

```sql
-- All customers and employees; remove duplicate rows
SELECT FirstName, LastName FROM dbo.Customers
UNION
SELECT FirstName, LastName FROM dbo.Employees;

-- Keep duplicates
SELECT FirstName, LastName FROM dbo.Customers
UNION ALL
SELECT FirstName, LastName FROM dbo.Employees;

-- Rows common to both inputs
SELECT FirstName, LastName FROM dbo.Customers
INTERSECT
SELECT FirstName, LastName FROM dbo.Employees;

-- Rows in the first input but not the second
SELECT FirstName, LastName FROM dbo.Customers
EXCEPT
SELECT FirstName, LastName FROM dbo.Employees;
```

**Additional SQL Server note:** Each input must return the same number of columns in the same order, with compatible data types. `UNION`, `INTERSECT`, and `EXCEPT` return distinct rows; `UNION ALL` preserves duplicates and avoids duplicate elimination. Row order is not guaranteed unless the combined query has a final `ORDER BY`.

## 12. Transactions and savepoints

**Purpose:** Treat multiple changes as a unit: commit them together or roll them back.

```sql
BEGIN TRANSACTION;

UPDATE dbo.Products
SET Price = Price * 1.05
WHERE Category = N'Electronics';

COMMIT TRANSACTION;
```

```sql
BEGIN TRANSACTION;

UPDATE dbo.Products
SET Price = Price * 1.05
WHERE Category = N'Electronics';

-- Undo the transaction's uncommitted changes
ROLLBACK TRANSACTION;
```

| Command | Purpose |
|---|---|
| `BEGIN TRANSACTION` | Starts an explicit transaction. |
| `COMMIT TRANSACTION` | Makes the transaction's changes permanent. |
| `ROLLBACK TRANSACTION` | Undoes uncommitted work. |
| `SAVE TRANSACTION name` | Creates a savepoint inside the transaction. |
| `ROLLBACK TRANSACTION name` | Rolls back to a named savepoint, not to the start of the transaction. |
| `SET TRANSACTION ISOLATION LEVEL ...` | Sets the session's isolation level for subsequent transactions. |

```sql
BEGIN TRANSACTION;

UPDATE dbo.Products
SET Price = Price * 1.05
WHERE Category = N'Electronics';

SAVE TRANSACTION BeforeDelete;

DELETE FROM dbo.Customers
WHERE CustomerId = -1; -- Example condition; replace with the intended predicate.

ROLLBACK TRANSACTION BeforeDelete;
COMMIT TRANSACTION;

SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
```

**T-SQL compatibility:** SQL Server uses `SAVE TRANSACTION savepoint_name` and `ROLLBACK TRANSACTION savepoint_name`; `SAVEPOINT` and `ROLLBACK TO SAVEPOINT` from other dialects are not the SQL Server syntax. SQL Server does not provide a separate `RELEASE SAVEPOINT` command.

**Additional SQL Server note:** Isolation levels include `READ UNCOMMITTED`, `READ COMMITTED`, `REPEATABLE READ`, `SERIALIZABLE`, and `SNAPSHOT` (subject to database configuration). Set the level intentionally; it affects concurrency and what a transaction can observe.

## 13. Source-example corrections at a glance

| Source issue | Correct T-SQL guidance |
|---|---|
| The source uses date functions such as `DATE_ADD`, `DATE_PART`, `EXTRACT`, and `TO_CHAR`. | Use `DATEADD`, `DATEPART`, `CONVERT` (or `FORMAT` where appropriate). |
| The source uses `IF()` and `CURRENT_DATE()` / `CURRENT_TIME()`. | Use `CASE`/`IIF`; cast `GETDATE()` or `SYSDATETIME()` to `date`/`time`. |
| The source uses `NATURAL JOIN`. | SQL Server does not support it; use an explicit `JOIN ... ON`. |
| The source emulates `FULL JOIN` with `LEFT JOIN UNION RIGHT JOIN`. | Use `FULL OUTER JOIN`; the union workaround can mishandle duplicates. |
| The source's self-join example compares a row to itself. | Use aliases and a meaningful relationship, such as `Employee.ManagerId = Manager.EmployeeId`. |
| One source `UPDATE` example omits `WHERE`. | Add a precise predicate unless every row is intentionally being changed. |
| The source uses non-T-SQL savepoint commands. | Use `SAVE TRANSACTION` and `ROLLBACK TRANSACTION savepoint_name`. |
| The source presents several generic string and date signatures. | Follow T-SQL signatures and version requirements documented in the relevant sections above. |

---

**Scope:** This reference follows the topics in the supplied PDF. SQL Server equivalents and compatibility notes are identified where the source uses another SQL dialect or an example needs correction; unrelated SQL Server topics are intentionally omitted.
