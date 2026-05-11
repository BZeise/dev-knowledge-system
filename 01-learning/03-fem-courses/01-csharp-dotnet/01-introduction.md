# Introduction to .NET

*From "C# and .NET: From Beginner to Professional" by Spencer Schneidenbach (Frontend Masters)*

## Course Overview

This is an introductory course on C# and .NET designed for beginners aspiring toward professional development. Instructor Spencer Schneidenbach emphasizes practical, real-world application over theoretical concepts.

### About the Instructor

Spencer Schneidenbach brings substantial credentials:
- President and CTO of Aviron Software, a .NET-focused consulting firm
- Microsoft MVP for nearly a decade in Developer Technologies
- Extensive consulting background across startups and enterprises
- International conference speaker on .NET topics

His teaching philosophy centers on preparing students for professional-level development using lessons from real projects. He emphasizes practicality over exhaustive coverage and explicitly acknowledges his opinionated perspective while encouraging pragmatic decision-making over perfectionism.

---

## Why .NET?

Schneidenbach opens with a bold assertion: **.NET is the best platform for building web apps in the world.** He characterizes it as modern and continuously evolving, distinct from legacy versions.

### Six Reasons to Choose .NET

1. **Strong job market demand** — High-paying positions with good availability
2. **Developer enjoyment and usability** — Modern language features and productivity
3. **Open-source, cross-platform availability** — Not locked to Windows
4. **Active community ecosystem** — Rich set of libraries and frameworks
5. **Performance and scalability capabilities** — Fast, handles large workloads
6. **Microsoft backing and regular updates** — Actively maintained, frequent improvements

---

## .NET vs CLR vs C# — Core Concepts

### .NET
The overarching development platform that encompasses various tools, libraries, and frameworks. 
- Open-source and cross-platform
- Supports multiple languages
- Includes extensive standard libraries
- The "frame of the house"

### CLR (Common Language Runtime)
The execution layer that manages .NET applications.
- **Manages memory allocation and deallocation** through garbage collection
- **Ensures type safety and security** across all .NET languages
- All .NET languages compile to **Intermediate Language (IL)**, which the CLR then executes
- The "foundation/plumbing/electricity"

### C#
A modern, object-oriented, type-safe programming language.
- Syntax similar to C-style languages
- Continuously evolving with new features
- The "building materials"

### Supporting Ecosystem

**Other .NET Languages:**
- **VB.NET** — Visual Basic's object-oriented version
- **F#** — A functional-first option

**NuGet** — .NET's package manager for sharing libraries and managing dependencies (the "hardware store")

### Helpful Analogy
> ".NET is the frame of the house, CLR is the foundation/plumbing/electricity, C# is the building materials, and NuGet is the hardware store."

---

## A Brief History of .NET

### Original Motivation (2001 onwards)
Microsoft created .NET as a direct response to Java's rising market dominance, seeking a similarly productive, high-level programming alternative. The platform launched with C# and VB.NET as primary languages, establishing ".NET Framework" as foundational infrastructure comparable to modern core libraries.

Initial tools included:
- **WinForms** — for desktop applications
- **WebForms** — for web development

### Significant Adoption Barriers

Three major limitations hindered widespread adoption:
1. **Platform dependency on Windows exclusively** — couldn't run on Linux/Mac
2. **Proprietary, closed-source codebase** — not transparent
3. **Opaque development methodology** — users couldn't see what was happening

### Community Innovation: Mono Project

An independent community initiative created an open-source, cross-platform .NET implementation. Though not Microsoft-backed, Mono addressed accessibility concerns and continues serving embedded systems today.

### Transformational Shift: .NET Core (2014)

Microsoft's announcement of .NET Core fundamentally repositioned the platform by introducing:
- **Cross-platform functionality** — runs on Windows, Linux, macOS
- **Complete open-source licensing** — transparent development
- **GitHub-based development** — community can see everything
- **Active community participation** — anyone can contribute

This transition resolved previous limitations and established .NET's contemporary relevance in professional development environments.

---

## Use Cases for .NET

### Terminal Applications
.NET is extremely versatile for building:
- Command-line interfaces
- System utilities
- Automation scripts

### Web Development

**ASP.NET Core:**
- Web applications
- APIs
- Microservices

**Blazor:**
- Interactive web UIs
- Single-page applications

### Cross-Platform Development
**.NET MAUI (Multi-platform App UI)** enables developers to create:
- iOS applications
- Android applications
- macOS applications
- Windows applications
- All from a single codebase

### Windows Desktop
- **WPF (Windows Presentation Foundation)** — Rich desktop applications
- **Windows Forms** — Maintaining legacy systems

### Cloud Solutions
- **Azure Functions** — Serverless computing
- Containerized applications using **Docker**
- Cloud-native development

### Game Development
C# integrates with both:
- **Unity** game engine
- **Godot** game engine

---

## Tooling Overview

### Installation: dotnet CLI

**Step 1: Install .NET SDK**
Visit https://dot.net and download the .NET SDK for your operating system.

**Step 2: Verify Installation**
Open a terminal and run:
```bash
dotnet --version
```

### Development Environments

**Visual Studio Code (Recommended for Learning)**
1. Download from https://code.visualstudio.com/
2. Install for your operating system
3. Launch the editor
4. Install the **C# Dev Kit extension by Microsoft** via Extensions marketplace (Ctrl+Shift+X or Cmd+Shift+X)
   - Enables IntelliSense and debugging features
   - Free for individuals (licensing requirements apply for businesses)

**Alternative IDEs:**
- **JetBrains Rider** — Full-featured, paid
- **ReSharper** — Visual Studio plugin, paid
- **Visual Studio** — Full IDE by Microsoft, free Community edition available

---

## Essential dotnet CLI Commands

### Creating Projects
```bash
dotnet new <template> -n <project-name>
```

Available templates:
- `console` — Console applications
- `classlib` — Class libraries
- `webapi` — ASP.NET Core API projects
- `blazor` — Blazor web applications
- `wpf` — Windows desktop applications
- `winforms` — Windows Forms applications

**Example:**
```bash
dotnet new console -n MyConsoleApp
```

### Building
```bash
dotnet build
```
Builds a .NET project and all its dependencies.

### Running
```bash
dotnet run
```
Runs source code without any explicit compile or launch commands.

### Publishing
```bash
dotnet publish -c Release
```
Prepares the application for deployment. The `-c Release` flag optimizes for production.

### Restoring Dependencies
```bash
dotnet restore
```
Retrieves project dependencies from NuGet.

### Testing
```bash
dotnet test
```
Executes unit tests in your project.

---

## Using the dotnet CLI: Creating Your First Console App

### Step 1: Create the Project
Open a terminal in your desired project directory and execute:
```bash
dotnet new console -n MyFirstConsoleApp
cd MyFirstConsoleApp
```

### Step 2: Explore the Project Structure
Open the folder in Visual Studio Code:
- Review the generated `Program.cs` file (entry point)
- Review the `MyFirstConsoleApp.csproj` file (project configuration)
- Install the C# Dev Kit extension if needed

### Step 3: Build and Run
**Compile:**
```bash
dotnet build
```

**Execute:**
```bash
dotnet run
```

### Key Takeaway
The dotnet CLI provides a streamlined workflow for creating, building, and executing C# console applications directly from the terminal without requiring a full IDE.

---

## Next Steps

Now that you understand what .NET is and how to set up your environment, proceed to:
- [02-Basics of C#](./02-basics.md) — Learn the C# language fundamentals
- [Core Concepts](../../02-concepts/) — Understanding OOP and design principles

---

*Content licensed under CC-BY-NC-4.0 | Code samples under Apache 2.0*
