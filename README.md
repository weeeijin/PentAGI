# WIC2007 — Human-AI Penetration Testing Assessment
> AI-assisted web application security audit and defensive hardening of DVWA v1.10

## Overview

This project documents a full penetration testing cycle conducted on DVWA (Damn Vulnerable Web Application) 
using PentAGI — an autonomous AI penetration testing agent — combined with human oversight for validation, 
logic auditing, and defensive remediation.

The assessment achieved **100% task completion across 11 vulnerability subtasks**, reducing exploitation 
effectiveness from **100% to 0%** across all three critical vulnerability classes after patching.

## Tech Stack

| Component | Tool |
|---|---|
| Target Application | DVWA v1.10 |
| AI Penetration Agent | PentAGI (Qwen2.5 72B) |
| Knowledge Graph | Neo4j |
| Recon Tools | nikto, curl, whatweb |
| Environment | Docker (containerized DVWA) |

## Vulnerabilities Identified & Patched

| Vulnerability | Severity | Status |
|---|---|---|
| SQL Injection | Critical | ✅ Patched |
| Reflected XSS | High | ✅ Patched |
| Command Injection | Critical | ✅ Patched |
| Directory Indexing | Medium | ✅ Confirmed |
| Credential Exposure | Critical | ✅ Confirmed |
| Missing Security Headers | Low | ✅ Confirmed |

## Methodology

### Phase 1 — AI-Driven Reconnaissance
PentAGI autonomously conducted reconnaissance using nikto, curl, and whatweb, 
populating a Neo4j knowledge graph with discovered assets, services, and vulnerabilities.

### Phase 2 — Exploitation
The agent and human tester executed targeted exploits against identified vulnerabilities:

**SQL Injection**
```php
// Payload
1' OR '1'='1
// Result: Full user database dump
```

**Command Injection**
```bash
# Payload
127.0.0.1; cat /etc/passwd
# Result: Full /etc/passwd file exposed
```

**Reflected XSS**
```html
<!-- Payload -->
<script>alert(1)</script>
<!-- Result: JavaScript executed in browser -->
```

### Phase 3 — Defensive Hardening

**SQL Injection Fix — Prepared Statements**
```php
$stmt = mysqli_prepare($conn, "SELECT first_name, last_name FROM users WHERE user_id = ?");
mysqli_stmt_bind_param($stmt, 's', $id);
mysqli_stmt_execute($stmt);
```

**XSS Fix — Output Encoding**
```php
$name = htmlspecialchars($_GET['name'], ENT_QUOTES, 'UTF-8');
```

**Command Injection Fix — Input Validation + Escaping**
```php
if(filter_var($target, FILTER_VALIDATE_IP)) {
    $cmd = shell_exec('ping -c 4 ' . escapeshellarg($target));
}
```

### Phase 4 — Logic Audit
Human oversight identified the following AI agent issues via Neo4j Cypher queries:

| Issue | Type | Severity |
|---|---|---|
| Repeated reconnaissance 3x unnecessarily | Logic Error | High |
| Agent switched to Chinese/Russian mid-session | Logic Error | Medium |
| SQL Injection reported without actual testing | Hallucination | High |
| Wrong target IP on initial attempt | Logic Error | Medium |

## Results

| Phase | SQL Injection | XSS | Command Injection | Overall |
|---|---|---|---|---|
| Before Patching | 100% | 100% | 100% | 100% |
| After Patching | 0% | 0% | 0% | 0% |

## Key Takeaways

- AI agents are effective for automated reconnaissance and initial vulnerability discovery
- Human oversight is critical to catch hallucinations — the agent reported SQL Injection 
  without actually testing it (confidence: 0.40, source: agent_hypothesis)
- Prepared statements, output encoding, and input validation are the three most effective 
  defenses against OWASP Top 10 injection-class vulnerabilities
- Neo4j knowledge graphs are a powerful structure for modelling attack paths, 
  pivot points, and agent logic auditing in automated pentests

## Reports

| Report | Description |
|---|---|
| `defense_verification_report.pdf` | Vulnerability patches with before/after proof |
| `logic_audit_report.pdf` | Neo4j Cypher queries for AI logic and hallucination auditing |

## Course Info

**Course:** WIC2007 Cybersecurity — Alternative Assessment  
**Institution:** Universiti Malaya  
**Session:** Academic Session 2025/2026 Semester 2  
**Student:** Tai Jin Wei (23080642)
