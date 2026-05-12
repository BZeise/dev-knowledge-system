# Diving Into ASP.NET Core

*From "Building APIs with C# & ASP.NET Core" by Spencer Schneidenbach (Frontend Masters)*

---

## Lesson — Setting the Stage

---

The course guides learners through ASP.NET Core by building a progressively complex application — starting with a simple in-memory web API for managing a list of employees, and culminating in a complete web application with a database.

**How the code is structured:** Each lesson builds on the previous one. To find the solution for a given lesson, look at the starting code of the *next* lesson. For example, the solution for "Writing our first tests" appears as the starting code for "Refactoring to request classes and adding a PUT request."

---

## Lesson — Introduction to Minimal APIs

---

### Creating the Project

```bash
dotnet new webapi -n TheEmployeeAPI
```

After removing the unneeded parts from `Program.cs`, you're left with:

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();

app.Run();
```

### What is a Minimal API?

Minimal APIs are a relatively new way to define routes and handlers in ASP.NET Core. They are a simple and lightweight way to define routes and handlers.

They have two basic ways to define endpoints:

- **Delegate** — any ol' function
- **RequestDelegate** — e.g. `(HttpContext context) => { ... }`. In practice, this is rarely used.

### Adding the Employee Class

Add a new file `Employee.cs`:

```csharp
public class Employee
{
    public int Id { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
}
```

The compiler complains about `FirstName` and `LastName` not being initialized. We could mark them as nullable with `?`, but employees have names and they're not nullable. Mark them as `required`:

```csharp
public class Employee
{
    public int Id { get; set; }
    public required string FirstName { get; set; }
    public required string LastName { get; set; }
}
```

🌶️🌶️🌶️ Nullability is relatively new to .NET and the real world is filled with projects where it's simply impractical to rewrite all code to be non-nullable. Spencer uses nullability in this course to model best practices, but with eyes wide open knowing not all real-world projects do.

Treat all warnings as errors by editing the `.csproj` file:

```xml
<TreatWarningsAsErrors>true</TreatWarningsAsErrors>
```

### Creating and Seeding the List

```csharp
var employees = new List<Employee>();
employees.Add(new Employee { Id = 1, FirstName = "John", LastName = "Doe" });
employees.Add(new Employee { Id = 2, FirstName = "Jane", LastName = "Doe" });
```

### Adding Endpoints

**GET all employees:**

```csharp
app.MapGet("/employees", () => {
    return Results.Ok(employees);
});
```

`Results` is a handy class that contains methods representing HTTP status codes you can return from your APIs.

🌶️🌶️🌶️ Spencer NEVER EVER returns anything that isn't tied directly to an HTTP status code.

**GET single employee:**

```csharp
app.MapGet("/employees/{id:int}", (int id) => {
    var employee = employees.SingleOrDefault(e => e.Id == id);
    if (employee == null)
    {
        return Results.NotFound();
    }
    return Results.Ok(employee);
});
```

The `{id:int}` syntax is a **route constraint** — it ensures the `id` parameter must be an integer.

**POST create employee:**

```csharp
app.MapPost("/employees", (Employee employee) => {
    employee.Id = employees.Max(e => e.Id) + 1; // Manually assign an ID since we're not using a DB
    employees.Add(employee);
    return Results.Created($"/employees/{employee.Id}", employee);
});
```

Sending an empty Employee object results in a `400 Bad Request` with a messy stack trace. We'll handle that better as the course progresses.

### Route Groups

Route groups allow grouping related routes together and sharing behaviors across them:

```csharp
var employeeRoute = app.MapGroup("employees");
```

Refactored endpoints:

```csharp
employeeRoute.MapGet(string.Empty, () => {
    return employees;
});

employeeRoute.MapGet("{id:int}", (int id) => {
    var employee = employees.SingleOrDefault(e => e.Id == id);
    if (employee == null)
    {
        return Results.NotFound();
    }
    return Results.Ok(employee);
});

employeeRoute.MapPost(string.Empty, (Employee employee) => {
    employee.Id = employees.Max(e => e.Id) + 1;
    employees.Add(employee);
    return Results.Created($"/employees/{employee.Id}", employee);
});
```

### Parameter Binding in Minimal APIs

In the second argument of `MapGet`/`MapPost`/etc., ASP.NET Core binds parameters in this specific order:

1. Parameters using `[Bind/From]` attributes
2. Special types like `HttpContext` and `CancellationToken`
3. Route parameters / query string
4. Services from the DI container
5. Body of the request

**Binding attributes example:**

```csharp
app.MapPost("/employees/{id}", (
    [FromRoute] int id,
    [FromBody] Employee employee,
    [FromServices] ILogger<Program> logger,
    [FromQuery] string search) =>
{
    return Results.Ok(employee);
});
```

Each attribute binds the parameter from a different part of the request:

- `[FromRoute]` — from the route
- `[FromBody]` — from the request body
- `[FromServices]` — from the DI container
- `[FromQuery]` — from the query string

🌶️🌶️🌶️ Spencer always recommends using binding attributes. They make code more explicit and remove guesswork.

---

## Lesson — Writing Our First Tests

---

### Why Write Tests?

- **Verify code behaves as expected** — catch bugs before they reach production.
- **Enable confident refactoring** — change code without fear of breaking things.
- **Create a de-facto spec** — tests document expected behavior.

### Setup Requirements

1. Add an XUnit test project to the solution.
2. Install the testing package:
   ```bash
   dotnet add package Microsoft.AspNetCore.Mvc.Testing
   ```
3. Modify the API project to expose `Program` to the test project by declaring it as a partial public class:
   ```csharp
   public partial class Program { }
   ```

### Test Class Structure

Using `IClassFixture<WebApplicationFactory<Program>>` instructs XUnit to create an instance of `WebApplicationFactory<Program>` for each test class and dispose of it after. This provides a `WebApplication` instance and allows creating test `HttpClient` instances.

```csharp
public class EmployeeTests : IClassFixture<WebApplicationFactory<Program>>
{
    private readonly WebApplicationFactory<Program> _factory;

    public EmployeeTests(WebApplicationFactory<Program> factory)
    {
        _factory = factory;
    }
}
```

### Example Tests

Four test cases:

- **GetAllEmployees** — `GET /employees` returns success status code
- **GetEmployeeById** — `GET /employees/{id}` returns the correct employee
- **CreateEmployee (Success)** — valid POST returns 201 Created
- **CreateEmployee (Failure)** — invalid POST (empty object to trigger validation) returns 400 Bad Request

```csharp
[Fact]
public async Task CreateEmployee_ReturnsBadRequestResult()
{
    var client = _factory.CreateClient();
    var invalidEmployee = new CreateEmployeeRequest(); // Empty object triggers validation errors

    var response = await client.PostAsJsonAsync("/employees", invalidEmployee);

    Assert.Equal(HttpStatusCode.BadRequest, response.StatusCode);
}
```

Run tests with: `dotnet test`

---

## Lesson — Refactoring to Request Classes and Adding a PUT Request

---

### The Problem

When new security requirements emerge — such as preventing updates to sensitive fields like social security numbers — modifying the `Employee` entity directly becomes messy and leads to a mismatch between the database schema and the API contract.

### The Solution: Separate Request/Response Classes

> "Things that serve a singular purpose are easier to understand, test, and maintain."

Three distinct classes replace the single `Employee` entity in the API layer:

| Class | Purpose |
|-------|---------|
| `CreateEmployeeRequest` | For POST operations — includes all fields including SSN |
| `UpdateEmployeeRequest` | For PUT operations — only allows updating address and contact info, not names or SSN |
| `GetEmployeeResponse` | For GET operations — excludes SSN and other sensitive identity fields |

### Key Benefit

This separation allows the API contract and database schema to evolve independently. The refactored endpoints:

- Accept specific request types instead of the full `Employee` entity
- Return `GetEmployeeResponse` objects that exclude sensitive data
- Map explicitly between storage and API layers

All existing tests continue to pass after this refactor, confirming no regression.

---

## Lesson — Refactoring to the Repository Pattern

---

### Design Patterns Discussed

- **Repository** — separates data storage from data access
- **Factory** — separates object creation from usage
- **Unit of Work** — groups multiple operations into a single unit

### What is the Repository Pattern?

The Repository pattern separates the concerns of data storage (e.g. a database) from data access logic. Problems with the initial in-memory list approach:

- Data isn't persisted between requests
- `List<T>` methods don't model real database operations well

### Implementation Steps

**1. Create a Generic Repository Interface:**

```csharp
public interface IRepository<T>
{
    T? GetById(int id);
    IEnumerable<T> GetAll();
    T Create(T entity);
    T Update(T entity);
    void Delete(T entity);
}
```

**2. Implement EmployeeRepository:**

The concrete implementation wraps the in-memory list with ID generation logic and CRUD operations.

**3. Register and Inject:**

```csharp
builder.Services.AddSingleton<IRepository<Employee>, EmployeeRepository>();
```

Endpoints receive the interface as a parameter rather than a direct implementation:

```csharp
employeeRoute.MapGet(string.Empty, (IRepository<Employee> repository) => {
    return Results.Ok(repository.GetAll());
});
```

### Best Practices

- Use interfaces (`IRepository<Employee>`) rather than concrete implementations in dependency injection.
- This enables swapping implementations — in-memory for tests, real database for production.
- Start with functional code first, then refactor into patterns.

### Testing Note

Tests failed initially because the repository wasn't registered in the test factory. The test constructor creates initial employee data for testing purposes.

---

## Lesson — Adding Validation to Your API

---

### Built-in Data Annotations

The `System.ComponentModel.DataAnnotations` namespace provides validation attributes:

| Attribute | Description | Example |
|-----------|-------------|---------|
| `[Required]` | Ensures the property is not null or empty | `[Required] public string FirstName { get; set; }` |
| `[StringLength]` | Validates string length within a range | `[StringLength(10, MinimumLength = 3)] public string FirstName { get; set; }` |
| `[Range]` | Validates a numeric value is within bounds | `[Range(18, 65)] public int Age { get; set; }` |
| `[EmailAddress]` | Validates email format | `[EmailAddress] public string Email { get; set; }` |
| `[Phone]` | Validates phone number format | `[Phone] public string Phone { get; set; }` |
| `[Url]` | Validates URL format | `[Url] public string Website { get; set; }` |

### What Happens Without Validation

Posting an invalid (empty) object returns a nasty stack trace — not something clients can easily interpret, and a security risk in production:

```
Microsoft.AspNetCore.Http.BadHttpRequestException: Failed to read parameter "CreateEmployeeRequest employee" from the request body as JSON.
 ---> System.Text.Json.JsonException: JSON deserialization for type 'CreateEmployeeRequest' was missing required properties...
```

### Adding ProblemDetails

Register ProblemDetails for better structured error responses:

```csharp
builder.Services.AddProblemDetails();
```

### Implementing Data Annotations on the Request Class

Properties are marked nullable so the JSON parser allows empty strings — letting validation logic handle the errors rather than the runtime:

```csharp
public class CreateEmployeeRequest
{
    [Required(AllowEmptyStrings = false)]
    public string? FirstName { get; set; }
    [Required(AllowEmptyStrings = false)]
    public string? LastName { get; set; }
    public string? SocialSecurityNumber { get; set; }
    public string? Address1 { get; set; }
    public string? Address2 { get; set; }
    public string? City { get; set; }
    public string? State { get; set; }
    public string? ZipCode { get; set; }
    public string? PhoneNumber { get; set; }
    public string? Email { get; set; }
}
```

### Validating in the Endpoint

```csharp
employeeRoute.MapPost(string.Empty, (CreateEmployeeRequest employeeRequest, IRepository<Employee> repository) => {
    var validationProblems = new List<ValidationResult>();
    var isValid = Validator.TryValidateObject(employeeRequest, new ValidationContext(employeeRequest), validationProblems, true);
    if (!isValid)
    {
        return Results.BadRequest(validationProblems.ToValidationProblemDetails());
    }

    var newEmployee = new Employee {
        FirstName = employeeRequest.FirstName!,
        LastName = employeeRequest.LastName!,
        // ...other properties
    };
    repository.Create(newEmployee);
    return Results.Created($"/employees/{newEmployee.Id}", employeeRequest);
});
```

### Extension Method: Converting to ValidationProblemDetails

```csharp
public static class Extensions
{
    public static ValidationProblemDetails ToValidationProblemDetails(this List<ValidationResult> validationResults)
    {
        var problemDetails = new ValidationProblemDetails();

        foreach (var validationResult in validationResults)
        {
            foreach (var memberName in validationResult.MemberNames)
            {
                if (problemDetails.Errors.ContainsKey(memberName))
                {
                    problemDetails.Errors[memberName] = problemDetails.Errors[memberName]
                        .Concat([validationResult.ErrorMessage]).ToArray()!;
                }
                else
                {
                    problemDetails.Errors[memberName] = new List<string> { validationResult.ErrorMessage! }.ToArray();
                }
            }
        }

        return problemDetails;
    }
}
```

This returns a proper `ProblemDetails` format:

```json
{
    "type": "https://tools.ietf.org/html/rfc9110#section-15.5.1",
    "title": "One or more validation errors occurred.",
    "status": 400,
    "errors": {
        "LastName": [
            "The LastName field is required."
        ]
    }
}
```

### Updated Test

```csharp
[Fact]
public async Task CreateEmployee_ReturnsBadRequestResult()
{
    var client = _factory.CreateClient();
    var invalidEmployee = new CreateEmployeeRequest();

    var response = await client.PostAsJsonAsync("/employees", invalidEmployee);

    Assert.Equal(HttpStatusCode.BadRequest, response.StatusCode);

    var problemDetails = await response.Content.ReadFromJsonAsync<ValidationProblemDetails>();
    Assert.NotNull(problemDetails);
    Assert.Contains("FirstName", problemDetails.Errors.Keys);
    Assert.Contains("LastName", problemDetails.Errors.Keys);
    Assert.Contains("The FirstName field is required.", problemDetails.Errors["FirstName"]);
    Assert.Contains("The LastName field is required.", problemDetails.Errors["LastName"]);
}
```

🌶️🌶️🌶️ In the real world, testing validation on every property of every request is largely a waste of time. Identify the most important behaviors to validate and focus on those first.

### Why Spencer Doesn't Like Built-in Validation

1. A bit inflexible.
2. Custom validation logic is cumbersome — involves implementing `IValidatableObject`.
3. The validation logic itself is convoluted:
   ```csharp
   var isValid = Validator.TryValidateObject(employeeRequest, new ValidationContext(employeeRequest), validationProblems, true);
   ```
4. The validation pipeline for .NET doesn't support async validation.

### FluentValidation (Preferred Approach)

[FluentValidation](https://docs.fluentvalidation.net/en/latest/) solves all of the above:

- Separate validation classes from request classes
- Supports both sync and async validation
- Clean, truly fluent API

**Installation:**

```bash
dotnet add package FluentValidation
dotnet add package FluentValidation.DependencyInjectionExtensions
```

**Create a validator:**

```csharp
using FluentValidation;

public class CreateEmployeeRequestValidator : AbstractValidator<CreateEmployeeRequest>
{
    public CreateEmployeeRequestValidator()
    {
        RuleFor(x => x.FirstName).NotEmpty();
        RuleFor(x => x.LastName).NotEmpty();
    }
}
```

**Register all validators from the assembly:**

```csharp
builder.Services.AddValidatorsFromAssemblyContaining<Program>();
```

**Use in the endpoint:**

```csharp
employeeRoute.MapPost(string.Empty, async (CreateEmployeeRequest employeeRequest,
    IRepository<Employee> repository, IValidator<CreateEmployeeRequest> validator) =>
{
    var validationResults = await validator.ValidateAsync(employeeRequest);
    if (!validationResults.IsValid)
    {
        return Results.ValidationProblem(validationResults.ToDictionary());
    }

    var newEmployee = new Employee {
        FirstName = employeeRequest.FirstName!,
        LastName = employeeRequest.LastName!,
        // ...other properties
    };
    repository.Create(newEmployee);
    return Results.Created($"/employees/{newEmployee.Id}", employeeRequest);
});
```

**Updated test (note the different error message format from FluentValidation):**

```csharp
Assert.Contains("'First Name' must not be empty.", problemDetails.Errors["FirstName"]);
Assert.Contains("'Last Name' must not be empty.", problemDetails.Errors["LastName"]);
```

### Advanced FluentValidation Features

**Async validation (e.g., SSN uniqueness):**

```csharp
public class CreateEmployeeRequestValidator : AbstractValidator<CreateEmployeeRequest>
{
    private readonly EmployeeRepository _repository;

    public CreateEmployeeRequestValidator(EmployeeRepository repository)
    {
        this._repository = repository;

        RuleFor(x => x.FirstName).NotEmpty();
        RuleFor(x => x.LastName).NotEmpty();
        RuleFor(x => x.SocialSecurityNumber)
            .Cascade(CascadeMode.Stop)
            .NotEmpty().WithMessage("SSN cannot be empty.")
            .MustAsync(BeUnique).WithMessage("SSN must be unique.");
    }

    private async Task<bool> BeUnique(string ssn, CancellationToken token)
    {
        return await _repository.GetEmployeeBySsn(ssn) != null;
    }
}
```

**Conditional validation using `When()`:**

```csharp
When(r => r.Address1 != null, () => {
    RuleFor(x => x.Address1).NotEmpty();
    RuleFor(x => x.City).NotEmpty();
    RuleFor(x => x.State).NotEmpty();
    RuleFor(x => x.ZipCode).NotEmpty();
});
```

---

## Lesson — Reviewing Everything We've Done

---

In this section, we covered:

- Learned about HTTP
- Learned about ASP.NET Core and how it handles HTTP endpoints
- How to create GET, PUT, and POST requests using the minimal API paradigm
- How to write tests for ASP.NET Core
- The repository pattern and how to implement a basic version of it
- How to add validation using both built-in attributes and FluentValidation

---

*Content licensed under CC-BY-NC-4.0 | Code samples under Apache 2.0*
