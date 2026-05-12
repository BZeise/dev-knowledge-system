# Introduction

*From "Building APIs with C# & ASP.NET Core" by Spencer Schneidenbach (Frontend Masters)*

---

## Lesson — Introduction

---

Hi, I'm Spencer, and I'm super excited to be here at Frontend Masters to teach you about building APIs with ASP.NET Core.

### A Bold Declaration

**.NET is the best platform for building web apps in the world.**

And by extension, that makes ASP.NET Core the best web framework in the world.

### About the Instructor

Spencer Schneidenbach:

- President and CTO of [Aviron Software](https://avironsoftware.com), a software consulting firm specializing in .NET development, Microsoft Azure, React, and React Native.
- Recognized as a Microsoft MVP for nearly 10 years in Developer Technologies, specifically .NET.
- .NET consultant with many years of hands-on experience across dozens of codebases from startups to enterprises.
- Conference speaker around the world on many different .NET topics.

### The Goal of This Course

The goal of this course is to teach you how Spencer builds web APIs using .NET and ASP.NET Core. It's built from the ground up to be practical, pragmatic, and demonstrate how **real applications** are built.

There are **no ivory towers** here. The course is meant for productivity. Spencer wants to highlight what has worked well and what he's avoided to be successful and productive in .NET.

Spencer believes in minimizing "magic" in code — it should be easy for other developers to understand what is happening. Thus, the course digs into actual decompiled source code to understand what's going on under the hood.

### Why Choose ASP.NET Core?

1. **LOTS AND LOTS OF JOBS!** — Huge demand in the marketplace for .NET skills.
2. **Fun to use and write** — More fun than any other language and platform (Spencer's opinion).
3. **Open Source and Cross-Platform** — .NET is fully open source and runs on multiple platforms.
4. **Strong Developer Ecosystem** — Benefit from the "network effect" of a large, active community.
5. **High Performance and Scalability** — Modern .NET is designed for efficiency and handles projects of any size.
6. **Modern and Secure** — Backed by Microsoft and regularly kept up-to-date.
7. **Plays nicely with others** — Pairs nicely with frontend frameworks like React.

### What You'll Gain

- **Real-World Experience** — ASP.NET Core as it's used in actual projects, not just theory.
- **Professional Best Practices** — How experienced developers write ASP.NET Core, including what pitfalls to avoid.
- **Source Code Deep Dives** — Examination of real ASP.NET Core source code regularly.
- **Practical Examples** — Real-world scenarios throughout to solidify understanding.

### One Last Thing

Spencer is pretty opinionated. Opinions are useful because they help you make quick decisions — just don't be dogmatic about 'em. Quick decisions are almost always better than analysis paralysis. Mistakes will happen, and ultimately that's a good thing.

> Some mistakes will be made along the way. That's _good_. **Because at least some decisions are being made along the way.** And we'll find the mistakes. We'll fix 'em. — Steve Jobs

> I used to have a saying to put things into perspective when things were getting really crazy at work and we were freaking out over the Daily Crisis: Breathe. It's just software, we're not saving babies here. — Scott Hanselman

Spicy takes will be denoted with: 🌶️🌶️🌶️

---

## Lesson — Tooling Overview

---

### Install the dotnet CLI

1. Visit the official .NET website: [https://dot.net](https://dot.net)
2. Download and install the .NET SDK for your operating system
3. Verify installation by opening a terminal and running:

```bash
dotnet --version
```

> **macOS Users:** If the `dotnet` command is not found, you may need to create a sym link after running the installer:
> ```bash
> sudo ln -s /usr/local/share/dotnet/dotnet /usr/local/bin/
> ```

### Install Visual Studio Code

1. Visit: [https://code.visualstudio.com/](https://code.visualstudio.com/)
2. Download the installer for your OS (Windows, macOS, or Linux)
3. Run the installer and follow the wizard

#### Install C# Extension for VS Code

1. Open VS Code
2. Go to Extensions (`Ctrl+Shift+X` / `Cmd+Shift+X`)
3. Search for "C#"
4. Install **C# Dev Kit** by Microsoft

This provides IntelliSense, debugging, and more. The extension has licensing requirements for businesses but is free for individuals.

Spencer also strongly recommends [**NuGet Gallery**](https://marketplace.visualstudio.com/items?itemName=patcx.vscode-nuget-gallery) — provides a nice UI for exploring NuGet packages, much nicer than using the CLI.

### Install Postman (Optional)

Postman is a powerful tool for testing APIs. Used alongside Swagger in this course.

1. Visit: [https://www.postman.com/downloads/](https://www.postman.com/downloads/)
2. Download and install for your OS

### Using the dotnet CLI

Essential commands:

- **`dotnet new`** — Creates a new project, configuration file, or solution based on a template.

  ```bash
  dotnet new console -n MyConsoleApp
  ```

  Common templates:
  - `console` — Console application
  - `classlib` — Class library
  - `web` — ASP.NET Core empty web app
  - `webapi` — ASP.NET Core Web API
  - `mvc` — ASP.NET Core Web App (Model-View-Controller)
  - `blazorserver` — Blazor Server App
  - `blazorwasm` — Blazor WebAssembly App
  - `wpf` — Windows Presentation Foundation Application
  - `winforms` — Windows Forms Application

  List all available templates:
  ```bash
  dotnet new list
  ```

- **`dotnet build`** — Builds a .NET project and all its dependencies.
  ```bash
  dotnet build
  ```

- **`dotnet publish`** — Publishes the application and dependencies for deployment.
  ```bash
  dotnet publish -c Release
  ```

- **`dotnet run`** — Runs source code without any explicit compile or launch commands.
  ```bash
  dotnet run
  ```

- **`dotnet restore`** — Restores the dependencies and tools of a project.
  ```bash
  dotnet restore
  ```

- **`dotnet test`** — Runs unit tests using the test runner specified in the project.
  ```bash
  dotnet test
  ```

### Other IDEs/Tools

- **Visual Studio** — Full-featured IDE for Windows with comprehensive .NET development capabilities.
- **ReSharper** — Powerful extension for Visual Studio. Spencer considers it 100% essential for VS development.
- **JetBrains Rider** — Cross-platform .NET IDE. Spencer considers it the best .NET IDE available.

---

## Lesson — HTTP Overview and the Bridge to ASP.NET Core

---

Before talking about ASP.NET Core, it's worth covering the major parts of HTTP so we know how each thing fits into the bigger picture.

### What is HTTP?

HTTP is a protocol that allows clients to communicate with servers. It's the protocol that powers the web.

### A Typical Request/Response

Example GET request from client to server:

```
GET /index.html HTTP/1.1
Host: www.example.com
Accept: text/html
Accept-Language: en-US
```

| Line | Description |
|------|-------------|
| `GET /index.html HTTP/1.1` | The request line: HTTP method (GET), resource (/index.html), and HTTP version (1.1). |
| `Host: www.example.com` | Specifies the domain name of the server to which the request is sent. |
| `Accept: text/html` | Tells the server what content types the client can understand. |
| `Accept-Language: en-US` | Indicates the natural language and locale the client prefers. |

The response:

```
HTTP/1.1 200 OK
Content-Type: text/html

<!DOCTYPE html>
<html lang="en">
...
```

| Line | Description |
|------|-------------|
| `HTTP/1.1 200 OK` | Status code line: HTTP version, status code (200), and description (OK). |
| `Content-Type: text/html` | Specifies the MIME type of the response body. |

### HTTP Methods

| Method | Purpose |
|--------|---------|
| `GET` | Requests a representation of the specified resource. |
| `POST` | Submits an entity to a resource, often causing a change in state or side effects. |
| `PUT` | Replaces the specified resource with the request payload. |
| `DELETE` | Deletes the specified resource. |
| `PATCH` | Applies partial modifications to a resource. *(less common)* |
| `HEAD` | Like GET but without the response body. *(less common)* |
| `OPTIONS` | Describes the communication options for the target resource. *(less common)* |

Most of the time, people use the top four: GET, POST, PUT, and DELETE.

### HTTP Status Codes

Status codes are three-digit codes that indicate the result of a request. Five classes:

- **1xx** — Informational: Request received, continuing process
- **2xx** — Success: The action was successfully received, understood, and accepted
- **3xx** — Redirection: Further action must be taken to complete the request
- **4xx** — Client Error: The request contains bad syntax or cannot be fulfilled
- **5xx** — Server Error: The server failed to fulfill an apparently valid request

Most common status codes for this course:

| Code | Description |
|------|-------------|
| 200 | OK — The request was successful |
| 201 | Created — A new resource was successfully created |
| 400 | Bad Request — The request is invalid |
| 401 | Unauthorized — Authentication is required |
| 403 | Forbidden — Access is forbidden |
| 404 | Not Found — The requested resource was not found |
| 500 | Internal Server Error — The server encountered an error |

### POST Requests and the Body

`GET` requests don't support bodies, but `POST` does. Example:

```
POST /employees/create HTTP/1.1
Host: www.example.com
Content-Type: application/json

{
    "name": "John Doe",
    "email": "john.doe@example.com",
    "message": "Hello, world!"
}
```

- The `POST` method submits data to the server.
- The `Content-Type` header specifies the MIME type of the request body (JSON in this case).

### What is ASP.NET Core?

ASP.NET Core is a framework for building web applications and APIs. It's built on the .NET runtime and is designed to be a high-performance, modular web framework.

One command creates a new ASP.NET Core project:

```bash
dotnet new webapi -n MyWebApp
```

### The Generated Program.cs

```csharp
var builder = WebApplication.CreateBuilder(args);

// Add services to the container.
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

// Configure the HTTP request pipeline.
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();

var summaries = new[]
{
    "Freezing", "Bracing", "Chilly", "Cool", "Mild", "Warm", "Balmy", "Hot", "Sweltering", "Scorching"
};

app.MapGet("/weatherforecast", () =>
{
    var forecast = Enumerable.Range(1, 5).Select(index =>
        new WeatherForecast
        (
            DateOnly.FromDateTime(DateTime.Now.AddDays(index)),
            Random.Shared.Next(-20, 55),
            summaries[Random.Shared.Next(summaries.Length)]
        ))
        .ToArray();
    return forecast;
})
.WithName("GetWeatherForecast")
.WithOpenApi();

app.Run();

record WeatherForecast(DateOnly Date, int TemperatureC, string? Summary)
{
    public int TemperatureF => 32 + (int)(TemperatureC / 0.5556);
}
```

Breaking it down:

```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();
var app = builder.Build();
```
Configures the services that the application will use.

```csharp
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}
```
Configures the HTTP request pipeline. Any place where the API starts with `Use` or `Get` will typically be adding something to the HTTP middleware pipeline.

### What is Swagger?

Swagger is a tool that allows us to document our API. It generates interactive UI for testing endpoints directly from the browser.

### What is Middleware?

Middleware are the things that handle HTTP requests and responses in ASP.NET Core.

Middleware are executed in the order they are added to the pipeline, which makes order **extremely important**.

Swagger middleware example:

```csharp
/// <summary>
/// Register the Swagger middleware with provided options
/// </summary>
public static IApplicationBuilder UseSwagger(this IApplicationBuilder app, SwaggerOptions options)
{
    return app.UseMiddleware<SwaggerMiddleware>(options);
}
```

You can add custom middleware:

```csharp
app.Use(async (HttpContext context, RequestDelegate next) =>
{
    context.Response.Headers.Append("Content-Type", "text/html");
    await context.Response.WriteAsync("<h1>Welcome to Ahoy!</h1>");
});
```

This middleware skips the rest of the pipeline and writes its own response.

### What is `HttpContext`?

`HttpContext` is the main class that represents the incoming HTTP request — and all related things around handling that request — in ASP.NET Core. It contains all the information about the request and response.

Most important properties:

| Property | Description |
|----------|-------------|
| `Request` | The request object (headers, method, URL, body, etc.) |
| `Response` | The response object |
| `Items` | A dictionary for storing data during the request/response process |
| `User` | The user object — discussed more in authentication |
| `Features` | A collection of features for storing data or adding behaviors |

Spencer mostly reads the `Request` or `User` objects.

### The Rest of Program.cs

```csharp
app.UseHttpsRedirection();
```
Redirects HTTP requests to HTTPS. Good practice for ensuring HTTPS usage.

```csharp
app.MapGet("/weatherforecast", () => { ... })
.WithName("GetWeatherForecast")
.WithOpenApi();
```
Handles the GET request for the `/weatherforecast` endpoint. Returns random weather forecast data. `.WithName` and `.WithOpenApi` add metadata that OpenAPI uses to generate documentation.

```csharp
app.Run();
```
Starts the application after all services and middleware have been registered.

---

*Content licensed under CC-BY-NC-4.0 | Code samples under Apache 2.0*
