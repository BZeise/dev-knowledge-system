# Claude and Claude Code

## Claude Code Overview

Claude Code is Anthropic's agentic coding tool — a CLI that can autonomously read, write, and execute code in your project, run tests, and interact with Git.

## Key Commands and Setup

- `/install-github-app` — Connects Claude Code to GitHub, enabling automatic PR creation
- System prompts can keep responses concise and focused on diffs

## Working Effectively with Claude Code

- Be explicit about constraints: file paths, which tests to run, what not to touch
- For large tasks, break work into phases and review each before proceeding
- Claude Code can run in the background on long-running tasks; context-switch and come back

## Custom Subagents

Claude Code supports defining specialized subagents for specific tasks:

- **UX Reviewer** — Evaluates UI changes against accessibility/usability standards
- **API Designer** — Reviews or generates API contracts (REST, GraphQL)
- **Security Reviewer** — Scans for vulnerabilities, injection risks, auth issues
- **Test Runner** — Writes and executes tests, reports coverage
- **Database Admin** — Generates migrations, optimizes queries, reviews schema changes

## MCPs (Model Context Protocol)

MCP is an open standard allowing LLMs to connect to external tools and data sources via a defined protocol — a plugin system for AI agents.

Connect an MCP server and the model gains ability to query that service as a tool.

**Examples:** Databases, Figma/Canva designs, GitHub (PRs and issues), filesystem tools, custom internal APIs.

---

## Resources

- [Claude Code Documentation](https://claude.com/claude-code)
- [How I Use Claude Code (YouTube)](https://youtube.com)
- [Coding with AI Learning Path — Frontend Masters](https://frontendmasters.com)
- [6 Months of Claude Code Lessons — YouTube](https://youtube.com)

---

*Expand this section with your personal Claude Code tips and workflows*
