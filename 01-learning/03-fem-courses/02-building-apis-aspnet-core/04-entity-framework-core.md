# Entity Framework Core

*From "Building APIs with C# & ASP.NET Core" by Spencer Schneidenbach (Frontend Masters)*

---

## Lesson — Introducing EF Core

---

Entity Framework Core (EF Core) is **an object-relational mapping (ORM) framework for .NET** that enables developers to interact with databases using .NET objects in an object-oriented manner rather than raw SQL.

### Why Use EF Core?

1. **Code-First Development** — Define database schema using .NET classes. Tables are generated automatically. Promotes agile design.
2. **Object-Relational Mapping (ORM)** — Maps .NET objects to database tables, simplifying object-oriented database interactions.
3. **Query Abstraction** — Provides a powerful and flexible query abstraction layer enabling expressive, readable queries while abstracting the underlying SQL.
4. **Change Tracking** — Built-in mechanisms manage in-memory data changes and database synchronization, reducing inconsistencies.
5. **Database Migrations** — Enables incremental schema modifications in a controlled, version-managed way.

### Core Components

**DbContext** — The main class representing a database session. Manages connections, query creation, and command execution.

**DbSet\<T\>** — A collection property representing database entities; provides querying and manipulation methods.

```csharp
public class MyDbContext : DbContext
{
    public DbSet<Employee> Employees { get; set; }
}
```

LINQ queries on `DbSet` automatically translate to SQL:

```csharp
var employees = context
    .Employees
    .Where(e => e.Department == "Sales")
    .ToList();
```

Equivalent SQL:

```sql
SELECT * FROM Employees
WHERE Department = 'Sales';
```

### Installation

```bash
dotnet add package Microsoft.EntityFrameworkCore
```

Supported database providers: SQL Server, SQLite, PostgreSQL, MySQL. This course uses **SQLite** for accessibility.

---

## Lesson — IQueryable and IEnumerable

---

### IEnumerable\<T\>

`IEnumerable<T>` is an interface that represents a collection of objects that can be enumerated over. It's one of the main drivers of LINQ and is used to query, filter, and manipulate collections.

```csharp
public class Example
{
    public void PrintItems(IEnumerable<string> items)
    {
        foreach (var item in items)
        {
            Console.WriteLine(item);
        }
    }
}
```

### Lazy Evaluation (Deferred Execution)

Lazy evaluation means values or sequences are not generated or fetched until they are actually needed. Methods like `Select`, `Where`, and `Take` do NOT immediately execute when called — they set up a query pipeline. Actual iteration happens only when elements are accessed via a `foreach` loop or terminal operation like `ToList()`.

```csharp
var numbers = new[] { 1, 2, 3, 4, 5 };

var query = numbers.Where(n => n % 2 == 0);

Console.WriteLine("Query is defined, but not executed yet.");

foreach (var number in query)
{
    Console.WriteLine(number);  // Only now is the query executed
}
```

### IQueryable\<T\>

`IQueryable<T>` is similar to `IEnumerable<T>` but **meant for querying data from non-in-memory data sources** — typically databases. It retains LINQ's expressiveness but adds a translation layer that converts LINQ calls into the native language of the target data source (e.g., SQL).

### Expression Trees

What's the difference between these two lines?

```csharp
Func<string, string> toLower = s => s.ToLower();
Expression<Func<string, string>> toLower = s => s.ToLower();
```

- `Func<string, string>` — a **delegate** that executes the function
- `Expression<Func<string, string>>` — a **description** of the function as a tree structure that can be compiled or translated

**Functions do the thing. Expression trees describe the thing that does the thing.**

An expression tree is a representation of code in a tree structure. Each node represents an expression.

🌶️🌶️🌶️ Spencer has a whole talk on expression trees — worth watching if curious: [YouTube link (5 years old but still relevant)](https://www.youtube.com/watch?v=Ptnxc6tVIPE)

This matters significantly when querying databases with EF Core — covered in the coming lessons.

---

## Lesson — Setting Up Your First DbContext and DbSet with Migrations

---

### Install the Package

```bash
dotnet add package Microsoft.EntityFrameworkCore.Sqlite
```

This also installs `Microsoft.EntityFrameworkCore` as a dependency.

### Setting Up Your DbContext

Create a new class in the project root (or a Data/Infrastructure folder — no strong opinion):

```csharp
using Microsoft.EntityFrameworkCore;

public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options)
    {
    }

    public DbSet<Employee> Employees { get; set; }
}
```

Breaking it down:
- Inherits from `DbContext` for all its goodness
- Constructor takes `DbContextOptions<AppDbContext>` — configuration (typically the connection string)
- `DbSet<Employee> Employees` — the collection of `Employee` entities mapped to the `Employees` table

### Registering with Dependency Injection

In `Program.cs`:

```csharp
using Microsoft.EntityFrameworkCore;

builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlite(builder.Configuration.GetConnectionString("DefaultConnection")));
```

Connection string in `appsettings.json`:

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Data Source=employees.db"
  }
}
```

### Setting Up Migrations

Migrations allow incremental changes to the database schema over time — without manually writing SQL.

Install the CLI tools globally:

```bash
dotnet tool install --global dotnet-ef
```

**Create the database:**

```bash
dotnet ef database update
```

This creates `employees.db` and a `__EFMigrationsHistory` table to track migration history.

**Create a migration:**

```bash
dotnet ef migrations add Init
```

This generates a migration file in the `Migrations/` folder:

```csharp
public partial class Init : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.CreateTable(
            name: "Employees",
            columns: table => new
            {
                Id = table.Column<int>(type: "INTEGER", nullable: false)
                    .Annotation("Sqlite:Autoincrement", true),
                FirstName = table.Column<string>(type: "TEXT", nullable: false),
                LastName = table.Column<string>(type: "TEXT", nullable: false),
                SocialSecurityNumber = table.Column<string>(type: "TEXT", nullable: true),
                Address1 = table.Column<string>(type: "TEXT", nullable: true),
                // ...other columns
            },
            constraints: table =>
            {
                table.PrimaryKey("PK_Employees", x => x.Id);
            });

        migrationBuilder.CreateTable(
            name: "EmployeeBenefits",
            columns: table => new
            {
                Id = table.Column<int>(type: "INTEGER", nullable: false)
                    .Annotation("Sqlite:Autoincrement", true),
                EmployeeId = table.Column<int>(type: "INTEGER", nullable: false),
                BenefitType = table.Column<int>(type: "INTEGER", nullable: false),
                Cost = table.Column<decimal>(type: "TEXT", nullable: false)
            },
            constraints: table =>
            {
                table.PrimaryKey("PK_EmployeeBenefits", x => x.Id);
                table.ForeignKey(
                    name: "FK_EmployeeBenefits_Employees_EmployeeId",
                    column: x => x.EmployeeId,
                    principalTable: "Employees",
                    principalColumn: "Id",
                    onDelete: ReferentialAction.Cascade);
            });
    }

    protected override void Down(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.DropTable(name: "EmployeeBenefits");
        migrationBuilder.DropTable(name: "Employees");
    }
}
```

- `Up` — defines the forward migration (what changes to make)
- `Down` — defines how to revert the changes

EF Core automatically creates the `EmployeeBenefits` table because `Employee` has a `List<EmployeeBenefit>` navigation property.

**Remove the last migration:**

```bash
dotnet ef migrations remove
```

### Navigation Properties

Navigation properties allow navigating between related entities. In a one-to-many relationship between `Employee` and `EmployeeBenefit`:

```csharp
public class Employee
{
    // ...
    public List<EmployeeBenefit> Benefits { get; set; } = new List<EmployeeBenefit>();
}
```

Adding the reverse navigation property to `EmployeeBenefit`:

```csharp
public class EmployeeBenefits
{
    public int Id { get; set; }
    public int EmployeeId { get; set; }
    public BenefitType BenefitType { get; set; }
    public decimal Cost { get; set; }

    public Employee Employee { get; set; } = null!;
}
```

🌶️🌶️🌶️ With nullability turned on, note the `= null!` syntax — a compromise to make nullability work nicely with EF Core. See [Microsoft docs](https://learn.microsoft.com/en-us/ef/core/miscellaneous/nullable-reference-types).

### SQLTools Extension

Install the **SQLTools** VS Code extension with the SQLite driver to view and manage the `employees.db` database directly in the editor.

---

## Lesson — Seeding Your Database

---

### Built-in EF Core Seeding (Not Recommended)

EF Core provides seeding through `OnModelCreating`, but the instructor doesn't use it and doesn't know anyone who does.

### Recommended Approach: Static SeedData Class

```csharp
public static class SeedData
{
    public static void Seed(IServiceProvider services)
    {
        using var scope = services.CreateScope();
        var context = scope.ServiceProvider.GetRequiredService<AppDbContext>();

        if (!context.Employees.Any())
        {
            context.Employees.AddRange(
                new Employee { FirstName = "John", LastName = "Doe", /* ... */ },
                new Employee { FirstName = "Jane", LastName = "Smith", /* ... */ }
            );
            context.SaveChanges();
        }
    }
}
```

Call from `Program.cs` after `app.Build()`.

### The Scoped Service Problem

Calling `Seed` directly from startup causes an error:

```
Cannot resolve scoped service 'AppDbContext' from root provider
```

**Why:** `DbContext` is a scoped service — it requires an HTTP request context to be resolved. No scope exists during app startup.

**Solution:** Create an explicit scope:

```csharp
using var scope = app.Services.CreateScope();
SeedData.Seed(scope.ServiceProvider);
```

---

## Lesson — Querying Basics

---

### LINQ for Database Queries

The main way to query the database is through LINQ. It allows filtering, ordering, and selecting data in an expressive way.

### Replacing the Repository with DbContext Directly

🌶️🌶️🌶️ `DbContext` already implements both the Repository and Unit of Work patterns. Wrapping it in a custom `IRepository<T>` adds complexity without sufficient benefit. Spencer prefers using `DbContext` directly. (The community is split ~50/50 on this.)

### Basic Queries

```csharp
// Get all employees
var employees = await _dbContext.Employees.ToArrayAsync();

// Include related data (Benefits will be empty array without this)
var employees = await _dbContext.Employees
    .Include(e => e.Benefits)
    .ToArrayAsync();
```

Without `Include()`, related entities like `Benefits` return empty arrays by default (lazy loading is not enabled).

### Filtering and Pagination

**Request class:**

```csharp
public class GetAllEmployeesRequest
{
    public int? Page { get; set; }
    public int? RecordsPerPage { get; set; }
    public string? FirstNameContains { get; set; }
    public string? LastNameContains { get; set; }
}
```

**Implementation using `IQueryable<T>` for dynamic query building:**

```csharp
int page = request.Page ?? 1;
int numberOfRecords = request.RecordsPerPage ?? 100;

IQueryable<Employee> query = _dbContext.Employees
    .Include(e => e.Benefits)
    .Skip((page - 1) * numberOfRecords)
    .Take(numberOfRecords);

if (!string.IsNullOrWhiteSpace(request.FirstNameContains))
    query = query.Where(e => e.FirstName.Contains(request.FirstNameContains));

if (!string.IsNullOrWhiteSpace(request.LastNameContains))
    query = query.Where(e => e.LastName.Contains(request.LastNameContains));

var employees = await query.ToArrayAsync();
```

Defaults: page 1, 100 records per page. Validate that `RecordsPerPage` doesn't exceed 100.

### Testing Setup

Use **SQLite in-memory** for tests — NOT EF Core's own in-memory provider (Microsoft explicitly discourages using EF Core's in-memory provider):

```csharp
public class CustomWebApplicationFactory : WebApplicationFactory<Program>
{
    protected override void ConfigureWebHost(IWebHostBuilder builder)
    {
        builder.ConfigureServices(services =>
        {
            // Remove existing DbContext registration
            // Add SQLite in-memory connection
            services.AddSingleton<DbConnection>(container =>
            {
                var connection = new SqliteConnection("DataSource=:memory:");
                connection.Open();
                return connection;
            });

            services.AddDbContext<AppDbContext>((container, options) =>
            {
                var connection = container.GetRequiredService<DbConnection>();
                options.UseSqlite(connection);
            });
        });
    }
}
```

Run migrations automatically during test setup via a `MigrateAndSeed` method.

---

## Lesson — Adding and Removing Objects from Your Database

---

### Creating Objects (POST)

```csharp
var newEmployee = new Employee
{
    FirstName = request.FirstName!,
    LastName = request.LastName!,
    // ...other properties
};

_dbContext.Employees.Add(newEmployee);
await _dbContext.SaveChangesAsync();

return CreatedAtAction(nameof(GetEmployee), new { id = newEmployee.Id }, MapToResponse(newEmployee));
```

Alternative: `_dbContext.Add(newEmployee)` also works.

### Updating Objects (PUT)

```csharp
var existingEmployee = await _dbContext.Employees.FindAsync(id);
if (existingEmployee == null) return NotFound();

existingEmployee.Address1 = request.Address1;
// ...update other allowed properties

_dbContext.Entry(existingEmployee).State = EntityState.Modified;
await _dbContext.SaveChangesAsync();
return Ok(MapToResponse(existingEmployee));
```

### Change Tracking

By default, EF Core tracks all loaded entities and checks if they should be updated on every `SaveChangesAsync` call.

> "If you load 10,000 entities using EF Core, it'll happily track them all — and then check to see if they should be updated once you call SaveChangesAsync."

**Disable tracking globally (recommended):**

```csharp
options.UseQueryTrackingBehavior(QueryTrackingBehavior.NoTracking);
```

When disabled, explicitly mark entities as modified:

```csharp
_dbContext.Entry(existingEmployee).State = EntityState.Modified;
```

Or opt-in per query:

```csharp
var employee = await _dbContext.Employees.AsTracking().FindAsync(id);
```

### Deleting Objects (DELETE)

```csharp
var employee = await _dbContext.Employees.FindAsync(id);
if (employee == null) return NotFound();

_dbContext.Employees.Remove(employee);
await _dbContext.SaveChangesAsync();
return NoContent();
```

Returns **204 No Content** on success.

### Testing Emphasis

Comprehensive tests catch overlooked details and prevent false confidence. Tests revealed gaps in implementation throughout this lesson.

---

## Lesson — Extending Your Data Model and Model Configuration

---

### Rethinking the Benefits Model

The original `EmployeeBenefits` class with a `BenefitType` enum was a simplification that no longer fits a real payroll system. A real system needs a proper junction table.

Remove `EmployeeBenefits` and `BenefitType` enum entirely. `BenefitType` is promoted from an enum to a full entity.

**New entities:**

```csharp
public class Benefit
{
    public int Id { get; set; }
    public required string Name { get; set; }
    public required string Description { get; set; }
    public decimal BaseCost { get; set; }
}

public class EmployeeBenefit
{
    public int Id { get; set; }
    public int EmployeeId { get; set; }
    public Employee Employee { get; set; } = null!;
    public int BenefitId { get; set; }
    public Benefit Benefit { get; set; } = null!;
    public decimal? CostToEmployee { get; set; }
}
```

> All models have an `Id` property — this is super common in databases and useful for easier querying and indexing.

### Migrating the Old Table

```bash
dotnet ef migrations add RemoveOldBenefitsTable
```

Review the migration before applying. It drops the `EmployeeBenefits` table.

🌶️🌶️🌶️ Never trust migration files without reviewing them first.

🌶️🌶️🌶️ In production, dropping a table is a Pretty Big Deal™️ — you need to migrate old data, test extensively, etc. In this course we skip that, but know the implications.

### Adding New Entities to DbContext

```csharp
public class AppDbContext : DbContext
{
    public DbSet<Employee> Employees { get; set; }
    public DbSet<Benefit> Benefits { get; set; }
    public DbSet<EmployeeBenefit> EmployeeBenefits { get; set; }
}
```

### Configuring a Unique Constraint

An employee and a benefit should have exactly 0 or 1 records together — a unique constraint enforces this:

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<EmployeeBenefit>()
        .HasIndex(eb => new { eb.EmployeeId, eb.BenefitId })
        .IsUnique();
}
```

### More on OnModelCreating

This method is a powerful tool for configuring the data model. Use it for:

```csharp
// Relationships
modelBuilder.Entity<Employee>()
    .HasMany(e => e.Benefits)
    .WithOne(eb => eb.Employee)
    .HasForeignKey(eb => eb.EmployeeId);

// Composite primary keys
modelBuilder.Entity<EmployeeBenefit>()
    .HasKey(eb => new { eb.EmployeeId, eb.BenefitId })
    .HasName("PK_EmployeeBenefit");

// Column data types
modelBuilder.Entity<Employee>()
    .Property(e => e.FirstName)
    .HasColumnType("varchar(100)");
```

### Adding the Benefits Migration

```bash
dotnet ef migrations add AddBenefitsAndEmployeeBenefits
```

Review migration for accuracy before running.

### Updated Seed Method

```csharp
public static void MigrateAndSeed(IServiceProvider serviceProvider)
{
    var context = serviceProvider.GetRequiredService<AppDbContext>();
    context.Database.Migrate();

    if (!context.Employees.Any())
    {
        var employees = new List<Employee>
        {
            new Employee
            {
                FirstName = "John", LastName = "Doe",
                SocialSecurityNumber = "123-45-6789",
                Address1 = "123 Main St", City = "Anytown", State = "NY",
                ZipCode = "12345", PhoneNumber = "555-123-4567",
                Email = "john.doe@example.com"
            },
            new Employee
            {
                FirstName = "Jane", LastName = "Smith",
                SocialSecurityNumber = "987-65-4321",
                Address1 = "456 Elm St", Address2 = "Apt 2B",
                City = "Othertown", State = "CA",
                ZipCode = "98765", PhoneNumber = "555-987-6543",
                Email = "jane.smith@example.com"
            }
        };

        context.Employees.AddRange(employees);
        context.SaveChanges();
    }

    if (!context.Benefits.Any())
    {
        var benefits = new List<Benefit>
        {
            new Benefit { Name = "Health", Description = "Medical, dental, and vision coverage", BaseCost = 100.00m },
            new Benefit { Name = "Dental", Description = "Dental coverage", BaseCost = 50.00m },
            new Benefit { Name = "Vision", Description = "Vision coverage", BaseCost = 30.00m }
        };

        context.Benefits.AddRange(benefits);
        context.SaveChanges();

        var healthBenefit = context.Benefits.Single(b => b.Name == "Health");
        var dentalBenefit = context.Benefits.Single(b => b.Name == "Dental");
        var visionBenefit = context.Benefits.Single(b => b.Name == "Vision");

        var john = context.Employees.Single(e => e.FirstName == "John");
        john.Benefits = new List<EmployeeBenefit>
        {
            new EmployeeBenefit { Benefit = healthBenefit, CostToEmployee = 100m },
            new EmployeeBenefit { Benefit = dentalBenefit }
        };

        var jane = context.Employees.Single(e => e.FirstName == "Jane");
        jane.Benefits = new List<EmployeeBenefit>
        {
            new EmployeeBenefit { Benefit = healthBenefit, CostToEmployee = 120m },
            new EmployeeBenefit { Benefit = visionBenefit }
        };

        context.SaveChanges();
    }
}
```

### Updated GetBenefitsForEmployee Endpoint

```csharp
[HttpGet("{employeeId}/benefits")]
[ProducesResponseType(typeof(IEnumerable<GetEmployeeResponseEmployeeBenefit>), StatusCodes.Status200OK)]
[ProducesResponseType(StatusCodes.Status404NotFound)]
[ProducesResponseType(StatusCodes.Status500InternalServerError)]
public async Task<IActionResult> GetBenefitsForEmployee(int employeeId)
{
    var employee = await _dbContext.Employees
        .Include(e => e.Benefits)
        .ThenInclude(e => e.Benefit)
        .SingleOrDefaultAsync(e => e.Id == employeeId);

    if (employee == null) return NotFound();

    var benefits = employee.Benefits.Select(b => new GetEmployeeResponseEmployeeBenefit
    {
        Id = b.Id,
        Name = b.Benefit.Name,
        Description = b.Benefit.Description,
        Cost = b.CostToEmployee ?? b.Benefit.BaseCost  // Use employee cost if set, otherwise base cost
    });

    return Ok(benefits);
}
```

**Notes:**

- The `ThenInclude` call after `Include` is essential for loading nested related entities. Try commenting it out to see what happens!
- We stopped returning benefits in the Employee GET endpoints for convenience in this refactor. This is a valid design decision — think about the implications for your application.

🌶️🌶️🌶️ SQL skills are highly undervalued by developers. Being able to differentiate yourself in understanding the strengths, weaknesses, and capabilities of SQL as a language and data store is immensely valuable.

---

## Lesson — Reviewing Everything We've Done

---

In the EF Core section, we:

- Replaced the in-memory repository with a real SQLite database
- Added database seeding with initial data
- Enhanced the GET employees endpoint with filtering and pagination
- Redesigned benefits modeling into a proper many-to-many relationship with junction tables

### The Value of Request Objects

Using request objects rather than entities for data transfer:

- **Leaves entities to do what they do best** — represent and store data
- **Avoids awkward JSON attribute workarounds** to suppress empty navigation properties
- **Single Responsibility Principle** — each thing has one job and it's easy to understand what that job is

### For the Remainder of the Course

Upcoming topics:
- Entity Framework and ASP.NET Core advanced techniques
- Testcontainers for more realistic testing environments
- Security considerations

---

## Lesson — Auditing and Automatic Application of Properties

---

### Determining Who Made a Change and When

For auditing purposes, use a common base class with audit properties:

```csharp
public abstract class AuditableEntity
{
    public string? CreatedBy { get; set; }
    public DateTime? CreatedOn { get; set; }
    public string? LastModifiedBy { get; set; }
    public DateTime? LastModifiedOn { get; set; }
}
```

(Declared nullable to make migrations easier — in production you'd typically require values.)

Make `Employee` extend this class:

```csharp
public class Employee : AuditableEntity
{
    // ...
}
```

Add a migration:

```bash
dotnet ef migrations add EmployeeAuditableFields
```

### Automatically Populating Audit Fields

Override `SaveChanges` and `SaveChangesAsync` in `DbContext` to automatically set audit properties — no need to set them manually in every endpoint:

```csharp
public class AppDbContext : DbContext
{
    public override int SaveChanges()
    {
        UpdateAuditFields();
        return base.SaveChanges();
    }

    public override Task<int> SaveChangesAsync(CancellationToken cancellationToken = default)
    {
        UpdateAuditFields();
        return base.SaveChangesAsync(cancellationToken);
    }

    private void UpdateAuditFields()
    {
        var entries = ChangeTracker.Entries<AuditableEntity>();

        foreach (var entry in entries)
        {
            if (entry.State == EntityState.Added)
            {
                entry.Entity.CreatedBy = "TheCreateUser";
                entry.Entity.CreatedOn = DateTime.UtcNow;
            }

            if (entry.State == EntityState.Modified)
            {
                entry.Entity.LastModifiedBy = "TheUpdateUser";
                entry.Entity.LastModifiedOn = DateTime.UtcNow;
            }
        }
    }
}
```

After a PUT request, browse with SQLTools and verify that `LastModifiedBy` and `LastModifiedOn` are being set.

**NOTE:** In a real application, capture the actual authenticated user rather than hardcoded strings. This will be addressed in the security section.

### Making Timestamps Testable

The challenge: `DateTime.UtcNow` makes exact timestamp assertions impossible in tests.

Solution: inject `ISystemClock` (from `Microsoft.Extensions.Internal`) instead of calling `DateTime.UtcNow` directly:

**Register in `Program.cs`:**

```csharp
using Microsoft.Extensions.Internal;

builder.Services.AddSingleton<ISystemClock, SystemClock>();
```

**Use in `AppDbContext`:**

```csharp
public class AppDbContext : DbContext
{
    private readonly ISystemClock _systemClock;

    public AppDbContext(DbContextOptions<AppDbContext> options, ISystemClock systemClock) : base(options)
    {
        this._systemClock = systemClock;
    }

    private void UpdateAuditFields()
    {
        var entries = ChangeTracker.Entries<AuditableEntity>();

        foreach (var entry in entries)
        {
            if (entry.State == EntityState.Added)
            {
                entry.Entity.CreatedBy = "TheCreateUser";
                entry.Entity.CreatedOn = _systemClock.UtcNow.UtcDateTime;
            }

            if (entry.State == EntityState.Modified)
            {
                entry.Entity.LastModifiedBy = "TheUpdateUser";
                entry.Entity.LastModifiedOn = _systemClock.UtcNow.UtcDateTime;
            }
        }
    }
}
```

### Updated Test Factory and TestSystemClock

```csharp
public class CustomWebApplicationFactory : WebApplicationFactory<Program>
{
    public static TestSystemClock SystemClock { get; } = new TestSystemClock();

    protected override void ConfigureWebHost(IWebHostBuilder builder)
    {
        builder.ConfigureServices(services =>
        {
            // ... DbContext setup ...

            var systemClockDescriptor = services.Single(d => d.ServiceType == typeof(ISystemClock));
            services.Remove(systemClockDescriptor);
            services.AddSingleton<ISystemClock>(SystemClock);
        });
    }
}

public class TestSystemClock : ISystemClock
{
    public DateTimeOffset UtcNow { get; } = DateTimeOffset.Parse("2022-01-01T00:00:00Z");
}
```

### Updated Test

```csharp
[Fact]
public async Task UpdateEmployee_ReturnsOkResult()
{
    var client = _factory.CreateClient();
    var response = await client.PutAsJsonAsync("/employees/1", new Employee
    {
        FirstName = "John",
        LastName = "Doe",
        Address1 = "123 Main Smoot"
    });

    if (!response.IsSuccessStatusCode)
    {
        var content = await response.Content.ReadAsStringAsync();
        Assert.Fail($"Failed to update employee: {content}");
    }

    using var scope = _factory.Services.CreateScope();
    var db = scope.ServiceProvider.GetRequiredService<AppDbContext>();
    var employee = await db.Employees.FindAsync(1);
    Assert.Equal("123 Main Smoot", employee.Address1);
    Assert.Equal(CustomWebApplicationFactory.SystemClock.UtcNow.UtcDateTime, employee.LastModifiedOn);
}
```

---

## Lesson — Testcontainers and Real Databases

---

### Three Approaches to Database Testing

| Approach | Quality | Notes |
|----------|---------|-------|
| EF Core in-memory provider | Not good | Microsoft explicitly discourages it |
| SQLite in-memory | Better | A real database, but you probably aren't using SQLite in production |
| Production DB in a container | Best | Real-world accuracy, catches prod-specific bugs earlier |

### Testcontainers

Testcontainers is a library that enables spinning up **real databases in containers** for your tests. Requires Docker to be installed.

**Setup:**

Remove SQLite references, install PostgreSQL provider and Testcontainers:

```bash
dotnet add package Npgsql.EntityFrameworkCore.PostgreSQL
dotnet add package Testcontainers.PostgreSql
```

Remove previous migrations and create new ones for PostgreSQL.

### Bugs Discovered During Migration

Switching from SQLite to PostgreSQL surfaced real bugs that were masked by SQLite:

1. A test was creating duplicate John Doe records — only visible with PostgreSQL
2. The seed method failed to properly add `EmployeeBenefits` records with the target database

### Recommendation

Start with your target production database in your tests from the beginning. Testcontainers makes this practical by spinning up and tearing down containers automatically.

---

*Content licensed under CC-BY-NC-4.0 | Code samples under Apache 2.0*
