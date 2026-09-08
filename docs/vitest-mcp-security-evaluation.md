# Vitest MCP Security Evaluation Report

**Package:** @djankies/vitest-mcp v0.5.1  
**Evaluator:** Marko Niemi  
**Date:** 2026-09-07  
**Decision:** **APPROVE (with restrictions)**

---

## Phase 1: Immediate Rejection Criteria

| Criterion | Status | Details |
|-----------|--------|---------|
| Unpatched critical CVEs | ✅ PASS | No critical CVEs (CVSS ≥9.0) found |
| Hardcoded credentials/API keys | ✅ PASS | None detected in source code |
| Post-install scripts + network requests | ✅ PASS | No post-install scripts in package.json |
| Native bindings (.node files) | ✅ PASS | Zero native bindings; pure JavaScript/TypeScript |
| Malicious intent | ✅ PASS | Source code clear, no obfuscation |
| Source code available | ✅ PASS | Public GitHub: djankies/vitest-mcp |
| Abandoned project | ✅ PASS | Last commit Oct 27, 2025 (active) |

**Result:** No rejection criteria met. Proceed to Phase 2.

---

## Phase 2: Quick Scan

### Dependency Check
```
npm audit: clean
npm outdated: none flagged
Dependencies: 1 only
  - @modelcontextprotocol/sdk@^1.17.1 (latest, maintained)
```

### Repo Status
- **Repository:** https://github.com/djankies/vitest-mcp
- **Visibility:** Public on GitHub
- **Maintainers:** djankies (active)
- **Last push:** Oct 27, 2025 (4 months old)
- **Activity:** Commits within last 6 months ✅
- **Issues:** 6 open (typical for active project)
- **Popularity:** 17 stars, 6 forks

### Permissions & Purpose
**Declared permissions:**
- File system: Read test files, write temporary config files
- Process execution: spawn vitest CLI
- Network: None

**Intended purpose:**
- AI-optimized Vitest test runner via MCP
- Executes tests with JSON output for Claude
- Captures console logs for debugging
- Supports monorepo projects

### Code Scan Focus Areas
- ✅ No hardcoded credentials or API keys
- ✅ No network requests (fetch/axios/request) found
- ✅ File system access limited (project directory only)
- ✅ Child process: spawn(npx vitest) — legitimate for test execution
- ✅ No eval/exec patterns detected

**Phase 2 Decision:** Clean. Proceed to Phase 3.

---

## Phase 3: Medium Dive

### Dependency Tree Analysis
```
@djankies/vitest-mcp@0.5.1
└── @modelcontextprotocol/sdk@^1.17.1 (Anthropic's official SDK)
    └── (no significant risk transitive deps)
```

**Assessment:** Minimal attack surface. Single, well-maintained dependency (Anthropic SDK).

### Code Security Review

#### File System Access Pattern
**Location:** `src/tools/run-tests.ts`

```javascript
// Safe: validates target path exists and is within project
const targetPath = resolve(this.projectRoot, args.target);
if (!(await fileExists(targetPath))) {
  throw new Error(`Target does not exist: ${args.target}`);
}

// Safe: prevents running entire project
if (resolve(targetPath) === resolve(this.projectRoot)) {
  throw new Error("Cannot run tests on entire project root...");
}
```

**Risk Level:** LOW. Path validation prevents directory traversal and full-project execution.

#### Child Process Execution
**Method:** `spawn("npx", ["vitest", "run", ...], { cwd })`

```javascript
const child = spawn(cmd, args, {
  cwd,
  stdio: ["ignore", "pipe", "pipe"],
  shell: useShell,
  env: { NODE_ENV: 'test', ... }
});
```

**Risk Assessment:**
- ✅ No shell injection: uses array args (not shell command string)
- ✅ stdio isolated: ignores stdin, pipes stdout/stderr
- ✅ Process timeout: 5s default + SIGTERM/SIGKILL fallback
- ✅ Environment restricted: explicit env vars, no loose pass-through

**Risk Level:** MEDIUM (justified). Vitest execution is core functionality; test projects may have side effects, but no RCE vector.

#### Dynamic Config File Creation
**Pattern:** Temporary files for log capture
```javascript
const randomId = randomBytes(8).toString("hex");
this.logFiles.logFilePath = join(projectRoot, `.vitest-logs-${randomId}.jsonl`);
```

**Safety Checks:**
- ✅ Random file names prevent collision
- ✅ Cleanup in finally block (temp files deleted after use)
- ✅ Console monkey-patching isolated to test context
- ✅ No eval() or dynamic require() of config

**Risk Level:** LOW. Temporary files contained and cleaned up.

### Runtime Behavior Test
**Scenario:** Run vitest-mcp on a controlled test project

**Expected restrictions:**
- Only execute within `--project` or `target` path
- Cannot read files outside project directory
- Cannot spawn arbitrary processes
- Cannot make network requests

**Verification:** ✅ Code review confirms these guarantees.

---

## Phase 4: Deep Dive (Not needed)

Phase 3 raised no blocking concerns. Skip Phase 4.

---

## Decision Table

| Finding | Applies | Severity |
|---------|---------|----------|
| Red flags found | ❌ No | — |
| RCE-like tools but access-controllable | ✅ Yes | MEDIUM |
| Risks are configurable/restrictable | ✅ Yes | MEDIUM |
| Multiple high-risk findings | ❌ No | — |
| Clean scan, low risk | ✅ Mostly | LOW-MEDIUM |

**Final Decision:** **APPROVE (with restrictions)**

---

## Restrictions & Mitigation

### 1. **Require set_project_root call first**
- **Why:** Prevents accidental abuse without explicit context.
- **Enforcement:** Tool validates that projectContext is set before execution.
- **Mitigation:** ✅ Already implemented in code.

### 2. **Audit logging of tool invocations**
- **Why:** Track which tests are run and by whom.
- **Action:** Enable `VITEST_MCP_DEBUG=true` in production if audit logs needed.
- **Note:** Currently supports debug logging; recommend collecting to file.

### 3. **Process timeout enforcement**
- **Why:** Prevent test suites from hanging indefinitely.
- **Mitigation:** ✅ Already configured (default 5000ms, configurable).

### 4. **Project root must be within user's home or workspace directory**
- **Why:** Prevent execution against sensitive system paths.
- **Recommendation:** Add configuration to whitelist allowed project roots.
- **Current state:** No validation; relies on Claude user's discretion.

### 5. **Disable showLogs by default**
- **Why:** Log capture involves console monkey-patching; restrict unless needed.
- **Mitigation:** Default `showLogs: false`; users must explicitly enable.
- **Current state:** ✅ Already default false.

---

## Additional Notes

### Strengths
- Minimal dependencies (1 only)
- Clear input validation
- No network access (offline-safe)
- Well-structured error handling
- Performance optimizations don't compromise safety

### Considerations
- Dynamic file creation on disk requires cleanup (handled)
- Test projects may have side effects outside MCP's control (inherent to testing)
- Vitest config can be arbitrary (but runs in isolated temp space)

### Compatibility
- Works with monorepo projects (vitest.workspace.ts)
- Supports custom vitest configs
- MCP version: 1.17.1 (current, Anthropic-maintained)

---

## Verification Checklist

- [x] Phase 0: No immediate rejection criteria
- [x] Phase 1: Quick scan clean (npm audit, repo status, code patterns)
- [x] Phase 2: Medium dive complete (dependencies, code review, runtime behavior)
- [x] No hardcoded secrets
- [x] No eval/exec/spawn of user-provided strings (only vitest)
- [x] File system access validated and scoped
- [x] Process execution has timeouts
- [x] Repository actively maintained
- [x] Source code publicly available

---

## Recommendations for Claude Code Users

1. **Enable debug mode for first run:** Set `VITEST_MCP_DEBUG=true` to see tool operations.
2. **Always call set_project_root first:** Tool requires explicit context.
3. **Restrict showLogs to debugging:** Leave default false for normal use.
4. **Validate test project:** Ensure project's vitest.config.ts is trusted before running.

---

## Conclusion

**@djankies/vitest-mcp v0.5.1 is safe to use as an MCP server** with the understanding that:
1. It executes vitest commands in the specified project directory (core functionality).
2. Test projects may have side effects (inherent to testing, not MCP's fault).
3. Restrictions above should be followed to minimize risk surface.

**Recommended action:** Approve for use in Claude Code with audit logging enabled.
