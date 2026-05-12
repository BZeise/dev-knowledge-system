# Controllers and Advanced API Topics

*From "Building APIs with C# & ASP.NET Core" by Spencer Schneidenbach (Frontend Masters)*

---

## Lesson — Introducing Controllers

---

### What is a Controller?

A controller is **a class that handles HTTP requests**. Minimal APIs have limitations compared to controllers:

- Less flexibility
- Increased boilerplate as projects grow
- Lower prevalence in production code
- Less clean dependency management

🌶️🌶️🌶️ Spencer recommends using controllers for any solution anticipated to exceed approximately 10 endpoints.

### What Does a Controller Look Like?

```csharp
[ApiController]
[Route("[controller]")]
public class CustomersController : Controller
{
    [HttpGet]
    public IActionResult GetCustomers()
    {
        return Ok(_repo.GetAll());
    }

    [HttpGet("{id}")]
    public IActionResult GetCustomer(int id)
    {
        var customer = _repo.GetById(id);
        if (customer == null) return NotFound();
        return Ok(customer);
    }
}
```

Controllers inherit from `Controller` or `ControllerBase`.

### Key Annotations

**`[ApiController]`** — Enables special behaviors:
- Requires attribute-based routing
- Returns `ProblemDetails` for error codes
- Automatically validates model binding and returns 400 for invalid objects
- Infers parameter binding (no need for explicit `[FromBody]` in most cases)

**`[Route("[controller]")]`** — Routes requests using the controller name (without the "Controller" suffix). `CustomersController` becomes `/customers`.

**`[HttpGet("{id}")]`** — Marks a method as a GET endpoint with an optional route parameter.

### How Do You Make ASP.NET Core Use Controllers?

In `Program.cs`:

```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllers();

var app = builder.Build();
app.MapControllers();
app.Run();
```

---

## Lesson — Converting Our Existing APIs to Controllers

---

### Creating a Base Controller

Create an abstract base controller that all other controllers inherit from, applying shared attributes once:

```csharp
[ApiController]
[Route("api/[controller]")]
public abstract class BaseController : ControllerBase
{
}
```

### Building EmployeesController

The controller gets the repository injected via constructor and implements four HTTP operations:

- `GET /api/employees` — get all employees
- `GET /api/employees/{id}` — get employee by ID
- `POST /api/employees` — create employee
- `PUT /api/employees/{id}` — update employee

### Project Structure: Horizontal vs Vertical Slicing

The instructor prefers **horizontal slicing** — grouping all employee-related files together (Employee, EmployeesController, CreateEmployeeRequest, etc.) rather than the conventional Microsoft approach of separate Controllers/, Models/, DTOs/ folders.

Both are valid — pick one and be consistent.

### Reducing Validation Boilerplate

**Initial approach:** inject `IValidator<CreateEmployeeRequest>` into the controller constructor and call it directly.

**Improved approach:** move validation to a generic method in `BaseController` that dynamically retrieves validators from DI:

```csharp
protected async Task<IActionResult?> ValidateAsync<T>(T request)
{
    var validator = HttpContext.RequestServices.GetService<IValidator<T>>();
    if (validator == null) return null;

    var result = await validator.ValidateAsync(request);
    if (!result.IsValid)
    {
        result.AddToModelState(ModelState);
        return ValidationProblem();
    }
    return null;
}
```

This removes the need to inject validators into every controller.

### Property Mapping Note

Mapping properties between entities and response objects can be verbose. **AutoMapper** is a common library that automates this transformation, though it's not covered in depth here.

### Essential Setup

Enable controller routing in `Program.cs` before `app.Run()`:

```csharp
app.MapControllers();
```

---

## Lesson — Adding Logging to Our Controllers

---

### Why Logging?

Logging helps in tracking the flow of the application, troubleshooting issues, and monitoring the overall health of the system.

### ILogger\<T\>

In .NET, logging is handled through the `ILogger<T>` interface, injectable into any class. The `T` provides context about which part of the application is generating log messages.

```csharp
public class MyService
{
    private readonly ILogger<MyService> _logger;

    public MyService(ILogger<MyService> logger)
    {
        _logger = logger;
    }

    public void DoWork()
    {
        _logger.LogInformation("Starting work...");
    }
}
```

### Log Levels (Severity Order)

| Level | Purpose | Example |
|-------|---------|---------|
| **Trace** | Highly detailed debugging info | "Starting DoWork method" |
| **Debug** | Development diagnostics | "DoWork processed employee 123456" |
| **Information** | General application flow | "DoWork method completed successfully" |
| **Warning** | Unexpected but handled | "FileNotFoundException for file log.txt, but continued processing" |
| **Error** | Application can't function as expected | "Failed to connect to database" |
| **Critical** | Failure requiring immediate attention | "Application is shutting down due to unrecoverable error" |

```csharp
_logger.LogTrace("This is a trace log.");
_logger.LogDebug("This is a debug log.");
_logger.LogInformation("This is an informational log.");
_logger.LogWarning("This is a warning log.");
_logger.LogError("This is an error log.");
_logger.LogCritical("This is a critical log.");
```

### Implementing Logging in EmployeesController

```csharp
public class EmployeesController : BaseController
{
    private readonly IRepository<Employee> _repository;
    private readonly ILogger<EmployeesController> _logger;

    public EmployeesController(IRepository<Employee> repository, ILogger<EmployeesController> logger)
    {
        _repository = repository;
        _logger = logger;
    }
```

Adding logging to the `UpdateEmployee` method:

```csharp
[HttpPut("{id}")]
public IActionResult UpdateEmployee(int id, [FromBody] UpdateEmployeeRequest employeeRequest)
{
    _logger.LogInformation("Updating employee with ID: {EmployeeId}", id);

    var existingEmployee = _repository.GetById(id);
    if (existingEmployee == null)
    {
        _logger.LogWarning("Employee with ID: {EmployeeId} not found", id);
        return NotFound();
    }

    _logger.LogDebug("Updating employee details for ID: {EmployeeId}", id);
    existingEmployee.Address1 = employeeRequest.Address1;
    // ...other property updates

    try
    {
        _repository.Update(existingEmployee);
        _logger.LogInformation("Employee with ID: {EmployeeId} successfully updated", id);
        return Ok(existingEmployee);
    }
    catch (Exception ex)
    {
        _logger.LogError(ex, "Error occurred while updating employee with ID: {EmployeeId}", id);
        return StatusCode(500, "An error occurred while updating the employee");
    }
}
```

**Key notes:**
1. Use structured logging with `{EmployeeId}` placeholder — that becomes a searchable field in logging systems.
2. Log exceptions with context to provide more information about where errors occurred.

### Testing That Logging Occurs

Using Moq to verify logging behavior:

```csharp
[Fact]
public async Task UpdateEmployee_LogsAndReturnsOkResult()
{
    var loggerMock = new Mock<ILogger<EmployeesController>>().SetupAllProperties();
    var repositoryMock = new Mock<IEmployeeRepository>();
    var controller = new EmployeesController(loggerMock.Object, repositoryMock.Object);

    var employeeId = 1;
    var updateRequest = new UpdateEmployeeRequest { FirstName = "John", LastName = "Doe" };

    repositoryMock.Setup(r => r.GetById(employeeId)).Returns(new Employee { Id = employeeId });

    var result = controller.UpdateEmployee(employeeId, updateRequest);

    Assert.IsType<OkObjectResult>(result);
    loggerMock.VerifyLog(logger => logger.LogInformation("Updating employee with ID: {EmployeeId}", employeeId));
    loggerMock.VerifyLog(logger => logger.LogInformation("Employee with ID: {EmployeeId} successfully updated", employeeId));
}
```

(`ILogger.Moq` extension method is needed to verify against the underlying `ILogger.Log` method cleanly.)

🌶️🌶️🌶️ Would Spencer bother doing this? Yes, in the case where a business-critical process needs to log and the behavior should never change. Not something done most of the time though.

---

## Lesson — Middleware and Filters

---

### Reviewing Middleware

Middleware are the things that handle HTTP requests and responses in ASP.NET Core. They execute in the order they're added to the pipeline — order is **extremely important**.

```csharp
app.Use(async (HttpContext context, RequestDelegate next) =>
{
    context.Response.Headers.Append("Content-Type", "text/html");
    await context.Response.WriteAsync("<h1>Welcome to Ahoy!</h1>");
});
```

Try moving middleware to different positions in the pipeline. If placed after `MapControllers()`, a valid controller endpoint call will not execute the middleware.

### Standard Middleware Order

```csharp
app.UseHttpsRedirection();
app.UseStaticFiles();

app.UseRouting();

app.UseAuthentication();
app.UseAuthorization();

app.MapRazorPages();
app.MapDefaultControllerRoute();
```

### Does Spencer Write Custom Middleware?

Not often, but it's useful for things like multi-tenant SaaS products where each tenant is identified by a different subdomain:

```csharp
app.Use(async (HttpContext context, RequestDelegate next) =>
{
    var tenant = context.Request.Host.Host.Split('.')[0];
    var tenantId = await _tenantService.GetTenantIdBySubdomain(tenant);
    context.Items["TenantId"] = tenantId;
    await next(context);
});
```

### Filters

Filters modify request behavior in ASP.NET Core **after** the request has been routed to a controller method. This is their key advantage over middleware: **filters engage after route matching and model binding have occurred.**

Think of filters as micro-middleware invoked after all main middleware run and a route has been matched.

Filters can be applied globally, or on specific controllers or actions.

| Filter Type | When It Executes | Where It Can Be Defined |
|-------------|-----------------|------------------------|
| Authorization | Before the action is invoked | Global or per controller |
| Resource | Before and after the action | Global, controller, or action |
| Action | Before and after the action | Global, controller, or action |
| Exception | When an exception is thrown | Global or per controller |
| Result | After the action is invoked | Global, controller, or action |

**Common built-in filters:**

- `[Authorize]` — Checks for valid auth tokens
- `[EnableCors]` — Enables CORS
- `[ResponseCache]` — Adds response caching
- `[RequireHttps]` — Redirects to HTTPS
- `[ValidateAntiForgeryToken]` — Validates anti-forgery tokens

### Writing a Custom FluentValidation Filter

The `[ApiController]` attribute is supposed to trigger validation behavior, but **it doesn't work well with FluentValidation because it only supports synchronous validation**. (See [GitHub issue #31905](https://github.com/dotnet/aspnetcore/issues/31905) — filed by the creator of FluentValidation.)

Solution: a custom `IAsyncActionFilter`:

```csharp
public class FluentValidationFilter : IAsyncActionFilter
{
    private readonly IServiceProvider _serviceProvider;
    private readonly ProblemDetailsFactory _problemDetailsFactory;

    public FluentValidationFilter(IServiceProvider serviceProvider, ProblemDetailsFactory problemDetailsFactory)
    {
        _serviceProvider = serviceProvider;
        _problemDetailsFactory = problemDetailsFactory;
    }

    public async Task OnActionExecutionAsync(ActionExecutingContext context, ActionExecutionDelegate next)
    {
        foreach (var parameter in context.ActionDescriptor.Parameters)
        {
            if (context.ActionArguments.TryGetValue(parameter.Name, out var argumentValue) && argumentValue != null)
            {
                var argumentType = argumentValue.GetType();
                var validatorType = typeof(IValidator<>).MakeGenericType(argumentType);
                var validator = _serviceProvider.GetService(validatorType) as IValidator;

                if (validator != null)
                {
                    ValidationResult validationResult = await validator.ValidateAsync(
                        new ValidationContext<object>(argumentValue));

                    if (!validationResult.IsValid)
                    {
                        validationResult.AddToModelState(context.ModelState);
                        var problemDetails = _problemDetailsFactory.CreateValidationProblemDetails(
                            context.HttpContext, context.ModelState);
                        context.Result = new BadRequestObjectResult(problemDetails);
                        return;
                    }
                }
            }
        }

        await next();
    }

    public void OnActionExecuted(ActionExecutedContext context) { }
}
```

What this filter does:
- Implements `IAsyncActionFilter` for async filter behavior
- Gets each parameter of the controller action from `context.ActionDescriptor.Parameters` — this is why filters are better than middleware here; middleware has no awareness of controller actions or parameters
- Uses `IServiceProvider` to get the `IValidator` for the current parameter type
- Runs validation and sets a `BadRequestObjectResult` if validation fails

Register globally:

```csharp
builder.Services.AddControllers(options =>
{
    options.Filters.Add<FluentValidationFilter>();
});
```

After registering the filter, you can delete the `ValidateAsync` call from controller actions and the helper method from `BaseController`.

🌶️🌶️🌶️ While you're free to use this class, Spencer wouldn't recommend it as it's pretty simplistic. There's a NuGet package called [FluentValidation.AutoValidation](https://github.com/SharpGrip/FluentValidation.AutoValidation) that does the same thing but much more comprehensively.

### Refactoring the PUT Endpoint with Advanced Validation

A validator that prevents nulling out an `Address` field that already has a value:

```csharp
public class UpdateEmployeeRequestValidator : AbstractValidator<UpdateEmployeeRequest>
{
    private readonly HttpContext _httpContext;
    private readonly IRepository<Employee> _repository;

    public UpdateEmployeeRequestValidator(IHttpContextAccessor httpContextAccessor, IRepository<Employee> repository)
    {
        this._httpContext = httpContextAccessor.HttpContext!;
        this._repository = repository;

        RuleFor(x => x.Address1)
            .MustAsync(NotBeEmptyIfItIsSetOnEmployeeAlreadyAsync)
            .WithMessage("Address1 must not be empty.");
    }

    private async Task<bool> NotBeEmptyIfItIsSetOnEmployeeAlreadyAsync(string? address, CancellationToken token)
    {
        await Task.CompletedTask;

        var id = Convert.ToInt32(_httpContext.Request.RouteValues["id"]);
        var employee = _repository.GetById(id);

        if (employee!.Address1 != null && string.IsNullOrWhiteSpace(address))
        {
            return false;
        }

        return true;
    }
}
```

- `IHttpContextAccessor` — gets the current `HttpContext` to access the employee ID from the route.
- `IRepository<Employee>` — fetches the employee from the database.
- `Convert.ToInt32(_httpContext.Request.RouteValues["id"])` — extracts the `id` from the route.
- `MustAsync` — runs custom validation logic asynchronously.

This validator is automatically registered by FluentValidation's `AddValidatorsFromAssemblyContaining<Program>()`. The only additional registration needed:

```csharp
builder.Services.AddHttpContextAccessor();
```

**Tests for the refactored PUT endpoint:**

```csharp
[Fact]
public async Task UpdateEmployee_ReturnsOkResult()
{
    var client = _factory.CreateClient();
    var response = await client.PutAsJsonAsync("/employees/1",
        new Employee { FirstName = "John", LastName = "Doe", Address1 = "123 Main St" });

    response.EnsureSuccessStatusCode();
}

[Fact]
public async Task UpdateEmployee_ReturnsBadRequestWhenAddress()
{
    var client = _factory.CreateClient();
    var invalidEmployee = new UpdateEmployeeRequest(); // Empty object to trigger validation

    var response = await client.PutAsJsonAsync($"/employees/{_employeeId}", invalidEmployee);

    Assert.Equal(HttpStatusCode.BadRequest, response.StatusCode);

    var problemDetails = await response.Content.ReadFromJsonAsync<ValidationProblemDetails>();
    Assert.NotNull(problemDetails);
    Assert.Contains("Address1", problemDetails.Errors.Keys);
}
```

---

## Lesson — Configuring Controllers for OpenAPI

---

### The Problem

After removing minimal APIs, the OpenAPI documentation loses descriptions and tags that Swagger previously auto-generated.

### ProducesResponseType Attributes

Add response type metadata to controller endpoints:

```csharp
[HttpGet]
[ProducesResponseType(typeof(IEnumerable<GetEmployeeResponse>), StatusCodes.Status200OK)]
[ProducesResponseType(StatusCodes.Status500InternalServerError)]
public async Task<IActionResult> GetEmployees() { ... }
```

This tells OpenAPI which response types each endpoint can return.

### XML Comments for Documentation

XML comments add human-readable descriptions to endpoints.

**1. Enable documentation file generation in `.csproj`:**

```xml
<PropertyGroup>
    <GenerateDocumentationFile>true</GenerateDocumentationFile>
    <NoWarn>$(NoWarn);1591</NoWarn>
</PropertyGroup>
```

**2. Configure Swagger to include the XML file in `Program.cs`:**

```csharp
builder.Services.AddSwaggerGen(options =>
{
    options.IncludeXmlComments(
        Path.Combine(AppContext.BaseDirectory, "TheEmployeeAPI.xml"));
});
```

XML comments and OpenAPI create a mostly seamless integration for documenting REST API endpoints, allowing comprehensive documentation to be generated automatically from code.

---

## Lesson — Querying Sub-Resources

---

### Sub-Resources

Sub-resources are related data attached to a primary resource — e.g., an employee's benefits. Nested resources are well-supported by OpenAPI and most API clients.

### Two Approaches

| Approach | When to Use |
|----------|------------|
| Include with the parent object | Best for small collections (e.g., addresses) |
| Query separately via a dedicated endpoint | Better for large collections (e.g., orders, benefits) |

### New Classes for Employee Benefits

```csharp
public class EmployeeBenefits
{
    public int Id { get; set; }
    public int EmployeeId { get; set; }
    public BenefitType BenefitType { get; set; }
    public decimal Cost { get; set; }
}

public enum BenefitType
{
    Health,
    Dental,
    Vision
}
```

### Option 1: Include with the Original Object

Modify `GetEmployeeResponse` to include a `Benefits` list. Add a factory method to avoid duplication:

```csharp
private static GetEmployeeResponse EmployeeToGetEmployeeResponse(Employee e)
{
    return new GetEmployeeResponse
    {
        Id = e.Id,
        FirstName = e.FirstName,
        LastName = e.LastName,
        Benefits = e.Benefits.Select(BenefitToBenefitResponse).ToList()
    };
}
```

### Option 2: Separate Endpoint

Creates a dedicated endpoint for the sub-resource using a nested route pattern:

```
GET /employees/{employeeId}/benefits
```

```csharp
[HttpGet("{employeeId}/benefits")]
public IActionResult GetBenefitsForEmployee(int employeeId)
{
    var employee = _repository.GetById(employeeId);
    if (employee == null) return NotFound();

    var benefits = employee.Benefits.Select(BenefitToBenefitResponse);
    return Ok(benefits);
}

private static GetEmployeeResponseEmployeeBenefit BenefitToBenefitResponse(EmployeeBenefits b)
{
    return new GetEmployeeResponseEmployeeBenefit
    {
        Id = b.Id,
        BenefitType = b.BenefitType,
        Cost = b.Cost
    };
}
```

Factory methods like `EmployeeToGetEmployeeResponse` and `BenefitToBenefitResponse` eliminate duplication and maintain clean separation between entities and response objects.

---

*Content licensed under CC-BY-NC-4.0 | Code samples under Apache 2.0*
