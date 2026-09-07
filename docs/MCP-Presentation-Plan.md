# MCP Presentation Plan: When to Use MCPs vs. CLIs

**Audience:** Developers using Cline (React + Spring Boot + Robot Framework + ArangoDB stack)  
**Duration:** 50 minutes  
**Goal:** Adopt local MCPs in Cline after security team approval by showing what MCPs enable (direct tool access) and why the approval cost is worth it  
**Note:** Installation details are out of scope — this presentation focuses on value proposition, not setup

---

## Overall Structure & Timing

| Section | Time | Type |
|---------|------|------|
| 1. Opening: What We're Exploring | 2 min | Talk |
| 2. What is an MCP? | 5 min | Slides + Explanation |
| 3. Which MCPs We're Requesting | 2 min | Slides |
| 4. Demo 1: Playwright MCP | 12 min | Live Demo |
| 5. Demo 2: rf-mcp | 12 min | Live Demo |
| 6. When CLI is Enough | 5 min | Honest Assessment |
| 7. Security Approval Path | 10 min | Slides + Process |
| 8. Q&A + Next Steps | 2 min | Discussion |

---

## Section 1: Opening (2 minutes)

### Goal
Introduce the core question: Cline can help with your tools in two ways. Which is better?

### Script
> "You use Cline to help with coding, testing, debugging. But here's the question: how does Cline access your tools? Two ways:
> 
> **One:** You tell Cline 'run this playwright test,' Cline executes it in a terminal, parses raw text output — potentially 1000 lines of logs, noise, and formatting. Cline has to read and extract the signal from all that.
> 
> **Two:** Cline has direct access via MCP. It runs the tool, gets back concise, structured JSON. No parsing. Just clean data to reason about.
> 
> That's the difference. MCPs make your AI development loop more reliable and much faster — Cline doesn't have to guess the right Windows or Linux command, doesn't waste time parsing noise, just acts directly on clean data."

### Slides
- Title: "Cline + Your Development Tools"
- Subheading: "Two ways to connect: Terminal or MCP?"
- Simple visual: Two paths — Cline → Terminal (indirect) vs. Cline → MCP (direct access)

---

## Section 2: What is an MCP? (5 minutes)

### Goal
Define MCP clearly. Show the difference between local and remote MCPs. Establish why local matters for your team.

### Script
> "MCP stands for **Model Context Protocol**. It's a standard way for Cline to access tools and data sources.
>
> Think of it like a plugin. Instead of Cline being limited to what you tell it in a terminal, an MCP gives Cline direct access to your tools: testing frameworks, databases, documentation, APIs. Cline can run them, read the results, and integrate that knowledge into its suggestions.
>
> Now, MCPs come in two flavors:
>
> **Remote MCPs** — Run in the cloud. Example: Cline connects to a remote service to fetch data. Disadvantage: your data leaves your machine. Your security team will want to audit where data goes.
>
> **Local MCPs** — Run on your machine. Example: Cline runs Playwright tests on your laptop using the Playwright MCP. Advantage: your code and data stay local. Your database stays on your machine. Security team's main concern is what the MCP code does, not where data travels. Technically, Cline spawns the MCP as a child process and communicates with it via stdio — it's not a server. Just a lightweight subprocess that handles the tool call and returns the result.
>
> **That's why we're focusing on local MCPs.** Your testing, your code, your databases — all stay local. Security can audit the code, approve it, and you're good to go."

### Slides (3 slides)

**Slide 1: What is an MCP?**
- Simple definition: "A protocol that gives Cline access to tools and data"
- Example analogy: "Like a plugin, but standardized"
- Visual: Cline (AI assistant) ↔ MCP ↔ Your Tool (Playwright, Robot Framework, Database, etc.)

**Slide 2: Remote vs Local MCPs**
```
Remote MCP:
Your Machine → MCP (cloud service) → External data source
Data travels to cloud ⚠️

Local MCP:
Your Machine → MCP (local) → Playwright, ArangoDB, etc.
Data stays local ✓
```

**Slide 3: Why Local MCPs For Us**
- Code stays on your machine
- Data stays local (databases, test artifacts, everything)
- Security team audits the code, not where data travels
- Easier approval process
- We're requesting local MCPs only
- Works across all your IDEs and environments (one approval, everywhere)

### Key Talking Points
- "MCP is a standard. It's not ArangoDB-specific or Playwright-specific — it's a universal way for Cline to access tools."
- "Remote MCPs send data to cloud services. Local MCPs keep everything on your machine."
- "We're asking security to approve local MCPs because your data never leaves your machine, and the approval works everywhere."

---

## Section 3: Which MCPs We're Requesting (2 minutes)

### Goal
List the specific MCPs we're asking security to approve and their purpose.

### Script
> "We're asking security to approve six local MCPs:
>
> **Maven MCP** — Run tests, compile, and manage the full build lifecycle. Get structured results instead of parsing verbose output.
>
> **JUnit MCP** — Direct access to test execution and failure analysis. Cline understands exactly which assertions failed and why.
>
> **Vitest MCP** — Run frontend tests, get clean JSON results. Same tight feedback loop as Maven.
>
> **Playwright MCP** — Write and debug e2e tests seamlessly in Cline.
>
> **rf-mcp** — Cline understands Robot Framework test failures and suggests fixes.
>
> **context7** — Fetch current documentation without leaving your editor.
>
> All six are local, auditable, and work everywhere — VS Code, IntelliJ, CLI."

### Slides (1 slide)

**Slide: Which MCPs We're Requesting**
- Maven MCP — Build and test lifecycle, structured results
- JUnit MCP — Test execution and failure analysis
- Vitest MCP — Run frontend tests, get clean JSON output
- Playwright MCP — E2E test writing and debugging
- rf-mcp — Robot Framework test analysis
- context7 — Documentation lookup in-IDE
- All local, auditable, cross-IDE compatible

---

## Section 4: Demo 1 — Playwright MCP (12 minutes)

### Goal
Show what Playwright MCP enables: Cline runs e2e tests against a live application and identifies bugs by analyzing test interactions.

### Demo Scenario
Run e2e tests from the `e2e/` directory against the app in `src/`, and ask Cline to spot errors in the application based on test behavior.

### Pre-Demo Setup Checklist
- [ ] **Application started** — `npm run dev` running on http://localhost:5173
- [ ] Cline open with Playwright MCP configured
- [ ] E2E test files exist in `e2e/` directory
- [ ] Ready to run `npm run test:e2e` via Playwright MCP
- [ ] Backup screenshots: test running, passing/failing output, Cline's error analysis
- [ ] **Note:** Installation details are not demoed — assume MCPs are already set up

### Demo Flow (12 min breakdown)

**1. Setup (1 min)**
- Show the app running in browser (or split view)
- Say: "We have a running application and a suite of e2e tests. With Playwright MCP, Cline can run these tests, see the interactions, and spot bugs in our app. Let's see what it finds."

**2. Cline runs e2e tests via Playwright MCP (4 min)**
- Prompt Cline: "Use Playwright MCP to run e2e tests."
- Cline executes the tests via Playwright MCP (direct access)
- Show test output: which tests passed, which failed, browser interactions logged
- Say: "Cline ran all the tests against localhost:5173 and got the full output. It can see exactly what the app did during each interaction."

**3. Ask Cline to spot app errors (6 min)**
- Prompt Cline: "Look at the test output and the app interactions. Can you identify any bugs or unexpected behavior in the application? What should the app be doing differently?"
- Cline analyzes:
  - Test assertions that failed
  - App responses that didn't match expectations
  - UI elements that didn't behave as intended
  - Data flow issues
- Cline explains: "The app is doing X, but the test expects Y. Here's what's broken: [specific error]."
- Show Cline's analysis inline
- Say: "Cline doesn't just run tests — it understands the test intent and spots where the app is misbehaving."

**4. Wrap up (1 min)**
- Say: "That's the MCP advantage: Cline sees your app in action through your tests and can diagnose real bugs. No manual log reading. Direct integration."

### Script/Talking Points
> "We have e2e tests and a running application. Let's see what Cline can discover by running the tests via Playwright MCP."

> "Cline runs the tests and captures every interaction the app makes. It's seeing your app behave in real time."

> "Now here's the key: Cline can analyze the test results and spot bugs in the application itself. It's not just running tests — it's diagnosing app failures."

> "This is what MCP enables: Cline has direct, structured access to your test execution and can reason about app behavior."

### Backup Plan (if app is not running or tests fail)
- Have pre-recorded screenshots showing:
  1. E2E test running (test output)
  2. Browser interactions captured
  3. Cline's analysis identifying the bug
  4. Explanation of what the app should do differently
- Narrate: "This is what it looks like — e2e tests run via MCP, Cline sees the interactions, and it identifies exactly what's broken in the app and why."

### Slide (1 slide)
- Title: "Playwright MCP: Run Tests and Spot App Bugs"
- Annotated screenshots showing: "E2E test running → browser interactions → Cline's bug analysis"

---

## Section 5: Demo 2 — rf-mcp (12 minutes)

### Goal
Show how Cline can convert Playwright tests to Robot Framework tests, run them via rf-mcp, and analyze them for best practices.

### Demo Scenario
Take a Playwright test, convert it to Robot Framework using Cline + rf-mcp, run it, and review best practices.

### Pre-Demo Setup Checklist
- [ ] **Application started** — `npm run dev` running on http://localhost:5173
- [ ] Playwright test generated from Demo 1 (or sample Playwright test ready)
- [ ] rf-mcp configured in Cline
- [ ] Robot Framework installed locally (for testing)
- [ ] Browser library available for Robot (via requirements.txt or pip)
- [ ] Backup: screenshots showing codegen → Playwright test → converted Robot test → analysis
- [ ] **Note:** Installation details are not demoed — assume rf-mcp is already set up

### Demo Flow (12 min breakdown)

**1. Start Playwright codegen (2 min)**
- Say: "We're starting fresh. Let's record a test using Playwright's codegen — `npx playwright codegen`."
- Run `npx playwright codegen http://localhost:5173` to open the browser with codegen active
- Show the Playwright Inspector panel recording interactions
- Say: "Playwright is now watching everything we do. As we interact with the app, it records the steps."

**2. Record test interactions (3 min)**
- Perform a simple user flow in the app (e.g., navigate, fill a form, submit, verify result)
- Say: "See how Playwright captures each click, text input, and assertion. It's automatically building a test as we go."
- Stop codegen and show the generated Playwright test code
- Say: "Cline now has the raw Playwright test. It's readable, but what if we want to run this in Robot Framework instead?"

**3. Cline converts Playwright to Robot (2 min)**
- Ask Cline: "Convert this Playwright test to a Robot Framework test. Use the Browser library. Keep the same test flow."
- Cline generates Robot Framework test code with proper syntax:
  - `*** Settings ***` with Browser library import
  - `*** Test Cases ***` with the test
  - Keywords properly structured
- Say: "Cline understands both frameworks and converts the test automatically."

**4. Run Robot test via rf-mcp (2 min)**
- Ask Cline to run the Robot test using rf-mcp
- Cline executes the test directly with MCP access
- Show the test running and passing
- Say: "The test runs and passes. Cline has direct access to Robot Framework via rf-mcp — it runs the test and gets the full output."

**5. Analyze for best practices (2 min)**
- Ask Cline: "Review this Robot test for best practices. Suggest improvements in keyword extraction, documentation, and variable naming."
- Cline provides feedback on structure and maintainability
- Say: "Cline analyzes the test and suggests improvements. That's the workflow: record with Playwright, convert to Robot, run via rf-mcp, and improve with Cline's guidance."

**6. Wrap up (1 min)**
- Say: "From live interaction to Playwright recording to Robot conversion to execution via rf-mcp — all in one flow. MCPs enable this seamless translation between frameworks."

### Script/Talking Points
> "We have a Playwright test from the previous demo. Let's see how Cline converts it to Robot Framework."

> "Cline understands both Playwright and Robot syntax. It generates valid Robot code with proper structure and imports."

> "Now Cline runs the Robot test directly via rf-mcp. It's not just generating code — it's executing it and seeing the results."

> "Here's the powerful part: Cline analyzes the Robot test against best practices and suggests improvements. Better keyword extraction, documentation, variable naming — all automatically."

### Backup Plan (if demo fails or conversion is incomplete)
- Have pre-recorded screenshots showing:
  1. Original Playwright test
  2. Converted Robot Framework test
  3. Robot test running and passing (output)
  4. Cline's best practices analysis
- Narrate: "This is the workflow — Playwright to Robot conversion, execution via rf-mcp, and automated best practices review. Cline handles all the framework knowledge for you."

### Slide (1 slide)
- Title: "rf-mcp: Convert, Run, and Analyze Robot Tests"
- Annotated screenshots showing: "Playwright test → Robot conversion → test passing → best practices analysis"

---

## Section 6: Being Realistic About MCPs (5 minutes)

### Goal
Show that MCPs aren't required for everything — some workflows don't need them.

### Script
> "MCPs are powerful, but they're not the answer for everything. Be realistic about when the approval cost is worth it.
> 
> **Where MCPs don't add much:**
> - **One-off tasks** — If you're checking GitLab MR status once a day, MCP doesn't save friction. CLI is fine.
> - **Simple queries** — If you're running a quick database query, the CLI works fine. arangosh is simpler.
> - **CI/CD** — In your pipeline, tests run headlessly. No IDE, no interactive feedback. CLI is the right tool.
> 
> **Where MCPs make a real difference:**
> - **Iterative debugging** — Write → test → fail → debug → fix cycle. Claude reasoning + direct tool access = tight feedback.
> - **Complex output parsing** — When test output is verbose or hard to understand, Claude analyzing it directly is faster than you reading logs.
> - **Mid-coding interruptions** — You're writing code, need docs. MCP fetches them in-IDE. No context switch.
> 
> The rule: **If you're in a tight iteration loop with feedback, MCP saves time. If it's a one-shot command, CLI is fine.**"

### Slides (2 slides)

**Slide 1: CLI is Fine For**
- One-off commands (glab MR status, database queries)
- CI/CD pipelines (no IDE integration benefit)
- Non-interactive tasks
- Quick lookups

**Slide 2: MCP Shines For**
- Tight iteration loops (write → test → debug → fix)
- Complex output that needs reasoning
- In-IDE workflows (docs, suggestions)
- Feedback-driven development

### Key Talking Points
- "We're asking security to approve six MCPs that solve real friction. We're not trying to MCP everything."
- "This is honest: some workflows don't need the approval overhead. That's fine."
- "We're being selective about what's worth it."

---

## Section 7: Security Approval Path (10 minutes)

### Goal
Remove uncertainty. Show exactly how approval works and what security reviews.

### Script (2-part)

**Part 1: Why Security Reviews MCPs (3 min)**
> "MCPs are new network interfaces between Cline and your tools. Security needs to vet them because:
> - They can access your codebase, databases, APIs
> - They run as background services or plugins
> - They need IAM/credentials to function
> 
> It's the same reason you can't just install random npm packages — they touch your system. Security reviews are actually good here."

**Part 2: Our Approach (4 min)**
> "We're being proactive with security. Instead of asking them to evaluate from scratch, we're:
> 
> 1. **Create a security evaluation plan** — Document what we're assessing for each MCP: error handling, credential handling, data flow, logging, potential risks.
> 2. **Write security reports** — For each of the six MCPs, we've already analyzed: what it does, what permissions it needs, what could go wrong, and how we mitigate it.
> 3. **Submit to security** — Hand them the evaluation plan and all six reports upfront. No surprises, no back-and-forth on 'what do these do.'
> 4. **Security reviews** — They verify our analysis, ask targeted questions if needed (usually minor clarifications).
> 5. **Approval** — Either 'approved' or 'approved with restrictions.' We move fast because we've done the homework.
> 6. **Rollout** — Teams start using the approved MCPs. One approval works everywhere: VS Code, IntelliJ, CLI, any environment.
> 
> Timeline: Usually 1 week, sometimes faster because security doesn't need to reverse-engineer the MCPs."

**Part 3: Your Role (3 min)**
> "Here's what we need from you:
> 
> 1. **Feedback on the evaluation plan** — Does it cover the risks you care about? Are we missing something?
> 2. **Questions or concerns about specific MCPs** — Ask now, and we'll address them in the security reports.
> 3. **After approval** — Setup instructions will be shared tailored to your IDE or environment.
> 
> **Key point:** One security approval covers all environments. You don't need separate approvals for VS Code vs IntelliJ vs CLI — it's all the same MCPs.
> 
> Timeline: We're submitting the full evaluation plan and reports next week. You'll hear when security approves."

### Slides (3 slides)

**Slide 1: Why Security Reviews MCPs**
- MCPs access your code, databases, APIs
- They run as background services
- They need IAM/credentials
- This is similar to reviewing npm packages

**Slide 2: Our Approval Approach**
```
Evaluation Plan + Security Reports → Security Reviews → Approved/Mitigated → Rollout
(~1 week, we do the homework upfront)
```

**Slide 3: What We Need From You**
- Review the evaluation plan — does it cover your concerns?
- Questions about specific MCPs — ask now
- After approval: setup instructions tailored to your IDE
- Timeline: submitting reports next week

---

## Section 8: Q&A + Next Steps (2 minutes)

### Goal
Address concerns, gather feedback, clarify adoption path.

### Anticipated Questions & Answers

**Q: "What if security says no?"**  
A: "We'll understand their concerns. Usually it's about credentials or logging. We'll work with them to mitigate — maybe 'context7 only on non-prod' or 'rf-mcp with audit logging.' Worst case, we have the CLI as backup. MCPs are an optimization, not a blocker."

**Q: "Do I have to use MCPs?"**  
A: "No. If you prefer the CLI, that's fine. MCPs are tools we're making available. Use them where they help, skip where they don't."

**Q: "How do I set up the MCP once it's approved?"**  
A: "One config file. We'll provide a template, walkthrough docs, and a guide. Should take 5 minutes."

**Q: "What about other MCPs I find online?"**  
A: "Those still need security approval. Don't add unofficial MCPs to your setup without asking. We'll have a simple request process."

**Q: "Can I use MCPs offline?"**  
A: "These are local MCPs — they run on your machine, not cloud. Once set up, they work offline (as long as your tools work offline)."

### Talking Points for Closing
- "We're doing this thoughtfully — not rushing, thinking about security, being honest about when they help and when they don't."
- "The goal is reducing friction for the developers who want it, without forcing it on anyone."
- "Questions now, or reach out anytime. Thanks."

### Slide (1 slide)
- Title: "Questions?"
- Bullet points:
  - Security concerns? Ask now.
  - Setup questions? We'll send docs.
  - Want to try them early? Let us know.
  - Contact: [your email]

---

## Demo Backup Materials

### If Playwright Demo Fails
**Backup option:** Pre-recorded GIF or screenshot showing:
1. Test code being generated
2. Test running
3. Failure message
4. Claude suggesting fix

**Fallback narration:**
> "If this were running live, you'd see Cline generate the test, run it, see that selector failing, and suggest the fix — all in one flow. Since the live version is being flaky, here's what it looks like when it works smoothly."

### If rf-mcp Demo Fails
**Backup option:** Pre-recorded screenshot or text showing:
1. Complex Robot test failure log
2. Claude's analysis explaining the root cause
3. Claude's suggested fix

**Fallback narration:**
> "This is a real example of a Robot test failure. Notice how the log is hard to parse — lots of noise. Cline analyzes that and pinpoints: the element lookup failed because the page wasn't ready. Instead of you digging through logs, Cline does that work for you."

---

## Preparation Checklist

- [ ] **Slides created** in your preferred tool (PowerPoint, Google Slides, etc.)
- [ ] **Playwright MCP configured** in Cline (test live)
- [ ] **rf-mcp configured** (test with a sample Robot test)
- [ ] **Demo scripts written out** (talking points for each demo)
- [ ] **Backup screenshots/videos captured** (if live demos fail)
- [ ] **Example code/tests ready** (don't improvise during demo)
- [ ] **Timing checked** — Run through once, hit the 60-minute target
- [ ] **Security approval request drafted** (what you'll send to security team)
- [ ] **Setup documentation prepared** (what you'll share after approval)

---

## Timing Checkpoints

- **At 15 min mark:** Should be wrapping Section 2 (MCP vs CLI). If you're earlier, you can extend Q&A later.
- **At 27 min mark:** Should be starting Demo 2 (rf-mcp). First demo should have gone smoothly.
- **At 42 min mark:** Should be wrapping Demo 2, moving to "When CLI is Enough."
- **At 47 min mark:** Should be starting Security Approval Path.
- **At 57 min mark:** Should be in Q&A, wrapping up naturally.

If you're running behind:
- **Compress "When CLI is Enough"** to 2 minutes (skip one bullet per topic).
- **Shorten Q&A** to 3 minutes, offer to take additional questions via email.
- **Cut specific slides** from Security section if needed (keep the flow chart).

---

## Notes for Delivery

- **Pacing:** Your script is conversational. Read it once, then ad-lib based on audience reactions. Don't sound robotic.
- **Demos:** Assume they fail. Have backups. It's okay to say "let me show you what this looks like" and pull up a screenshot.
- **Honesty is your strength:** You're not overselling MCPs. You're showing trade-offs. That builds trust.
- **After presentation:** Offer to send the slide deck and setup guide to attendees. Ask for feedback on what was most/least useful.
