# Claude Code Directives

System prompts and automated behaviors for Claude Code sessions in this project.

---

## Activity Log Automation

**Purpose:** Maintain a daily record of programming work across Claude, GitHub, and other platforms.

**Behavior:** At the end of most sessions, prompt the user to summarize their work for the activity log.

### Implementation

- **When to prompt:** End of session (after substantive work)
- **What to ask:** `"What did you accomplish today? (Brief summary for activity log)"`
- **How to format:** Convert user response into bullet points, one per action
- **Where to save:** `03-personal/01-activity-log/YYYY.md` (current year)
- **Append location:** Under current month section, then date header

### Examples of Good Entries

- "Added 4 commits to dev-knowledge-system repo"
- "Planned authentication system for project XYZ"
- "Fixed N+1 query performance bug in API endpoint"
- "Completed Chapter 5 of SQL course on PostgreSQL functions"
- "Reviewed Claude Code best practices and MCP integration"

### Examples of What NOT to Log

- ❌ Very detailed technical descriptions (keep it short)
- ❌ Multiple actions on same line (one action = one bullet)
- ❌ Work from other days (only today's work)
- ❌ Personal life/non-coding activities

---

## Project Context

**Project Name:** Dev Knowledge System  
**Location:** `c:\Code\dev-knowledge-system\`  
**Git User:** Ben Zeise  
**Email:** ben.zeise@gmail.com

**Purpose:** Comprehensive software development knowledge system organized by:
- **01-Learning:** Courses, concepts, and frameworks
- **02-Professional:** Interview prep, tools, references
- **03-Personal:** Activity logs, projects, milestones

---

## Key Files

- `README.md` — Main overview and navigation
- `TODO.md` — Incomplete sections and course expansion tasks
- `03-personal/01-activity-log/2026.md` — Current year activity log
- `CLAUDE.md` — This file; directives and behaviors

---

## Cross-Linking Standards

All internal links use relative markdown syntax for VS Code and GitHub compatibility:

```markdown
[Link text](./file.md)              # Same folder
[Link text](../other/file.md)       # Different folder
[Link text](./file.md#heading)      # Specific section
```

---

## Future Automations

Planned but not yet implemented:

- [ ] GitHub integration to auto-detect daily commits and suggest entries
- [ ] Weekly summary generation from activity log entries
- [ ] Sync with project milestones and goals
- [ ] Quarterly reflection prompts

---

**Last Updated:** 2026-05-11  
**Status:** Active
