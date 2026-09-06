# MCP Security Evaluation Guide

Evaluating a local MCP (Model Context Protocol server) is about understanding what it can access and whether you trust it. This guide walks through how to do that systematically.

Local MCPs run directly on your machine with access to your files, environment variables, and processes. That's why evaluation matters—a compromised MCP can compromise your system.

---

## What You're Looking For

Three things:
1. **Is the code from a trusted source?** (public, maintained, no obvious red flags)
2. **What can it actually do?** (file access, network access, dangerous tools)
3. **Can you restrict what it does?** (configuration options, firewall rules, OS permissions)

---

## When to Reject Immediately

Don't waste time on a full evaluation if any of these are true:

- Unpatched critical CVEs (CVSS ≥9.0) with no fix in sight
- Hardcoded credentials or API keys in the source code
- Post-install scripts that make network requests (runs arbitrary code at install time)
- Native bindings (.node files) from unknown or untrusted sources
- Clear evidence of malicious intent—obfuscation, credential stealing, exfiltration
- No source code available (closed source or removed from public repos)
- Project abandoned for 2+ years with open security issues
- Tool descriptions that are explicitly malicious or suspicious

If you find any of these: stop, document it, and reject. No further review needed.

---

## Available Tools

Before you start manual analysis, use these automated tools. They're fast and catch obvious issues.

**Web-based (no setup required):**
- **MCPScan.ai** — Paste a GitHub URL, get a security scan in 30 seconds. Catches command injection, poisoning, data exfiltration issues.
- **Socket.dev** — Analyzes npm packages for risky behavior (post-install scripts, native bindings, sketchy patterns).

**CLI tools (if you want deeper analysis):**
- **Snyk agent-scan** — Detects prompt injection, tool poisoning, tool shadowing. Scans MCPs and AI agents.
- **Semgrep** — Static code analysis. Good at finding security patterns and bugs.

**MCP registries:**
- **MCPVerified** — Directory of MCPs with security reviews.
- **Official MCP Registry** — List of published MCPs at modelcontextprotocol.io.

---

## The Evaluation Phases

You can do this in phases. Stop early if something looks wrong.

### Quick Scan (30 minutes)

Start here. You're just checking if this is a legitimate project worth looking at deeper.

- [ ] Run MCPScan.ai on the GitHub repo (takes 1 minute)
- [ ] Check npm audit (or equivalent for your language): `npm audit --audit-level=moderate`
- [ ] Is the repo public on GitHub? Is it actively maintained (commits in the last 6 months)?
- [ ] Does the README explain what it does and what permissions it needs?
- [ ] Any hardcoded credentials lying around in package.json or config?

If everything checks out here, keep going. If you find something concerning, investigate deeper before deciding.

---

### Medium Dive (2 hours)

Now you're looking at the actual code and how it works.

**Dependencies:**

- [ ] Check the dependency tree: what does this thing actually pull in?
  - `npm ls --all` for JavaScript/Node
  - `pip list` for Python
  - Look for post-install scripts (red flag)
  - Flag native bindings—are they from trusted sources?

- [ ] Search the source for risky patterns: `eval`, `exec`, `spawn`, things that run code dynamically
- [ ] Any obvious hardcoded secrets? Search for "password", "secret", "api_key", "token"
- [ ] Check if there are deprecated packages or packages that haven't been updated in years

**Runtime behavior:**

- [ ] Create some test files and try to access them. Can the MCP read/write only in its declared scope, or can it access your whole disk?
- [ ] Try running it as a restricted user (not admin). Does it work normally, or does it need elevated privileges?
- [ ] Can it see sensitive files like .ssh directories? Try to verify it can't.

Use Socket.dev if you want quick analysis of the npm package. Use Semgrep if you want to grep the code automatically for security patterns.

---

### Deep Dive (4+ hours)

For critical MCPs or when the medium dive raised questions. This is where you verify runtime behavior.

**Code review:**

- [ ] Read through the source carefully. Look at error handling—do error messages leak paths or secrets?
- [ ] Check the permission model: does it follow least privilege or is it overpermissive?
- [ ] Audit the transitive dependencies. Not just direct deps, but what do *they* pull in?

**Runtime testing:**

- [ ] Use Process Monitor (Windows) to watch what the MCP actually does
  - What files does it open?
  - What processes does it spawn?
  - What environment variables does it read?
  - Does it write outside its declared directories?

- [ ] If it makes network calls, inspect them with Wireshark or mitmproxy
  - Where is it calling?
  - Is TLS working properly?
  - Are credentials in the clear anywhere?

- [ ] Run static analysis (Semgrep, Snyk) if the code is complex

---

## MCP Configuration: What Can You Actually Restrict?

Every MCP is different, but here's what you should look for:

**File system:** Can you limit it to a specific directory? Is there a flag like `--workspace-root` or `--file-access restricted`?

**Network:** Can you restrict which domains it can reach? Look for `--allowed-origins`, `--network-whitelist`, or similar.

**Tools/capabilities:** Can you disable specific functions? Some MCPs let you say "use this tool but not that one." Others expose everything.

**Process:** Can you set resource limits (memory, CPU, timeout)?

Check the README first. If that doesn't say, run `--help`. If that doesn't work, ask the maintainer on GitHub.

---

## Questions to Ask Before Approval

Get answers to these. They'll guide your decision.

1. **File access:** Does this thing need to read/write files? If so, which directories? Read-only or read-write?

2. **Network:** Does it phone home to external servers? If so, which ones? (You'll need this for firewall rules.)

3. **Privileges:** Can it run as a normal user, or does it need admin rights?

4. **Secrets:** Does it read environment variables or config files? Could those contain credentials?

5. **Dependencies:** Are the direct and transitive dependencies all from trusted sources? Any post-install scripts doing weird stuff?

---

## Making the Decision

After you've done the review, here's how to decide:

**Did you find red flags (unpatched CVEs, hardcoded secrets, malicious code)?**  
→ Reject. Done.

**Did the automated tools flag critical issues?**  
→ Investigate further. If it's a false positive, document why. If it's real, reject.

**Can the risky capabilities be restricted?**  
→ Good. Document what you need to restrict (firewall rules, config flags, OS permissions).

**Are the optional dependencies a problem?**  
→ If the MCP offers modular features, only install what you need. SeleniumLibrary for web testing? Only if you're doing web testing.

**Is there a clear way to monitor what it does?**  
→ Can you watch file access, network traffic, process creation? If yes, approval is safer.

**Final decision table:**

- All red flags passed + risks are configurable/restrictable → **Approve**
- Red flags passed + some risks but good mitigations available → **Approve with restrictions**
- RCE-like tools present but can be access-controlled → **Approve (with monitoring)**
- Multiple high-risk findings with no way to mitigate → **Escalate to security team**
- Red flags found or no possible mitigations → **Reject**

---

## Running the Evaluation

Here's the workflow:

1. **Check red flags** (5 min) — If any are true, stop and reject
2. **Quick scan** (30 min) — Automated tools + basic checks
3. **Medium dive** (2 hours) — Code review + dependency audit + runtime checks
4. **Deep dive** (4+ hours) — Process Monitor + Wireshark + detailed analysis (only if needed)
5. **Decision** — Use the decision table above

---

## Windows Tools Reference

You'll need some tools for the deep dive on Windows. Here's what to use:

**File & process monitoring:**
- **Process Monitor** (Sysinternals) — Watch what the MCP accesses
  - Download from Microsoft's site
  - Run as admin, filter by process name
  - Look for unexpected file reads/writes, registry access, process spawning

**Network monitoring:**
- **netstat** — Built-in Windows command, see active connections
  - `netstat -ano | findstr node.exe` to see what a node process is connecting to
- **Wireshark** — Detailed packet inspection if you need to see HTTPS traffic
  - Download from wireshark.org

**Testing permissions:**
- **Local Users and Groups** (lusrmgr.msc) — Create a restricted test user
- **runas** command — Run the MCP as that user
- **NTFS permissions** — Set file access restrictions via Properties

---

## Common Commands

Checking dependencies:

```powershell
# npm/Node
npm audit --audit-level=moderate
npm outdated
npm ls --all
Get-ChildItem node_modules -Recurse -Filter *.node  # Find native bindings

# Python
pip list
pip check  # Look for dependency conflicts
```

Searching for risky patterns:

```powershell
# Search for eval, exec, spawn
Get-ChildItem -Path src -Recurse -Include *.js,*.ts | Select-String -Pattern "eval|exec|spawn"

# Search for hardcoded secrets
Get-ChildItem -Path src -Recurse -Include *.js,*.ts,*.json | Select-String -Pattern "password|secret|api_key|token"
```

Testing file access:

```powershell
# Try to access a restricted file as the MCP user
runas /user:mcp_user "type C:\Windows\System32\config\sam"
# Should fail: "Access Denied"
```

Monitoring:

```powershell
# See what the process is connecting to
netstat -ano | findstr node.exe

# Check what files it's accessing (use Process Monitor GUI, not command line)
```

---

## What Happens Next

Once you've done the evaluation:

1. **Document your findings** — What did you check? What did you find? Any restrictions needed?
2. **Make the call** — Approve, approve with restrictions, or reject
3. **If approved with restrictions** — Document them clearly:
   - Firewall rules needed
   - Config flags to enable/disable
   - OS permissions to set
   - User account requirements
   - Monitoring strategy
4. **Set up monitoring** — Even if you approve, watch what it actually does in production

---

## Resources

- **OWASP Security Coding Practices** — owasp.org
- **npm Security Docs** — docs.npmjs.com/cli/v9/using-npm/security
- **Node.js Security** — nodejs.org/en/docs/guides/security/
- **MCP Specification** — modelcontextprotocol.io/docs
- **Sysinternals Process Monitor** — microsoft.com/en-us/sysinternals/downloads/procmon
- **Wireshark** — wireshark.org
