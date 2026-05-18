# Rules of Engagement (RoE) Document

**Course:** WIC2007 Cyber Security — Alternative Assessment  
**Session:** Academic Session 2025/2026 Semester 2  
**Prepared by:** Tai Jin Wei  
**Student ID:** 23080642  
**Date:** 16 May 2026  

---

## 1. Introduction

This Rules of Engagement document defines the scope, boundaries, and operational constraints for the AI agent conducting penetration testing as part of the WIC2007 Alternative Assessment. 

## 2. Test Scope

| Field | Details |
|-------|---------|
| Target Application | Damn Vulnerable Web Application (DVWA) |
| Target IP Address | 172.23.0.6 |
| Permitted Ports | Port 80 (HTTP) only |
| Application URL | http://172.23.0.6:80 |
| Environment | Isolated Docker network (pentagi-network) |
| Testing Period | May — June 2026 |


## 3. Permitted Activities

- Port scanning and service enumeration
- Web application vulnerability scanning
- SQL Injection testing
- Cross-Site Scripting (XSS) testing
- Authentication bypass testing
- Directory enumeration
- HTTP header analysis

<div style="page-break-before: always;"></div>

## 4. Prohibited Techniques

The following techniques are strictly **PROHIBITED** for the AI agent:

**4.1 Denial of Service (DoS) Attacks**  
The agent must NOT perform any flooding, resource exhaustion, or availability-disrupting attacks against the target or any other system.

**4.2 Attacks Against Out-of-Scope Targets**  
The agent must ONLY target 172.23.0.6. Any scanning or exploitation attempt against other IP addresses, including the host machine or external internet targets, is strictly prohibited.

**4.3 Destructive Exploitation**  
The agent must NOT delete, corrupt, or permanently modify any data or system files on the target beyond what is necessary to demonstrate vulnerability.

**4.4 Privilege Escalation Beyond Web Application**  
The agent must NOT attempt OS-level privilege escalation, container breakout, or lateral movement beyond the designated DVWA container.

**4.5 Social Engineering**  
The agent must NOT perform phishing, pretexting, or any human-targeted manipulation techniques.


## 5. Budget Constraints

| Field | Details |
|-------|---------|
| Maximum Token Budget | $0.50 USD per session |
| LLM Model | qwen/qwen2.5-72b-instruct |
| Input Cost | $0.25 per 1M tokens |
| Output Cost | $0.75 per 1M tokens |
| Kill-switch | Agent automatically halts when cumulative cost reaches $0.50 |


## 6. Escalation Procedure

If the AI agent attempts any prohibited action, the following steps must be taken immediately:

**Step 1 — IMMEDIATE HALT**  
Terminate the agent session immediately via the PentAGI interface or budget kill-switch.

**Step 2 — LOG THE INCIDENT**  
Record the prohibited action, timestamp, and tool used in `agent_execution_log.json`.

**Step 3 — REVIEW & ANALYSE**  
Review the Neo4j knowledge graph to identify what triggered the prohibited behaviour.

**Step 4 — RECONFIGURE & RESTART**  
Adjust the agent system prompt to add explicit restrictions before restarting the session.


## 7. Authorisation

I confirm that this penetration test is conducted solely within the designated Docker environment for academic purposes under WIC2007 Cyber Security assessment.

| Field | Details |
|-------|---------|
| Name | Tai Jin Wei |
| Student ID | 23080642 |
| Date | 16 May 2026 |
| Authorised by | WIC2007 Course Lecturer, Universiti Malaya |