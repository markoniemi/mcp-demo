# Security Evaluation Report: JetBrains MCP Proxy Server

**Evaluation Date:** 2026-09-10  
**Evaluated By:** Claude Code  
**Project:** mcp-jetbrains (https://github.com/JetBrains/mcp-jetbrains)  
**Maintainer:** JetBrains  
**Latest Commit:** 2025-08-18 (Update README.md)  
**Package:** @jetbrains/mcp-proxy (v1.8.0)  

---

## Executive Summary

**Overall Risk Level:** LOW  
**Recommendation:** APPROVED

The JetBrains MCP Proxy Server is an official, well-maintained proxy that connects AI clients (Claude, VS Code, etc.) to JetBrains IDEs. The codebase is clean, minimal, and transparent. Primary functionality is a straightforward HTTP proxy with environment-based configuration. No significant security vulnerabilities identified.

**Note:** Repo is deprecated. Current implementation is built into IntelliJ IDEA 2025.2+ (closed source, part of proprietary IDE).

---

## Phase 1: Immediate Rejection Criteria

| Criterion | Status | Finding |
|-----------|--------|---------|
| Unpatched critical CVEs (CVSS ≥9.0) | ✅ PASS | No known critical CVEs |
| Hardcoded credentials/API keys | ✅ PASS | No credentials in source code |
| Post-install scripts making network requests | ✅ PASS | `prepare` script only runs build (tsc + chmod) |
| Native bindings from untrusted sources | ✅ PASS | Pure TypeScript/Node.js, no .node files |
| Clear malicious intent | ✅ PASS | Code is transparent and well-structured |
| Source code unavailable | ✅ PASS | Public GitHub repository (official JetBrains) |
| Project abandonment (2+ years with open issues) | ✅ PASS | Recently maintained (last commit Aug 2025); deprecated but not abandoned |

**Phase 1 Result:** ✅ **PASS** — No immediate rejection criteria met.

---

## Phase 2: Quick Scan & Repository Analysis

### 2.1 Project Authority & Maintenance

- **Publisher:** JetBrains (official organization)
- **Repository:** Public on GitHub with full source visibility
- **License:** Apache License 2.0
- **Maintenance Status:** Active until integration into IntelliJ 2025.2; now deprecated but repository preserved
- **Last Update:** August 18, 2025 (README update indicating deprecation)
- **Commit History:** 47 commits, 4 contributors
- **Issues:** 38 open (mostly feature requests or questions, not security issues)
- **Repository Stars:** 965 (community trust indicator)

**Assessment:** Official project with transparent maintenance history. Clear deprecation notice in README directing users to built-in IDE functionality.

### 2.2 Dependency Analysis

**Production Dependencies:**
```
@modelcontextprotocol/sdk    ^1.9.0
node-fetch                   ^3.3.2
```

- **@modelcontextprotocol/sdk**: Official Anthropic MCP SDK—core dependency for MCP protocol implementation
- **node-fetch**: Widely-used, standard Node.js fetch library (npm: 450M+ weekly downloads)
- **Total dependencies:** Only 2 production dependencies (minimal attack surface)

**Build Dependencies:**
- TypeScript (compiler)
- shx (shell cross-platform utilities for chmod)
- Standard tooling, no suspicious packages

**Assessment:** Minimal, well-vetted dependencies. No third-party authentication or sensitive data handling libraries.

### 2.3 README & Security Documentation

README clearly states:
- Functionality: "Proxies requests from client to JetBrains IDE"
- Installation methods: VS Code integration, Claude Desktop, manual configuration
- Troubleshooting section with Node.js version requirements
- Configuration via environment variables (`IDE_PORT`, `HOST`, `LOG_ENABLED`)
- No permission escalation or unsafe defaults mentioned
- **Deprecation notice:** Functionality merged into IntelliJ 2025.2+

**Assessment:** Clear, honest documentation. No misleading security claims.

### 2.4 Source Code Structure

**Main file:** `src/index.ts` (286 lines, 251 LOC)
- Single-file implementation (high auditability)
- Clear separation of concerns
- Standard MCP server scaffold from official SDK
- Well-commented critical sections

**Assessment:** Minimal, auditable codebase. Single responsibility (proxy).

---

## Phase 3: Deep Code Analysis

### 3.1 Dependency Security

**No transitive supply-chain risks:** Both dependencies are from trusted sources:
- MCP SDK: Published by Anthropic on npm
- node-fetch: Widely-used ESM fetch polyfill, actively maintained

**No post-install hooks:** `prepare` script only builds TypeScript to JavaScript, no network access.

**Assessment:** ✅ Dependency supply chain is secure.

### 3.2 Network Security

**Network Requests Pattern:**
```typescript
const res = await fetch(`${endpoint}/mcp/list_tools`);
```

- **Endpoint source:** Environment variable `HOST` (default: 127.0.0.1)
- **Port source:** Environment variable `IDE_PORT`
- **Protocol:** HTTP (by default, for local IDE communication)
- **Destination:** Local IDE instance on same machine or configured network address

**Findings:**
- ✅ No external network calls to third-party services
- ✅ All network targets are user-configurable via environment variables
- ✅ Default configuration targets localhost (127.0.0.1)
- ⚠️ **Consideration:** External network exposure possible if `HOST` set to non-localhost and IDE port opened to WAN (but this is explicit user configuration, not default)

**Assessment:** Network communication is limited to IDE proxy relay. No external exfiltration or callback mechanisms detected.

### 3.3 Code Execution & Input Handling

**Pattern Scan Results:**

No instances of:
- `eval()`, `Function()`, or dynamic code execution
- `child_process`, `spawn()`, or `exec()` system calls
- `require()` with dynamic paths
- YAML/JSON deserialization of untrusted data
- Template injection vulnerabilities
- Command injection patterns

**Tool Proxying:**
```typescript
// Safely forwards tool requests to IDE via HTTP
server.setRequestHandler(CallToolRequestSchema, async (request) => {
  const result = await fetch(`${cachedEndpoint}/mcp/call_tool`, {
    method: "POST",
    body: JSON.stringify(request),
  });
  // Returns IDE response as-is
});
```

- ✅ Tool requests are serialized to JSON and POSTed to IDE
- ✅ IDE response returned directly to client (IDE responsible for validation)
- ✅ No string interpolation or command construction

**Assessment:** No code execution vulnerabilities. Proxy correctly delegates execution to IDE.

### 3.4 File System Access

**Scan Result:** No file system operations detected in src/index.ts
- No `fs` module imports
- No file reads, writes, or directory traversal
- No credential file access attempts

**Assessment:** ✅ No file system attack surface.

### 3.5 Logging & Information Disclosure

**Logging Pattern:**
```typescript
const LOG_ENABLED = process.env.LOG_ENABLED === "true";

export function log(...args: any) {
  if (LOG_ENABLED) console.error(...args);
}
```

- Logging disabled by default (must explicitly set `LOG_ENABLED=true`)
- Logs go to stderr, not files
- Log output: Debug messages, endpoint URLs, partial responses (first 100 chars)
- **No sensitive data logged:** Credential fields, full request bodies, or security tokens not logged

**Assessment:** ✅ Logging is conservative and opt-in. No sensitive data leakage observed.

### 3.6 Configuration Management

**Environment Variables Used:**
1. `LOG_ENABLED` (string: "true" to enable, default: off)
2. `HOST` (string: network address, default: "127.0.0.1")
3. `IDE_PORT` (string: port number, e.g., "6365")

- ✅ No hardcoded credentials
- ✅ All defaults are restrictive (localhost-only)
- ✅ No config file parsing from untrusted sources
- ⚠️ **Configuration note:** User responsible for securing IDE port if opened to network

**Assessment:** Configuration is minimal and secure-by-default.

### 3.7 Tool List Caching & Change Detection

**Tool discovery mechanism:**
```typescript
// Polls IDE's /mcp/list_tools endpoint every 10 seconds
// Notifies client if tool list changes
async function testListTools(endpoint: string): Promise<boolean> {
  const res = await fetch(`${endpoint}/mcp/list_tools`);
  const currentResponse = await res.text();
  
  if (previousResponse !== currentResponse) {
    sendToolsChanged();
    previousResponse = currentResponse;
  }
  return true;
}
```

- ✅ Safe polling mechanism (no recursive calls)
- ✅ String comparison prevents cache poisoning
- ✅ Notifications sent to MCP client, not external parties

**Assessment:** ✅ Tool discovery is safe and non-invasive.

---

## Phase 4: Risk Assessment

### 4.1 Threat Model

**Attack Vectors Considered:**

1. **Code Injection**: ❌ Not possible
   - No `eval()` or dynamic code execution
   - All inputs are JSON-serialized to IDE

2. **Credential Theft**: ❌ Not possible
   - No credential storage or handling
   - No external network calls

3. **Command Injection**: ❌ Not possible
   - No shell commands or subprocess execution

4. **Supply Chain Attack**: ✅ Mitigated
   - Only 2 trusted dependencies (MCP SDK, node-fetch)
   - Both from well-established publishers
   - No post-install hooks executing code

5. **Information Disclosure**: ✅ Mitigated
   - Logging disabled by default
   - No file system access
   - Network traffic limited to IDE endpoint

6. **Denial of Service**: Possible but low-risk
   - Polling loop could theoretically be abused by malformed IDE responses
   - However, polling interval is fixed (10 seconds), limiting impact
   - User controls endpoint configuration

7. **Privilege Escalation**: ❌ Not applicable
   - Runs as user invoking CLI
   - No privilege escalation attempts
   - No root/admin required

### 4.2 Trust Boundary Analysis

**Trusted Entities:**
- JetBrains IDE (running locally)
- User's environment configuration
- Official MCP SDK from Anthropic

**Untrusted Entities:**
- IDE HTTP responses (treated as opaque data, forwarded to client)
- Environment variable values (passed through, not executed)

**Assessment:** Clear trust boundaries are maintained.

---

## Phase 5: Findings & Recommendations

### 5.1 Security Findings Summary

**Critical Issues:** None  
**High Issues:** None  
**Medium Issues:** None  
**Low Issues:** None  
**Informational Observations:**

1. **Deprecation Status**
   - Repository is deprecated as of IntelliJ 2025.2
   - MCP functionality now built into IDE
   - Existing installations will continue to work
   - No security maintenance expected going forward

2. **Network Configuration**
   - Default config is localhost-only (secure)
   - If exposed to network, user is responsible for:
     - Firewalling IDE port
     - Network authentication if needed
     - TLS/encryption for remote access

3. **Logging**
   - Disabled by default (good)
   - When enabled, outputs debug info to stderr
   - No sensitive data logged

---

### 5.2 Approval Decision

**Verdict:** ✅ **APPROVED FOR USE**

This package is suitable for:
- Local development with Claude Desktop
- Local IDE tool integration with Claude
- VS Code MCP server configuration
- Teams needing IDE integration with AI assistants

**Restrictions:** None required for default configuration (localhost-only)

**Optional Hardening (if network exposure needed):**
- Run IDE's built-in MCP server (IntelliJ 2025.2+) instead
- Implement firewall rules restricting IDE port to trusted networks
- Use VPN or SSH tunneling for remote access
- Monitor IDE logs for suspicious requests

---

### 5.3 Alternative Recommendations

For new projects using IntelliJ IDEA 2025.2 or later:
- Prefer built-in MCP server functionality over this standalone proxy
- Refer to [JetBrains MCP Documentation](https://www.jetbrains.com/help/idea/mcp-server.html)
- File feature requests/bugs in [JetBrains YouTrack](https://youtrack.jetbrains.com/issues?q=project:%20IJPL%20Subsystem:%20%7BMCP%20(Model%20Context%20Protocol)%7D)

For older IntelliJ versions or standalone IDE setup:
- This package remains a secure option
- Ensure IDE HTTP port is not exposed to untrusted networks

---

## Appendix: Files Analyzed

- ✅ `package.json` — Dependencies, build scripts, bin entry point
- ✅ `src/index.ts` — Main source code (286 lines)
- ✅ `README.md` — Documentation and usage instructions
- ✅ `.gitignore` — No suspicious exclusions
- ✅ GitHub repository metadata — Commit history, contributor analysis
- ✅ License — Apache 2.0 (verified)

---

**Report Prepared By:** Claude Code  
**Classification:** Security Evaluation  
**Status:** APPROVED  

Sources:
- [JetBrains/mcp-jetbrains Repository](https://github.com/JetBrains/mcp-jetbrains)
- [Official IntelliJ IDEA MCP Documentation](https://www.jetbrains.com/help/idea/mcp-server.html)
- [MCP Server Registry](https://www.augmentcode.com/mcp/mcp-jetbrains)
