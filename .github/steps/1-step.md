# Module 8 Exercise — MCP Tools in Practice

This exercise has three parts. Part 0 is a warm-up where you discover what MCP tools are available. Part 1 teaches you how to use Playwright MCP for UI verification. Part 2 shows you how Context7 improves coding accuracy when APIs change after your training data.

## Part 0: Discover MCP tools (warm-up)

### Objective
Get familiar with the MCP ecosystem and what tools are available beyond the most common ones.

### Task
1. Search for "MCP servers" or "Model Context Protocol tools" online
2. Find 3-4 MCP tools you haven't heard of
3. For each tool, read its documentation or description
4. For each tool, write one sentence about what it does and when you might use it

Examples of tools to look for (but find your own too):
- Tools for specific cloud platforms (AWS, GCP, Azure)
- Tools for communication platforms (Slack, Discord, email)
- Tools for data or analytics services
- Tools for monitoring or observability systems

---

## Part 1: Playwright MCP — Debugging failing e2e tests

### Objective
Install Playwright MCP, run a React app with failing e2e tests, and use Claude with Playwright to diagnose why the tests are failing by observing what the app actually does.

### Setup
1. In Claude Code, use `/plugin` to search for "Playwright MCP"
2. Install the plugin (if it requires authentication, follow the setup steps)
3. After installation, use `/reload-plugins` to load the new plugin into the current session

### Task
1. Run your React app locally (e.g., `npm install` and `npm dev`)
2. Run the e2e tests and let them fail (don't debug yet) (`npm test:e2e`)
3. ** Optional: try to fix the code yourself **
4. Ask Claude to:
    - Use Playwright to navigate to the app
    - Perform the same interactions the tests do
    - Fix the test by modifying the code

Example prompt:
> I have a failing e2e test. The app is running at http://localhost:5177. Please use Playwright to:
> 1. Navigate to the app
> 2. Perform the same interactions the test does
> 3. Fix the test by modifying the code

### Debrief
After Claude diagnoses the issue, ask yourself:
- What did Claude observe with Playwright that you couldn't see by reading the test code alone?
- Would you have spotted the issue faster by having Claude navigate the app visually?
- How much time would Claude have saved you compared to manually reproducing the test steps?

---

## Part 2: Context7 — Documentation-assisted coding with API changes

### Objective
Install Context7, implement a function using a library that had significant API changes after Claude's training cutoff, and see how live documentation prevents outdated code.

### Task
1. Application has outdated dependencies.
2. Ask Claude to create a migration plan for the dependencies to docs/MigrationPlan.md.
3. Use `/clear` to clear context.
3. In Claude Code, use `/plugin` to search for "Context7"
4. Install the plugin (it requires no additional authentication)
5. After installation, use `/reload-plugins` to load the new plugin into the current session
6. Ask Claude to use Context7 and to create a migration plan for the dependencies to docs/MigrationPlanWithContext7.md.
7. Ask Claude to compare the two migration plans.

### Debrief
After comparing the two migration plans (MigrationPlan.md vs. MigrationPlanWithContext7.md), ask yourself:

- How did the two migration plans differ?
- Were there breaking changes in the newer versions that Claude missed in the first plan but caught with Context7?
- Did Context7 help Claude discover newer, better patterns or APIs?
- Which migration plan would you be more confident following in a real project and why?

---

### Automated Checks
The CI pipeline will verify:
- Tests pass
- doc/MigrationPlan.md exists
- doc/MigrationPlanWithContext7.md exists

<details>
<summary>Having trouble? 🤷</summary><br/>

**Part 1 (Playwright MCP):**
* If Playwright MCP won't install, check that you're using `/plugin` in Claude Code and not trying to install manually.
* If the plugin loads but tests still fail after Claude's fixes, paste the full error output and ask Claude to observe the app with Playwright to diagnose further.
* If Playwright can't interact with the app (buttons not clickable, selectors not found), the app may not be running. Verify `npm run dev` is still executing and the correct port is in the Playwright config.
* If the Playwright config has the wrong base URL or port, update it and re-run the tests.

**Part 2 (Context7):**
* If Context7 won't install, check that you've selected the correct plugin in `/plugin`.
* If Context7 finds outdated docs or the migration plan seems incomplete, ask Claude to search for breaking changes in the CHANGELOG or release notes of each library.

**General:**
* Make sure to use `/reload-plugins` after installing a plugin so Claude can access it in the current session.
* If you're stuck, paste the error message or full test output and ask Claude what you should fix.
* If you still need help, ask in the **#ai-driven-developer-support** Slack channel.

</details>


---

<img alt="Amin 2.0" src="../images/amin2_smile.png" align="right" height="125px" />

Please, follow the steps above.
I'll watch your progress in the background to provide feedback. 🧐