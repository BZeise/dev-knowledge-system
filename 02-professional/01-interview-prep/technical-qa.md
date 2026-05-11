# Technical Interview Q&A

Interview questions and answers organized by topic. Includes questions from various technical interviews.

## OOP & C#

### What is method overloading?

Defining multiple methods in the same class with the same name but different parameter lists (different number, type, or order). The compiler resolves which to call based on arguments at the call site. This is compile-time (static) polymorphism.

```csharp
public int Add(int a, int b) => a + b;
public double Add(double a, double b) => a + b;
public int Add(int a, int b, int c) => a + b + c;
```

### What is a partial class?

A partial class splits the definition of a single class across multiple files. All parts are combined by the compiler.

Most common use: code-generated files — generated code goes in one partial file, your custom code goes in another, so regenerating doesn't overwrite your additions.

### What is deferred execution in LINQ?

LINQ queries are not executed when defined — they execute when results are first enumerated (via `foreach`, `.ToList()`, `.Count()`). Calling `.ToList()` forces immediate execution.

### Can you explain dependency injection?

Supply dependencies from outside rather than creating them internally, typically via constructor parameters. Enables loose coupling and testability. Built into ASP.NET Core.

See [Dependency Injection in FEM Course](../../03-fem-courses/01-csharp-dotnet/06-hosting-api-development.md#dependency-injection-di)

---

## SQL

### What are database indexes and what are they used for?

A separate data structure SQL Server maintains to speed up data retrieval — like a book index. Speeds up reads, slows down writes. Types: clustered, non-clustered, covering, composite.

### What is database normalization?

Organizing a schema to reduce data redundancy and improve integrity. Proceeds through normal forms:
- **1NF** — Atomic values, no repeating groups
- **2NF** — Full dependency on entire primary key
- **3NF** — No transitive dependencies (practical target for most OLTP)

### What are Inner vs. Outer Joins?

- **INNER JOIN** — Only rows with match in both tables
- **LEFT OUTER** — All left rows plus matches; unmatched right-side columns are NULL
- **FULL OUTER** — All rows from both tables with NULLs where no match

---

## Coding Exercises

### Find duplicate records
```sql
SELECT Email, COUNT(*) as Count
FROM Customers
GROUP BY Email
HAVING COUNT(*) > 1;
```

### Find bad zip codes (not 5 digits)
```sql
SELECT * FROM Addresses
WHERE ZipCode NOT LIKE '[0-9][0-9][0-9][0-9][0-9]'
   OR LEN(ZipCode) <> 5;
```

### Active members in last 10 days
```sql
SELECT * FROM Members
WHERE IsActive = 1
  AND LastActivityDate >= DATEADD(DAY, -10, GETDATE());
```

---

## Expand This Section

This is a stub. Add:
- More company-specific interview questions
- Your personal interview prep notes
- Practice problems and solutions
- Interview experience retrospectives
