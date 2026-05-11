# Object-Oriented Programming (OOP)

## The Four Fundamental Concepts (A PIE)

### Abstraction
**Expose only what's necessary; hide the complexity underneath.**

A car's steering wheel is an abstraction over the steering system. Users interact with a simple interface without understanding hydraulics, gears, and sensors.

In code:
```csharp
public interface IPaymentProcessor
{
    Task ProcessPaymentAsync(decimal amount);
}

// Users don't need to know HOW payment is processed
// Just that they can call this method
var processor = new StripePaymentProcessor();
await processor.ProcessPaymentAsync(99.99);
```

### Polymorphism
**One name, many forms. A method or interface behaves differently depending on the concrete type behind it.**

Two types:
- **Compile-time (Method Overloading)** — Resolved at compile time based on parameters
- **Runtime (Method Overriding)** — Resolved at runtime based on actual type

```csharp
// Polymorphism in action
Animal dog = new Dog();      // Upcast to base type
Animal cat = new Cat();

dog.MakeSound();  // "Woof!"
cat.MakeSound();  // "Meow!"
```

### Inheritance
**A class can extend another class, inheriting its members and behavior.**

- C# supports **single inheritance for classes** (one parent only)
- A class can **implement multiple interfaces**

```csharp
public class Animal
{
    public string Name { get; set; }
    public virtual void MakeSound() { }
}

public class Dog : Animal
{
    public override void MakeSound() => Console.WriteLine("Woof!");
}
```

### Encapsulation
**Bundle data and methods together, and control access via access modifiers.**

Prevents outside code from putting an object into an invalid state.

```csharp
public class BankAccount
{
    private decimal _balance;  // Private — only accessible internally
    
    public decimal Balance => _balance;  // Public read-only access
    
    public void Deposit(decimal amount)
    {
        if (amount <= 0)
            throw new ArgumentException("Amount must be positive");
        _balance += amount;
    }
}
```

---

## Plain-English Explanation: Pokémon

> "Use Pokémon. Let's create a class: Pokémon. What do we know about Pokémon? They have numbers, they have attack stats, they have nicknames, they have four moves. We can create subclasses. A Bulbasaur is a Pokémon that has access to certain moves, certain attack and defense stats. What's the difference between a class and an instance of a class? Well, we've programmed the recipe for a Bulbasaur, but if we create a Bulbasaur named Henry, Henry is a single instance of that Bulbasaur."

Or more generally:
- Think of anything in terms of **what it is** and **what it can do**
- A car has a color and wheels (**properties**); it can be driven and its doors can open (**methods**)
- A **class** is the blueprint; an **object** is one concrete thing made from that blueprint

---

## Access Modifiers

### public
```csharp
public void PublicMethod() { }
// Accessible to all consumers everywhere
```

### private
```csharp
private void PrivateMethod() { }
// Accessible only within this class — encapsulation
```

### protected
```csharp
protected void ProtectedMethod() { }
// Accessible within this class and derived classes
```

### internal
```csharp
internal void InternalMethod() { }
// Accessible only within this assembly
```

---

## Abstract Classes

An abstract class **cannot be instantiated directly** — it exists to be inherited from. It can contain:
- Fully implemented methods (inherited as-is)
- Abstract methods (which subclasses **must** override)

```csharp
public abstract class Shape
{
    // Abstract method — subclasses must implement
    public abstract double Area();
    
    // Concrete method — subclasses inherit as-is
    public void Describe() => Console.WriteLine($"Area: {Area()}");
}

public class Circle : Shape
{
    public double Radius { get; set; }
    
    // Must override abstract method
    public override double Area() => Math.PI * Radius * Radius;
}

// Shape s = new Shape();  // ERROR — cannot instantiate abstract class
```

**Use when:** You want to share implementation code across related classes while requiring subclasses to fill in specific behavior.

---

## Abstract Class vs. Interface

| Aspect | Abstract Class | Interface |
|--------|---|---|
| Can have implementation | Yes | Yes (C# 8.0+) |
| Can have fields/state | Yes | No |
| A class can inherit | One only | Multiple |
| Use when... | Sharing code among related types | Defining a contract for unrelated types |
| Provides what? | Abstract and concrete behavior | Contract only (traditionally) |
| Access modifiers | Yes, can be private | No, always public |
| Default access modifiers | Can be private by default | Public by default |
| Can inherit from | Another abstract class or interface | Only another interface |

---

## Generic Classes

A generic class is parameterized by a type, written as `ClassName<T>`. Allows a single class definition to work safely with any data type without sacrificing type safety.

```csharp
public class Box<T>
{
    public T Value { get; set; }
    public Box(T value) { Value = value; }
}

var intBox = new Box<int>(42);
var strBox = new Box<string>("hello");
```

**Advantages:**
- Type safety at compile time
- No boxing/unboxing overhead
- Code reusability

Prefer generic types from `System.Collections.Generic` over obsolete non-generic alternatives like `ArrayList`.

---

## Anonymous Types

Create lightweight, read-only objects without defining a named class. Most useful in LINQ select projections when you need a temporary shape for query results.

```csharp
var employee = new { Amount = 108, Message = "Hello" };
Console.WriteLine(employee.Amount + employee.Message);
```

---

## Method Overloading

Defining multiple methods in the same class with the same name but different parameter lists (different number, type, or order).

The compiler resolves which to call based on arguments at the call site. This is **compile-time (static) polymorphism**.

```csharp
public int Add(int a, int b) => a + b;
public double Add(double a, double b) => a + b;
public int Add(int a, int b, int c) => a + b + c;
```

---

## Partial Classes

Split the definition of a single class across multiple files. All parts are combined by the compiler.

**Most common use:** Code-generated files — generated code goes in one partial file, your custom code goes in another.

```csharp
// File: Order.cs
public partial class Order
{
    public int Id { get; set; }
    public DateTime CreatedAt { get; set; }
}

// File: Order.Business.cs
public partial class Order
{
    public bool IsValid() => Id > 0 && CreatedAt != default;
}
```

Regenerating the generated file won't overwrite your custom code.

---

## Next Steps

- [SOLID Principles](./solid.md) — Design principles for maintainable code
- [Design Patterns](./design-patterns.md) — Proven solutions to recurring problems
- [FEM C# and .NET](../03-fem-courses/01-csharp-dotnet/03-types-classes-structures.md) — Deep dive into C# OOP implementation
