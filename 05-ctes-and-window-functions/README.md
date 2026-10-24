# CTE و توابع پنجره‌ای

## عبارت جدول مشترک (CTE)

CTE با `WITH` تعریف می‌شود و به یک پرس‌وجو نام موقت و خوانا می‌دهد. دامنهٔ آن همان دستور بعدی است:

```sql
WITH DepartmentTotals AS
(
    SELECT DepartmentId, COUNT(*) AS EmployeeCount
    FROM dbo.Employees
    GROUP BY DepartmentId
)
SELECT DepartmentId, EmployeeCount
FROM DepartmentTotals
WHERE EmployeeCount > 10;
```

CTE به خوانایی پرس‌وجوهای چندمرحله‌ای کمک می‌کند، اما به‌خودی‌خود تضمین نمی‌کند دادهٔ میانی ذخیره یا فقط یک‌بار محاسبه شود. برای کارایی، طرح اجرایی را بررسی کنید.

## توابع پنجره‌ای

توابع پنجره‌ای روی مجموعه‌ای از ردیف‌های مرتبط محاسبه انجام می‌دهند، بی‌آنکه هر گروه را به یک ردیف خلاصه کنند. `OVER` پنجره را تعریف می‌کند:

```sql
SELECT
    DepartmentId,
    EmployeeId,
    Salary,
    ROW_NUMBER() OVER
    (
        PARTITION BY DepartmentId
        ORDER BY Salary DESC, EmployeeId
    ) AS SalaryRank
FROM dbo.Employees;
```

- `PARTITION BY` گروه محاسبه را تعیین می‌کند.
- `ORDER BY` ترتیب درون هر گروه را مشخص می‌کند.
- `ROW_NUMBER`, `RANK` و `DENSE_RANK` برای شماره‌گذاری یا رتبه‌بندی به‌کار می‌روند.
- `SUM(...) OVER (...)` می‌تواند مجموع جاری یا مجموع یک گروه را کنار هر ردیف نشان دهد.

ترتیب پنجره باید برای خروجی قطعی کافی باشد؛ در صورت وجود مقدارهای برابر، یک ستون یکتا به ترتیب اضافه کنید.
