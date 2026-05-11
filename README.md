# Personal Software Development Knowledge System

A comprehensive, organized guide to software development concepts, tools, frameworks, and best practices. Built for learning and reference.

## 📚 Structure

### [01-Courses](./01-courses/) — Personal Course Notes
Learning materials from professional courses and workshops.

- [Enterprise UI Development](./01-courses/enterprise-ui-dev.md) — Steve Kinney (Frontend Masters)
- [Complete Intro to React, v9](./01-courses/react-v9.md) — Brian Holt (Frontend Masters)
- [AI for Software Engineers](./01-courses/ai-software-engineers.md) — Frontend Masters

### [02-Concepts](./02-concepts/) — Core Programming Principles

Foundational concepts applicable across languages and domains.

- [Object-Oriented Programming](./02-concepts/oop.md) — Abstraction, Polymorphism, Inheritance, Encapsulation
- [SOLID Principles](./02-concepts/solid.md) — Design principles for maintainable code
- [Design Patterns](./02-concepts/design-patterns.md) — Creational, Structural, and Behavioral patterns
- [Testing](./02-concepts/testing.md) — Unit, Integration, E2E, Component, and Smoke Testing

### [03-Languages-Frameworks](./03-languages-frameworks/) — Language & Framework Deep Dives

In-depth guides for specific languages and frameworks.

- [C# and .NET](./03-languages-frameworks/csharp-dotnet-comprehensive.md) — See FEM course below for comprehensive coverage
- [SQL and Database Development](./03-languages-frameworks/sql-database.md) — Query optimization, indexes, schema design, performance tuning
- [ASP.NET MVC](./03-languages-frameworks/aspnet-mvc.md) — MVC architecture, ViewData/ViewBag/TempData, validation, CORS, dependency injection
- [CSS](./03-languages-frameworks/css.md) — Layout, sizing, flexbox vs grid, semantic HTML, typography, accessibility

---

### [03-FEM-Courses](./03-fem-courses/) — Frontend Masters Courses

In-depth educational content from Frontend Masters.

#### [C# and .NET: From Beginner to Professional](./03-fem-courses/01-csharp-dotnet/)

Comprehensive guide by Spencer Schneidenbach covering modern .NET development.

- [01-Introduction to .NET](./03-fem-courses/01-csharp-dotnet/01-introduction.md)
- [02-Basics of C#](./03-fem-courses/01-csharp-dotnet/02-basics.md)
- [03-Types, Classes, and Structures](./03-fem-courses/01-csharp-dotnet/03-types-classes-structures.md)
- [04-Collections, Generics, and LINQ](./03-fem-courses/01-csharp-dotnet/04-collections-generics-linq.md)
- [05-Common Libraries and Advanced Topics](./03-fem-courses/01-csharp-dotnet/05-common-libraries-advanced.md)
- [06-Hosting Model and API Development](./03-fem-courses/01-csharp-dotnet/06-hosting-api-development.md)

### [04-Interview-Prep](./04-interview-prep/) — Technical Interview Preparation

Interview questions, answers, and preparation materials organized by topic and company.

- [Technical Q&A](./04-interview-prep/technical-qa.md) — OOP, C#, SQL fundamentals, coding exercises
- **Company-Specific Preparation:**
  - [CVS](./04-interview-prep/company-specific-cvs.md) — Campaign/patient outreach focus, SQL emphasis, data design
  - [SDG](./04-interview-prep/company-specific-sdg.md) — Static classes and members focus
  - [Rayus / Mahasti](./04-interview-prep/company-specific-rayus.md) — .NET/SQL heavy, textbook questions with real-world examples
  - [Genus Technologies](./04-interview-prep/company-specific-genus.md) — Production experience, design patterns, communication emphasis

### [05-Tools](./05-tools/) — Development Tools & Workflows

Tools, frameworks, and optimization techniques for development.

- [Dev Optimization Tools](./05-tools/dev-tools.md) — Gulp, Grunt, Vite, webpack, imagemin, build automation
- [Claude and Claude Code](./05-tools/claude-code.md) — Working with AI coding assistants, MCPs, custom agents

### [06-Reference](./06-reference/) — External Resources & Links
Curated links to official documentation, tutorials, and learning resources.

- [Learning Resources](./06-reference/resources.md) — Organized by topic

---

## 🎯 How to Use This Repository

1. **For Learning**: Start with [02-Concepts](./02-concepts/) to build foundational understanding, then dive into specific [01-Courses](./01-courses/) or [03-FEM-Courses](./03-fem-courses/).

2. **For Reference**: Use the table of contents above to jump to what you need. All files link to related topics.

3. **For Interview Prep**: Review [04-Interview-Prep](./04-interview-prep/) before technical interviews.

4. **For Tools**: Check [05-Tools](./05-tools/) when setting up new projects or workflows.

## 🔗 Cross-File Navigation

All files use relative markdown links for easy navigation in VS Code and GitHub:
- `[Link text](./other-file.md)` — links to another file in the same directory
- `[Link text](../other-section/file.md)` — links across directories
- `[Link text](./file.md#section-heading)` — links to specific sections within files

## 📝 License

This knowledge system is a personal learning resource. All content is either original work or properly attributed to original sources (courses, textbooks, official documentation).

---

**Last Updated**: 2026-05-11  
**Status**: Active - continuously updated as new concepts are learned and courses are completed
