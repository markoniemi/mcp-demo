# MCP Security Evaluation Checklist

## Phase 1

1. Unpatched critical CVEs (CVSS ≥9.0) with no fix
2. Hardcoded credentials or API keys in source code
3. Post-install scripts making network requests
4. Native bindings (.node files) from untrusted sources
5. Clear malicious intent (obfuscation, credential theft, exfiltration)
6. No source code available (closed source)
7. Project abandoned 2+ years with open security issues

**Action:** If any are true, reject and stop.

---

## Phase 2

1. Use **package-scan.vercel.app** for automated dependency analysis (no clone needed):
   - Navigate to https://package-scan.vercel.app
   - Enter package name (e.g., `@djankies/vitest-mcp`)
   - Review: Vulnerabilities count, license, stability metrics
2. Verify: Is repo public on GitHub? Actively maintained (commits last 6 months)?
   - Check last commit date from PackSage or npm registry
   - Confirm GitHub URL is accessible
3. Read: README for permissions and security explanation
4. Analyze source code (from PackSage display or GitHub), focusing on:
   - Network requests (fetch, axios, request)
   - File system access (fs, path)
   - Child process execution (child_process)
   - Native bindings (.node files)
5. **AI Prompt:** "Review these dependency and security findings: [paste PackSage output]"

**Decision:** If nothing concerning, proceed to Phase 3. If suspicious, investigate deeper.

---

## Phase 3

**Dependencies:**

1. Run: `npm ls --all` (view full dependency tree)
2. Run: `npm outdated` (check for deprecated packages)
3. Run: `Get-ChildItem node_modules -Recurse -Filter *.node` (find native bindings)
4. Search for code execution patterns:
   ```powershell
   Get-ChildItem -Path src -Recurse -Include *.js,*.ts | Select-String -Pattern "eval|exec|spawn|child_process"
   ```

**Runtime behavior:**

5. Create test files in your home directory
6. Run the MCP and verify it can only access declared directories
7. Try accessing sensitive files (.ssh, .aws): should fail
8. **AI Prompt:** "Analyze this dependency tree for suspicious or risky packages: [paste npm ls output]"
9. **AI Prompt:** "Review this code for security vulnerabilities: [paste code sections]"

**Decision:** If all looks good, proceed to Phase 5 only if needed. Otherwise, document findings.

---

## Phase 4 (only if Phase 3 raised concerns)

**Process monitoring:**

1. Download **Process Monitor** (Sysinternals from microsoft.com)
2. Run as admin, filter by MCP process name
3. Watch for:
   - Unexpected file reads/writes
   - Registry access
   - Process spawning
4. Check network connections:
   ```powershell
   netstat -ano | findstr node.exe
   ```
5. **AI Prompt:** "I captured this Process Monitor output from the MCP. What security concerns do you see? [paste output]"

**Code review:**

6. Read source code carefully for:
   - Error handling (do errors leak paths or secrets?)
   - Permission model (least privilege or overpermissive?)
   - Transitive dependencies (audit what dependencies pull in)

---

## Phase 5: Document Findings

Create a security review document with:

1. **What was checked:** Which phases completed, which tools used
2. **What was found:** Summary of findings (or "no issues")
3. **Restrictions needed (if any):**
   - Firewall rules (if network access needed)
   - Config flags to enable/disable features
   - OS permissions to restrict
   - User account requirements
   - Monitoring requirements

---

## Common Commands Reference

**Dependency checks:**
```powershell
npm audit --audit-level=moderate
npm outdated
npm ls --all
Get-ChildItem node_modules -Recurse -Filter *.node
```

**Search for risky patterns:**
```powershell
# Code execution patterns
Get-ChildItem -Path src -Recurse -Include *.js,*.ts | Select-String -Pattern "eval|exec|spawn|child_process"

# Hardcoded secrets
Get-ChildItem -Path src -Recurse -Include *.js,*.ts,*.json | Select-String -Pattern "password|secret|api_key|token"
```

**Network/process inspection:**
```powershell
netstat -ano | findstr node.exe
```

**Test file access (as restricted user):**
```powershell
runas /user:mcp_user "type C:\Windows\System32\config\sam"
# Should fail: "Access Denied"
```
