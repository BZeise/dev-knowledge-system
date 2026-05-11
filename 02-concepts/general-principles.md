# General Programming Principles

Foundational concepts applicable across languages, frameworks, and domains.

---

## Parameters vs Arguments

A **parameter** is a variable in a function definition — a placeholder with no concrete value. An **argument** is the actual value passed at the point of invocation. Arguments fill the slots that parameters reserved.

```csharp
// 'name' and 'age' are parameters
void DisplayPerson(string name, int age)
{
    Console.WriteLine($"{name} is {age} years old");
}

// "John" and 30 are arguments
DisplayPerson("John", 30);
```

---

## Chesterton's Fence

*"Don't remove a fence until you know why it was put up in the first place."*

Applied to code: before deleting or refactoring something that seems unnecessary, understand why it exists. It may be guarding against a non-obvious edge case, a legacy workaround, or a constraint you don't immediately see.

**In practice:**
- Before removing code, check git blame and commit history for context
- Ask teammates why a pattern exists before proposing alternatives
- Document why something is there if the reason isn't obvious

---

## Stateful vs Stateless

### Stateful Applications
Retain data between sessions or requests. Example: a shopping cart that persists across page loads.

**Characteristics:**
- Store user context in memory or session storage
- Server remembers client state across requests
- Scaling horizontally is harder (state must be shared or replicated)

### Stateless Applications
Treat each request as independent with no memory of prior ones. Example: a pure REST API that requires all context with each call.

**Characteristics:**
- No server-side session storage
- All context comes with the request
- Easy to scale horizontally (any server can handle any request)
- Each request is self-contained

**Trade-off:** Stateless systems are generally easier to scale, but stateful systems can be more efficient and provide better user experience for interactive applications.

---

## Patterns and Anti-Patterns

### Design Pattern
A reusable, **proven solution** to a commonly recurring problem. Captures best practices and abstracts away complexity.

Examples: Factory Pattern, Observer Pattern, Singleton Pattern

### Anti-Pattern
The opposite: a response that seems reasonable on the surface but **consistently causes more problems than it solves**, and for which a better alternative exists.

**Common anti-patterns:**
- **God Object** — A single class that does too much (violates Single Responsibility Principle)
- **Anemic Domain Model** — Models with only getters/setters, all logic in services (separates data from behavior)
- **Spaghetti Code** — Tangled control flow with no clear structure
- **Lava Flow** — Keeping unused code "just in case" (dead code accumulation)
- **Copy-Paste Coding** — Duplicating code instead of extracting shared logic

**Key skill:** Recognizing both patterns and anti-patterns helps you name what you're seeing and steer toward better solutions instead of repeating mistakes.

---

## Next Steps

- [OOP - Object-Oriented Programming](./oop.md) — Core OOP concepts
- [SOLID Principles](./solid.md) — Design principles that work with these foundations
- [Design Patterns](./design-patterns.md) — Proven solutions to common problems
