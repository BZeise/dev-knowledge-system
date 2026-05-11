# Testing Fundamentals

## Types of Tests

### Unit Tests
**Isolated; test one function or method with specific inputs and assert on output.**

- No network, no database
- Dependencies are stubbed or mocked
- Fast, cheap, first line of defense against regressions

```csharp
[Fact]
public void Add_TwoNumbers_ReturnsSum()
{
    var calc = new Calculator();
    int result = calc.Add(2, 3);
    Assert.Equal(5, result);
}
```

### Integration Tests
**Test two or more units working together.**

Verify that the seams between components work correctly.

```csharp
[Fact]
public async Task GetUser_WithValidId_ReturnsUser()
{
    var context = new TestDatabaseContext();
    var repository = new UserRepository(context);
    
    var user = await repository.GetUserAsync(1);
    
    Assert.NotNull(user);
    Assert.Equal("John", user.Name);
}
```

### End-to-End (E2E) Tests
**Drive the entire system from the outside, simulating a real user.**

- Authentication → API calls → UI rendering
- Most valuable; hardest to set up
- Treat as a safety net, not primary strategy

Tools: Cypress, Playwright, Selenium (for web applications)

### Component Testing
**Render a single UI component in isolation.**

- Faster than full browser tests
- More realistic than pure unit tests
- Common in React/Vue (React Testing Library, Vue Test Utils)

### Smoke / Build Verification Tests
**Minimal set run after each build to confirm app starts and basic functionality works.**

If smoke tests fail, don't run the rest of the suite.

### Sanity Testing
**Targeted check after smoke tests pass, covering the specific functionality that changed.**

### Regression Testing
**Re-running a defined set of tests to confirm existing functionality hasn't broken.**

The suite grows over time — each bug fix should ideally add a test.

### Usability Testing
**Observing real users to find friction points and UX gaps.**

Not automated — qualitative research.

---

## The Testing Pyramid

```
        /\
       /E2E\
      /______\
     /        \
    / Integration\
   /____________\
  /              \
 /   Unit Tests   \
/________________\
```

**Many unit tests at the base** (fast, cheap)  
**Fewer integration tests in the middle**  
**Small number of E2E tests at the top** (slow, expensive)

### Anti-pattern: Inverted Pyramid

Relying heavily on E2E tests leads to slow, fragile test suites. Avoid this.

---

## Mocking and Test Doubles

Replace dependencies with controlled substitutes when unit testing.

### Stub
Returns a fixed value. Doesn't care how it's called.

```csharp
var stubRepository = new Mock<IUserRepository>();
stubRepository
    .Setup(r => r.GetUser(1))
    .Returns(new User { Id = 1, Name = "John" });
```

### Mock
Records how it was called. You can assert on it.

```csharp
var mockLogger = new Mock<ILogger>();

var service = new UserService(mockLogger.Object);
service.CreateUser("Alice");

mockLogger.Verify(l => l.LogInformation(It.IsAny<string>()), Times.Once);
```

### Fake
A lightweight, working implementation (e.g., an in-memory database instead of real one).

```csharp
public class InMemoryUserRepository : IUserRepository
{
    private List<User> _users = new();
    
    public void Add(User user) => _users.Add(user);
    public User GetUser(int id) => _users.FirstOrDefault(u => u.Id == id);
}

// Used in tests instead of real database
var repository = new InMemoryUserRepository();
```

### Libraries

**Moq** and **NSubstitute** are common mocking libraries for .NET.

---

## xUnit Testing in .NET

xUnit is the dominant testing framework for modern .NET.

### Key Attributes

**[Fact]** — Single test
```csharp
[Fact]
public void Add_TwoNumbers_ReturnsSum()
{
    var calc = new Calculator();
    Assert.Equal(5, calc.Add(2, 3));
}
```

**[Theory]** with **[InlineData]** — Parameterized tests
```csharp
[Theory]
[InlineData(2, 3, 5)]
[InlineData(-1, 1, 0)]
[InlineData(0, 0, 0)]
public void Add_Various_ReturnsCorrectSum(int a, int b, int expected)
{
    var calc = new Calculator();
    Assert.Equal(expected, calc.Add(a, b));
}
```

### Common Assertions

```csharp
Assert.Equal(expected, actual);
Assert.NotEqual(notExpected, actual);
Assert.True(condition);
Assert.False(condition);
Assert.Null(value);
Assert.NotNull(value);
Assert.Empty(collection);
Assert.NotEmpty(collection);
Assert.Contains(item, collection);
Assert.Throws<ArgumentException>(() => /* code */);
```

### Test Structure (AAA Pattern)

**Arrange, Act, Assert**

```csharp
[Fact]
public void Transfer_ValidAmount_UpdatesBothAccounts()
{
    // Arrange
    var fromAccount = new Account { Balance = 1000 };
    var toAccount = new Account { Balance = 500 };
    var transferService = new TransferService();
    
    // Act
    transferService.Transfer(fromAccount, toAccount, 200);
    
    // Assert
    Assert.Equal(800, fromAccount.Balance);
    Assert.Equal(700, toAccount.Balance);
}
```

---

## Best Practices

### Test Naming

Use descriptive names that explain what is being tested:
```
MethodName_Scenario_ExpectedResult

Transfer_ValidAmount_UpdatesBothAccounts
ValidateEmail_InvalidFormat_ReturnsFalse
Calculate_DivideByZero_ThrowsException
```

### Avoid Test Interdependence

Tests should be independent and run in any order:

```csharp
// BAD — TestB depends on TestA being run first
[Fact]
public void TestA_CreatesUser()
{
    _userId = _service.CreateUser("John");  // Shared state
}

[Fact]
public void TestB_UpdatesUser()
{
    _service.UpdateUser(_userId, "Jane");   // Depends on TestA
}

// GOOD — Each test is self-contained
[Fact]
public void CreateUser_ValidName_ReturnsUser()
{
    var result = _service.CreateUser("John");
    Assert.NotNull(result);
}

[Fact]
public void UpdateUser_ValidName_UpdatesSuccessfully()
{
    var user = _service.CreateUser("John");
    _service.UpdateUser(user.Id, "Jane");
    var updated = _service.GetUser(user.Id);
    Assert.Equal("Jane", updated.Name);
}
```

### One Assert Per Test (When Possible)

```csharp
// OK — Related assertions
[Fact]
public void CreateUser_ValidInput_CreatesUserCorrectly()
{
    var result = _service.CreateUser("John", 30);
    
    Assert.NotNull(result);
    Assert.Equal("John", result.Name);
    Assert.Equal(30, result.Age);
}

// BETTER — Focused test
[Fact]
public void CreateUser_ValidInput_ReturnsUserWithCorrectName()
{
    var result = _service.CreateUser("John", 30);
    Assert.Equal("John", result.Name);
}
```

### Test at the Right Level

```csharp
// BAD — Testing implementation details
[Fact]
public void Calculate_ValidInput_CallsHelperMethod()
{
    var mockHelper = new Mock<ICalculationHelper>();
    var calc = new Calculator(mockHelper.Object);
    
    calc.Calculate(5);
    
    mockHelper.Verify(h => h.DoMath(5), Times.Once);
}

// GOOD — Testing behavior
[Fact]
public void Calculate_ValidInput_ReturnsCorrectResult()
{
    var calc = new Calculator(new RealCalculationHelper());
    int result = calc.Calculate(5);
    
    Assert.Equal(25, result);
}
```

### Mock Sparingly

```csharp
// BAD — Over-mocking makes tests brittle
var mockRepository = new Mock<IRepository>();
var mockLogger = new Mock<ILogger>();
var mockCache = new Mock<ICache>();

// Any refactoring breaks the test

// GOOD — Use real implementations when possible
var repository = new InMemoryRepository();
var logger = new TestLogger();

var service = new UserService(repository, logger);
```

---

## Running Tests

```bash
# Run all tests
dotnet test

# Run tests in a specific project
dotnet test ./MyProject.Tests

# Run specific test class
dotnet test --filter "Namespace.ClassName"

# Run with coverage
dotnet test /p:CollectCoverage=true
```

---

## Test-Driven Development (TDD)

**Red → Green → Refactor**

1. **Red** — Write a failing test
2. **Green** — Write minimal code to make it pass
3. **Refactor** — Improve the code while keeping tests passing

```csharp
// RED — Write test first, code doesn't exist yet
[Fact]
public void Reverse_ValidString_ReturnsReversed()
{
    var result = StringHelper.Reverse("hello");
    Assert.Equal("olleh", result);
}

// GREEN — Minimal implementation to pass
public class StringHelper
{
    public static string Reverse(string str)
    {
        return new string(str.Reverse().ToArray());
    }
}

// REFACTOR — Improve while keeping test passing
public class StringHelper
{
    public static string Reverse(string str)
    {
        if (string.IsNullOrEmpty(str))
            return str;
        return new string(str.Reverse().ToArray());
    }
}
```

---

## Next Steps

- [Design Patterns](./design-patterns.md) — Patterns that enable testability
- [SOLID Principles](./solid.md) — Principles that make testing easier
- [FEM C# and .NET](../03-fem-courses/01-csharp-dotnet/) — Practical testing examples
