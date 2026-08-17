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
