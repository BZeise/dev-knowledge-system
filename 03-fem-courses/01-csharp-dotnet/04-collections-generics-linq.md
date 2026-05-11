# Collections, Generics, and LINQ

*From "C# and .NET: From Beginner to Professional" by Spencer Schneidenbach (Frontend Masters)*

## Generics

Generics enable developers to create reusable code by defining placeholder types while maintaining compile-time type safety.

### Overview

Generics were introduced in .NET 3.5 and C# 2. They represent polymorphism in action, allowing classes, methods, and interfaces to work with any data type safely.

### Generic Classes

```csharp
public class Box<T>
{
    private T _item;
    
    public void SetItem(T item) => _item = item;
    public T GetItem() => _item;
}

// Usage
var intBox = new Box<int>();
intBox.SetItem(42);
int value = intBox.GetItem();  // Type-safe: int

var stringBox = new Box<string>();
stringBox.SetItem("Hello");
string text = stringBox.GetItem();  // Type-safe: string
```

### Generic Methods

```csharp
public class Utilities
{
    public T GetDefaultValue<T>()
    {
        return default(T);  // int: 0, string: null, bool: false
    }
}

var util = new Utilities();
int defaultInt = util.GetDefaultValue<int>();        // 0
string defaultString = util.GetDefaultValue<string>(); // null
```

### Multiple Type Parameters

```csharp
public class Dictionary<TKey, TValue>
{
    private TKey _key;
    private TValue _value;
}

var userIds = new Dictionary<string, int>();
// Maps usernames (string) to user IDs (int)
```

### Type Constraints

Restrict what types can be used with a generic:

```csharp
// T must be a class
public class Repository<T> where T : class
{
    // ...
}

// T must implement IDisposable
public class UsingHelper<T> where T : IDisposable
{
    // ...
}

// Multiple constraints
public class SpecialClass<T> where T : class, IDisposable, new()
{
    // T is a class, implements IDisposable, has parameterless constructor
}
```

### Repository Pattern with Generics

```csharp
public interface IRepository<T>
{
    T GetById(int id);
    IEnumerable<T> GetAll();
    void Add(T item);
    void Delete(int id);
}

public class CustomerRepository : IRepository<Customer>
{
    // Implement for Customer
}

public class ProductRepository : IRepository<Product>
{
    // Implement for Product — same code structure, type-safe
}
```

---

## Collections Overview

### Arrays and Their Limitations

Arrays provide fixed-size storage but cannot dynamically resize:

```csharp
int[] numbers = { 1, 2, 3 };
// Adding a 4th element requires creating a new array
int[] newNumbers = new int[4];
Array.Copy(numbers, newNumbers, numbers.Length);
newNumbers[3] = 4;
```

### Legacy Collection Types (Avoid)

Before generics, developers used non-generic collections:

```csharp
ArrayList list = new ArrayList();      // Lacks type safety
Hashtable table = new Hashtable();     // No compile-time checking
```

These are now considered obsolete. Always use generic alternatives.

### IEnumerable<T>

The most commonly used interface in .NET. All modern collections implement it:

```csharp
public interface IEnumerable<T>
{
    IEnumerator<T> GetEnumerator();
}
```

Enables iteration with `foreach`:

```csharp
List<int> numbers = new List<int> { 1, 2, 3 };
foreach (int num in numbers)
{
    Console.WriteLine(num);
}
```

### Modern Collection Types

**List<T>** — Resizable, flexible collection:

```csharp
var names = new List<string> { "Alice", "Bob", "Charlie" };
names.Add("Diana");
names.Remove("Bob");
names.Insert(0, "Zoe");
names.Sort();
names.Reverse();
```

**Dictionary<TKey, TValue>** — Key-value pairs with fast lookups:

```csharp
var ages = new Dictionary<string, int>();
ages["Alice"] = 30;
ages["Bob"] = 25;

if (ages.ContainsKey("Alice"))
{
    int age = ages["Alice"];
}

// TryGetValue for safe retrieval
if (ages.TryGetValue("Charlie", out int age))
{
    Console.WriteLine(age);
}
else
{
    Console.WriteLine("Charlie not found");
}
```

**HashSet<T>** — Stores unique values only:

```csharp
var uniqueNumbers = new HashSet<int> { 1, 2, 3, 1, 2 };
// Contains: 1, 2, 3 (duplicates removed)
```

**Queue<T>** and **Stack<T>** — FIFO and LIFO:

```csharp
// Queue (First In, First Out)
var queue = new Queue<string>();
queue.Enqueue("First");
queue.Enqueue("Second");
string first = queue.Dequeue();  // "First"

// Stack (Last In, First Out)
var stack = new Stack<int>();
stack.Push(1);
stack.Push(2);
int top = stack.Pop();  // 2
```

**ImmutableArray<T>** — Fixed-size, immutable collections:

```csharp
var numbers = ImmutableArray.Create(1, 2, 3);
// Cannot modify the array
// Modifications create new instances
var updated = numbers.Add(4);  // new array with 4
```

### Selection Guidance

- **ImmutableArray<T>** — When data doesn't change
- **List<T>** — When you need indexed access and resizing
- **Dictionary<TKey, TValue>** — When you need key-based lookups
- **HashSet<T>** — When you need unique values
- **Queue<T>** / **Stack<T>** — When order matters (FIFO/LIFO)

---

## Lambda Expressions, Func<T>, and Action<T>

### Lambda Expressions

Anonymous functions defined inline using `=>` (pronounced "goes into"):

```csharp
// Traditional method
public int Square(int x)
{
    return x * x;
}

// Lambda expression equivalent
Func<int, int> square = x => x * x;

// Usage
int result = square(5);  // 25
```

### Func<T>

Represents methods that **return a value**. The final type parameter is the return type:

```csharp
// Func<parameter-types, return-type>
Func<int, int> square = x => x * x;
Func<int, int, int> add = (x, y) => x + y;
Func<string, int, bool> checkLength = (str, length) => str.Length > length;

// Usage
int result = add(3, 4);              // 7
bool isLong = checkLength("hello", 3); // true
```

### Action<T>

Represents methods that **return void**:

```csharp
// Action<parameter-types>
Action<string> print = x => Console.WriteLine(x);
Action<int, string> log = (code, message) => 
    Console.WriteLine($"[{code}] {message}");

// Usage
print("Hello");
log(500, "Server Error");
```

### Closures

Lambda expressions can capture variables from surrounding scope:

```csharp
int multiplier = 2;
Func<int, int> multiply = x => x * multiplier;

Console.WriteLine(multiply(5));  // 10

multiplier = 3;
Console.WriteLine(multiply(5));  // 15 — closure captures by reference

// Counter example
int counter = 0;
Action increment = () => counter++;

increment();
increment();
Console.WriteLine(counter);  // 2 — changes persist
```

**Important:** Closures capture by reference, not by value.

---

## Introduction to LINQ

**LINQ** stands for **Language INtegrated Query**. It enables writing concise, readable queries directly in C# to work with collections, databases, XML, and other data sources.

### Why LINQ?

Compare traditional approach vs. LINQ:

```csharp
// Traditional approach
var employees = new List<Employee> { /* ... */ };
var highEarners = new List<Employee>();

foreach (var employee in employees)
{
    if (employee.Salary > 100000)
    {
        highEarners.Add(employee);
    }
}

highEarners.Sort((a, b) => a.Name.CompareTo(b.Name));

// LINQ approach — cleaner and more readable
var highEarners = employees
    .Where(e => e.Salary > 100000)
    .OrderBy(e => e.Name)
    .ToList();
```

### Two Syntax Approaches

**Lambda Syntax** (preferred in practice):

```csharp
var highEarners = employees
    .Where(e => e.Salary > 100000)
    .OrderBy(e => e.Name)
    .Select(e => e.Name);
```

**Query Syntax** (SQL-like alternative):

```csharp
var highEarners = from e in employees
                  where e.Salary > 100000
                  orderby e.Name
                  select e.Name;
```

In modern code, lambda syntax dominates (~99% of code in the wild).

### Technologies Behind LINQ

LINQ relies on several features working together:
- **Generics** — Type safety
- **IEnumerable<T>** — Iteration capability
- **Extension methods** — Adding methods to interfaces
- **Lambda expressions** — Anonymous functions
- **Delegates** — Func and Action types

### Lazy Evaluation (Deferred Execution)

LINQ queries do not execute when defined — they execute when results are enumerated:

```csharp
var query = employees.Where(e => e.Salary > 100000);
// Query does NOT execute here

foreach (var emp in query)
{
    // Query executes HERE when iterating
    Console.WriteLine(emp.Name);
}
```

Methods like `ToList()` and `ToArray()` force immediate execution:

```csharp
var query = employees
    .Where(e => e.Salary > 100000)
    .ToList();  // Forces execution — creates list in memory
```

---

## Projection, Anonymous Types, and Select

### The Select Method

Transforms collections by applying a function to each element:

```csharp
var employees = new List<Employee>
{
    new() { Name = "Alice", Salary = 120000 },
    new() { Name = "Bob", Salary = 95000 }
};

var names = employees.Select(e => e.Name);
// Result: ["Alice", "Bob"]

var salaries = employees.Select(e => e.Salary * 1.1);
// Result: [132000, 104500] — 10% raise
```

### Anonymous Types

Dynamically created classes without explicit declaration:

```csharp
var employee = new { Name = "Alice", Salary = 120000 };
Console.WriteLine(employee.Name);    // "Alice"
Console.WriteLine(employee.Salary);  // 120000
```

**Characteristics:**
- Read-only properties
- Automatic `Equals` and `GetHashCode` based on structural equality
- Useful for temporary data projections

### Structural Equality

```csharp
var e1 = new { Name = "John", Salary = 50000 };
var e2 = new { Name = "John", Salary = 50000 };
var e3 = new { Name = "Jane", Salary = 55000 };

e1.Equals(e2)  // true — same properties and values
e1.Equals(e3)  // false — different values
```

### Practical Example: Flattening Nested Data

```csharp
var employeeData = employees.Select(e => new 
{ 
    ManagerName = e.Manager.Name,
    e.Salary,
    DepartmentName = e.Department.Name
}).ToList();
```

### Common Mistakes to Avoid

1. **Using Select as a foreach substitute:**
```csharp
// WRONG — Select for side effects
employees.Select(e => Console.WriteLine(e.Name));

// RIGHT — foreach for side effects
foreach (var e in employees)
{
    Console.WriteLine(e.Name);
}
```

2. **Mutating objects in Select:**
```csharp
// WRONG
employees.Select(e => { e.Salary *= 1.1; return e; });

// RIGHT — Select transforms, doesn't mutate
var updated = employees.Select(e => new Employee 
{ 
    Name = e.Name, 
    Salary = e.Salary * 1.1 
});
```

---

## Filtering with Where

### Basic Filtering

The `Where` method filters collections based on a predicate (a function returning true/false):

```csharp
var employees = new List<Employee> { /* ... */ };

var highEarners = employees.Where(e => e.Salary > 100000);
```

### How Where Works

```csharp
public static IEnumerable<TSource> Where<TSource>(
    this IEnumerable<TSource> source, 
    Func<TSource, bool> predicate)
{
    foreach (TSource element in source)
    {
        if (predicate(element))
        {
            yield return element;
        }
    }
}
```

### Practical Examples

**Single condition:**
```csharp
var highEarners = employees.Where(e => e.Salary > 100000);
```

**Chaining Where with other operations:**
```csharp
var highEarnerNames = employees
    .Where(e => e.Salary > 100000)
    .Select(e => e.Name);
```

**Multiple filters:**
```csharp
var seniorHighEarners = employees
    .Where(e => e.Salary > 100000)
    .Where(e => e.YearsOfExperience > 10);
```

### OfType for Type-Based Filtering

```csharp
var mixedObjects = new object[] 
{ 
    "string", 
    42, 
    new Employee(), 
    3.14 
};

var employees = mixedObjects.OfType<Employee>();
// Result: [new Employee()] — only Employee instances
```

Useful with non-generic collections like `ArrayList`.

---

## Ordering with OrderBy

### Basic Sorting

```csharp
var employees = new List<Employee> { /* ... */ };

// Ascending order
var bySalary = employees.OrderBy(e => e.Salary);

// Descending order
var byNameDesc = employees.OrderByDescending(e => e.Name);
```

### Return Type: IOrderedEnumerable<T>

Methods like `OrderBy` return `IOrderedEnumerable<T>`, which maintains order information and enables secondary sorting.

### Secondary Sorting with ThenBy

```csharp
var sorted = employees
    .OrderBy(e => e.Department)      // Primary sort
    .ThenBy(e => e.Name);            // Secondary sort
    .ThenByDescending(e => e.Salary); // Tertiary sort

// Result: sorted by department, then name, then salary descending
```

---

## Selecting Single Values

Methods for retrieving individual elements from sequences.

### ElementAt / ElementAtOrDefault

Access elements by index:

```csharp
var numbers = new[] { 10, 20, 30, 40 };

int third = numbers.ElementAt(2);           // 30
int fifth = numbers.ElementAtOrDefault(4);  // default(int) = 0
```

### First / FirstOrDefault

Get the initial element (or first matching element):

```csharp
int first = numbers.First();                    // 10
int firstOver25 = numbers.First(n => n > 25);  // 30

int firstOver100 = numbers.FirstOrDefault(n => n > 100);  // 0
```

### Single / SingleOrDefault

Assert exactly one element exists:

```csharp
var users = new[] { new User { Id = 1 } };
User user = users.Single();                           // OK
User adminUser = users.Single(u => u.IsAdmin);      // Throws if 0 or 2+
User guest = users.SingleOrDefault(u => u.IsGuest); // null if not found
```

### Spencer's Recommendation

> "In most scenarios, I overwhelmingly prefer Single or SingleOrDefault because they enforce the expectation of one matching element, preventing subtle bugs."

Using `First` when assuming uniqueness silently returns the first of multiple matches, creating production issues.

---

## Aggregate Methods

Aggregate methods perform calculations over sequences of elements.

### Sum

```csharp
var numbers = new List<int> { 1, 2, 3, 4, 5 };
int total = numbers.Sum();              // 15
int salarySum = employees.Sum(e => e.Salary);
```

### Max and Min

```csharp
int max = numbers.Max();                // 5
int min = numbers.Min();                // 1
decimal highestSalary = employees.Max(e => e.Salary);
decimal lowestSalary = employees.Min(e => e.Salary);
```

### Average

```csharp
double average = numbers.Average();     // 3
double avgSalary = employees.Average(e => e.Salary);
```

### Handling Empty Collections

**Critical caveat:** Aggregate methods throw exceptions on empty collections:

```csharp
var emptyList = new List<int>();
int sum = emptyList.Sum();      // 0 — Sum handles empty
int max = emptyList.Max();      // Throws InvalidOperationException

// Solution: DefaultIfEmpty
int safeMax = emptyList.DefaultIfEmpty(0).Max();  // 0
```

---

## Other Important LINQ Methods

### Distinct

Removes duplicate elements:

```csharp
var numbers = new[] { 1, 2, 2, 3, 3, 3 };
var unique = numbers.Distinct();  // [1, 2, 3]

// For custom objects
var employees = new[] { /* ... */ };
var uniqueByDept = employees.DistinctBy(e => e.Department);
```

### GroupBy

Categorizes elements into groups:

```csharp
var grouped = employees.GroupBy(e => e.Department);

foreach (var group in grouped)
{
    Console.WriteLine($"Department: {group.Key}");
    foreach (var emp in group)
    {
        Console.WriteLine($"  - {emp.Name}");
    }
}
```

### SelectMany

Flattens nested collections:

```csharp
var managers = new[]
{
    new { Name = "Alice", Employees = new[] { "Bob", "Charlie" } },
    new { Name = "Diana", Employees = new[] { "Eve", "Frank" } }
};

var allEmployees = managers.SelectMany(m => m.Employees);
// Result: ["Bob", "Charlie", "Eve", "Frank"]
```

### Join

Combines data from two sequences using common keys:

```csharp
var departments = new[] { /* ... */ };
var employees = new[] { /* ... */ };

var result = employees
    .Join(departments,
        e => e.DepartmentId,
        d => d.Id,
        (e, d) => new { e.Name, e.Department = d.Name });
```

**Note:** Spencer rarely uses Join in LINQ because the syntax is verbose. Usually, navigational properties handle relationships instead.

---

## Next Steps

Master these skills:
- [05-Common Libraries and Advanced Topics](./05-common-libraries-advanced.md) — Async, file I/O, JSON
- [Design Patterns](../../02-concepts/design-patterns.md) — Real-world patterns

---

*Content licensed under CC-BY-NC-4.0 | Code samples under Apache 2.0*
