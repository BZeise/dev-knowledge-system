# Software Design Patterns

Design patterns are proven, reusable solutions to recurring design problems. Not code to copy — **templates to adapt**.

Grouped into three categories: **Creational**, **Structural**, and **Behavioral**.

---

## Creational Patterns — How Objects Are Created

### Singleton

Ensures a class has only one instance and provides a global point of access to it.

```csharp
public class Logger
{
    private static Logger _instance;
    
    private Logger() { }  // Private constructor prevents external instantiation
    
    public static Logger GetInstance()
    {
        if (_instance == null)
            _instance = new Logger();
        return _instance;
    }
}

var logger1 = Logger.GetInstance();
var logger2 = Logger.GetInstance();
// logger1 and logger2 are the same instance
```

**Use for:** Configuration objects, database connections, logging.

**Caveat:** Introduces shared global state, making testing harder.

### Builder

Constructs a complex object step by step using chained methods rather than a constructor with many parameters.

```csharp
public class QueryBuilder
{
    private string _select = "*";
    private string _from = "";
    private string _where = "";
    
    public QueryBuilder Select(string columns)
    {
        _select = columns;
        return this;  // Return this for chaining
    }
    
    public QueryBuilder From(string table)
    {
        _from = table;
        return this;
    }
    
    public QueryBuilder Where(string condition)
    {
        _where = condition;
        return this;
    }
    
    public string Build()
    {
        return $"SELECT {_select} FROM {_from} WHERE {_where}";
    }
}

// Usage — readable fluent interface
var query = new QueryBuilder()
    .Select("id, name")
    .From("users")
    .Where("age > 18")
    .Build();
```

### Factory Method

Defines an interface for creating an object, but lets a factory class/method decide which concrete class to instantiate.

```csharp
public interface ILogger
{
    void Log(string message);
}

public class LoggerFactory
{
    public static ILogger CreateLogger(string type)
    {
        return type.ToLower() switch
        {
            "console" => new ConsoleLogger(),
            "file" => new FileLogger(),
            "cloud" => new CloudLogger(),
            _ => throw new ArgumentException("Unknown logger type")
        };
    }
}

// Usage — no direct instantiation
ILogger logger = LoggerFactory.CreateLogger("file");
```

Instead of `new ConcreteClass()` throughout your code, you call a factory. Easier to swap implementations without changing call sites.

### Abstract Factory

A factory of factories. Groups related factory methods for creating families of related objects.

```csharp
public interface IUIFactory
{
    IButton CreateButton();
    ICheckbox CreateCheckbox();
}

public class WindowsUIFactory : IUIFactory
{
    public IButton CreateButton() => new WindowsButton();
    public ICheckbox CreateCheckbox() => new WindowsCheckbox();
}

public class MacUIFactory : IUIFactory
{
    public IButton CreateButton() => new MacButton();
    public ICheckbox CreateCheckbox() => new MacCheckbox();
}
```

Use when you need to create families of related objects (complete UI toolkits for different operating systems).

---

## Structural Patterns — How Objects Are Composed

### Facade

Provides a simplified interface to a complex subsystem. Instead of exposing many classes and methods, a facade exposes a small, clean surface.

```csharp
// Complex subsystem with many moving parts
public class DatabaseConnection { }
public class AuthenticationService { }
public class PaymentGateway { }

// Facade — simple, unified interface
public class OrderService
{
    private readonly DatabaseConnection _db = new();
    private readonly AuthenticationService _auth = new();
    private readonly PaymentGateway _payment = new();
    
    public void PlaceOrder(User user, Order order)
    {
        _auth.Authenticate(user);
        _payment.Charge(user, order.Total);
        _db.SaveOrder(order);
    }
}

// Client doesn't need to know about the subsystems
var orderService = new OrderService();
orderService.PlaceOrder(user, order);
```

Common in layered architectures — a service class coordinating multiple repository calls behind a single method.

### Adapter

Wraps an incompatible interface so it can work with code that expects a different interface.

```csharp
// Old system expects IPrinter
public interface IPrinter
{
    void Print(string text);
}

// New third-party library provides Printer3000
public class Printer3000
{
    public void Output(string content) { }
}

// Adapter wraps Printer3000 to implement IPrinter
public class PrinterAdapter : IPrinter
{
    private readonly Printer3000 _printer3000;
    
    public PrinterAdapter(Printer3000 printer) => _printer3000 = printer;
    
    public void Print(string text) => _printer3000.Output(text);
}

// Old code can now use Printer3000
IPrinter printer = new PrinterAdapter(new Printer3000());
printer.Print("Hello");
```

Like a physical power adapter — the underlying thing doesn't change, but its plug does.

### Decorator

Wraps an object to add new behavior without modifying the original class. Can be stacked.

```csharp
public interface IDataStream
{
    string Read();
}

public class FileStream : IDataStream
{
    public string Read() => /* read from file */;
}

public class CompressionDecorator : IDataStream
{
    private readonly IDataStream _stream;
    
    public CompressionDecorator(IDataStream stream) => _stream = stream;
    
    public string Read()
    {
        var data = _stream.Read();
        return Decompress(data);
    }
}

public class EncryptionDecorator : IDataStream
{
    private readonly IDataStream _stream;
    
    public EncryptionDecorator(IDataStream stream) => _stream = stream;
    
    public string Read()
    {
        var data = _stream.Read();
        return Decrypt(data);
    }
}

// Stack decorators — can be composed
IDataStream stream = new FileStream();
stream = new CompressionDecorator(stream);
stream = new EncryptionDecorator(stream);
```

---

## Behavioral Patterns — How Objects Communicate

### Observer

Defines a one-to-many dependency so that when one object changes state, all dependents are notified automatically.

Foundation of event systems and pub/sub messaging.

```csharp
public interface IObserver
{
    void Update(string notification);
}

public class NewsAgency
{
    private List<IObserver> _subscribers = new();
    
    public void Subscribe(IObserver observer) => _subscribers.Add(observer);
    
    public void PublishNews(string news)
    {
        foreach (var observer in _subscribers)
            observer.Update(news);
    }
}

public class NewsViewer : IObserver
{
    public void Update(string notification) => Console.WriteLine($"Breaking: {notification}");
}

// Usage
var agency = new NewsAgency();
agency.Subscribe(new NewsViewer());
agency.Subscribe(new NewsViewer());

agency.PublishNews("Major event!");  // Both viewers notified
```

### Strategy

Defines a family of algorithms, encapsulates each one, and makes them interchangeable. The client picks the strategy at runtime.

```csharp
public interface ISortStrategy
{
    void Sort(int[] array);
}

public class BubbleSort : ISortStrategy
{
    public void Sort(int[] array) { /* bubble sort logic */ }
}

public class QuickSort : ISortStrategy
{
    public void Sort(int[] array) { /* quicksort logic */ }
}

public class Sorter
{
    private ISortStrategy _strategy;
    
    public Sorter(ISortStrategy strategy) => _strategy = strategy;
    
    public void Sort(int[] array) => _strategy.Sort(array);
}

// Usage — swap strategies at runtime
var sorter = new Sorter(new QuickSort());
sorter.Sort(array);

// Later, switch algorithms without changing Sorter
sorter = new Sorter(new BubbleSort());
```

### Iterator

Provides a way to access elements of a collection sequentially without exposing the underlying structure.

In .NET, this is `IEnumerable<T>` / `IEnumerator<T>`. `foreach` works on anything implementing `IEnumerable`.

```csharp
public IEnumerable<int> GetEvenNumbers(int max)
{
    for (int i = 0; i <= max; i += 2)
        yield return i;  // Lazy iteration
}

// Usage
foreach (int num in GetEvenNumbers(10))
    Console.WriteLine(num);
```

### Command

Encapsulates a request as an object, allowing you to parameterize clients with requests, queue or log them, and support undoable operations.

```csharp
public interface ICommand
{
    void Execute();
    void Undo();
}

public class WriteCommand : ICommand
{
    private string _text;
    
    public WriteCommand(string text) => _text = text;
    
    public void Execute() => Console.WriteLine($"Writing: {_text}");
    public void Undo() => Console.WriteLine("Undo write");
}

public class CommandQueue
{
    private Stack<ICommand> _history = new();
    
    public void Execute(ICommand command)
    {
        command.Execute();
        _history.Push(command);
    }
    
    public void Undo()
    {
        if (_history.TryPop(out var command))
            command.Undo();
    }
}
```

---

## When to Apply Patterns

**Don't design with patterns preemptively.** Use patterns when you encounter recurring problems:

1. **Singleton** — When accessing shared resource (logger, config)
2. **Factory** — When object creation becomes complex or variable
3. **Adapter** — When integrating incompatible libraries
4. **Decorator** — When adding optional behaviors without modifying original
5. **Observer** — When changes in one object affect many others
6. **Strategy** — When you have multiple algorithms to choose from

**Premature pattern design is as bad as ignoring them entirely.**

---

## Next Steps

- [SOLID Principles](./solid.md) — Principles that patterns embody
- [Testing](./testing.md) — Verify pattern implementations work correctly
- [FEM C# and .NET](../03-fem-courses/01-csharp-dotnet/) — See patterns in action
