# Security Evaluation Report: mcp-server-arangodb

**Evaluation Date:** 2026-09-08  
**Evaluated By:** Claude Code  
**Project:** mcp-server-arangodb (https://github.com/ravenwits/mcp-server-arangodb)  
**Maintainer:** ravenwits  
**Latest Commit:** 2026-02-15 (approximately 7 months old)  

---

## Executive Summary

**Overall Risk Level:** MEDIUM  
**Recommendation:** CONDITIONAL APPROVAL

mcp-server-arangodb is a TypeScript MCP server for ArangoDB database interaction. Project is actively maintained with clean code structure. **Primary security concern: Direct database query execution without built-in parameterization protection.** With proper configuration and access controls, risk is manageable.

**Key Issues:** Database query execution requires careful input validation at MCP client level.

---

## Phase 0: Immediate Rejection Criteria

| Criterion | Status | Finding |
|-----------|--------|---------|
| Unpatched critical CVEs (CVSS ≥9.0) | ✅ PASS | No known critical CVEs |
| Hardcoded credentials/API keys | ✅ PASS | No credentials in source code |
| Post-install scripts making network requests | ✅ PASS | No post-install scripts |
| Native bindings from untrusted sources | ✅ PASS | Pure TypeScript/Node.js, no native bindings |
| Clear malicious intent | ✅ PASS | Code is well-structured and transparent |
| Source code unavailable | ✅ PASS | Public GitHub repository |
| Project abandonment (2+ years) | ✅ PASS | Recent commits (Feb 2026) |

**Phase 0 Result:** ✅ **PASS** — No immediate rejection criteria found.

---

## Phase 1: Quick Scan (30 minutes)

### 1.1 Project Purpose & Permissions

**What it does:**
- Connects to ArangoDB instance via HTTP REST API
- Exposes query execution tools to MCP clients
- Supports AQL (ArangoDB Query Language) queries
- Manages database connections and authentication

**Permissions required:**
- Network access to ArangoDB server (typically HTTP/HTTPS)
- Database credentials (username/password or token)
- Execution of arbitrary AQL queries on configured database

### 1.2 Project Maintenance

- **Repository:** Public on GitHub
- **Last commit:** February 2026 (7 months old)
- **Activity:** Moderate, appears actively maintained
- **Status:** ✅ Maintained, not abandoned

### 1.3 Dependency Analysis

```
mcp-server-arangodb
├── @modelcontextprotocol/sdk (MCP framework)
├── arangojs (ArangoDB JavaScript client library)
└── (minimal additional dependencies)
```

**Assessment:**
- ✅ MCP SDK: Official Anthropic SDK, well-maintained
- ✅ arangojs: Official ArangoDB Node.js client, actively maintained
- ✅ Minimal dependencies reduce attack surface

### 1.4 Code Patterns (Suspicious Code Search)

| Pattern | Result | Risk |
|---------|--------|------|
| Hardcoded secrets | None found | ✅ SAFE |
| Eval/dynamic code execution | None found | ✅ SAFE |
| Child process execution | None found | ✅ SAFE |
| Network requests (unvalidated) | Only ArangoDB server | ✅ SAFE |
| SQL/AQL injection patterns | Potential risk | ⚠️ REVIEW |

**Phase 1 Result:** ✅ **PASS** — Proceed to Phase 2

---

## Phase 2: Medium Dive (2 hours)

### 2.1 AQL Query Execution Risk

**Finding:** Direct AQL query execution without client-side parameterization.

**Details:**
- Tools accept user-provided AQL query strings
- Queries passed directly to arangojs without escaping
- No built-in query parameter binding at MCP level
- Risk depends on MCP client's input validation

**Risk Level:** MEDIUM

**Evidence:**
- Tool definitions allow arbitrary AQL strings
- arangojs itself supports parameterized queries, but MCP server doesn't enforce usage

**Mitigation:**
- ✅ arangojs has built-in parameterization support (binding variables)
- ✅ ArangoDB server validates AQL syntax
- ❌ No client-side validation at MCP tool level

**Recommendation:** Document requirement for parameterized queries in README.

---

### 2.2 Database Credentials & Connection Security

**Finding:** Good credential handling with environment variable support.

**Details:**
- Database URL configurable via environment variables
- Credentials not embedded in code
- arangojs handles HTTPS/TLS to ArangoDB
- Connection pooling via arangojs

**Risk Level:** LOW

**Mitigation:** ✅ Environment-based configuration, no hardcoded secrets

---

### 2.3 Network Security

**Finding:** Outbound HTTPS connection to ArangoDB server only.

**Details:**
- Only connects to configured ArangoDB instance
- Uses arangojs HTTPS support for TLS encryption
- No external network requests (CDN, telemetry, etc.)
- Firewall can restrict to ArangoDB server IP

**Risk Level:** LOW

**Evidence:**
- Network calls limited to arangojs → ArangoDB interaction
- No additional HTTP libraries or external API calls

**Mitigation:** ✅ Firewall can restrict to specific ArangoDB server

---

### 2.4 File System Access

**Finding:** Minimal file system access.

**Details:**
- No file reading/writing by design
- Configuration from environment variables only
- No temporary files or logging to disk

**Risk Level:** LOW ✅

---

### 2.5 MCP Protocol Implementation

**Finding:** Standard MCP implementation with proper tool registration.

**Details:**
- Follows MCP SDK patterns
- Error handling returns MCP error results
- No exception leakage to client
- Tool inputs validated by SDK

**Risk Level:** LOW ✅

---

## Phase 3: Deep Dive (Not Needed)

Phase 2 identified medium-level risk (AQL injection) but no blockers. Risk is manageable with configuration controls and client-side validation.

---

## Phase 4: Decision

| Criteria | Result | Notes |
|----------|--------|-------|
| Red flags found | ❌ None | All Phase 0 checks passed |
| Unpatched CVEs | ❌ None | No known vulnerabilities |
| Dependencies clean | ✅ Yes | Official MCP SDK and arangojs |
| AQL injection risk | ⚠️ Medium | Mitigatable with parameterized queries |
| Credentials secure | ✅ Yes | Environment-based config |
| Network access | ✅ Restricted | ArangoDB server only |

**APPROVAL RECOMMENDATION: ✅ CONDITIONAL APPROVAL**

---

## Phase 5: Restrictions & Conditions

**Condition 1 — AQL Query Parameterization (MANDATORY):**
- Require all AQL queries use parameterized binding (`:param` syntax)
- Document in README how to use query variables
- Example: `FOR doc IN collection FILTER doc.id == @id RETURN doc` with `bindVars: { id: userInput }`

**Condition 2 — Database Access Control (MANDATORY):**
- ArangoDB user account must have minimal required privileges
- Restrict to specific databases and collections (not system collections)
- Use ArangoDB role-based access control (RBAC)

**Condition 3 — Network Isolation (RECOMMENDED):**
- Firewall restrict to specific ArangoDB server IP/port
- Use HTTPS-only connection (no plaintext HTTP)
- Disable ArangoDB's default anonymous access

**Condition 4 — Connection Limits (RECOMMENDED):**
- Configure connection pool size to prevent resource exhaustion
- Set connection timeout to prevent hanging queries
- Monitor active connections

---

## Recommendations for MCP Users

1. **Validate all inputs before passing to AQL queries**
   - Never pass raw user input directly to query strings
   - Use parameterized query binding exclusively

2. **Restrict ArangoDB user privileges**
   - Create dedicated read-only account if possible
   - Limit to specific collections/databases
   - Disable admin operations

3. **Monitor database access**
   - Enable ArangoDB audit logging
   - Alert on unusual query patterns
   - Track tool invocations

4. **Test query isolation**
   - Verify queries cannot access unintended collections
   - Test access control boundaries
   - Validate parameterization works as expected

---

## Risk Summary

| Concern | Severity | Mitigated |
|---------|----------|-----------|
| AQL Injection | MEDIUM | Via parameterized queries |
| Database Exposure | MEDIUM | Via RBAC configuration |
| Credential Compromise | LOW | Environment variables |
| Resource Exhaustion | LOW | Connection pooling + timeouts |

**Overall Assessment:** MEDIUM risk, manageable with proper configuration.

---

## Approval Status

**Status:** ✅ **CONDITIONAL APPROVAL**  
**Conditions:** AQL parameterization, database access control, network isolation  
**Valid Until:** Implementation of conditions within 30 days

---

## References

- [GitHub: ravenwits/mcp-server-arangodb](https://github.com/ravenwits/mcp-server-arangodb)
- [ArangoDB Documentation](https://www.arangodb.com/docs/)
- [arangojs Documentation](https://github.com/arangodb/arangojs)
- [MCP Specification](https://modelcontextprotocol.io/docs)
- [MCP Security Evaluation Guide](./mcp-security-evaluation-plan.md)

---

**Report Version:** 1.0  
**Last Updated:** 2026-09-08
