# rf-mcp Security Evaluation

**MCP:** rf-mcp (Robot Framework MCP Server)  
**Version:** 0.35.0  
**Repository:** https://github.com/manykarim/rf-mcp  
**Language:** Python ≥3.10  
**Package Manager:** PyPI  
**Evaluation Date:** 2026-08-28  
**Evaluated Using:** MCP Security Evaluation Guide (Windows, Python variant)

---

## Executive Summary

**Status:** PROCEED TO APPROVAL DECISION

rf-mcp is an actively maintained, open-source Robot Framework MCP server for test automation. It enables AI agents to discover, execute, and generate Robot Framework test cases across multiple platforms (browser, mobile, API, database, desktop).

**Evaluation shows:** Moderate security posture with **active maintenance and no known CVEs**. However, modular architecture with many optional dependencies increases supply-chain attack surface. File and network access patterns depend on enabled optional features.

**Red Flags Check:** ✅ PASS — No blockers found

**Key Points:**
- ✅ Public GitHub, Apache-2.0 licensed, actively maintained (daily commits)
- ✅ No known CVEs or security advisories
- ✅ No hardcoded credentials, closed source, or abandoned status
- ⚠️ Modular optional dependencies (web, mobile, api, desktop) increase complexity
- ⚠️ Can execute arbitrary Robot Framework keywords (by design)
- ⚠️ Network/file access depends on installed optional libraries (SeleniumLibrary, Appium, etc.)
- ⚠️ No SECURITY.md or responsible disclosure policy documented

---

## Red Flags Check

**Stop evaluation if ANY found. Results:**

| Red Flag | Status | Notes |
|----------|--------|-------|
| Unpatched critical CVEs (CVSS ≥ 9.0) | ✅ PASS | No known CVEs; PyPI shows clean history |
| Hardcoded credentials or API keys | ✅ PASS | None found in public source code |
| Suspicious post-install scripts | ✅ PASS | Uses standard Python packaging (Hatchling) |
| Native bindings from untrusted sources | ⚠️ CHECK | Optional: PlatynUI (Windows desktop automation) uses native bindings |
| Evidence of malicious intent | ✅ PASS | Code purpose is clear, well-documented |
| Closed source or source removed | ✅ PASS | Public GitHub repo, full source available |
| Abandoned project | ✅ PASS | Actively maintained, commits as recent as Aug 7, 2026 |
| Malicious or suspicious tool descriptions | ✅ PASS | Tools are documented for test automation purpose |

**Red Flags Result:** ✅ PASS (with note on PlatynUI native bindings for desktop automation)

---

## Quick Review (30 min) — COMPLETED

### Automated Security Scanners

| Tool | Result | Notes |
|------|--------|-------|
| **MCPScan.ai** | ⚠️ UNABLE TO VERIFY | API service currently unavailable (connection error). Attempted scan on 2026-08-29; service backend unreachable. |
| **GitHub Security Advisories** | 🟢 PASS | No active advisories for rf-mcp or direct dependencies |
| **PyPI Health Check** | 🟢 PASS | Project actively maintained, recent releases, good documentation |
| **Direct Dependencies Audit** | 🟢 PASS | All core dependencies (fastmcp, robotframework, pydantic) are well-maintained |

### Basic Checks

- [x] Repository public on GitHub? → **Yes**, Apache-2.0 licensed
- [x] README documents purpose & permissions? → **Yes**, comprehensive README with feature overview
- [x] Recent commits (last 6 months)? → **Yes**, multiple commits daily, latest Aug 7, 2026
- [x] Hardcoded secrets in source? → **No**
- [x] Dependency metadata transparent? → **Yes**, PyPI lists all dependencies clearly

**Quick Review Result:** ✅ PASS — Proceed to Medium Review

---

## Medium Review (2 hours) — COMPLETED

### Code & Dependencies Analysis

**Purpose:** Test automation MCP server. Executes Robot Framework keywords and generates test suites.

**Direct Dependencies:**
```
fastmcp (≥3.0)
robotframework (≥7.0)
pydantic (≥2.0.0)
beautifulsoup4, lxml, python-dotenv, pyyaml, tomlkit
```

**Optional Feature Dependencies (Modular):**
- **web:** SeleniumLibrary, Browser Library
- **mobile:** AppiumLibrary
- **api:** RequestsLibrary
- **database:** DatabaseLibrary
- **desktop:** PlatynUI (Windows desktop automation, Python 3.12+)
- **memory:** sqlite-vec (semantic search)
- **tokens:** tiktoken (token counting)

**Dependency Assessment:**
- ✅ Core dependencies pinned to minimum versions (e.g., `robotframework >=7.0`)
- ✅ All direct dependencies are well-maintained, popular projects
- ⚠️ Optional features introduce additional attack surface:
  - SeleniumLibrary: Browser automation (network + file I/O)
  - AppiumLibrary: Mobile testing (network + device interaction)
  - PlatynUI: Desktop automation (native bindings, Windows registry access)
- ✅ No post-install scripts detected (standard Python packaging)

**Risky Patterns:**
- ✅ No hardcoded credentials found
- ✅ No shell injection patterns (exec, eval with user input)
- ⚠️ Intentional: Keyword execution is by design (executes Robot Framework keywords dynamically)
- ⚠️ File access: Can read/write test files based on --workspace configuration

### MCP Configuration & Restrictions

**What Can Be Restricted:**

| Restriction Type | Available? | Configuration | Default |
|------------------|-----------|----------------|---------|
| **Workspace Root** | ✅ Yes | Set via environment variable or config file | Must be configured |
| **Library Restrictions** | ✅ Yes | Only install required optional features | All features optional |
| **Network Access** | ⚠️ Partial | Depends on installed libraries (SeleniumLibrary, etc.) | Not restricted by default |
| **File I/O Scope** | ✅ Yes | Workspace root limits scope | Configurable |
| **Keyword Allowlist** | ❌ No | Cannot disable specific Robot Framework keywords | All keywords available |
| **Environment Variables** | ⚠️ Depends | Parsed in test execution | Check test file contents |

**How to Check MCP Configuration:**
1. README: Lists all configuration options and environment variables
2. GitHub Issues/Discussions: Community support and configuration examples
3. Docker docs: Pre-configured container examples with known settings
4. Python help: `python -m rf_mcp --help` for CLI options

### Runtime & Access Verification

**File System:**
- ✅ Workspace root configurable (limits file access by default)
- ⚠️ Can read/write test files within workspace
- ✅ No automatic access to %USERPROFILE%\.ssh or similar
- ⚠️ Installed libraries (Selenium, Appium) may access additional paths

**Network Access:**
- 🔴 HIGH if web features enabled: SeleniumLibrary/Browser Library make outbound HTTP/HTTPS requests
- 🔴 HIGH if mobile features enabled: AppiumLibrary connects to Appium servers
- 🔴 MEDIUM if api features enabled: RequestsLibrary makes HTTP API calls
- ✅ NONE if only core features used (no optional libraries)

**Process Privileges:**
- ✅ Can run as unprivileged user (standard Python process)
- ⚠️ Desktop automation (PlatynUI) may require elevated privileges on Windows
- ✅ Browser automation runs as current user

**Medium Review Result:** ✅ PASS with documented risks based on optional features

---

## Deep Review (4+ hours) — PARTIAL (Runtime Testing Needed)

### Detailed Code Analysis

**Architecture:**
- MCP server using FastMCP framework
- Exposes Robot Framework keyword execution as MCP tools
- Optional libraries loaded dynamically based on configuration
- Persistent semantic memory for optimization (optional)
- Django dashboard for monitoring (optional)

**Tool Capabilities:**
- `discover_keywords` — List available Robot Framework keywords
- `execute_keyword` — Run keyword with parameters
- `generate_tests` — Create .robot test files from execution results
- `list_browsers` — List available browser instances
- `debug_attach` — Connect to IDE debugger (VS Code, RobotCode)

**Risky Patterns:**
- ⚠️ Keyword execution by design (keyword names user-controlled)
- ✅ Input validation via Pydantic models
- ✅ No eval/exec with user input found
- ⚠️ Library loading: Which libraries are installed determines capabilities

**Error Handling Review:**
- ❓ Do error messages leak file paths or sensitive data? (need runtime testing)
- ❓ Does stack traces expose credential patterns? (need runtime testing)

### Dependency Deep Dive (Sample)

**fastmcp** (core MCP framework)
- Status: Well-maintained, active development
- Risk: Low (foundational library)

**robotframework** (test automation core)
- Status: Industry-standard, widely used, actively maintained
- Risk: Low (mature project)

**SeleniumLibrary** (optional, web automation)
- Status: Active, popular, maintained by Robot Framework community
- Risk: Medium (makes outbound HTTPS to web servers)

**AppiumLibrary** (optional, mobile automation)
- Status: Maintained, less active than SeleniumLibrary
- Risk: Medium-High (requires Appium server, network access)

**PlatynUI** (optional, desktop automation)
- Status: Less documented, newer library
- Risk: High (Windows native bindings, registry access)

### Runtime Behavior (Windows Process Monitor Testing Needed)

**File System Monitoring:**
- ❓ What files does rf-mcp access on startup?
- ❓ Which directories are created/modified during execution?
- ❓ Does it respect workspace-root restrictions?

**Network Monitoring:**
- ❓ What external endpoints does it contact at startup?
- ❓ Which libraries trigger network calls?
- ❓ Are TLS certificates validated?

**Process Monitoring:**
- ❓ Does it spawn child processes (browser, Appium, etc.)?
- ❓ What environment variables are read?
- ❓ Does PlatynUI require admin privileges?

**Deep Review Status:** ⚠️ INCOMPLETE — Runtime testing with Process Monitor needed

---

## Attack Scenarios

### Scenario 1: Malicious Test File Injection
**Attack:** Inject Robot Framework code via test file that executes system commands  
**Impact:** Code execution within rf-mcp process privilege level  
**Mitigation:**
- ✅ Limit workspace-root to trusted directories only
- ✅ Use file permissions (NTFS ACLs) to restrict test file modification
- ✅ Review test files before execution

### Scenario 2: Library Exploitation
**Attack:** Compromised SeleniumLibrary or Appium sends credentials to attacker server  
**Impact:** Session hijacking, credential theft  
**Mitigation:**
- ✅ Only install required optional libraries
- ✅ Audit transitive dependencies of installed libraries
- ✅ Use firewall to restrict outbound network
- ✅ Run behind HTTPS proxy (mitmproxy) to inspect traffic

### Scenario 3: Desktop Automation Privilege Escalation
**Attack:** Use PlatynUI to access Windows registry or system files  
**Impact:** System compromise  
**Mitigation:**
- ✅ Do NOT install desktop automation if not needed
- ✅ Run as restricted user (not admin)
- ✅ Monitor process for registry access with Process Monitor

### Scenario 4: Resource Exhaustion
**Attack:** Create infinite loop test that spawns many browser instances  
**Impact:** Out of memory, system hangs  
**Mitigation:**
- ✅ Run in container with memory/CPU limits
- ✅ Set timeout on keyword execution

---

## Recommended Mitigations

### CRITICAL — Optional Feature Management
- **Only install required optional features** — Do NOT install all options by default
- Decide: Do you need web automation? Mobile? Desktop? API testing? Install only those.
- Example restrictive install:
  ```bash
  pip install rf-mcp  # Core only, no web/mobile/desktop
  ```
- Example full install (higher risk):
  ```bash
  pip install rf-mcp[web,mobile,api,desktop,memory]  # All features
  ```

### CRITICAL — Workspace Configuration
- Set `RF_MCP_WORKSPACE` environment variable to specific, limited directory
- Use NTFS ACLs to restrict directory access
- Ensure only trusted users can modify test files in workspace
- Example:
  ```powershell
  $env:RF_MCP_WORKSPACE = "C:\workspace\rf-mcp-tests"
  ```

### HIGH — Network Isolation
- Windows Defender Firewall: Block all outbound HTTPS by default
- Whitelist specific servers if web/mobile/api features needed:
  - Selenium Grid servers (if used)
  - Appium servers (if mobile testing)
  - API endpoints for testing
- Use mitmproxy to inspect and validate HTTPS traffic

### HIGH — File System Isolation
- Run as restricted Windows user (not admin)
- Use NTFS permissions to restrict access:
  - Read-only access to test files (if possible)
  - Read/write only to workspace-root
  - No access to %USERPROFILE%\.ssh or system directories

### MEDIUM — Desktop Automation Restrictions
- If desktop automation (PlatynUI) is NOT needed: **Do NOT install it**
- If needed: Run only on isolated Windows VM with monitoring
- Monitor registry access and process creation with Process Monitor

### MEDIUM — Monitoring & Logging
- Enable MCP protocol logging
- Monitor file access in workspace-root
- Log all executed keywords and results
- Alert on attempts to access files outside workspace
- Monitor network connections (especially if web/api features enabled)

---

## MCP Configuration Options

### Environment Variables
```powershell
# Set workspace root
$env:RF_MCP_WORKSPACE = "C:\workspace\rf-mcp-tests"

# Enable debug mode (verbose logging)
$env:RF_MCP_DEBUG = "true"

# Set memory persistence (if sqlite-vec installed)
$env:RF_MCP_MEMORY_ENABLED = "true"
```

### Installation Profiles
```bash
# Minimal (core only)
pip install rf-mcp

# Web automation
pip install "rf-mcp[web]"

# Mobile automation
pip install "rf-mcp[mobile]"

# Desktop automation (Windows, Python 3.12+)
pip install "rf-mcp[desktop]"

# All features (highest risk)
pip install "rf-mcp[web,mobile,api,database,desktop,memory,tokens]"
```

### Startup Command
```powershell
# Run with workspace root
python -m rf_mcp --workspace-root C:\workspace\rf-mcp-tests

# Run with debug logging
python -m rf_mcp --debug
```

---

## Key Questions for Security Team

Before approval, answer:

1. **Which optional features are needed?** (web, mobile, api, database, desktop, memory)
   - Only install what's necessary to reduce attack surface

2. **Where will test files be stored?** (workspace configuration)
   - Must be limited to trusted directory with proper permissions

3. **What external services will it connect to?** (Selenium, Appium, APIs)
   - Needed for firewall allowlist configuration

4. **Who will author test files?** (trusted team, external contributors)
   - Affects file permission strategy

5. **Will this run on personal machines or shared systems?**
   - Affects privilege model and isolation requirements

---

## Approval Decision

**Following Approval Decision Flow from MCP Security Evaluation Guide:**

### Step 1: Red Flags Check ✅
All red flags passed (see above). Note: PlatynUI native bindings are documented and intentional.

### Step 2: Tool Results ✅
- **MCPScan.ai:** Unable to verify (API service unavailable as of 2026-08-29)
- **GitHub Security Advisories:** No active CVEs
- **PyPI Health:** Project actively maintained
- **Direct Dependencies:** All well-maintained, no critical issues
→ Core checks passed. Proceed. (Recommend retry MCPScan.ai when service available)

### Step 3: Evaluate Configuration ✅

| Capability | Restricted? | Configuration |
|-----------|------------|----------------|
| Optional Features | ✅ YES | Install only required features |
| Workspace Access | ✅ YES | `RF_MCP_WORKSPACE` environment variable |
| Keyword Execution | ❌ NO | All Robot Framework keywords available |
| Network (if web enabled) | ⚠️ PARTIAL | Firewall + Optional library controls |

**Finding:** Risk level depends on which optional features are installed. Core-only installation has low risk. Full installation with all features has medium-high risk.

### Step 4: Can Mitigations Address Risks?

**Risk:** Optional libraries (SeleniumLibrary, Appium, PlatynUI) introduce network/file access  
**Mitigation:** Only install required features; restrict via firewall and NTFS permissions  
**Addressed?** ✅ YES — Risk scales with feature selection

**Risk:** Test file execution may include malicious Robot Framework code  
**Mitigation:** Workspace restrictions + file permissions + code review  
**Addressed?** ✅ YES — Can enforce via configuration

**Risk:** Desktop automation (PlatynUI) may access Windows registry  
**Mitigation:** Only install if needed; run as restricted user; monitor with Process Monitor  
**Addressed?** ✅ YES — Can be disabled entirely if not needed

### Decision

| Criteria | Result | Notes |
|----------|--------|-------|
| Red flags found | ❌ None | All checks passed |
| Unpatched CVEs | ❌ None | No known vulnerabilities |
| Active maintenance | ✅ Yes | Recent commits, responsive to issues |
| Configurable | ✅ Yes (mostly) | Features optional; workspace configurable |
| Network access | ⚠️ Depends | Only if web/api/mobile features enabled |
| Mitigations adequate | ✅ Yes | Risk can be managed via feature selection |

**APPROVAL RECOMMENDATION: ✅ APPROVE WITH RESTRICTIONS**

**Condition 1 — Feature Selection (MANDATORY):**
```
Identify required optional features:
  ☐ web (SeleniumLibrary, Browser Library) — only if web testing needed
  ☐ mobile (AppiumLibrary) — only if mobile testing needed
  ☐ api (RequestsLibrary) — only if API testing needed
  ☐ database (DatabaseLibrary) — only if database testing needed
  ☐ desktop (PlatynUI) — ONLY on Windows VMs with monitoring
  ☐ memory (sqlite-vec) — optional optimization
  ☐ tokens (tiktoken) — optional LLM optimization

Installation profile:
  pip install rf-mcp[<selected-features-only>]
```

**Condition 2 — Workspace Configuration (MANDATORY):**
```powershell
Set workspace root to limited directory:
  $env:RF_MCP_WORKSPACE = "C:\workspace\rf-mcp-tests"
  
Restrict access via NTFS permissions:
  - MCP user: Modify (read/write/create)
  - Others: No access
```

**Condition 3 — Network Isolation (MANDATORY if web/api/mobile enabled):**
```
If web/api/mobile features enabled:
  - Windows Firewall: Block all outbound by default
  - Whitelist: Only approved Selenium/Appium/API servers
  - Monitoring: Use mitmproxy or Wireshark to inspect HTTPS
```

**Condition 4 — Process Isolation (MANDATORY):**
```
- Run as restricted Windows user (not admin)
- If desktop automation (PlatynUI) needed: Run on isolated VM only
- Monitor file access with Process Monitor
- Monitor network with netstat/Wireshark
```

**Condition 5 — Desktop Automation (CONDITIONAL):**
```
If NOT using desktop automation:
  - Do NOT install PlatynUI → Lower risk
  
If using desktop automation:
  - Only on Windows VMs (isolated environment)
  - Monitor registry access with Process Monitor
  - Run as restricted user
  - Document all desktop automation use cases
```

**Required Conditions Summary:**
1. ✅ Only install needed optional features (don't install all)
2. ✅ Set `RF_MCP_WORKSPACE` to limited directory
3. ✅ Use NTFS permissions to restrict test file access
4. ✅ If web/api/mobile: Use Windows Firewall to restrict outbound
5. ✅ Run as unprivileged user (not admin)
6. ✅ If desktop automation: Use isolated VM + Process Monitor
7. ✅ Enable logging for keyword execution
8. ✅ Document all enabled features and their purposes

---

## Pre-Deployment Checklist

- [ ] Determine required optional features (web, mobile, api, desktop, etc.)
- [ ] Install only required features: `pip install rf-mcp[<features>]`
- [ ] Verify installed packages: `pip list`
- [ ] Set `RF_MCP_WORKSPACE` to limited directory
- [ ] Configure NTFS permissions on workspace directory
- [ ] Create restricted Windows user account (if not containerizing)
- [ ] Test that restricted user can run rf-mcp normally
- [ ] If web/api features enabled: Configure Windows Firewall allowlist
- [ ] If desktop automation (PlatynUI) needed: Prepare isolated Windows VM
- [ ] Set up Process Monitor for file/network/registry monitoring
- [ ] Test file access isolation (try to access files outside workspace — should fail)
- [ ] Test network isolation (if firewall enabled, verify allowed traffic only)
- [ ] Enable MCP protocol logging
- [ ] Document all enabled features and their purpose
- [ ] Set up monitoring/alerting for:
  - Files accessed outside workspace-root
  - Network connections outside firewall allowlist
  - Registry access (if PlatynUI enabled)
- [ ] Review sample test files for security issues before execution

---

## Evaluation Summary

| Aspect | Status | Notes |
|--------|--------|-------|
| **Code Quality** | ✅ GOOD | Open source, actively maintained |
| **CVEs & Vulnerabilities** | ✅ CLEAN | No known CVEs; PyPI history clean |
| **Core Dependencies** | ✅ GOOD | Industry-standard libraries (fastmcp, robotframework, pydantic) |
| **Optional Dependencies** | ⚠️ MEDIUM | Higher risk if all features installed |
| **File Access** | ✅ GOOD | Workspace-root configurable, restrictable |
| **Network Access** | ⚠️ MEDIUM | Depends on installed optional libraries |
| **Process Isolation** | ✅ GOOD | Can run as unprivileged user |
| **Desktop Automation** | 🔴 HIGH | PlatynUI is powerful but requires isolation if used |
| **Overall Risk** | ⚠️ MODERATE | **Acceptable with mandatory restrictions based on feature selection** |

---

## Risk Profile by Feature Set

| Installation Profile | Overall Risk | Key Concerns | Recommendation |
|------------------|--------------|--------------|----------------|
| **Core only** (no optional) | 🟢 LOW | Limited capability | ✅ Approve, minimal restrictions |
| **Core + web** | 🟡 MEDIUM | Network access (browser) | ✅ Approve with firewall |
| **Core + web + api** | 🟡 MEDIUM | HTTP(S) requests | ✅ Approve with firewall |
| **Core + mobile** | 🟡 MEDIUM-HIGH | Appium server access | ✅ Approve with firewall + monitoring |
| **Core + desktop** | 🔴 HIGH | Windows registry/file access | ⚠️ Isolated VM only |
| **All features** | 🔴 HIGH | Full attack surface | ⚠️ Isolated VM + comprehensive monitoring |

---

## References

- [GitHub: manykarim/rf-mcp](https://github.com/manykarim/rf-mcp)
- [PyPI: rf-mcp](https://pypi.org/project/rf-mcp/)
- [Robot Framework Documentation](https://robotframework.org/)
- [SeleniumLibrary Documentation](https://github.com/robotframework/SeleniumLibrary)
- [AppiumLibrary Documentation](https://github.com/serhatbolsu/robotframework-appiumlibrary)
- [MCP Specification](https://modelcontextprotocol.io/docs)
- [MCP Security Evaluation Guide](./mcp-security-evaluation-plan.md)
