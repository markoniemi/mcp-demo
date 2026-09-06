# MCP Presentation Plan: When to Use MCPs vs. CLIs

**Audience:** Developers using Cline (React + Spring Boot + Robot Framework + ArangoDB stack)  
**Duration:** 60 minutes  
**Goal:** Adopt local MCPs in Cline after security team approval by showing what MCPs enable (direct tool access) and why the approval cost is worth it  
**Note:** Installation details are out of scope — this presentation focuses on value proposition, not setup

---

## Overall Structure & Timing

| Section | Time | Type |
|---------|------|------|
| 1. Opening: What We're Exploring | 2 min | Talk |
| 2. What is an MCP? | 5 min | Slides + Explanation |
| 3. Why MCPs Matter for Claude | 12 min | Slides + Explanation |
| 4. Demo 1: Playwright MCP | 12 min | Live Demo |
| 5. Demo 2: rf-mcp | 12 min | Live Demo |
| 6. When CLI is Enough | 5 min | Honest Assessment |
| 7. Security Approval Path | 10 min | Slides + Process |
| 8. Q&A + Next Steps | 2 min | Discussion |

---

## Section 1: Opening (2 minutes)

### Goal
Introduce the core question: Claude can help with your tools in two ways. Which is better?

### Script
> "You use Cline to help with coding, testing, debugging. But here's the question: how does Cline access your tools? Two ways:
> 
> **One:** You tell Cline 'run this playwright test,' Cline executes it in a terminal, returns the output. You manage the handoff between Cline and the tool.
> 
> **Two:** Cline has direct access via MCP. It runs the tool, sees results, integrates them into suggestions — seamlessly, within Cline.
> 
> Which is better? That's what we're exploring today. Spoiler: MCPs are smoother. The catch: they need security approval because they're new interfaces. But the payoff is worth it. So we're asking: which tools are worth that approval cost?"

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
> **Local MCPs** — Run on your machine. Example: Cline runs Playwright tests on your laptop using the Playwright MCP. Advantage: your code and data stay local. Your database stays on your machine. Security team's main concern is what the MCP code does, not where data travels.
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

## Section 3: Why MCPs Matter for Cline (12 minutes)

### Goal
Show what MCPs enable and why they're worth the security approval cost.

### Script
> "MCPs give Cline **direct, seamless access** to your tools. Here's why that matters:
> 
> **Without MCP:** You tell Cline 'run this test,' Cline executes a command, gets output, returns it to you. Cline sees raw text. You have to interpret it and decide what to do.
>
> **With MCP:** Cline runs the tool *and understands the results*. Test fails? Cline analyzes the failure, understands what went wrong, and suggests the fix — right in your editor. Cline interprets it for you; you don't read raw output.
>
> Now, the catch: **MCPs are new software**. They're connections between Cline and your infrastructure. Security needs to review them. Regular CLI tools don't need approval because they're already installed as dev dependencies.
>
> So the real question: **Is the seamless experience worth asking security to approve new interfaces?** We think so, for the tools that matter most to your workflow. Let's show you what that looks like."

### Slides (4 slides)

**Slide 1: Terminal vs MCP Access Model**
```
Terminal Path:
Cline → "Run test" → Terminal → Output → You interpret

MCP Path:
Cline → Direct Access → Runs tool → Understands results → Suggests fix
```

**Slide 2: What's Different?**
- Terminal: Cline gives you information; you reason about it
- MCP: Cline acts and interprets; you review suggestion
- Result: Faster feedback loops, Cline does the reasoning

**Slide 3: The Security Trade-off**
- **CLI tools:** Already approved as dev dependencies. No new review needed.
- **MCPs:** New interfaces between Cline and your tools. Security reviews them once. Worth it because of better integration across your IDEs.

**Slide 4: Which MCPs We're Asking For**
- Playwright MCP (write/debug tests seamlessly in Cline)
- rf-mcp (Cline understands Robot Framework test failures)
- context7 (fetch current docs without leaving your editor)
- All local, all auditable, all work everywhere (VS Code, IntelliJ, CLI)

### Key Talking Points
- "MCPs aren't about replacing terminal tools — it's about how Cline helps you *use* your tools better."
- "Terminal is fine if you like manual handoffs. MCP is smoother if you want Cline to do more of the reasoning."
- "Security approval is real, but these are solid, documented tools worth that cost. One approval works everywhere."

---

## Section 4: Demo 1 — Playwright MCP (12 minutes)

### Goal
Show what Playwright MCP enables: Cline runs e2e tests against a live application and identifies bugs by analyzing test interactions.

### Demo Scenario
Run e2e tests from the `e2e/` directory against the app in `src/`, and ask Cline to spot errors in the application based on test behavior.

### Pre-Demo Setup Checklist
- [ ] **Application started** — `npm start` or your dev server running on localhost (e.g., http://localhost:3000)
- [ ] Cline open with Playwright MCP configured
- [ ] E2E test files exist in `e2e/` directory (e.g., `e2e/example.spec.ts`)
- [ ] Backup screenshots: test running, passing/failing output, Cline's error analysis
- [ ] **Note:** Installation details are not demoed — assume MCPs are already set up

### Demo Flow (12 min breakdown)

**1. Setup (1 min)**
- Show the app running in browser (or split view)
- Say: "We have a running application and a suite of e2e tests. With Playwright MCP, Cline can run these tests, see the interactions, and spot bugs in our app. Let's see what it finds."

**2. Cline runs e2e tests via Playwright MCP (4 min)**
- Prompt Cline: "Run the e2e tests in the `e2e/` directory. Show me the test output and what interactions the app is performing."
- Cline executes the tests via Playwright MCP (direct access)
- Show test output: which tests passed, which failed, browser interactions logged
- Say: "Cline ran all the tests and got the full output. It can see exactly what the app did during each interaction."

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
- [ ] Playwright test generated from Demo 1 (or sample Playwright test ready)
- [ ] rf-mcp configured in Cline
- [ ] Robot Framework installed locally (for testing)
- [ ] Browser library available for Robot (via requirements.txt or pip)
- [ ] Backup: screenshots showing codegen → Playwright test → converted Robot test → analysis
- [ ] **Note:** Installation details are not demoed — assume rf-mcp is already set up

### Demo Flow (12 min breakdown)

**1. Start with Playwright test (1 min)**
- Show the Playwright test generated in Demo 1
- Say: "We recorded this Playwright test. Now let's see what happens when we convert it to Robot Framework using Cline's rf-mcp access."

**2. Cline converts Playwright to Robot (3 min)**
- Ask Cline: "Convert this Playwright test to a Robot Framework test. Use the Browser library. Keep the same test flow."
- Cline generates Robot Framework test code with proper syntax:
  - `*** Settings ***` with Browser library import
  - `*** Test Cases ***` with the test
  - Keywords properly structured
- Say: "Cline understands both frameworks and generates valid Robot syntax automatically."

**3. Run Robot test via rf-mcp (3 min)**
- Ask Cline to run the Robot test using rf-mcp
- Cline executes the test directly with MCP access
- Show the test running and passing
- Say: "The test runs and passes. Cline has direct access to Robot Framework via rf-mcp — it runs the test and gets the full output."

**4. Analyze for best practices (4 min)**
- Ask Cline: "Analyze this Robot test. Is it following Robot Framework best practices? Check keyword naming, test structure, library usage, documentation, and variable naming."
- Cline reviews the test and provides feedback:
  - Suggests better keyword extraction (breaking down steps into reusable keywords)
  - Recommends documentation format (*** Keywords ***, better test descriptions)
  - Suggests variable naming conventions (${BROWSER} vs ${browser})
  - Recommends wait strategies and implicit waits
  - Points out logging and reporting improvements
- Show Cline's analysis appearing inline
- Say: "Cline analyzes the test structure and suggests how to make it more maintainable and aligned with Robot Framework conventions."

**5. Wrap up (1 min)**
- Say: "That's the workflow: record in Playwright, convert to Robot, run via rf-mcp, and improve using Cline's understanding of best practices. MCP makes all of this seamless."

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
> - **Simple queries** — If you're running a quick database query, you don't need Claude's reasoning. arangosh works.
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
- "We're asking security to approve three MCPs that solve real friction. We're not trying to MCP everything."
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

**Part 2: The Approval Process (4 min)**
> "Here's what happens:
> 
> 1. **We request approval** — Submit three MCPs: playwright, rf-mcp, context7. Include: what they do, what permissions they need, why we need them.
> 2. **Security reviews** — They evaluate: Does it have the right error handling? Does it log securely? Can it leak credentials? (They'll ask us questions.)
> 3. **We answer** — Clarify any concerns, point them to documentation, show how we're using it safely.
> 4. **Approval or mitigation** — Either 'approved' or 'approved with restrictions' (e.g., 'only use on non-prod databases').
> 5. **Rollout** — Teams start using the approved MCPs. One approval works everywhere: VS Code, IntelliJ, CLI, any environment.
> 
> Timeline: Typically 1-2 weeks, sometimes faster if the tools are known and well-documented."

**Part 3: Your Role (3 min)**
> "After this presentation, you don't need to do anything immediately. We'll take the approval request to security. Once approved, we'll share setup documentation tailored to your IDE or environment.
> 
> **Key point:** One security approval covers all environments. You don't need separate approvals for VS Code vs IntelliJ vs CLI — it's all the same MCPs.
> 
> Questions about specific tools or concerns? Bring them now, and we'll include them in the security request."

### Slides (3 slides)

**Slide 1: Why Security Reviews MCPs**
- MCPs access your code, databases, APIs
- They run as background services
- They need IAM/credentials
- This is similar to reviewing npm packages

**Slide 2: The Approval Flow**
```
Request → Security Reviews → We Clarify → Approved/Mitigated → Rollout
(1-2 weeks typical)
```

**Slide 3: Next Steps**
- Security request submitted with playwright, rf-mcp, context7
- You'll hear when it's approved
- Setup instructions shared to all developers
- Questions? Raise them now or email [your email]

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
> "If this were running live, you'd see Claude generate the test, run it, see that selector failing, and suggest the fix — all in one flow. Since the live version is being flaky, here's what it looks like when it works smoothly."

### If rf-mcp Demo Fails
**Backup option:** Pre-recorded screenshot or text showing:
1. Complex Robot test failure log
2. Claude's analysis explaining the root cause
3. Claude's suggested fix

**Fallback narration:**
> "This is a real example of a Robot test failure. Notice how the log is hard to parse — lots of noise. Claude analyzes that and pinpoints: the element lookup failed because the page wasn't ready. Instead of you digging through logs, Claude does that work for you."

---

## Preparation Checklist

- [ ] **Slides created** in your preferred tool (PowerPoint, Google Slides, etc.)
- [ ] **Playwright MCP configured** in Claude Code (test live)
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
