# SQL and Database Development

## Performance Tuning — Where to Start

When a query or stored procedure is slow, first reach for the **Execution Plan**.

**In SSMS:** Ctrl+M or Include Actual Execution Plan

The plan shows exactly how SQL Server executed the query.

### Key Things to Look For

**Table Scan vs Index Seek**
- **Scan** reads every row
- **Seek** jumps to matching rows
- A scan on a large table is usually the culprit

**Key Lookup**
- Happens when a non-clustered index is used but SQL still must fetch columns not in the index
- May indicate you need a covering index

**Estimated vs Actual Rows**
- Large discrepancy means SQL's statistics are out of date

**Thick Arrows**
- In the graphical plan, thick arrows between operations indicate many rows flowing through
- Good place to focus optimization efforts

---

## Indexes

An index is a separate data structure SQL Server maintains to speed up data retrieval — like a book index.

**Trade-off:** Speeds up reads but slows down writes (every INSERT, UPDATE, DELETE must update all relevant indexes). Don't over-index.

### Clustered Index

- **Physically sorts and stores table rows** by the index key
- **One per table**
- By default, the primary key becomes the clustered index

```sql
CREATE CLUSTERED INDEX idx_users_id ON Users(Id);
```

### Non-Clustered Index

- A separate structure with pointers back to rows
- A table can have **many**
- Use on columns frequently in WHERE, JOIN, or ORDER BY

```sql
CREATE NONCLUSTERED INDEX idx_users_email ON Users(Email);
```

### Covering Index

- A non-clustered index that **INCLUDEs all columns** a query needs
- Eliminates key lookups entirely
- Most efficient for specific query patterns

```sql
CREATE NONCLUSTERED INDEX idx_users_name_email 
ON Users(Name) INCLUDE (Email, CreatedDate);
```

### Composite Index

- Index on multiple columns
- **Column order matters** — most useful when queries filter on a leading prefix

```sql
CREATE NONCLUSTERED INDEX idx_orders_customer_date 
ON Orders(CustomerId, OrderDate);

-- This index is efficient for:
-- WHERE CustomerId = 5
-- WHERE CustomerId = 5 AND OrderDate > '2026-01-01'

-- But NOT efficient for:
-- WHERE OrderDate > '2026-01-01'  (doesn't use leading column)
```

---

## Database Normalization

Organizing a schema to reduce data redundancy and improve integrity. Proceeds through normal forms:

### 1NF — First Normal Form
- Each column holds atomic values
- No repeating groups

```sql
-- BAD: Repeating group
CREATE TABLE Orders (
    OrderId INT,
    CustomerId INT,
    Items VARCHAR(MAX)  -- "Item1, Item2, Item3"
);

-- GOOD: Atomicity
CREATE TABLE Orders (
    OrderId INT,
    CustomerId INT
);

CREATE TABLE OrderItems (
    OrderItemId INT,
    OrderId INT,
    ItemId INT
);
```

### 2NF — Second Normal Form
- Everything in 1NF, plus
- Every non-key column is **fully dependent on the entire primary key**

### 3NF — Third Normal Form
- Everything in 2NF, plus
- No non-key column depends on another non-key column (no transitive dependencies)

**3NF is the practical target for most OLTP databases.**

---

## Joins

Reference: [SQL Joins Diagram](https://upload.wikimedia.org/wikipedia/commons/9/9d/SQL_Joins.svg)

| Join Type | Returns |
|-----------|---------|
| INNER JOIN | Only rows with a match in both tables. Most common. |
| LEFT (OUTER) JOIN | All rows from the left table, plus matched rows from the right. Unmatched right-side columns are NULL. |
| RIGHT (OUTER) JOIN | Mirror of LEFT JOIN. Usually rewritable as a LEFT JOIN by swapping table order. |
| FULL OUTER JOIN | All rows from both tables. NULLs fill in where there's no match on either side. |
| CROSS JOIN | Cartesian product — every row in A paired with every row in B. Rarely intentional; can produce enormous result sets. |

---

## Subqueries vs JOINs

When to use each:

| Aspect | Subquery | JOIN |
|--------|----------|------|
| **Readability** | Often clearer for filtering | Better for combining columns |
| **Performance** | Can be slower (especially correlated) | Usually faster |
| **Use case** | Filtering based on aggregates | Combining data from multiple tables |
| **Correlated** | Powerful but potentially slow | N/A |

---

## Primary Key vs Foreign Key

### Primary Key
- **Uniquely identifies each row**
- Cannot be NULL
- Usually an auto-incrementing INT
- **One per table**

```sql
CREATE TABLE Users (
    UserId INT PRIMARY KEY IDENTITY(1,1),
    Name NVARCHAR(100)
);
```

### Foreign Key
- A column that **references the primary key of another table**
- Enforces **referential integrity** — can't insert a value that doesn't exist in the referenced table
- Can't delete a referenced row without handling dependents

```sql
CREATE TABLE Orders (
    OrderId INT PRIMARY KEY IDENTITY(1,1),
    UserId INT FOREIGN KEY REFERENCES Users(UserId),
    OrderDate DATETIME
);
```

---

## SQL Injection Protection

SQL injection is an attack where malicious SQL is inserted into an input that gets embedded in a query.

### Defenses

1. **Parameterized queries / prepared statements** — PRIMARY DEFENSE
   ```csharp
   // GOOD
   using (SqlCommand cmd = new SqlCommand("SELECT * FROM Users WHERE Email = @email", connection))
   {
       cmd.Parameters.AddWithValue("@email", userEmail);
       // Parameters are treated as data, never as executable SQL
   }
   ```

2. **Stored procedures** — Naturally parameterized if written correctly

3. **Static analysis / linting** — Catch direct string concatenation into SQL at development time

4. **Input validation** — Restrict what characters/formats are accepted (secondary defense only)

5. **Principle of Least Privilege** — DB accounts used by the app should have only permissions they need
   ```sql
   CREATE USER appuser WITHOUT LOGIN;
   GRANT SELECT, INSERT, UPDATE ON dbo.Users TO appuser;
   -- Does NOT have DROP TABLE permission
   ```

---

## Triggers

A stored procedure that fires automatically in response to a DML event (INSERT, UPDATE, DELETE) on a table.

### Advantages
- Enforce complex business rules at the database layer
- Audit changes automatically
- Maintain derived data

### Disadvantages
- **Hidden logic** — behavior happens invisibly, not in application code
- Hard to debug and test
- Can cause performance problems on bulk operations
- Can create cascading chains that are extremely difficult to reason about

**General Guidance:** Use triggers **sparingly**. Prefer handling logic in application code or stored procedures where it's visible and testable.

---

## SQL Cursors

A cursor steps through a result set row by row.

**General advice: AVOID THEM.** Set-based SQL operations are far faster.

Before reaching for a cursor, ask if the same result can be achieved with a JOIN, UPDATE ... FROM, or window function.

---

## Temp Tables and Table Variables

| Aspect | Temp Table (#tmp) | Table Variable (@tbl) |
|--------|-------------------|----------------------|
| **Scope** | Session / current connection | Current batch / procedure |
| **Stored in** | tempdb | Memory (mostly) |
| **Supports indexes** | Yes | Limited (only via constraints) |
| **Statistics** | Yes (SQL can optimize) | No |
| **Use when** | Large data sets, need indexes/stats | Small data sets, simple lookups |

```sql
-- Temp table
CREATE TABLE #TempResults (ID INT, Name NVARCHAR(100))
INSERT INTO #TempResults SELECT ID, Name FROM Customers WHERE IsActive = 1

-- Table variable
DECLARE @Results TABLE (ID INT, Name NVARCHAR(100))
INSERT INTO @Results SELECT ID, Name FROM Customers WHERE IsActive = 1
```

---

## Optimizing a Stored Procedure

Start by identifying the bottleneck (execution plan, SQL Profiler, Extended Events), then apply targeted fixes:

**Performance Tips:**
- Use `SET NOCOUNT ON` at the top — suppresses row-count messages sent back to client
- Use schema-qualified names: `dbo.TableName`, `EXEC dbo.ProcName` — avoids ambiguity, allows plan caching
- Don't prefix procedures with `sp_` — SQL Server checks master first
- Prefer `sp_executesql` over `EXEC` for dynamic SQL — parameterizable and plans are cached
- Avoid `SELECT *` — fetch only needed columns
- Avoid implicit type conversions in WHERE clauses — prevents index seeks
- Avoid `LIKE '%value%'` with leading wildcard — prevents index seeks
- Prefer JOINs over correlated subqueries in WHERE clauses
- Create non-clustered covering indexes on frequently-filtered columns
- Use `EXISTS` instead of `COUNT(*) > 0` in existence checks — short-circuits on first match
- Keep transactions as short as possible — minimize lock hold time, reduce deadlock risk
- Break large procedures into smaller sub-procedures
- For large result sets, implement server-side paging

**Note on WITH (NOLOCK):** Allows dirty reads. Appropriate in some reporting scenarios where slightly stale data is acceptable. SQL Server 2005+ introduced Read Committed Snapshot Isolation (RCSI) as a cleaner alternative.

---

## GROUP BY

`GROUP BY` collapses rows sharing the same value(s) into a single row, allowing aggregate functions to apply to each group.

**Rule:** Every non-aggregate column in SELECT must appear in GROUP BY.

Use `HAVING` to filter groups (as opposed to WHERE which filters rows before grouping).

```sql
SELECT DepartmentID, COUNT(*) AS EmployeeCount, AVG(Salary) AS AvgSalary
FROM Employees
GROUP BY DepartmentID
HAVING COUNT(*) > 5
```

---

## Next Steps

- [FEM C# and .NET](../03-fem-courses/01-csharp-dotnet/) — Entity Framework, LINQ, data access patterns
- [SOLID Principles](../02-concepts/solid.md) — Design patterns for data layer
