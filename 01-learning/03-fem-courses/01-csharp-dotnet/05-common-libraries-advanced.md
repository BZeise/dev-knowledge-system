# Common Libraries and Advanced Topics

*From "C# and .NET: From Beginner to Professional" by Spencer Schneidenbach (Frontend Masters)*

## Overview

Programming extends beyond syntax alone. This section explores essential libraries and advanced concepts that enhance C# development capabilities.

### Eight Core Namespaces

1. **System.Collections** — Classes and interfaces for collections (lists, dictionaries, queues)
2. **System.IO** — Reading and writing to files and streams
3. **System.Net** — Protocols used on the Internet
4. **System.Text.Json** — Serialization and deserialization of JSON data
5. **System.Net.Http** — Making HTTP requests and receiving responses
6. **System.Linq** — Querying and manipulating data in collections
7. **System.Threading.Tasks** — Threads and asynchronous operations
8. **System.Reflection** — Inspecting and manipulating assemblies, modules, and types at runtime

---

## Asynchronous Programming

Asynchronous programming is crucial for maintaining responsiveness and efficiency, especially when tasks involve waiting for external resources.

### Core Concept

Free up threads during I/O-bound operations (database queries, network requests) so applications remain responsive.

### Benefits

- **Responsiveness** — Background tasks don't freeze the UI
- **Resource Efficiency** — Threads aren't idle during waits, enabling better scalability in web frameworks like ASP.NET

### The async/await Pattern

Simplifies asynchronous code without explicit thread management:

```csharp
public async Task<string> GetDataFromServerAsync()
{
    var client = new HttpClient();
    string result = await client.GetStringAsync("https://example.com");
    return result;
}

// Usage
string data = await GetDataFromServerAsync();
```

### Task Types

**Task:** Represents an asynchronous operation with no return value:

```csharp
public async Task ProcessDataAsync()
{
    await Task.Delay(1000);  // Wait 1 second
    Console.WriteLine("Done");
}
```

**Task<T>:** Represents an asynchronous operation returning a value:

```csharp
public async Task<int> GetCountAsync()
{
    await Task.Delay(1000);
    return 42;
}
```

**ValueTask<T>:** Optimized version reducing allocations for synchronous completions:

```csharp
public async ValueTask<string> GetCachedValueAsync()
{
    if (_cache.TryGetValue("key", out var value))
    {
        return value;  // Synchronous — no allocation
    }
    
    var result = await FetchFromDatabaseAsync();
    return result;
}
```

### Best Practices (Do's)

**Async All the Way Down:**
```csharp
// GOOD — async all the way
public async Task HandleRequestAsync()
{
    var data = await GetDataAsync();
    return ProcessData(data);
}
```

**Use Async APIs:**
```csharp
// GOOD
await Task.Delay(1000);  // Async waiting

// AVOID
Thread.Sleep(1000);  // Blocks thread
```

**Favor Async Alternatives:**
```csharp
// GOOD
var data = await JsonSerializer.DeserializeAsync<T>(stream);

// AVOID
var data = JsonSerializer.Deserialize<T>(json);  // Synchronous
```

### Anti-Patterns (Don'ts)

**Avoid `.Result` or `.Wait()`:**
```csharp
// WRONG — can cause deadlocks
string data = GetDataAsync().Result;

// RIGHT
string data = await GetDataAsync();
```

**Don't Overuse Async:**
```csharp
// WRONG — not actually async
public async Task<int> Add(int a, int b)
{
    return a + b;  // No awaits!
}

// RIGHT
public int Add(int a, int b)
{
    return a + b;  // Synchronous is fine
}
```

**Never Use async void (Except Event Handlers):**
```csharp
// WRONG — exceptions can't be caught by caller
public async void ProcessData()
{
    await SomeOperationAsync();  // Exception crashes app!
}

// RIGHT — return Task
public async Task ProcessDataAsync()
{
    await SomeOperationAsync();
}

// EXCEPTION: Event handlers
private async void Button_Click(object sender, EventArgs e)
{
    await ProcessDataAsync();
}
```

**Don't Wrap Synchronous Methods:**
```csharp
// WRONG — creates inefficiency
public async Task<int> GetNumberAsync()
{
    return await Task.Run(() => GetSyncNumber());
}

// RIGHT — just call it
public Task<int> GetNumberAsync()
{
    return Task.FromResult(GetSyncNumber());
}
```

---

## Dispose Pattern

The `Dispose` method releases unmanaged resources held by objects.

### Why Dispose?

Cleanup prevents resource leaks:
- File handles
- Network connections
- Database connections
- Native memory allocations

### IDisposable Interface

```csharp
public interface IDisposable
{
    void Dispose();
}

public class FileReader : IDisposable
{
    private FileStream _stream;
    
    public FileReader(string path)
    {
        _stream = File.OpenRead(path);
    }
    
    public void Dispose()
    {
        _stream?.Dispose();
        _stream = null;
    }
}
```

### Using Statement

Automatically calls `Dispose()` when exiting a block:

```csharp
// Block syntax (clear)
using (var reader = new FileReader("file.txt"))
{
    // Use reader
}  // Dispose() called automatically

// Declaration syntax (inline)
using var reader = new FileReader("file.txt");
// Use reader
// Dispose() called when exiting scope
```

### Async Disposal

For asynchronous operations, use `IAsyncDisposable`:

```csharp
public class AsyncResource : IAsyncDisposable
{
    public async ValueTask DisposeAsync()
    {
        await CloseConnectionAsync();
    }
}

// Usage
await using var resource = new AsyncResource();
// Use resource
// DisposeAsync() called automatically
```

**Guideline:** "If your method is async and your resource is disposable, use `await using`"

---

## Streams

A stream is an abstraction representing a sequence of bytes for input/output operations.

### Common Stream Types

**FileStream** — Disk file operations:
```csharp
using (var stream = File.OpenRead("file.txt"))
{
    byte[] buffer = new byte[1024];
    int bytesRead = stream.Read(buffer, 0, buffer.Length);
}
```

**MemoryStream** — In-memory data storage:
```csharp
var stream = new MemoryStream();
byte[] data = Encoding.UTF8.GetBytes("Hello");
stream.Write(data, 0, data.Length);
```

**NetworkStream** — Network communication:
```csharp
using (var client = new TcpClient("example.com", 80))
{
    NetworkStream stream = client.GetStream();
}
```

### Common Patterns

**Reading Files:**
```csharp
using (var fileStream = File.OpenRead("data.txt"))
using (var reader = new StreamReader(fileStream))
{
    string content = reader.ReadToEnd();
}
```

**Writing Files:**
```csharp
using (var fileStream = File.Create("output.txt"))
using (var writer = new StreamWriter(fileStream))
{
    writer.WriteLine("Hello, World!");
}
```

**Memory Operations:**
```csharp
byte[] data = Encoding.UTF8.GetBytes("Hello");
using (var stream = new MemoryStream(data))
using (var reader = new StreamReader(stream))
{
    string text = reader.ReadToEnd();
}
```

### Warning: MemoryStream Misuse

**Avoid loading entire files into memory:**
```csharp
// WRONG — entire file in memory
byte[] data = File.ReadAllBytes("large-file.bin");
var stream = new MemoryStream(data);

// RIGHT — stream directly
using (var stream = File.OpenRead("large-file.bin"))
{
    ProcessStream(stream);
}
```

---

## File APIs

Working with files through three main classes: `File`, `FileInfo`, and path utilities.

### File Class (Static Methods)

**Checking Existence:**
```csharp
bool exists = File.Exists("path/to/file.txt");
```

**Reading Operations:**
```csharp
// Entire file as string
string content = File.ReadAllText("file.txt");

// File as lines
string[] lines = File.ReadAllLines("file.txt");

// Raw bytes
byte[] data = File.ReadAllBytes("file.bin");
```

**Writing Operations:**
```csharp
// Overwrite or create
File.WriteAllText("file.txt", "Hello, World!");

// Append
File.AppendAllText("file.txt", "\nMore text");

// Write lines
string[] lines = { "Line 1", "Line 2" };
File.WriteAllLines("file.txt", lines);
```

**File Operations:**
```csharp
// Copy
File.Copy("source.txt", "destination.txt");

// Move
File.Move("old-name.txt", "new-name.txt");

// Delete
File.Delete("file.txt");
```

### FileInfo Class (Instance-Based)

```csharp
var fileInfo = new FileInfo("document.pdf");

long sizeInBytes = fileInfo.Length;
DateTime created = fileInfo.CreationTime;
DateTime modified = fileInfo.LastWriteTime;

fileInfo.Delete();
```

### Directory and Path Utilities

**Creating Directories:**
```csharp
Directory.CreateDirectory("path/to/new/folder");
// Creates all missing parent directories
```

**Path Manipulation:**
```csharp
string combined = Path.Combine("folder", "subfolder", "file.txt");
string directory = Path.GetDirectoryName(combined);  // "folder/subfolder"
string filename = Path.GetFileName(combined);        // "file.txt"
string extension = Path.GetExtension(combined);      // ".txt"

// Temporary files
string tempPath = Path.GetTempFileName();
string randomName = Path.GetRandomFileName();
```

---

## Making HTTP Requests

Using `HttpClient` for HTTP communication.

### HttpClient Overview

```csharp
var client = new HttpClient();
```

### HttpRequestMessage

Represents an outgoing HTTP request:

```csharp
var request = new HttpRequestMessage(HttpMethod.Get, "https://api.example.com/users");
request.Headers.Authorization = new AuthenticationHeaderValue("Bearer", token);

HttpResponseMessage response = await client.SendAsync(request);
```

### Default Headers

```csharp
client.DefaultRequestHeaders.Add("User-Agent", "MyApp/1.0");
client.DefaultRequestHeaders.Authorization = 
    new AuthenticationHeaderValue("Bearer", token);
```

### HTTP Methods

**GET Requests:**
```csharp
HttpResponseMessage response = await client.GetAsync("https://api.example.com/users");
string content = await response.Content.ReadAsStringAsync();
```

**POST Requests:**
```csharp
var data = new { Name = "John", Age = 30 };
var json = JsonSerializer.Serialize(data);
var content = new StringContent(json, Encoding.UTF8, "application/json");

HttpResponseMessage response = await client.PostAsync("https://api.example.com/users", content);
```

**PUT/DELETE:**
```csharp
await client.PutAsync(url, content);
await client.DeleteAsync(url);
```

### Response Handling

```csharp
HttpResponseMessage response = await client.GetAsync(url);

// Check success
if (response.IsSuccessStatusCode)
{
    string body = await response.Content.ReadAsStringAsync();
    var user = JsonSerializer.Deserialize<User>(body);
}

// Throw on failure
response.EnsureSuccessStatusCode();

// Deserialize JSON directly
var user = await response.Content.ReadFromJsonAsync<User>();
```

### Best Practices

**Reuse HttpClient:**
```csharp
// GOOD — reuse single instance
public class UserService
{
    private static readonly HttpClient _client = new();
    
    public async Task<User> GetUserAsync(int id)
    {
        return await _client.GetFromJsonAsync<User>($"/users/{id}");
    }
}

// BETTER — use IHttpClientFactory in ASP.NET Core
public class UserService
{
    private readonly IHttpClientFactory _clientFactory;
    
    public async Task<User> GetUserAsync(int id)
    {
        var client = _clientFactory.CreateClient();
        return await client.GetFromJsonAsync<User>($"/users/{id}");
    }
}
```

**Don't Dispose HttpClient:**
```csharp
// WRONG
using (var client = new HttpClient())
{
    // Use it
}

// RIGHT — reuse the same instance
static HttpClient client = new HttpClient();
```

---

## JSON: Serialization and Deserialization

Working with JSON data in .NET.

### Historical Context

**XML** was once primary data interchange format but was verbose and complex.  
**JSON** emerged as simpler, more readable alternative.

### Two Primary Libraries

**System.Text.Json** (Recommended for New Projects):
- Introduced in .NET Core 3.0
- Emphasizes "speed, accuracy, exactness, security"
- Better performance

```csharp
var person = new Person { Name = "Alice", Age = 30 };

// Serialize
string json = JsonSerializer.Serialize(person);

// Deserialize
var restored = JsonSerializer.Deserialize<Person>(json);
```

**Newtonsoft.Json (Json.NET)** (For Existing Code):
- Older, widely established
- "Relaxed and forgiving" approach
- Handles non-standard JSON (comments, trailing commas)

```csharp
string json = JsonConvert.SerializeObject(person);
var restored = JsonConvert.DeserializeObject<Person>(json);
```

### Practical Recommendations

**For New Projects:** Use `System.Text.Json`

**For Existing Codebases:** Avoid mixing both libraries. If Newtonsoft is already used, maintain consistency unless performance becomes critical.

### Special Consideration: Enums

**Strongly recommend:** Configure serializers to output enums as strings:

```csharp
public enum Status { Pending, Active, Inactive }

// GOOD OUTPUT
{ "status": "Active" }

// BAD OUTPUT (default)
{ "status": 1 }
```

**Configuration:**
```csharp
var options = new JsonSerializerOptions
{
    Converters = { new JsonStringEnumConverter() }
};

string json = JsonSerializer.Serialize(obj, options);
```

---

## Reflection and Attributes

Understanding metadata inspection and manipulation.

### Reflection Overview

Reflection enables inspecting and manipulating code metadata at runtime.

**Key Capabilities:**
- Print object properties to console
- Set property values dynamically
- Invoke methods at runtime
- Instantiate types without knowing them at compile time

### Practical Examples

**Printing Properties:**
```csharp
var person = new Person { Name = "Alice", Age = 30 };

Type type = person.GetType();
PropertyInfo[] properties = type.GetProperties();

foreach (var prop in properties)
{
    Console.WriteLine($"{prop.Name}: {prop.GetValue(person)}");
}
```

**Setting Property Values:**
```csharp
var person = new Person();
PropertyInfo nameProp = person.GetType().GetProperty("Name");
nameProp.SetValue(person, "Bob");
```

**Invoking Methods:**
```csharp
MethodInfo method = type.GetMethod("Speak");
method.Invoke(person, null);
```

### Why Reflection Matters

Not immediately obvious to beginners. Real-world applications emerge when studying frameworks like ASP.NET Core, which heavily use reflection.

### Attributes

Metadata annotations that frameworks use to modify application behavior.

### Common Attributes

**Data Validation:**
```csharp
public class User
{
    [Required]
    public string Username { get; set; }
    
    [EmailAddress]
    public string Email { get; set; }
    
    [Range(18, 120)]
    public int Age { get; set; }
}
```

**Custom Attributes:**
```csharp
[AttributeUsage(AttributeTargets.Class | AttributeTargets.Method)]
public class LoggingAttribute : Attribute
{
    public string Description { get; set; }
}

[Logging(Description = "User creation")]
public class UserService
{
    [Logging]
    public void CreateUser(User user) { }
}
```

**Custom Attribute with Reflection:**
```csharp
var type = typeof(UserService);
var attribute = type.GetCustomAttribute<LoggingAttribute>();
if (attribute != null)
{
    Console.WriteLine($"Description: {attribute.Description}");
}
```

---

## Next Steps

Master application building:
- [06-Hosting Model and API Development](./06-hosting-api-development.md) — Build your first API
- [Testing](../../02-concepts/testing.md) — Ensure reliability

---

*Content licensed under CC-BY-NC-4.0 | Code samples under Apache 2.0*
