# Types, Classes, and Structures

*From "C# and .NET: From Beginner to Professional" by Spencer Schneidenbach (Frontend Masters)*

## Overview of Types

Understanding value types vs. reference types is fundamental to C# development.

### Value Types vs. Reference Types

**Value Types:**
- Store actual data directly
- Passed by value (methods receive a copy)
- Live on the stack
- Faster, direct access
- Automatic cleanup

Examples: `int`, `double`, `bool`, `struct`

**Reference Types:**
- Store memory references (pointers)
- Passed by reference (methods receive access to same object)
- Occupy the heap
- Indirect access through references
- Managed by garbage collection

Examples: `class`, `string`, `interface`, `delegate`, `arrays`

### Memory Storage

**Stack:**
- Stores value types
- Very fast, automatic cleanup
- Limited size

**Heap:**
- Stores reference types
- Larger memory space
- Managed by garbage collector

### Null Handling

**Reference types can be null:**
```csharp
string text = null;           // OK
object obj = null;            // OK
```

**Value types cannot be null by default:**
```csharp
int number = null;            // ERROR
int? nullable_number = null;  // OK — nullable int
```

Use the `?` syntax to make value types nullable.

### The Null Reference Exception

The most prevalent C# error: accessing members of a null object throws `NullReferenceException`:
```csharp
string text = null;
int length = text.Length;  // CRASH: NullReferenceException
```

**Critical guideline:** Always check for null before accessing reference type members.

### Modern Nullable Reference Types

Enable explicit nullability declarations in your project:
```csharp
#nullable enable

public class Person
{
    public string Name { get; set; }        // non-nullable
    public string? Nickname { get; set; }   // nullable
}
```

**Spencer's recommendation:** In new projects, enable nullable reference types and treat warnings as errors for stricter null safety and fewer bugs.

### Boxing and Unboxing

**Boxing** — wrapping value types as reference types:
```csharp
int number = 42;
object box = number;  // Boxing: value type wrapped in object
```

**Unboxing** — extracting value types from reference types:
```csharp
object box = 42;
int number = (int)box;  // Unboxing: extracts the int
```

**Note:** While boxing creates heap overhead, practical performance impact is typically minimal in modern .NET.

---

## Classes

Classes are fundamental OOP building blocks that encapsulate data and methods.

### Basic Class Definition

```csharp
public class Person
{
    public string Name { get; set; }
    public int Age { get; set; }
    
    public void Greet()
    {
        Console.WriteLine($"Hello, I'm {Name}");
    }
}
```

### Fields

Fields store object data as variables directly in the class:
```csharp
public class Account
{
    public string Username;      // public field
    private string _password;    // private field (convention: underscore prefix)
}
```

### Properties

Properties provide controlled access to data with a concise syntax:
```csharp
public class Person
{
    // Auto-property — compiler creates backing field
    public string Name { get; set; }
    
    // Property with backing field
    private int _age;
    public int Age
    {
        get { return _age; }
        set { _age = value; }
    }
    
    // Expression-bodied property (read-only)
    public string FullName => $"{Name} ({Age} years old)";
}
```

### Mutable vs. Immutable Properties

**Mutable** (can change after initialization):
```csharp
public string Name { get; set; }

var person = new Person { Name = "Alice" };
person.Name = "Bob";  // OK
```

**Immutable** (set only during construction):
```csharp
public string Name { get; init; }

var person = new Person { Name = "Alice" };
person.Name = "Bob";  // ERROR
```

**Required Properties** (must be initialized):
```csharp
public class Person
{
    public required string Name { get; set; }
}

// Must provide Name or compilation error
var person = new Person { Name = "Alice" };
```

### Constructors

Special methods that initialize objects when instantiated:
```csharp
public class Person
{
    public string Name { get; set; }
    public int Age { get; set; }
    
    // Constructor
    public Person(string name, int age)
    {
        Name = name;
        Age = age;
    }
}

var person = new Person("Alice", 30);
```

### Multiple Constructors

Classes can have multiple constructors using overloading:
```csharp
public class Person
{
    public string Name { get; set; }
    public int Age { get; set; }
    
    public Person(string name, int age)
    {
        Name = name;
        Age = age;
    }
    
    public Person(string name) 
        : this(name, 0)  // Calls other constructor
    {
    }
}
```

Use `this()` to call one constructor from another, reducing code duplication.

### Primary Constructors

Simplified syntax (C# 12+):
```csharp
public class Person(string name, int age)
{
    public string Name => name;
    public int Age => age;
}

var person = new Person("Alice", 30);
```

### Object Initialization

Modern syntax using object initializers:
```csharp
var person = new Person 
{ 
    Name = "Alice", 
    Age = 30 
};
```

More readable than sequential assignments.

### Equality Comparison

Classes are reference types compared by memory location, not value:
```csharp
var p1 = new Person { Name = "Alice", Age = 30 };
var p2 = new Person { Name = "Alice", Age = 30 };

p1 == p2  // false — different objects in memory
p1.Equals(p2)  // false — uses reference comparison by default
```

**To customize equality:**

```csharp
public class Person : IEquatable<Person>
{
    public string Name { get; set; }
    public int Age { get; set; }
    
    public override bool Equals(object? obj)
    {
        return Equals(obj as Person);
    }
    
    public bool Equals(Person? other)
    {
        return other is not null && 
               Name == other.Name && 
               Age == other.Age;
    }
    
    public override int GetHashCode()
    {
        return HashCode.Combine(Name, Age);
    }
}

var p1 = new Person { Name = "Alice", Age = 30 };
var p2 = new Person { Name = "Alice", Age = 30 };

p1 == p2  // Now true — value equality
```

---

## Structs and Records

### Structs

Value types optimized for small, immutable data structures:

```csharp
public struct Point
{
    public int X { get; set; }
    public int Y { get; set; }
}

Point p1 = new Point { X = 10, Y = 20 };
Point p2 = p1;  // Copy by value
p2.X = 30;
Console.WriteLine(p1.X);  // Still 10 — p1 unaffected
```

**Key characteristic:** Structs are value types stored on the stack and copied by value.

### Immutable Structs

Recommended to prevent side effects:

```csharp
public struct Point
{
    public int X { get; }
    public int Y { get; }
    
    public Point(int x, int y)
    {
        X = x;
        Y = y;
    }
}

var p1 = new Point(10, 20);
var p2 = new Point(10, 20);
p1 == p2  // true — value equality by default
```

### Records

Reference types providing value-based equality semantics for immutable data:

```csharp
public record Person(string Name, int Age);

var p1 = new Person("Alice", 30);
var p2 = new Person("Alice", 30);
p1 == p2  // true — structural equality
```

**With Expressions** — create modified copies without altering originals:

```csharp
var person1 = new Person("Alice", 30);
var person2 = person1 with { Age = 31 };

// person1 is unchanged
// person2 is new instance with Age = 31
```

### When to Use Each

**Structs:** High performance for small immutable types
- Geometric points, complex numbers
- Use when size matters and mutation isn't needed

**Records:** Data-centric models
- DTOs (Data Transfer Objects)
- Configuration objects
- Immutable data transfers

**Classes:** Complex objects with behavior
- Domain models with business logic
- Services with state management
- Objects with multiple methods

---

## Inheritance and Abstract Classes

### Inheritance Basics

All C# objects inherit from `object`. A derived class inherits from a base class using `:` syntax:

```csharp
public class Animal
{
    public string Name { get; set; }
    
    public virtual void MakeSound()
    {
        Console.WriteLine("Generic animal sound");
    }
}

public class Dog : Animal
{
    public override void MakeSound()
    {
        Console.WriteLine("Woof!");
    }
}

Animal dog = new Dog { Name = "Rex" };
dog.MakeSound();  // "Woof!"
```

### Benefits of Inheritance

- **Code reusability** — reduces redundancy
- **Hierarchical organization** — improves maintainability
- **Polymorphism** — flexible design patterns

### Virtual and Override

**Virtual Methods** — allow derived classes to provide custom implementations:

```csharp
public class Animal
{
    public virtual void MakeSound() { }
}

public class Cat : Animal
{
    public override void MakeSound()
    {
        Console.WriteLine("Meow!");
    }
}
```

### Sealed Classes and Methods

**Prevent further derivation:**

```csharp
public sealed class FinalClass
{
    // Cannot be inherited
}

public class BaseClass
{
    public sealed override void Method() { }
    // Derived classes cannot override this method
}
```

The `string` class is sealed in C# — you cannot create a subclass of string.

### Abstract Classes

Serve as templates for derived classes and cannot be instantiated directly:

```csharp
public abstract class Shape
{
    // Abstract method — must be implemented by derived classes
    public abstract double Area();
    
    // Virtual method — can be overridden or inherited as-is
    public virtual void Describe()
    {
        Console.WriteLine($"Area: {Area()}");
    }
}

public class Circle : Shape
{
    public double Radius { get; set; }
    
    public override double Area() => Math.PI * Radius * Radius;
}

// Shape s = new Shape();  // ERROR — cannot instantiate
Shape circle = new Circle { Radius = 5 };
circle.Describe();  // Inherited implementation using overridden Area()
```

### Access Modifiers in Inheritance Context

- `public` — Accessible to all consumers
- `protected` — Accessible to derived classes only
- `private` — Internal access only
- `internal` — Limited to the same assembly

```csharp
public class BaseClass
{
    public void PublicMethod() { }
    protected void ProtectedMethod() { }  // Derived can access
    private void PrivateMethod() { }      // Derived cannot access
}
```

### Spencer's Design Perspective

He recommends **avoiding excessive inheritance hierarchies**, typically staying within one level.

**Alternative approaches to deep inheritance:**
- Use property discriminators (enums) to differentiate behavior
- Use service classes for complex logic instead of inheritance chains

**DRY vs. MOIST:**
> "Total elimination of duplication is usually a waste of time."
> Refactor after duplicating code twice, not before.

---

## Interfaces

Interfaces define contracts that classes must implement.

### Basic Interface

```csharp
public interface IRepository<T>
{
    T GetById(int id);
    void Add(T item);
    void Delete(int id);
}

public class CustomerRepository : IRepository<Customer>
{
    public Customer GetById(int id) { /* ... */ }
    public void Add(Customer item) { /* ... */ }
    public void Delete(int id) { /* ... */ }
}
```

### Key Benefits

**Multiple Implementation:**
Unlike classes (single inheritance), a class can implement multiple interfaces:

```csharp
public class Logger : ILogger, IAuditable, IDisposable
{
    // Must implement all three interfaces
}
```

**Composability:**
Different classes share behaviors without strict inheritance hierarchies:

```csharp
public interface IDrawable
{
    void Draw();
}

public interface IMovable
{
    void Move(int x, int y);
}

public class GameObject : IDrawable, IMovable
{
    public void Draw() { }
    public void Move(int x, int y) { }
}
```

**Contract Definition:**
> "Interfaces define the thing's shape without opinions about the behavior"
> They specify what must exist, not how it should work.

### Advanced Interface Features (C# 8.0+)

**Default Implementations:**
```csharp
public interface ILogger
{
    void Log(string message);
    
    void LogInfo(string message)
    {
        Log($"INFO: {message}");  // Default implementation
    }
}
```

**Static Abstract Methods (C# 11.0):**
```csharp
public interface IFactory<T>
{
    static abstract T Create();
}
```

### Practical Applications

1. **Service Architecture:**
```csharp
public interface IUserService
{
    User GetUser(int id);
    void CreateUser(User user);
}
```

2. **Marker Interfaces:**
```csharp
public interface IAuditable { }

if (entity is IAuditable)
{
    // Track changes for audit
}
```

3. **Avoiding Deep Hierarchies:**
Use composition over extensive inheritance chains.

---

## Extension Methods

Extension methods enable adding functionality to existing classes without modifying source code.

### Basic Syntax

```csharp
public static class StringExtensions
{
    public static bool IsPalindrome(this string text)
    {
        var cleaned = new string(
            text.Where(c => !char.IsWhiteSpace(c))
                .ToArray()
        ).ToLower();
        
        var reversed = new string(cleaned.Reverse().ToArray());
        return cleaned == reversed;
    }
}

// Usage
string word = "racecar";
bool isPalindrome = word.IsPalindrome();  // true
```

**Key points:**
- Must be in a `static class`
- Must be a `static` method
- First parameter uses `this` keyword specifying target type

### Real-World Example: Logger Interface

Instead of embedding multiple methods in the interface:

```csharp
public interface ILogger
{
    void Log(LogLevel level, string message);
}

public static class LoggerExtensions
{
    public static void LogInformation(this ILogger logger, string message)
    {
        logger.Log(LogLevel.Information, message);
    }
    
    public static void LogWarning(this ILogger logger, string message)
    {
        logger.Log(LogLevel.Warning, message);
    }
    
    public static void LogError(this ILogger logger, string message)
    {
        logger.Log(LogLevel.Error, message);
    }
}

// Usage
logger.LogInformation("App started");
logger.LogError("An error occurred");
```

### Benefits

- **Clean API design** — keep interfaces minimal
- **Improved readability** — natural method syntax
- **Code organization** — related functionality in dedicated classes
- **Enhanced developer experience** — convenient utility methods

### Common Use Cases

1. **Inaccessible Classes** — extending third-party libraries
2. **Functional Gaps** — existing methods insufficient for needs
3. **Fluent APIs** — method chaining for readable code

---

## Next Steps

Continue learning:
- [04-Collections, Generics, and LINQ](./04-collections-generics-linq.md) — Master advanced data handling
- [SOLID Principles](../../02-concepts/solid.md) — Design principles for extensible code

---

*Content licensed under CC-BY-NC-4.0 | Code samples under Apache 2.0*
