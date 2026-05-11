# Behavioral & Soft Skills Interview Prep

Interview questions and discussion points focused on soft skills, communication, and professional behavior.

---

## Time Management During High-Volume / Crunch Periods

When facing high pressure and tight deadlines:

### 1. **Communicate with the team first** — surface the pressure, don't absorb it silently
   - Flag blocking issues early
   - Request help if needed
   - Keep stakeholders informed of progress

### 2. **Assess whether sprint stories are realistic** within the timeline
   - If not, reset expectations early
   - Propose scope reductions or timeline adjustments
   - Work with the team to prioritize

### 3. **Delegate where appropriate**
   - Identify which tasks others can take
   - Unblock teammates
   - Maintain team capacity

### 4. **Identify and rank priority tasks**
   - What's blocking others?
   - What has a hard deadline?
   - What has the most impact?

### 5. **Work top-priority tasks to a stopping point**, then move to the next
   - Avoid context switching overhead
   - Complete one item before starting another
   - Document decisions and handoffs

### 6. **Use the Pomodoro method** for sustained focus
   - 25 minutes focused work, 5-minute break
   - Rotate between tasks every few Pomodoros to keep the mind fresh
   - Prevents burnout from marathon sessions

---

## How to Troubleshoot and Fix Performance Issues

### For SQL / Database Performance

**1. Run the query with Actual Execution Plan enabled**
   - In SSMS: Ctrl+M or "Include Actual Execution Plan"
   - The plan shows exactly how SQL Server executed the query

**2. Look for bottlenecks in the plan**
   - **Table Scan vs Index Seek** — Scans read every row; seeks jump to matching rows. Scans on large tables are usually the culprit.
   - **Key Lookups** — Non-clustered index is used but SQL still fetches columns not in the index. May indicate need for a covering index.
   - **Estimated vs Actual rows** — Large discrepancy means statistics are out of date (UPDATE STATISTICS).
   - **Thick arrows** — In the graphical plan, thick arrows between operations indicate many rows flowing through. Focus optimization here.

**3. Check index usage**
   - Are relevant indexes being used?
   - Are there missing indexes?
   - Are there unused indexes slowing down writes?

**4. Review the query itself**
   - Can JOINs replace subqueries? (Usually faster)
   - Can SELECT * be narrowed to specific columns?
   - Add a covering index for the specific query pattern
   - Use EXISTS instead of COUNT(*) > 0 for existence checks (EXISTS short-circuits)

**5. Keep transactions short**
   - Minimize lock hold time
   - Reduce deadlock risk

### For Application-Level Performance

**1. Reproduce the issue in a controlled way**
   - Identify which operation is slow
   - Isolate the slow code path

**2. Add logging and profiling**
   - Profile around suspect code
   - Measure time spent in each operation
   - Look for unexpected delays

**3. Check for N+1 query problems**
   - In Entity Framework, lazy loading can hit the DB in a loop
   - Use .Include() for eager loading where needed

**4. Look at async patterns**
   - Is something blocking a thread unnecessarily?
   - Are you using async/await correctly?
   - Check for deadlocks in synchronous waits on async operations

**5. Profile memory**
   - Are large objects held in memory longer than needed?
   - Check for memory leaks
   - Look for unnecessary allocations in tight loops

---

## How Do You Typically Present Ideas to Your Team?

**Talking Point:** Frame your response around how you prepare and communicate ideas:

- **Preparation** — Do you research first? Get feedback? Build consensus?
- **Medium** — Do you start with a written doc, verbal discussion, or presentation?
- **Reading the room** — Do you adjust based on feedback? Listen to concerns?
- **Clarity** — Can you explain why, not just what?

This demonstrates:
- Communication skills
- Ability to think through ideas before proposing
- Respect for team input
- Flexibility in approach

---

## What Would Happen If Your Idea Isn't Chosen?

**Talking Point:** Demonstrate graciousness, ability to commit to a direction even if it wasn't yours, and that you can still advocate constructively.

Key points to convey:
- **Respect the decision** — Understand the reasoning (cost, timeline, other constraints)
- **Commit fully** — Support the chosen direction enthusiastically
- **Provide value anyway** — Help implement the chosen solution as well as you would your own
- **Learn from it** — Reflect on why the other direction was chosen
- **Constructive advocacy** — You can still surface concerns or alternative approaches later if issues arise, but you don't undermine the decision

This shows maturity and team orientation.

---

## What's a Typical Mitigation Strategy to You?

**Talking Point:** Demonstrate structured thinking about risk management. Frame your answer around:

**1. Identify the risk**
   - What could go wrong?
   - What's the failure mode?

**2. Assess likelihood and impact**
   - How likely is this to happen?
   - How bad would it be if it did?

**3. Define a response plan**
   - What actions will reduce the risk?
   - What's the contingency if the risk materializes?

**4. Assign ownership**
   - Who is responsible for executing the mitigation?
   - When will it be done?

**5. Communicate to stakeholders**
   - Make sure everyone understands the risk and the plan
   - Keep them updated on progress

**Examples:**
- **Production risk** — Mitigation: extensive testing, staging environment, rollback plan, monitoring
- **Data risk** — Mitigation: backups, encryption, audit logging, access controls
- **Timeline risk** — Mitigation: realistic estimates, buffer time, clear dependencies, early warning system

---

## Interview Tips

1. **Be specific** — Use real examples from your experience
2. **Show self-awareness** — Acknowledge what you don't know; show willingness to learn
3. **Demonstrate communication** — Explain your thinking clearly; ask clarifying questions
4. **Show team orientation** — Focus on how you work with others, not just individual heroics
5. **Handle pressure well** — Stay calm, think through problems systematically
6. **Respect the process** — Even if you disagree with a decision, support the team's direction

---

## Next Steps

- [Technical Q&A](./technical-qa.md) — Technical interview prep
- [Company-Specific Prep](./company-specific-cvs.md) — Tailored prep by company
