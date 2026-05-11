# Hosting Model and API Development

*From "C# and .NET: From Beginner to Professional" by Spencer Schneidenbach (Frontend Masters)*

## Generic Host and Web Host Model Overview

Modern applications need foundational infrastructure beyond syntax and logic.

### Common Application Requirements

Three essential elements that real-world applications require:

1. **Logging** — Essential for monitoring and debugging, capturing runtime information and errors
2. **Configuration** — Application settings and values specific to different environments
3. **Dependency Injection** — Promotes modularity and testability by managing how dependencies are provided to classes

### The Problem

Before ASP.NET Core standardized these patterns, developers implemented each piece separately, often resulting in inconsistent approaches across different projects.

### The Solution: Hosting Model

ASP.NET Core's hosting framework provides **built-in support for logging, configuration, and dependency injection** to reduce repetitive code and ensure consistency.

### Historical Context

The shift from developers building these systems from scratch to using a standardized framework was transformative. Now applications are scaffolded with these concerns already configured.

---

## Configuration in .NET

Managing application settings across environments.

### Core Concept

.NET's built-in configuration system uses the `IConfiguration` interface to manage application settings from multiple sources in a flexible, environment-aware manner.

### Configuration Providers

**Three Primary Sources:**

1. **appsettings.json** — Standard JSON-based configuration file:
```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=myapp"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information"
    }
  },
  "ApiSettings": {
    "Timeout": 30,
    "MaxRetries": 3
  }
}
```

2. **Environment Variables** — Used for environment-specific overrides, particularly in production:
```bash
export ConnectionStrings__DefaultConnection="Server=prod-db;Database=myapp"
```

3. **User Secrets** — For local development to keep sensitive data (API keys, credentials) separate from version control:
```bash
dotnet user-secrets set "ApiKey" "secret-value-here"
```

### Accessing Configuration Values

Using dependency injection:

```csharp
public class MyService
{
    private readonly IConfiguration _configuration;
    
    public MyService(IConfiguration configuration)
    {
        _configuration = configuration;
    }
    
    public void UseSettings()
    {
        string connectionString = _configuration["ConnectionStrings:DefaultConnection"];
        int timeout = int.Parse(_configuration["ApiSettings:Timeout"]);
    }
}
```

Colons (`:`) denote nested structure in configuration keys.

### Strongly-Typed Configuration

Binding configuration sections to typed objects provides compile-time safety:

```csharp
public class ApiSettings
{
    public int Timeout { get; set; }
    public int MaxRetries { get; set; }
}

// In startup code
var apiSettings = configuration.GetSection("ApiSettings").Get<ApiSettings>();
// or
configuration.GetSection("ApiSettings").Bind(apiSettings);

// Usage
Console.WriteLine(apiSettings.Timeout);  // Type-safe
```

### ASP.NET Core Integration

The framework integrates configuration automatically through the Generic Host pattern, supporting:
- JSON files
- Environment variables
- Command-line arguments

No manual setup needed in most cases.

### Recommendation

Examine Microsoft's `CreateDefaultBuilder` source code to understand default behaviors across .NET versions.

---

## Dependency Injection (DI)

A design pattern for loose coupling and improved testability.

### Core Concept

**Supply an object's dependencies from outside** rather than creating them internally.

### Without DI (Tightly Coupled)

```csharp
public class OrderService
{
    private readonly SqlOrderRepository _repository = new();
    
    public void ProcessOrder(Order order)
    {
        _repository.Save(order);
    }
}
```

**Problems:**
- Tightly coupled to `SqlOrderRepository`
- Hard to test (can't substitute mock repository)
- Difficult to change database implementations

### With DI (Loosely Coupled)

```csharp
public interface IOrderRepository
{
    void Save(Order order);
}

public class OrderService
{
    private readonly IOrderRepository _repository;
    
    public OrderService(IOrderRepository repository)
    {
        _repository = repository;
    }
    
    public void ProcessOrder(Order order)
    {
        _repository.Save(order);
    }
}

// Registration (in Program.cs or Startup.cs)
services.AddScoped<IOrderRepository, SqlOrderRepository>();
services.AddScoped<OrderService>();

// Usage
OrderService service = serviceProvider.GetRequiredService<OrderService>();
```

### Key Interfaces

**IServiceCollection:** Registers services and defines their lifetimes.

**IServiceProvider:** Resolves registered services and injects them into constructors.

### Service Lifetimes

**Transient** — New instance every time requested:
```csharp
services.AddTransient<ILogger, FileLogger>();

var logger1 = serviceProvider.GetService<ILogger>();
var logger2 = serviceProvider.GetService<ILogger>();
// logger1 and logger2 are different instances
```

Use for: Lightweight, stateless services (single-request processors).

**Scoped** — One instance per request (or per scope):
```csharp
services.AddScoped<IUserContext, UserContext>();
```

Use for: Services maintaining state within a single request (database contexts in web applications).

**Singleton** — One instance for application lifetime:
```csharp
services.AddSingleton<IConfiguration>(configuration);
```

Use for: Expensive-to-create services like logging, configuration.

**Warning:** Avoid relying on changing state within singleton services due to potential unexpected behavior.

### Service Resolution Methods

**Constructor Injection** (most common):
```csharp
public class UserService
{
    public UserService(IUserRepository repository) { }
}
```

**Method Injection:**
```csharp
public void ProcessOrder(IOrderRepository repository) { }
```

**Property Injection** (less common):
```csharp
public class MyClass
{
    public ILogger Logger { get; set; }
}
```

### Manual Configuration Example

```csharp
var services = new ServiceCollection();
services.AddScoped<IOrderRepository, SqlOrderRepository>();
services.AddScoped<OrderService>();

var serviceProvider = services.BuildServiceProvider();
var orderService = serviceProvider.GetRequiredService<OrderService>();
```

---

## Logging in .NET

Tracking application flow and troubleshooting issues.

### Core Concept

Logging is a common and critical part of modern applications for:
- Tracking application flow
- Troubleshooting and debugging
- System health monitoring

### ILogger Interface

The primary logging mechanism uses `ILogger<T>`:

```csharp
public class UserService
{
    private readonly ILogger<UserService> _logger;
    
    public UserService(ILogger<UserService> logger)
    {
        _logger = logger;
    }
    
    public void CreateUser(User user)
    {
        _logger.LogInformation("Creating user: {UserName}", user.Name);
        // ... create user logic ...
        _logger.LogInformation("User created successfully");
    }
}
```

The generic type parameter provides context about which component generates messages.

### Log Levels (Severity Tiers)

From least to most severe:

1. **Trace** — Highly detailed debugging information
2. **Debug** — Development-focused details
3. **Information** — General application flow events
4. **Warning** — Unexpected occurrences handled gracefully
5. **Error** — Functional impairment scenarios
6. **Critical** — Unrecoverable failures requiring immediate intervention

```csharp
_logger.LogTrace("Entering method with parameters: {@Parameters}", parameters);
_logger.LogDebug("Database query executed in {Duration}ms", stopwatch.ElapsedMilliseconds);
_logger.LogInformation("User {UserId} logged in", userId);
_logger.LogWarning("Retry attempt {Attempt} of {MaxAttempts}", attempt, maxAttempts);
_logger.LogError(exception, "Failed to process order {OrderId}", orderId);
_logger.LogCritical("Database connection failed. System halting.");
```

### Built-in Log Providers

Available through ASP.NET Core's default setup:
- **Console** — Terminal output
- **Debug** — Visual Studio debug window
- **Windows Event Log** — For Windows applications
- **EventSource** — Performance tracing
- **TraceSource** — Legacy tracing

### File Logging

Microsoft doesn't provide native file logging. Third-party solutions recommended:

**Serilog:**
```csharp
Log.Logger = new LoggerConfiguration()
    .WriteTo.File("logs/log-.txt", rollingInterval: RollingInterval.Day)
    .CreateLogger();
```

**NLog:**
```csharp
LogManager.LoadConfiguration("nlog.config");
```

### Cloud Integration

For production systems, use centralized logging:
- **Azure Application Insights**
- **Datadog**
- **Splunk**
- **ELK Stack**

### Structured Logging

Templated messages with placeholders enable searchable queries:

```csharp
_logger.LogInformation(
    "Order {OrderId} processed by {UserId} with total {Total}",
    orderId,
    userId,
    total
);
```

Logs contain structured data that's easily queried and aggregated.

### Log Filtering

Control verbosity through `appsettings.json`:

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning",
      "MyApplication.Services": "Debug"
    }
  }
}
```

Reduces framework verbosity while keeping application logs detailed.

---

## Creating Your First API

Building a functional web API using .NET.

### Project Creation

```bash
dotnet new webapi -n MyFirstApi
cd MyFirstApi
```

This scaffolds a new API project with generated files:
- **Program.cs** — Application entry point and configuration
- **appsettings.json** — Configuration file
- **WeatherForecast.cs** — Sample model
- **WeatherForecastController.cs** — Sample endpoint

### Running the Application

```bash
dotnet run
```

Navigate to `https://localhost:5001/weatherforecast` to see sample data.

### Project Structure

```
MyFirstApi/
├── Program.cs                      # Entry point
├── WeatherForecast.cs             # Model
├── Controllers/
│   └── WeatherForecastController.cs  # API endpoint
├── appsettings.json               # Configuration
├── MyFirstApi.csproj              # Project file
└── Properties/
    └── launchSettings.json        # Debug configuration
```

### Basic Controller Example

```csharp
[ApiController]
[Route("[controller]")]
public class UsersController : ControllerBase
{
    [HttpGet("{id}")]
    public ActionResult<User> GetUser(int id)
    {
        var user = new User { Id = id, Name = "John" };
        return Ok(user);
    }
    
    [HttpPost]
    public ActionResult<User> CreateUser([FromBody] CreateUserRequest request)
    {
        var user = new User { Name = request.Name };
        return CreatedAtAction(nameof(GetUser), new { id = user.Id }, user);
    }
}
```

### Recommended Improvements

**1. Service Architecture**

Extract business logic into a separate service layer:

```csharp
public interface IUserService
{
    User GetUser(int id);
    User CreateUser(string name);
}

public class UserService : IUserService
{
    public User GetUser(int id) => /* ... */;
    public User CreateUser(string name) => /* ... */;
}

// Register in Program.cs
services.AddScoped<IUserService, UserService>();

// Use in controller
[ApiController]
[Route("[controller]")]
public class UsersController : ControllerBase
{
    private readonly IUserService _userService;
    
    public UsersController(IUserService userService)
    {
        _userService = userService;
    }
    
    [HttpGet("{id}")]
    public ActionResult<User> GetUser(int id)
    {
        var user = _userService.GetUser(id);
        return user != null ? Ok(user) : NotFound();
    }
}
```

**2. Logging Implementation**

Add logging for debugging and monitoring:

```csharp
public class UserService : IUserService
{
    private readonly ILogger<UserService> _logger;
    
    public UserService(ILogger<UserService> logger)
    {
        _logger = logger;
    }
    
    public User GetUser(int id)
    {
        _logger.LogInformation("Retrieving user {UserId}", id);
        var user = /* ... fetch from repository ... */;
        _logger.LogInformation("User {UserId} retrieved successfully", id);
        return user;
    }
}
```

### Best Practices

**HTTP Status Codes:**
- **200 OK** — Request succeeded
- **201 Created** — Resource created successfully
- **204 No Content** — Request succeeded, no response body
- **400 Bad Request** — Invalid request
- **404 Not Found** — Resource not found
- **500 Internal Server Error** — Server error

**Endpoint Response Examples:**
```csharp
// Success with data
return Ok(user);

// Created
return CreatedAtAction(nameof(GetUser), new { id = user.Id }, user);

// No content
return NoContent();

// Not found
return NotFound();

// Bad request
return BadRequest("Invalid input");

// Server error
return StatusCode(500, "Internal server error");
```

---

## ASP.NET Core Fundamentals

### Program.cs Structure

```csharp
var builder = WebApplication.CreateBuilder(args);

// Add services (dependency injection)
builder.Services.AddControllers();
builder.Services.AddScoped<IUserService, UserService>();

var app = builder.Build();

// Configure middleware pipeline
if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
}

app.UseHttpsRedirection();
app.UseAuthorization();

app.MapControllers();

app.Run();
```

### Middleware Pipeline

Middleware processes requests in a specific order:

```
Request → HTTPS Redirect → Routing → Authorization → Custom Logic → Response
```

### Common Middleware

```csharp
// Error handling
app.UseExceptionHandler("/error");

// HTTPS redirection
app.UseHttpsRedirection();

// Static files
app.UseStaticFiles();

// Routing
app.UseRouting();

// Authentication
app.UseAuthentication();

// Authorization
app.UseAuthorization();

// Custom middleware
app.UseMiddleware<LoggingMiddleware>();

app.MapControllers();
```

---

## Next Steps

Build production applications:
- [Design Patterns](../../02-concepts/design-patterns.md) — Real-world patterns in action
- [Testing](../../02-concepts/testing.md) — Ensure API reliability
- Review [SQL & Database](./sql-database.md) for data persistence

---

*Content licensed under CC-BY-NC-4.0 | Code samples under Apache 2.0*
