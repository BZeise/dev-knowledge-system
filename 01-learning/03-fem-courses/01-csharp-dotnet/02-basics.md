# Basics of C#

*From "C# and .NET: From Beginner to Professional" by Spencer Schneidenbach (Frontend Masters)*

## Syntax Overview

C# is a modern, object-oriented programming language developed by Microsoft. The syntax derives from C and C++ but incorporates improvements for robustness and developer experience.

### Code Structure

**Code Blocks and Statements:**
- Uses curly braces `{}` to organize code sections
- All statements terminate with semicolons `;`
- While not whitespace-dependent, consistent indentation enhances readability

**Example:**
```csharp
if (condition)
{
    // code block
}
```

### Comments

**Single-line comments:**
```csharp
// This is a single-line comment
```

**Multi-line comments:**
```csharp
/* This is a 
   multi-line comment */
```

### Common Programming Mistakes to Avoid

1. **Omitting semicolons** after statements
2. **Unbalanced or mismatched braces** — every `{` needs a matching `}`
3. **Using assignment `=` instead of equality `==`** in conditions
4. **Declaring variables after attempting use** — declare first
5. **Incorrect letter casing** — C# is case-sensitive

**Key Principle:** C# is not whitespace-sensitive, but consistent indentation greatly improves code readability.

---

## Variables

### Declaration and Initialization

Variables require declaration before use following the syntax:
```csharp
dataType variableName;
```

Use **camel-case** naming conventions for variable identifiers:
```csharp
int age = 25;
string name = "John Doe";
double salary = 50000.50;
```

### Type Inference with `var`

C# supports type inference using the `var` keyword. The compiler infers the type automatically based on assigned values:
```csharp
var count = 42;           // inferred as int
var message = "Hello";    // inferred as string
var price = 19.99;        // inferred as double
```

The instructor almost exclusively uses `var` in practice.

### Constants

Unchangeable variables use the `const` keyword:
```csharp
const int MaxScore = 100;
const string ApiKey = "secret";
```

### Scope Levels

Variables operate within three scope categories:
- **Block level** — within `{ }`
- **Method level** — within a method
- **Class level** — within a class

Variables declared in narrower scopes shadow outer scopes.

### Type Categories

**Value Types** (stored directly on the stack):
- `int`, `float`, `bool`, `struct`, `decimal`
- Initialize with default values (0, 0.0, false, etc.)

**Reference Types** (stored on the heap):
- `class`, `interface`, `delegate`, `arrays`, `string`
- Typically nullable and initialize to `null`

---

## Methods

Methods encapsulate reusable logic. The fundamental structure includes an access modifier, return type, name, parameters, and body.

### Basic Structure

```csharp
[access-modifier] return-type MethodName(parameters) 
{
    // method body
}
```

**Naming Convention:** Use **PascalCase** for method names (e.g., `ThisIsPascalCase()`)

### Parameter Types

**Required Parameters:**
```csharp
public int Add(int a, int b)
{
    return a + b;
}
```

**Optional Parameters** (with default values):
```csharp
public void GreetUser(string name = "User")
{
    Console.WriteLine($"Hello, {name}!");
}
```

**Variable-Length Parameters:**
```csharp
public int Sum(params int[] numbers)
{
    int total = 0;
    foreach (int num in numbers)
    {
        total += num;
    }
    return total;
}
```

### Return Types

**Void Methods** (no return value):
```csharp
public void PrintMessage(string message)
{
    Console.WriteLine(message);
}
```

**Value-Returning Methods:**
```csharp
public double CalculateAverage(double[] values)
{
    return values.Sum() / values.Length;
}
```

The `return` statement serves dual purposes:
- Exits the method immediately
- Passes a value back to the caller

Every code path must terminate with a return statement (exceptions being an alternative exit path).

---

## Built-in Types

### Type Hierarchy

ALL types inherit from `object` — the foundation of C#'s type system.

### Text Types

**String:**
```csharp
string message = "Hello, world!";
```

**Character:**
```csharp
char letter = 'A';
```

### Numeric Types

**Integer (signed):**
- `sbyte` — 8-bit signed
- `short` — 16-bit signed
- `int` — 32-bit signed (most common)
- `long` — 64-bit signed

**Integer (unsigned):**
- `byte` — 8-bit unsigned
- `ushort` — 16-bit unsigned
- `uint` — 32-bit unsigned
- `ulong` — 64-bit unsigned

**Floating-Point:**
- `float` — 32-bit, precision ~6-9 digits
- `double` — 64-bit, precision ~15-17 digits
- `decimal` — 128-bit, high precision for financial data (prefer for money)

### Boolean

```csharp
bool isActive = true;
bool hasPermission = false;
```

### DateTime

```csharp
DateTime now = DateTime.UtcNow;
DateTime birthday = new DateTime(1990, 5, 15);

// Properties
int day = now.Day;
int month = now.Month;
int year = now.Year;
```

**Important:** Use `DateTime.UtcNow` in production code, not `DateTime.Now` due to machine-dependent time zone issues.

### Collections & Advanced Types

**Arrays:**
```csharp
int[] numbers = { 1, 2, 3, 4, 5 };
string[] names = new string[3];
```

**Enums:**
```csharp
public enum Status { Pending, Active, Inactive }
Status current = Status.Active;
```

**Tuples:**
```csharp
var point = (10, 20);  // anonymous tuple
var person = (Age: 30, Name: "Alice", IsEmployed: true);  // named tuple
```

**Null:**
Represents absence of value. Reference types are nullable by default.

**Void:**
Indicates no return from a method.

---

## Strings

### String Characteristics

Strings are a sequence of characters with three fundamental characteristics:
- **Unicode-encoded** — supports international characters
- **Immutable** — cannot be changed after creation
- **Reference types** — but behave like value types in practice

### String Types

**Standard Strings:**
```csharp
string text = "Hello, \"world\"!";  // use \" for quotes
string path = "C:\\Users\\John";     // use \\ for backslashes
string lines = "Line 1\nLine 2";     // use \n for newlines
string tab = "Col1\tCol2";           // use \t for tabs
```

**Verbatim Strings** (prefix with `@`):
```csharp
string path = @"C:\Users\John";      // backslashes literal
string multiline = @"Line 1
Line 2
Line 3";
```

**Raw Strings** (triple-quoted, C# 11+):
```csharp
string json = """
{
    "name": "John",
    "age": 30
}
""";
```

**Interpolated Strings:**
```csharp
var name = "Alice";
var age = 25;
string message = $"Hello, {name}! You are {age} years old.";
string calculation = $"2 + 2 = {2 + 2}";
```

### Essential String Methods

**Properties:**
- `string.Length` — Returns character count

**Searching:**
- `string.Contains(substring)` — Check if substring exists
- `string.StartsWith(prefix)` — Check beginning
- `string.EndsWith(suffix)` — Check ending
- `string.IndexOf(value)` — Find position
- `string.LastIndexOf(value)` — Find last position

**Manipulation:**
- `string.ToLower()` — Convert to lowercase
- `string.ToUpper()` — Convert to uppercase
- `string.Trim()` — Remove leading/trailing whitespace
- `string.TrimStart()` — Remove leading whitespace only
- `string.TrimEnd()` — Remove trailing whitespace only
- `string.Replace(old, new)` — Replace text
- `string.Substring(startIndex)` — Extract portion
- `string.Split(separator)` — Split into array

**Combining:**
- `string.Join(separator, values)` — Merge array into string

### Empty String Handling

Use `string.Empty` over `""`:
```csharp
if (text == string.Empty)
{
    // text is empty
}
```

Check with:
```csharp
string.IsNullOrEmpty(text)      // null or empty
string.IsNullOrWhiteSpace(text)  // null, empty, or whitespace
```

### String Comparison

Prefer using `StringComparison` parameters:
```csharp
// Good
bool isEqual = text.Equals("hello", StringComparison.OrdinalIgnoreCase);

// Avoid
bool isEqual = text == "hello";  // uses culture-specific comparison
```

### Performance: StringBuilder

For loops with string concatenation, use `StringBuilder`:
```csharp
// Inefficient — creates new strings each iteration
string result = "";
for (int i = 0; i < 1000; i++)
{
    result += i.ToString();  // Bad!
}

// Efficient — mutable string building
var sb = new StringBuilder();
for (int i = 0; i < 1000; i++)
{
    sb.Append(i);  // Good!
}
string result = sb.ToString();
```

---

## Control Flow

### Conditional Statements

**If Statement:**
```csharp
if (age >= 18)
{
    Console.WriteLine("You are an adult");
}
```

**If-Else Statement:**
```csharp
if (grade >= 90)
{
    Console.WriteLine("A");
}
else if (grade >= 80)
{
    Console.WriteLine("B");
}
else
{
    Console.WriteLine("C or lower");
}
```

**Switch Statements:**
```csharp
switch (dayOfWeek)
{
    case "Monday":
        Console.WriteLine("Start of week");
        break;
    case "Friday":
        Console.WriteLine("Almost weekend!");
        break;
    default:
        Console.WriteLine("Midweek day");
        break;
}
```

**Modern Switch Expressions:**
```csharp
string message = dayOfWeek switch
{
    "Monday" => "Start of week",
    "Friday" => "Almost weekend!",
    _ => "Midweek day"
};
```

### Loops

**For Loop** (most ubiquitous, though rarely used by the instructor):
```csharp
for (int i = 0; i < 10; i++)
{
    Console.WriteLine(i);
}
```

**While Loop:**
```csharp
int count = 0;
while (count < 10)
{
    Console.WriteLine(count);
    count++;
}

// Common pattern
while (true)
{
    if (condition)
        break;
}
```

**Do-While Loop** (executes at least once):
```csharp
int value;
do
{
    value = GetUserInput();
} while (value < 0);
```

**Foreach Loop** (instructor's favorite for iterating collections):
```csharp
int[] numbers = { 1, 2, 3, 4, 5 };
foreach (int number in numbers)
{
    Console.WriteLine(number);
}
```

### Flow Control Keywords

**Break:**
```csharp
for (int i = 0; i < 10; i++)
{
    if (i == 5)
        break;  // exits loop immediately
}
```

**Continue:**
```csharp
for (int i = 0; i < 10; i++)
{
    if (i % 2 == 0)
        continue;  // skips to next iteration
    Console.WriteLine(i);
}
```

---

## Exception Handling

Exception handling manages errors when code encounters unrecoverable problems.

### Try/Catch Blocks

```csharp
try
{
    int result = 10 / int.Parse("0");  // throws DivideByZeroException
}
catch (DivideByZeroException)
{
    Console.WriteLine("Cannot divide by zero");
}
catch (FormatException)
{
    Console.WriteLine("Invalid number format");
}
```

### Finally Block

Executes after try/catch regardless of outcome:
```csharp
try
{
    // risky code
}
catch (Exception ex)
{
    Console.WriteLine($"Error: {ex.Message}");
}
finally
{
    Console.WriteLine("Cleanup code runs here");
}
```

### Throwing Exceptions

Use the `throw` keyword with descriptive messages:
```csharp
public void SetAge(int age)
{
    if (age < 0)
        throw new ArgumentException("Age cannot be negative", nameof(age));
    
    _age = age;
}
```

### Custom Exception Classes

```csharp
public class InvalidOperationException : Exception
{
    public InvalidOperationException(string message) 
        : base(message)
    {
    }
}
```

**Note:** For typical applications, use standard exceptions like `ArgumentException`, `InvalidOperationException`, etc. Custom exceptions suit framework code.

---

## Pattern Matching

Pattern matching allows you to check a value against a pattern and execute different code based on the result. It serves as an enhanced alternative to traditional switch statements.

### Type Patterns

```csharp
object obj = "Hello";
if (obj is string str)
{
    Console.WriteLine($"String: {str}");
}
```

### Constant Patterns

```csharp
int number = 42;
if (number is 42)
{
    Console.WriteLine("The answer!");
}
```

### Relational Patterns

```csharp
int age = 25;
if (age is >= 18 and < 65)
{
    Console.WriteLine("Working age");
}
```

### Logical Patterns

```csharp
if (obj is not null and is string)
{
    Console.WriteLine("Non-null string");
}
```

### Property Patterns

```csharp
public record Person(string Name, int Age);

var person = new Person("Alice", 30);
if (person is { Name: "Alice", Age: >= 18 })
{
    Console.WriteLine("Adult Alice");
}
```

### Positional Patterns

```csharp
var point = (10, 20);
if (point is (0, 0))
{
    Console.WriteLine("Origin");
}
```

---

## Operators

### Arithmetic Operators

```csharp
int a = 10;
int b = 3;

a + b      // 13 — Addition
a - b      // 7 — Subtraction
a * b      // 30 — Multiplication
a / b      // 3 — Division (integer division)
a % b      // 1 — Modulus (remainder)
a++        // Increment (post-increment returns old value)
++a        // Pre-increment (returns new value)
a--        // Decrement
--a        // Pre-decrement
```

### Relational/Comparison Operators

```csharp
a == b     // Equal?
a != b     // Not equal?
a > b      // Greater than?
a < b      // Less than?
a >= b     // Greater than or equal?
a <= b     // Less than or equal?
```

### Logical Operators

```csharp
true && true   // AND — short-circuits (stops if first is false)
true || false  // OR — short-circuits (stops if first is true)
!true          // NOT — inverts boolean

true & true    // Bitwise AND — always evaluates both
true | false   // Bitwise OR — always evaluates both
```

### Assignment Operators

```csharp
a = 5         // Basic assignment
a += 3        // a = a + 3
a -= 2        // a = a - 2
a *= 2        // a = a * 2
a /= 2        // a = a / 2
a %= 3        // a = a % 3
```

---

## Type Conversion

Converting between types in C#.

### Implicit Conversion

The compiler automatically handles compatible type conversions:
```csharp
int number = 42;
double decimal_number = number;  // OK — int fits in double
```

Incompatible types require explicit conversion.

### Explicit Conversion (Casting)

```csharp
double value = 42.5;
int whole = (int)value;  // Explicit cast — result is 42
```

### Parse Method

Converts strings to specified types, throws exception on failure:
```csharp
int number = int.Parse("42");  // OK
int number = int.Parse("abc"); // Throws FormatException
```

### TryParse Method

Safely attempts conversion without throwing:
```csharp
if (int.TryParse("42", out int number))
{
    Console.WriteLine($"Parsed: {number}");
}
else
{
    Console.WriteLine("Invalid number");
}

// Modern syntax with inline declaration
if (int.TryParse(input, out int result))
{
    // result is available here
}
```

### Convert Class

```csharp
int number = Convert.ToInt32("42");     // Throws on failure
double decimal_value = Convert.ToDouble("3.14");
string text = Convert.ToString(42);
```

**Choosing the Right Method:**
- **Parse/Convert:** Use when you expect valid input
- **TryParse:** Use when input might be invalid and you need to handle gracefully

---

## Next Steps

Continue your journey:
- [03-Types, Classes, and Structures](./03-types-classes-structures.md) — Learn OOP fundamentals
- [Core OOP Concepts](../../02-concepts/oop.md) — Understand object-oriented principles

---

*Content licensed under CC-BY-NC-4.0 | Code samples under Apache 2.0*
