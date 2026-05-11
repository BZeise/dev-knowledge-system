# Knowledge System TODO List

Incomplete sections and items to be filled in as courses are reviewed and content is developed.

Organized under the new semantic block structure:
- **01-Learning** — Courses, Concepts, Frameworks
- **02-Professional** — Interview Prep, Tools, Reference
- **03-Personal** — Activity Logs, Projects, Milestones

---

## 🔍 High Priority

- [ ] **Review FEM Profile History** — Go through Frontend Masters course history to identify all courses taken and add them to [01-Learning/01-Courses](../01-learning/01-courses/) with appropriate course notes files

---

## 🎓 Courses - Content to Expand

### [01-Learning/01-Courses](../01-learning/01-courses/)

- [ ] **[Enterprise UI Development](../01-learning/01-courses/enterprise-ui-dev.md)** — Steve Kinney (Frontend Masters)
  - [ ] Expand beyond topic list to detailed notes
  - [ ] Add specific code examples
  - [ ] Include key takeaways and gotchas

- [ ] **[Complete Intro to React, v9](../01-learning/01-courses/react-v9.md)** — Brian Holt (Frontend Masters)
  - [ ] Complete useEffect/Portals and cleanup functions section
  - [ ] Error Boundaries detailed examples
  - [ ] Preload and Preconnect detailed implementation
  - [ ] Deployment notes expansion
  - [ ] Add notes on courses: Intermediate React, React with TypeScript

- [ ] **[AI for Software Engineers](../01-learning/01-courses/ai-software-engineers.md)** — Frontend Masters
  - [ ] Add LLMs and Transformers section completion
  - [ ] Expand on practical applications in code generation
  - [ ] Add prompt engineering best practices

### [01-Learning/03-FEM-Courses](../01-learning/03-fem-courses/01-csharp-dotnet/)

- [ ] **[C# and .NET: From Beginner to Professional](../01-learning/03-fem-courses/01-csharp-dotnet/)** — Spencer Schneidenbach
  - All modules appear to have placeholder content; expand as course is reviewed:
  - [ ] [01-Introduction to .NET](../01-learning/03-fem-courses/01-csharp-dotnet/01-introduction.md)
  - [ ] [02-Basics of C#](../01-learning/03-fem-courses/01-csharp-dotnet/02-basics.md)
  - [ ] [03-Types, Classes, and Structures](../01-learning/03-fem-courses/01-csharp-dotnet/03-types-classes-structures.md)
  - [ ] [04-Collections, Generics, and LINQ](../01-learning/03-fem-courses/01-csharp-dotnet/04-collections-generics-linq.md)
  - [ ] [05-Common Libraries and Advanced Topics](../01-learning/03-fem-courses/01-csharp-dotnet/05-common-libraries-advanced.md)
  - [ ] [06-Hosting Model and API Development](../01-learning/03-fem-courses/01-csharp-dotnet/06-hosting-api-development.md)

- [ ] **[Complete Intro to SQL & PostgreSQL](../01-learning/01-courses/sql-and-postgresql.md)**
  - [ ] Full course content to be added as you review the course
  - [ ] See checklist in file for required sections

---

## 📚 References

### [02-Professional/03-Reference](../02-professional/03-reference/)

- [ ] **[Learning Resources](../02-professional/03-reference/resources.md)**
  - [ ] Review and verify all links
  - [ ] Update any broken URLs
  - [ ] Add missing course references
  - [ ] Organize by category

---

## ✅ Completed Sections

The following sections are complete or well-populated:

- ✅ [01-Learning/02-Concepts](../01-learning/02-concepts/) — All core concepts documented
  - ✅ OOP, SOLID, Design Patterns, Testing, General Principles
- ✅ [01-Learning/04-Languages-Frameworks](../01-learning/04-languages-frameworks/) — Comprehensive coverage
  - ✅ SQL, CSS, ASP.NET MVC
- ✅ [02-Professional/01-Interview-Prep](../02-professional/01-interview-prep/) — Technical and behavioral prep
  - ✅ Technical Q&A, Behavioral/Soft Skills, Company-specific prep
- ✅ [02-Professional/02-Tools](../02-professional/02-tools/) — Development tools and workflows
  - ✅ Claude Code, Dev Tools
- ✅ [03-Personal/01-Activity-Log](./01-activity-log/) — Activity tracking system set up
  - ✅ 2026.md created, automation directive in CLAUDE.md

---

## 🔄 Review & Maintenance

- [ ] Fix cross-reference links in files after folder reorganization
- [ ] Verify all internal links work after reorganization
- [ ] Update course notes as new courses are completed
- [ ] Review and refresh Interview Prep content periodically
- [ ] Add new tools and frameworks as learned

---

## 🧹 Post-Reorganization Cleanup

After verifying the new structure works, run this PowerShell command from the project root to delete old directories:

```powershell
Remove-Item -Recurse -Force 01-courses, 02-concepts, 03-fem-courses, 03-languages-frameworks, 04-interview-prep, 05-tools, 06-reference
```

- [ ] Delete old directory: `01-courses/`
- [ ] Delete old directory: `02-concepts/`
- [ ] Delete old directory: `03-fem-courses/`
- [ ] Delete old directory: `03-languages-frameworks/`
- [ ] Delete old directory: `04-interview-prep/`
- [ ] Delete old directory: `05-tools/`
- [ ] Delete old directory: `06-reference/`

---

## 📝 How to Use This List

1. **Before reviewing a course:** Check the TODO list for that course
2. **While reviewing:** Take detailed notes and cross-reference with existing content
3. **After reviewing:** Check off the item and update the course notes file
4. **Maintenance:** Periodically review this list to ensure it stays current

---

**Last Updated:** 2026-05-11  
**Status:** Reorganization in progress — Semantic block structure implemented
