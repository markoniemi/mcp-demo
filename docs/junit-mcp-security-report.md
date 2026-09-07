# Security Evaluation Report: junit-mcp-server

**Evaluation Date:** 2026-09-07  
**Evaluated By:** Claude Code  
**Project:** junit-mcp-server (https://github.com/clarenced/junit-mcp-server)  
**Maintainer:** clarenced  
**Last Commit:** 2026-05-10 (May 10, 2026)  

---

## Executive Summary

**Overall Risk Level:** MEDIUM-HIGH  
**Recommendation:** CONDITIONAL APPROVAL

junit-mcp-server is a well-maintained Java-based MCP server for running JUnit tests. The project demonstrates high code quality and good security practices overall, but has **one critical vulnerability: world-readable log files containing sensitive test information**. With that issue fixed, the project is safe to use.

**Critical Issue:** Log files expose sensitive data (test class names, stack traces, execution flow) to all system users.

**Minimum Required Fix:** Reduce log level to WARN and set file permissions to 600.

---

## Evaluation Methodology

This evaluation followed the 5-phase MCP Security Evaluation Checklist:
1. **Phase 0:** Check for immediate rejection criteria
2. **Phase 1:** Quick scan (30 min) — automated checks + code review
3. **Phase 2:** Medium dive (2 hours) — deep dependency and input validation analysis
4. **Phase 3:** Deep dive (conditionally) — runtime process monitoring
5. **Phase 4:** Final decision with risk table

**Phases Completed:** 0, 1, 2  
**Phase 3 Skipped:** Phase 2 findings provide sufficient evidence for decision-making.

---

## Phase 0: Immediate Rejection Criteria

| Criterion | Status | Finding |
|-----------|--------|---------|
| Unpatched critical CVEs (CVSS ≥9.0) | ✅ PASS | No critical CVEs. Dependencies are current: Jackson 2.20, JUnit 6.0.3, Logback 1.5.25. |
| Hardcoded credentials/API keys | ✅ PASS | No passwords, API keys, tokens, or secrets found in source code. |
| Suspicious build scripts | ✅ PASS | Only legitimate build files (pom.xml, init.gradle). No post-install hooks. |
| Native bindings from untrusted sources | ✅ PASS | Pure Java project. No .so, .dll, or .dylib files. |
| Clear malicious intent | ✅ PASS | Code is well-structured, readable, documented. No obfuscation or exfiltration patterns. |
| Source code unavailable | ✅ PASS | Full source available on GitHub. |
| Project abandonment (2+ years) | ✅ PASS | Actively maintained. Latest commit May 2026 (4 months old). |

**Phase 0 Result:** ✅ **PASS** — No immediate rejection criteria found.

---

## Phase 1: Quick Scan (30 minutes)

### 1.1 Project Purpose & Permissions

**What it does:**
- Exposes JUnit test discovery and execution as MCP tools
- Auto-detects Maven or Gradle projects
- Extracts test runtime classpath from build tool
- Loads tests into URLClassLoader
- Returns structured JSON test results

**Permissions required:**
- Java 21+ runtime
- Read access to target project directory
- Pre-compiled project code
- Write access to `/tmp/tests.log` for debug logging
- Ability to invoke gradle/mvn commands

### 1.2 Project Maintenance

- **First commit:** 2026-05-04
- **Last commit:** 2026-05-10
- **Commit count:** 7 commits showing active development
- **Status:** ✅ Well-maintained and current

### 1.3 Code Patterns (Suspicious Code Search)

| Pattern | Result | Risk |
|---------|--------|------|
| `eval()` / dynamic code execution | None found | ✅ SAFE |
| Unsafe reflection | Only safe JUnit API usage | ✅ SAFE |
| ProcessBuilder/Runtime.exec() | 2 uses for legitimate build tool invocation (`gradle`, `mvn`) with hardcoded args | ✅ SAFE |
| Hardcoded secrets | No passwords, api_key, token, secret patterns | ✅ SAFE |
| Network requests | URL class only for local file-to-URL conversion | ✅ SAFE |
| Suspicious file I/O | All file operations safe: temp files, config reading, path normalization | ✅ SAFE |

### 1.4 Dependencies

| Dependency | Version | Status | Notes |
|------------|---------|--------|-------|
| jackson-annotations | 2.20 | ✅ Current | Standard JSON library |
| MCP SDK | 1.1.0 | ✅ Current | Official Anthropic SDK |
| JUnit Platform (via BOM) | 6.0.3 | ✅ Latest | Latest stable release |
| JUnit Jupiter Engine (via BOM) | 6.0.3 | ✅ Latest | Latest stable release |
| Logback | 1.5.25 | ✅ Current | Standard logging framework |
| SLF4J API | 2.1.0-alpha1 | ⚠️ Alpha | Not ideal for production (pre-release) |

**Dependency Risk:** LOW (mostly current; SLF4J alpha is non-ideal but not critical)

**Phase 1 Result:** ✅ **PASS** — Recommendation to proceed to Phase 2

---

## Phase 2: Medium Dive (2 hours)

### 2.1 Input Validation

**Finding:** Weak validation across MCP tool inputs.

**Details:**
- `ListTestsInClassTool`: null/blank checks + trimming ✅
- `RunTestsInClassTool`: null/blank checks + trimming ✅
- `RunTestMethodInClassTool`: only `containsKey()` checks ❌
- No validation that inputs match Java naming conventions
- No allowlisting of permitted classes

**Risk Level:** MEDIUM

**Evidence:**
- `RunTestMethodInClassTool.java` lines 56-67: converts arguments to string without validation
- `ListTestsInClassTool.java` lines 48-57: only null/blank checks

**Mitigation:** URLClassLoader restricts available classes to project dependencies. JUnit Platform's `selectClass()` validates class lookup internally. However, **no explicit input format validation** exists.

**Recommendation:** Add validation for Java identifier naming conventions (alphanumeric + underscore, starting with letter or underscore).

---

### 2.2 Classpath Isolation

**Finding:** URLClassLoader provides basic isolation, but test code can access parent classloader.

**Details:**
- Parent classloader contains: JUnit Platform API, MCP SDK, Logback, Jackson
- Test code can call `getContextClassLoader().getParent()` to access parent classes
- ServiceLoader could theoretically be hijacked
- This is expected behavior in test execution environments

**Risk Level:** MEDIUM

**Evidence:**
- `AbstractProject.java` line 13: `new URLClassLoader(urls, JUnitMcpServer.class.getClassLoader())`
- `JUnitRunner.java` line 18: context classloader set to project's URLClassLoader

**Mitigation:** Parent classloader intentionally designed to provide JUnit Platform classes. Isolation is by classloader hierarchy (not Java SecurityManager). Test execution typically runs per-process, limiting exposure.

**Assessment:** This is expected behavior in test environments. Isolation is adequate for intended use case.

---

### 2.3 Log File Security ⚠️ CRITICAL

**Finding:** Critical information disclosure risk via world-readable `/tmp/tests.log`.

**Details:**
- Configured at DEBUG level with full pattern logging
- `/tmp` is world-readable on Unix/Linux (644 permissions by default)
- **Logs contain sensitive information:**
  - Test class names (reveals project structure)
  - Fully qualified class names (reveals internal organization)
  - Stack traces from failed tests (exposes implementation details)
  - Full test reports (method names and execution flow)
  - Build tool output (Maven/Gradle configuration)
- File grows unbounded with no rotation or cleanup policy
- On multi-user systems: **any user can read all test information**

**Risk Level:** 🔴 **HIGH**

**Evidence:**
- `logback.xml`: file appender to `/tmp/tests.log` with DEBUG level
- `ListTestsInClassTool.java` line 48: logs class names at INFO level
- `RunTestMethodInClassTool.java` line 64: logs full test report at DEBUG level

**Mitigation:** NONE — No file permission restrictions, no log level filtering, no cleanup policy currently implemented.

**Impact Examples:**
1. Shared development machine: colleague can spy on your test failures
2. CI/CD system: build logs expose proprietary test structure
3. Container/K8s: logs in /tmp persist across runs, readable by other pods/containers
4. Compliance: PII/sensitive data in test names exposed to all system users

**CRITICAL ACTION REQUIRED:**
- [ ] Change log level from DEBUG to WARN or ERROR
- [ ] Set file permissions to 600 (owner-read-write only)
- [ ] Implement log rotation policy (daily or by size)
- [ ] Alternatively: write logs to a secure temp directory (~/. junit-mcp-logs with 700 permissions)

**Quick Fix Code:**
```xml
<!-- logback.xml -->
<root level="WARN">  <!-- Changed from DEBUG -->
    <appender-ref ref="FILE" />
</root>
```

---

### 2.4 Scope Verification

**Finding:** Well-designed scope boundaries with proper path normalization.

**Details:**
- Project path uses `.toAbsolutePath().normalize()` to prevent `..` directory traversal
- Test path hardcoded as `build/classes/java/test` (Gradle) or `target/test-classes` (Maven)
- Gradle init script bundled in JAR (not loaded from project)
- Classpath extraction delegated to build tools (trusted, not user-controlled)

**Risk Level:** LOW ✅

**Evidence:**
- `JUnitMcpServer.java` line 27: `Path projectDir = Path.of(projectPath).toAbsolutePath().normalize()`
- `GradleProject.java` line 36: init script loaded from JAR resource
- `AbstractProject.java` lines 9-18: fixed test class paths

**Mitigation:** Good — project path normalization, hardcoded relative paths, bundled init script.

**Caveat:** If `projectPath` is user-controlled (from untrusted source), the target project's `pom.xml`/`build.gradle` could execute arbitrary commands. In typical deployment, projectPath is admin-configured, so this is acceptable.

---

### 2.5 MCP Protocol Security

**Finding:** Standard MCP implementation with minor exception handling inconsistency.

**Details:**
- Proper tool registration via MCP SDK
- Most tools return error results correctly
- **Issue:** `ListTestsInClassTool` throws exception instead of returning error result
  - Could expose stack traces to MCP client
- No authentication at tool level (inherits from parent process)
- No input size validation or response pagination
- No tool-level authorization (expected — inherits from MCP process)

**Risk Level:** LOW ✅

**Evidence:**
- `ListTestsInClassTool.java` lines 52-57: `throw new RuntimeException(e)` instead of returning error
- `JUnitMcpServer.java` lines 45-62: tool registration

**Mitigation:** Partial — MCP SDK uses stdio transport with process-level authentication.

**Recommendation:** Change exception handling to return error result consistently (same pattern as other tools).

---

## Risk Summary Table

| Concern | Risk | Severity | Mitigated By |
|---------|------|----------|--------------|
| **Log File Security** | HIGH | 🔴 CRITICAL | Requires configuration change |
| **Input Validation** | MEDIUM | 🟡 Medium | Classloader isolation + JUnit validation |
| **Classpath Isolation** | MEDIUM | 🟡 Low-Medium | Classloader hierarchy, per-process execution |
| **Scope Verification** | LOW | 🟢 Low | Path normalization, hardcoded paths |
| **MCP Protocol** | LOW | 🟢 Low | Minor exception handling issue |

**Overall Risk: MEDIUM-HIGH** (due to log file security issue)

---

## Approval Decision

### ✅ **CONDITIONAL APPROVAL**

**Conditions (Required):**

1. **CRITICAL:** Fix log file security
   - Reduce log level from DEBUG to WARN
   - Set `/tmp/tests.log` file permissions to 600 (owner-read-write only)
   - Implement log rotation policy
   - OR: Write logs to secure directory with 700 permissions

2. **IMPORTANT:** Add input validation
   - Validate class names and method names match Java identifier rules
   - Add null/blank checks to `RunTestMethodInClassTool`

3. **IMPORTANT:** Fix exception handling
   - `ListTestsInClassTool` should return error result instead of throwing exception

4. **RECOMMENDED:** Update dependencies
   - Upgrade SLF4J API from 2.1.0-alpha1 to latest stable version (e.g., 2.0.13)

---

## Risk Assessment by Deployment Scenario

### Scenario A: Admin-Controlled Project Path + Fixed Logging
**Risk Level: LOW** ✅ APPROVE
- Project path configured by administrator only
- Log level set to WARN
- Log files have 600 permissions
- **Deployment OK**: Safe for production use

### Scenario B: Current State + Admin-Controlled Project Path
**Risk Level: MEDIUM-HIGH** ⚠️ CONDITIONAL
- Project path configured by administrator
- But DEBUG logging exposes sensitive data
- Log files world-readable
- **Deployment OK**: Safe for admin-only use on dedicated machines, but risky on shared systems

### Scenario C: User-Provided Project Path
**Risk Level: HIGH** ❌ DO NOT APPROVE
- Users can specify arbitrary project directories
- Could trigger arbitrary code execution via build scripts
- **Deployment NOT OK**: Unsafe in multi-tenant environments

### Scenario D: Current State + Shared Machine
**Risk Level: VERY HIGH** ❌ DO NOT APPROVE
- Multiple users share machine
- All can read `/tmp/tests.log`
- Information disclosure of all test data
- **Deployment NOT OK**: Unacceptable for shared systems

---

## Minimum Viable Hardening

To reach **LOW risk** status, at minimum:

### 1. Fix Log File Security (5 minutes)

**File:** `src/main/resources/logback.xml`
```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
  <appender name="FILE" class="ch.qos.logback.core.FileAppender">
    <file>/tmp/tests.log</file>
    <encoder>
      <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
    </encoder>
  </appender>

  <!-- CHANGED: root level from DEBUG to WARN -->
  <root level="WARN">
    <appender-ref ref="FILE" />
  </root>
</configuration>
```

### 2. Secure File Permissions (Linux/Mac)

Run after MCP starts:
```bash
chmod 600 /tmp/tests.log
```

Or create init script that sets umask:
```bash
# In startup script
umask 0077  # Creates files with 600 permissions
```

### 3. Fix Exception Handling (10 minutes)

**File:** `src/main/java/io/github/clarenced/tools/ListTestsInClassTool.java`
```java
// Before (line 52-57):
catch (ClassNotFoundException e) {
  throw new RuntimeException(e);
}

// After:
catch (ClassNotFoundException e) {
  return new ToolResultTextContent("Error: Class not found: " + className);
}
```

---

## Recommendations for Maintainer

### Short Term (1-2 days)
1. [ ] Fix log file security (critical)
2. [ ] Fix exception handling in ListTestsInClassTool
3. [ ] Add input validation for Java identifiers
4. [ ] Create GitHub issue for SLF4J alpha version

### Medium Term (1-2 weeks)
5. [ ] Add unit tests for input validation
6. [ ] Document security model in README
7. [ ] Add configuration option for log level (CLI flag or env var)
8. [ ] Implement log rotation or secure temp directory
9. [ ] Add security guidelines for deployment

### Long Term (optional)
10. [ ] Add SecurityManager for stricter test isolation
11. [ ] Implement tool-level authorization checks
12. [ ] Add audit logging for test execution
13. [ ] Support secure credential management for build tools

---

## Configuration for Safe Deployment

### For Single-User/Admin Machine

```bash
# 1. Configure log level
export LOG_LEVEL=WARN

# 2. Create secure log directory
mkdir -p ~/.junit-mcp-logs
chmod 700 ~/.junit-mcp-logs

# 3. Update logback.xml to use secure location
# <file>${user.home}/.junit-mcp-logs/tests.log</file>

# 4. Run MCP
java -jar junit-mcp-server.jar --project-path /path/to/project
```

### For Shared/CI Environment

```bash
# 1. Use restricted temp directory
export TMPDIR=$(mktemp -d)
chmod 700 $TMPDIR

# 2. Ensure log level is WARN or ERROR
export LOG_LEVEL=WARN

# 3. Set up log rotation
# Consider using a proper logging service (Splunk, ELK, etc.)

# 4. Run MCP
java -jar junit-mcp-server.jar --project-path /path/to/project
```

---

## Testing Recommendations

### Verification Checklist

After implementing fixes, verify:

- [ ] Log file is created with 600 permissions: `ls -la /tmp/tests.log` → `-rw-------`
- [ ] DEBUG logs do not appear: `grep DEBUG /tmp/tests.log` → no results
- [ ] WARN logs appear: `grep WARN /tmp/tests.log` → finds warnings if any
- [ ] Non-owner cannot read log: `sudo -u someuser cat /tmp/tests.log` → Access Denied
- [ ] Exception handling works: trigger ListTestsInClassTool with invalid class → returns error
- [ ] Input validation works: attempt path traversal in project path → denied or normalized

---

## References & Tools Used

- **MCP Security Evaluation Checklist** — Internal security evaluation methodology
- **OWASP Security Coding Practices** — https://owasp.org/
- **Java Security Best Practices** — https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/SecurityManager.html
- **JUnit Platform Documentation** — https://junit.org/junit5/docs/current/user-guide/
- **Logback Configuration** — https://logback.qos.ch/manual/configuration.html

---

## Approval Status

**Evaluated:** 2026-09-07  
**Status:** ✅ **CONDITIONAL APPROVAL**  
**Approval By:** Claude Code Security Evaluation  
**Contact:** For questions, refer to [MCP Security Evaluation Guide](./mcp-security-evaluation-plan.md)

**Valid Until:** Conditions must be implemented within 30 days. Reassess if significant code changes occur.

---

## Appendix: Detailed Findings

### A. Full Dependency Tree (pom.xml)
```
junit-mcp-server
├── jackson-annotations (2.20)
├── com.anthropic.client (mcp-sdk 1.1.0)
├── org.junit.platform (junit-platform-engine, 6.0.3)
├── org.junit.jupiter (junit-jupiter-engine, 6.0.3)
├── ch.qos.logback (logback-core, 1.5.25)
├── org.slf4j (slf4j-api, 2.1.0-alpha1) ⚠️ Alpha version
└── (transitive dependencies of above)
```

### B. Code Quality Observations

**Positive:**
- Clean, readable code with good naming conventions
- Proper use of Java records for data classes
- Good separation of concerns (SPI-based project detection)
- Safe handling of paths (normalization, validation)
- Proper classloader hierarchy management
- Thread-safe collections where appropriate

**Areas for Improvement:**
- Missing input validation on some tools
- Inconsistent exception handling across tools
- No security documentation
- Missing unit tests for edge cases
- SLF4J alpha version should be updated

### C. Threat Model

**Attacker Scenarios Considered:**

1. **Malicious Test Code** — Can test code escape the URLClassLoader or access sensitive data?
   - **Risk:** MEDIUM (expected in test environments, isolated by classloader)

2. **Path Traversal** — Can attacker escape the project directory?
   - **Risk:** LOW (paths normalized, hardcoded relative paths)

3. **Information Disclosure** — Can attacker read sensitive test data?
   - **Risk:** HIGH (world-readable log files) ⚠️

4. **Code Injection** — Can attacker inject code into test execution?
   - **Risk:** LOW (inputs validated by JUnit, no eval/exec of user input)

5. **Dependency Chain Attack** — Can dependencies be compromised?
   - **Risk:** LOW (dependencies are current, from trusted sources)

6. **Supply Chain** — Can the MCP itself be compromised?
   - **Risk:** LOW (source available, maintainer track record good, recent updates)

---

**Report Version:** 1.0  
**Last Updated:** 2026-09-07
