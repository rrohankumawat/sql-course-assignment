# SQL Server Practice Assignment
### Covers: Table Design, Constraints, Joins, Aggregate/Scalar Functions, Stored Procedures

---

## Setup Schema

Give students this schema to work with (or have them create it as Q1's warm-up):

```sql
CREATE TABLE Departments (
    DeptID INT PRIMARY KEY IDENTITY(1,1),
    DeptName VARCHAR(50) NOT NULL
);

CREATE TABLE Employees (
    EmpID INT PRIMARY KEY IDENTITY(1,1),
    EmpName VARCHAR(50) NOT NULL,
    DeptID INT FOREIGN KEY REFERENCES Departments(DeptID),
    Salary DECIMAL(10,2),
    HireDate DATE,
    Email VARCHAR(100) UNIQUE
);

INSERT INTO Departments (DeptName) VALUES
('IT'), ('HR'), ('Sales'), ('Finance');

INSERT INTO Employees (EmpName, DeptID, Salary, HireDate, Email) VALUES
('Amit Sharma', 1, 55000, '2021-03-15', 'amit@company.com'),
('Priya Verma', 2, 42000, '2020-07-01', 'priya@company.com'),
('Rahul Singh', 1, 61000, '2019-01-10', 'rahul@company.com'),
('Sneha Gupta', 3, 38000, '2022-05-20', 'sneha@company.com'),
('Vikram Rao', 3, 45000, '2021-11-11', 'vikram@company.com'),
('Neha Joshi', NULL, 30000, '2023-02-01', 'neha@company.com');
```

---

## Question 1 — DDL & Constraints (Basics)
Create the two tables above (`Departments`, `Employees`) with appropriate primary key, foreign key, `NOT NULL`, and `UNIQUE` constraints. Then insert the sample data.

**Answer:**
The `CREATE TABLE` and `INSERT` statements above are the expected answer. Key points to check:
- `DeptID` as PRIMARY KEY with IDENTITY
- `EmpID` as PRIMARY KEY with IDENTITY
- `DeptID` in Employees as FOREIGN KEY referencing Departments
- `Email` marked UNIQUE
- `EmpName` marked NOT NULL

---

## Question 2 — Joins
Write a query to display employee name, department name, and salary for all employees, **including those not assigned to any department** (like Neha Joshi).

**Answer:**
```sql
SELECT E.EmpName, D.DeptName, E.Salary
FROM Employees E
LEFT JOIN Departments D ON E.DeptID = D.DeptID;
```
*Teaching point: LEFT JOIN is required here because an INNER JOIN would drop Neha Joshi (DeptID is NULL).*

---

## Question 3 — Aggregate Functions & GROUP BY
Find the department-wise average salary, but only for departments having more than 1 employee. Also show the department with the highest average salary.

**Answer:**
```sql
-- Department-wise average salary with more than 1 employee
SELECT D.DeptName, AVG(E.Salary) AS AvgSalary, COUNT(*) AS EmpCount
FROM Employees E
JOIN Departments D ON E.DeptID = D.DeptID
GROUP BY D.DeptName
HAVING COUNT(*) > 1;

-- Department with highest average salary
SELECT TOP 1 D.DeptName, AVG(E.Salary) AS AvgSalary
FROM Employees E
JOIN Departments D ON E.DeptID = D.DeptID
GROUP BY D.DeptName
ORDER BY AvgSalary DESC;
```
*Teaching point: Tests GROUP BY + HAVING (filter on aggregate) vs WHERE (filter on rows), plus TOP with ORDER BY.*

---

## Question 4 — Scalar / String / Date Functions
Write a query that returns:
- Employee name in UPPERCASE
- Email domain only (text after `@`)
- Number of years the employee has worked (as of today)
- A column `Tenure` that shows `'Senior'` if years worked > 3, else `'Junior'` (use `CASE`)

**Answer:**
```sql
SELECT 
    UPPER(EmpName) AS EmpNameUpper,
    SUBSTRING(Email, CHARINDEX('@', Email) + 1, LEN(Email)) AS EmailDomain,
    DATEDIFF(YEAR, HireDate, GETDATE()) AS YearsWorked,
    CASE 
        WHEN DATEDIFF(YEAR, HireDate, GETDATE()) > 3 THEN 'Senior'
        ELSE 'Junior'
    END AS Tenure
FROM Employees;
```
*Teaching point: Combines string functions (UPPER, SUBSTRING, CHARINDEX, LEN), a date function (DATEDIFF), and CASE logic — ties together everything from the functions module.*

---

## Question 5 — Stored Procedure (Capstone)
Create a stored procedure `usp_GetEmployeesByDept` that:
1. Accepts a `@DeptName` parameter
2. Returns all employees in that department, ordered by salary descending
3. If the department doesn't exist or has no employees, it should print a message `'No employees found for this department'` instead of returning an empty result silently
4. Also include an optional `@MinSalary` parameter (default `0`) to filter employees with salary above that value

**Answer:**
```sql
CREATE PROCEDURE usp_GetEmployeesByDept
    @DeptName VARCHAR(50),
    @MinSalary DECIMAL(10,2) = 0
AS
BEGIN
    SET NOCOUNT ON;

    IF NOT EXISTS (
        SELECT 1 
        FROM Employees E
        JOIN Departments D ON E.DeptID = D.DeptID
        WHERE D.DeptName = @DeptName AND E.Salary > @MinSalary
    )
    BEGIN
        PRINT 'No employees found for this department';
        RETURN;
    END

    SELECT E.EmpName, E.Salary, E.HireDate
    FROM Employees E
    JOIN Departments D ON E.DeptID = D.DeptID
    WHERE D.DeptName = @DeptName AND E.Salary > @MinSalary
    ORDER BY E.Salary DESC;
END;
```
**Execution examples:**
```sql
EXEC usp_GetEmployeesByDept @DeptName = 'IT';
EXEC usp_GetEmployeesByDept @DeptName = 'Sales', @MinSalary = 40000;
EXEC usp_GetEmployeesByDept @DeptName = 'Legal';  -- triggers the "not found" message
```
*Teaching point: Parameters with defaults, conditional logic (IF NOT EXISTS), PRINT for feedback, RETURN to exit early, and dynamic filtering — a good capstone tying together everything covered.*

---

## Suggested Marking Scheme (out of 25)

| Question | Marks | Focus |
|---|---|---|
| Q1 | 3 | DDL, constraints |
| Q2 | 4 | Joins (esp. LEFT JOIN) |
| Q3 | 5 | GROUP BY, HAVING, aggregates |
| Q4 | 5 | Scalar functions, CASE |
| Q5 | 8 | Stored procedures, parameters, control flow |



# SQL Server Practice Assignment — Set 2
### Focus: Aggregate Functions, Ranking Functions, Date/Time Functions

Uses the same `Departments` and `Employees` tables from Set 1. Run this extra data first so ranking questions have enough rows to be meaningful (includes salary ties on purpose):

```sql
INSERT INTO Employees (EmpName, DeptID, Salary, HireDate, Email) VALUES
('Karan Mehta', 1, 55000, '2020-09-01', 'karan@company.com'),
('Divya Nair', 2, 42000, '2021-04-18', 'divya@company.com'),
('Arjun Kapoor', 3, 45000, '2018-12-05', 'arjun@company.com'),
('Isha Malhotra', 4, 70000, '2017-06-30', 'isha@company.com'),
('Rohan Das', 4, 68000, '2022-01-15', 'rohan@company.com');
```

---

## Question 1 — Aggregate Functions
Write a single query that returns, across the entire company:
- Total number of employees
- Total salary payout
- Highest and lowest salary
- Average salary rounded to 2 decimal places
- Number of employees who have **no department assigned**

**Answer:**
```sql
SELECT 
    COUNT(*) AS TotalEmployees,
    SUM(Salary) AS TotalPayout,
    MAX(Salary) AS HighestSalary,
    MIN(Salary) AS LowestSalary,
    ROUND(AVG(Salary), 2) AS AvgSalary,
    SUM(CASE WHEN DeptID IS NULL THEN 1 ELSE 0 END) AS UnassignedCount
FROM Employees;
```
*Teaching point: Multiple aggregates in one SELECT, plus a conditional COUNT pattern using SUM(CASE...) since COUNT can't directly filter on NULL logic like this.*

---

## Question 2 — RANK vs DENSE_RANK vs ROW_NUMBER
Rank all employees by salary (highest first), showing all three ranking styles side by side, so students can see how they differ when there are salary ties.

**Answer:**
```sql
SELECT 
    EmpName,
    Salary,
    ROW_NUMBER() OVER (ORDER BY Salary DESC) AS RowNum,
    RANK() OVER (ORDER BY Salary DESC) AS RankNum,
    DENSE_RANK() OVER (ORDER BY Salary DESC) AS DenseRankNum
FROM Employees
ORDER BY Salary DESC;
```
*Teaching point: With the tie at salary 55000 (Amit & Karan), ROW_NUMBER gives them different numbers, RANK gives them the same number but skips the next one, DENSE_RANK gives them the same number without skipping. This is the single most common ranking-function interview question.*

---

## Question 3 — PARTITION BY (Ranking within Groups)
Find the **top 2 highest paid employees in each department** using a window function.

**Answer:**
```sql
WITH RankedEmployees AS (
    SELECT 
        EmpName,
        D.DeptName,
        Salary,
        DENSE_RANK() OVER (PARTITION BY E.DeptID ORDER BY Salary DESC) AS DeptRank
    FROM Employees E
    JOIN Departments D ON E.DeptID = D.DeptID
)
SELECT * FROM RankedEmployees
WHERE DeptRank <= 2
ORDER BY DeptName, DeptRank;
```
*Teaching point: PARTITION BY restarts the ranking for every department — this is how "top N per group" problems are solved in SQL Server. Also introduces CTEs since you can't filter directly on a window function in the same SELECT's WHERE clause.*

---

## Question 4 — Date/Time Functions
Write a query that shows, for every employee:
- Hire date formatted as `DD-Mon-YYYY` (e.g., `15-Mar-2021`)
- The day of the week they were hired on (e.g., Monday)
- How many months they've been employed (as of today)
- A flag `AnniversaryThisMonth` = `'Yes'` if their hire-month matches the current month, else `'No'`

**Answer:**
```sql
SELECT 
    EmpName,
    FORMAT(HireDate, 'dd-MMM-yyyy') AS FormattedHireDate,
    DATENAME(WEEKDAY, HireDate) AS HireDayOfWeek,
    DATEDIFF(MONTH, HireDate, GETDATE()) AS MonthsEmployed,
    CASE 
        WHEN MONTH(HireDate) = MONTH(GETDATE()) THEN 'Yes'
        ELSE 'No'
    END AS AnniversaryThisMonth
FROM Employees;
```
*Teaching point: FORMAT() for display formatting, DATENAME() to extract a named part of a date, DATEDIFF() for elapsed time, and MONTH()/GETDATE() for comparisons. Mention that FORMAT() is flexible but slower on large datasets than CONVERT/style codes — worth a short discussion.*

---

## Question 5 — Combined: Aggregate + Ranking + Date Logic (Capstone)
Write a query to find, **for each department**, the most recently hired employee (i.e., the one with the latest `HireDate`) along with:
- Their name and hire date
- How many years ago they were hired
- The department's average salary (for comparison)
- Whether their own salary is above or below the department average (`'Above Avg'` / `'Below Avg'`)

**Answer:**
```sql
WITH DeptStats AS (
    SELECT 
        EmpID,
        EmpName,
        DeptID,
        Salary,
        HireDate,
        AVG(Salary) OVER (PARTITION BY DeptID) AS DeptAvgSalary,
        ROW_NUMBER() OVER (PARTITION BY DeptID ORDER BY HireDate DESC) AS RecencyRank
    FROM Employees
    WHERE DeptID IS NOT NULL
)
SELECT 
    D.DeptName,
    DS.EmpName,
    DS.HireDate,
    DATEDIFF(YEAR, DS.HireDate, GETDATE()) AS YearsAgoHired,
    ROUND(DS.DeptAvgSalary, 2) AS DeptAvgSalary,
    CASE 
        WHEN DS.Salary > DS.DeptAvgSalary THEN 'Above Avg'
        ELSE 'Below Avg'
    END AS SalaryComparison
FROM DeptStats DS
JOIN Departments D ON DS.DeptID = D.DeptID
WHERE DS.RecencyRank = 1
ORDER BY D.DeptName;
```
*Teaching point: This is the "everything together" question — window functions (AVG OVER, ROW_NUMBER with PARTITION BY), a CTE, date arithmetic, and conditional logic in one query. Good discussion point: AVG() OVER (PARTITION BY...) computes the group average without collapsing rows, unlike GROUP BY.*

---

## Suggested Marking Scheme (out of 25)

| Question | Marks | Focus |
|---|---|---|
| Q1 | 4 | Aggregate functions, conditional counting |
| Q2 | 4 | ROW_NUMBER vs RANK vs DENSE_RANK |
| Q3 | 5 | PARTITION BY, CTE, top-N-per-group |
| Q4 | 5 | Date/time functions, formatting |
| Q5 | 7 | Combined window functions + CTE + date logic |

**Discussion prompt for class:** Ask students to explain in their own words, using Q2's output, why RANK and DENSE_RANK behave differently after a tie — this is the concept students most often get wrong in interviews.
