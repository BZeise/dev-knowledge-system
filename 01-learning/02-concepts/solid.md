# SOLID Principles

**SOLID** is an acronym for five design principles that make software more maintainable, flexible, and scalable.

## S — Single Responsibility Principle

**A class should have only one reason to change — one job.**

If a class handles both business logic and database access, a schema change forces a change in your business logic class. Split them.

### Bad Example
```csharp
public class User
{
    public string Name { get; set; }
    public string Email { get; set; }
    
    // Business logic
    public bool ValidateEmail() => /* ... */;
    
    // Database responsibility
    public void SaveToDatabase() => /* ... */;
}
```

The class has two reasons to change:
1. Business validation rules change
2. Database structure changes

### Good Example
```csharp
public class User
{
    public string Name { get; set; }
    public string Email { get; set; }
}

public class UserValidator
{
    public bool ValidateEmail(User user) => /* ... */;
}

public class UserRepository
{
    public void Save(User user) => /* ... */;
}
```

Now each class has one clear responsibility.

---

## O — Open-Closed Principle

**Software entities should be open for extension but closed for modification.**

Add new behavior by writing new code (new subclasses, new implementations), **not by editing existing, working code**.

Instead of a giant switch statement that checks a type, define an interface and let each type implement it.

### Bad Example
```csharp
public class PaymentProcessor
{
    public void ProcessPayment(string paymentType, decimal amount)
    {
        switch (paymentType)
        {
            case "Credit":
                ProcessCreditCard(amount);
                break;
            case "PayPal":
                ProcessPayPal(amount);
                break;
            // Adding new payment type requires modifying this class!
        }
    }
}
```

To add a new payment type (Bitcoin), you must modify the existing class.

### Good Example
```csharp
public interface IPaymentMethod
{
    void Process(decimal amount);
}

public class CreditCardPayment : IPaymentMethod
{
    public void Process(decimal amount) => /* ... */;
}

public class PayPalPayment : IPaymentMethod
{
    public void Process(decimal amount) => /* ... */;
}

public class PaymentProcessor
{
    public void ProcessPayment(IPaymentMethod method, decimal amount)
    {
        method.Process(amount);  // Works for any IPaymentMethod
    }
}

// To add Bitcoin: Create new class, no existing code modified
public class BitcoinPayment : IPaymentMethod
{
    public void Process(decimal amount) => /* ... */;
}
```

---

## L — Liskov Substitution Principle

**If S is a subtype of T, then objects of type T may be replaced with objects of type S without breaking correctness.**

A subclass should be substitutable for its parent without surprising the caller.

### Bad Example (Violation)
```csharp
public class Rectangle
{
    public virtual int Width { get; set; }
    public virtual int Height { get; set; }
    public virtual int Area() => Width * Height;
}

public class Square : Rectangle
{
    public override int Width
    {
        get => base.Width;
        set { base.Width = value; base.Height = value; }  // Enforce equality
    }
}

// This breaks the contract!
Rectangle shape = new Square();
shape.Width = 5;
shape.Height = 10;
int area = shape.Area();  // Expected 50, got 100 — Square changed Height too!
```

The Square breaks the Rectangle contract — calling code can't substitute Square without surprising behavior.

### Good Example
```csharp
public interface IShape
{
    int Area();
}

public class Rectangle : IShape
{
    public int Width { get; set; }
    public int Height { get; set; }
    public int Area() => Width * Height;
}

public class Square : IShape
{
    public int Side { get; set; }
    public int Area() => Side * Side;
}

// Now each shape is substitutable for IShape
IShape shape = GetShape();
int area = shape.Area();  // Works correctly for any shape
```

---

## I — Interface Segregation Principle

**Clients should not be forced to depend on interfaces they don't use.**

Prefer many small, focused interfaces over one large catch-all interface.

### Bad Example
```csharp
public interface IWorker
{
    void Work();
    void Eat();
    void Sleep();
}

public class Robot : IWorker
{
    public void Work() => Console.WriteLine("Working");
    public void Eat() => throw new NotImplementedException();  // Doesn't make sense!
    public void Sleep() => throw new NotImplementedException();
}
```

The Robot is forced to implement irrelevant methods.

### Good Example
```csharp
public interface IWorkable
{
    void Work();
}

public interface IFeedable
{
    void Eat();
}

public interface IMaintainable
{
    void Sleep();
}

public class Robot : IWorkable, IMaintainable
{
    public void Work() => Console.WriteLine("Working");
    public void Sleep() => Console.WriteLine("Recharging");
}

public class Human : IWorkable, IFeedable, IMaintainable
{
    public void Work() => Console.WriteLine("Working");
    public void Eat() => Console.WriteLine("Eating");
    public void Sleep() => Console.WriteLine("Sleeping");
}
```

Now each class implements only the interfaces it actually needs.

---

## D — Dependency Inversion Principle

**High-level modules should not depend on low-level modules. Both should depend on abstractions.**

Program to interfaces, not concrete classes.

### Bad Example (Direct Dependency)
```csharp
public class OrderProcessor
{
    private SqlOrderRepository _repository = new SqlOrderRepository();
    
    public void ProcessOrder(Order order)
    {
        _repository.Save(order);  // Tightly coupled to SqlOrderRepository
    }
}
```

If you want to use a different database (NoSQL), you must modify OrderProcessor.

### Good Example (Depends on Abstraction)
```csharp
public interface IOrderRepository
{
    void Save(Order order);
}

public class OrderProcessor
{
    private readonly IOrderRepository _repository;
    
    public OrderProcessor(IOrderRepository repository)
    {
        _repository = repository;  // Depends on abstraction
    }
    
    public void ProcessOrder(Order order)
    {
        _repository.Save(order);
    }
}

// Easy to swap implementations
public class SqlOrderRepository : IOrderRepository
{
    public void Save(Order order) => /* SQL logic */;
}

public class MongoOrderRepository : IOrderRepository
{
    public void Save(Order order) => /* NoSQL logic */;
}
```

This principle is the foundation of **Dependency Injection**.

---

## Applying SOLID in Practice

**Don't over-engineer from the start.** These principles address pain points:

1. **Single Responsibility** — When a class becomes hard to test or name
2. **Open-Closed** — When adding features requires modifying many files
3. **Liskov Substitution** — When subclasses surprise callers
4. **Interface Segregation** — When classes implement methods they don't use
5. **Dependency Inversion** — When testing requires complex mocking

Refactor when you feel the pain, not preemptively.

---

## Next Steps

- [Design Patterns](./design-patterns.md) — Common solutions that apply SOLID principles
- [FEM C# and .NET](../03-fem-courses/01-csharp-dotnet/03-types-classes-structures.md) — See SOLID in action
