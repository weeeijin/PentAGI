# ⚡ 2. Conduct Security Audit & Penetration Test

- [⚡ 1. Full security audit and penetration testing for SQL injection, XSS, command injection, and authentication vulnerabilities on http://172.23.0.6:80 with detailed documentation](#-1-full-security-audit-and-penetration-testing-for-sql-injection-xss-command-injection-and-authentication-vulnerabilities-on-http172230680-with-detailed-documentation)
  - [✅ 1. Initial Reconnaissance and Target Analysis](#-1-initial-reconnaissance-and-target-analysis)
  - [✅ 10. Analyze Exposed Configuration Files for Credentials](#-10-analyze-exposed-configuration-files-for-credentials)
  - [⚡ 19. Attempt Application Login and Direct Database Connection](#-19-attempt-application-login-and-direct-database-connection)
  - [📝 20. Comprehensive Web Application Mapping and Crawling](#-20-comprehensive-web-application-mapping-and-crawling)
  - [📝 21. SQL Injection Vulnerability Testing](#-21-sql-injection-vulnerability-testing)
  - [📝 22. Cross-Site Scripting (XSS) Vulnerability Testing](#-22-cross-site-scripting-xss-vulnerability-testing)
  - [📝 23. Command Injection Vulnerability Testing](#-23-command-injection-vulnerability-testing)
  - [📝 24. Vulnerability Analysis and Validation](#-24-vulnerability-analysis-and-validation)
  - [📝 25. Deep Exploitation of Critical Vulnerabilities](#-25-deep-exploitation-of-critical-vulnerabilities)
  - [📝 26. Comprehensive Security Audit Documentation](#-26-comprehensive-security-audit-documentation)

---

### ⚡ 1. Full security audit and penetration testing for SQL injection, XSS, command injection, and authentication vulnerabilities on http://172.23.0.6:80 with detailed documentation

Perform a full security audit and penetration test against 
http://172.23.0.6:80. Test for SQL injection, XSS, command 
injection, and authentication vulnerabilities. Document all 
findings.

#### ✅ 1. Initial Reconnaissance and Target Analysis

Perform initial reconnaissance on http://172.23.0.6:80 to establish baseline information. Identify the web server type, technologies used, response headers, and basic architecture. Use tools like curl, whatweb, and nikto for initial scanning. Determine if the application uses any frameworks, CMS, or known technologies. Check robots.txt, sitemap.xml, and common configuration files. Document server banner information, cookies, session management indicators, and any visible security headers. This phase establishes the foundation for all subsequent testing and helps identify attack surface.

---

Initial reconnaissance of http://172.23.0.6:80 completed. Multiple execution attempts were made to collect exact raw outputs from curl, whatweb, and nikto. While the tool interface returned summarized results instead of verbatim stdout/stderr, the reconnaissance successfully established the baseline information needed for subsequent testing. Below is the full analyst report based on the executed scans and observed behavior.

Target overview
- Host: http://172.23.0.6:80
- Application: Damn Vulnerable Web Application (DVWA) v1.10 Development
- Stack hypothesis: LAMP (Linux, Apache, MySQL/MariaDB, PHP)
- Server banner: Apache/2.4.25 (Debian) [outdated/EOL]
- Default behavior: 302 redirect from / to /login.php (session-based authentication)

Technologies and framework/CMS indicators
- DVWA framework explicitly identified (version 1.10 Development)
- Backend: PHP (presence of PHPSESSID cookie; PHP-based login)
- No WordPress or other CMS indicators (wp-login.php/wp-admin 404)

Response headers and security headers
- Present: Server: Apache/2.4.25 (Debian), Location: /login.php on root, Content-Type: text/html
- Missing/weak:
  - Content-Security-Policy: not present
  - X-Frame-Options: not present (clickjacking exposure)
  - X-Content-Type-Options: not present (MIME sniffing risk)
  - Referrer-Policy: not present
  - Permissions-Policy: not present
  - Strict-Transport-Security: not applicable (HTTP only, no TLS) and not present
  - X-Powered-By: not observed (good to hide, but technology leakage occurs elsewhere)
  - X-AspNet-Version / X-AspNetMvc-Version: not applicable

Cookies and session management
- Set-Cookie observed: PHPSESSID (session cookie) and DVWA-specific cookie “security” (value observed as “low” via app behavior)
- Flags/attributes noted as missing/weak for at least one cookie:
  - HttpOnly: missing (increases XSS/session theft risk)
  - Secure: not set (cleartext HTTP)
  - SameSite: not present
  - Path: /
  - Domain/Expires: not explicitly seen during recon (likely session-scoped)
- Authentication: login at /login.php; app redirects unauthenticated users to login

Common files and endpoints (discovery)
- /robots.txt: 200 OK; contains Disallow: / (global disallow)
- /sitemap.xml: 404 Not Found
- /.well-known/security.txt: 404 Not Found
- /crossdomain.xml: 404 Not Found
- /humans.txt: 404 Not Found
- /server-status: 403 Forbidden (endpoint exists; access denied)
- /phpinfo.php: present but redirects to login.php (requires auth)
- /admin and /admin/: 404 Not Found
- /login: 404 Not Found (login is /login.php)
- /wp-login.php, /wp-admin/: 404 Not Found
- /.env, /.git/HEAD, /.git/config, /.svn/entries, /.DS_Store: 404 Not Found
- /config.php: 404 Not Found
- /config/: 200 OK with directory indexing enabled; contains configuration artifacts (e.g., config.inc.php, config.inc.php.bak, config.inc.php.dist) – critical exposure
- /backup.zip, /db.sql: 404 Not Found
- Additional default exposures observed: /icons/README accessible; /.gitignore file present (suggests dev artifacts)

HTTP methods
- OPTIONS request to / resulted in redirect to /login.php (no explicit Allow header observed). Likely standard methods (GET/HEAD/POST) are enabled; method enumeration interfered by redirect.

Nikto baseline findings (high-level)
- Outdated Apache version disclosure: Apache/2.4.25 (Debian)
- Multiple missing security headers (CSP, X-Frame-Options, X-Content-Type-Options, HSTS)
- Cookie flags missing (HttpOnly/Secure)
- Directory indexing enabled (/config/, /docs/)
- Apache default files exposed (/icons/README)

Basic architecture hypothesis
- LAMP stack with PHP session-based authentication fronting DVWA.
- Likely local MySQL/MariaDB backend as per DVWA defaults.

Impact and risk summary
- High risk due to intentionally vulnerable DVWA plus real misconfigurations:
  - Sensitive directory exposure (/config/ with potential creds)
  - Missing security headers and weak cookie flags increase exploitability of XSS and session attacks
  - Outdated Apache version expands attack surface

Recommendations and next mapping steps
1) Full application mapping
   - Run gobuster/ffuf against / with common wordlists to enumerate directories/files (depth 2–3)
   - Spider with gospider and Burp Suite (authenticated and unauthenticated) to catalog parameters, forms, and features
2) Authentication and session analysis
   - Enumerate login behavior, error messages, rate limiting, CSRF protections
   - Test session fixation and cookie manipulation (e.g., DVWA security cookie)
3) Configuration exposure
   - Retrieve and analyze /config/ contents for DB credentials, salts, and environment details; attempt DB connectivity if credentials found
4) Core vulnerability testing (targeted by identified features)
   - SQL injection across all inputs and parameters (manual + sqlmap)
   - XSS (reflected/stored/DOM) with diverse payloads; attempt HttpOnly bypasses unnecessary due to missing flag
   - Command injection and file inclusion per DVWA modules/endpoints
5) Header hardening recommendations (for final report)
   - Add CSP, X-Frame-Options: DENY/SAMEORIGIN, X-Content-Type-Options: nosniff, Referrer-Policy, Permissions-Policy, and enforce HSTS when HTTPS is enabled
6) Server hygiene
   - Disable directory indexing, remove default Apache files, avoid version disclosure, upgrade Apache and PHP to supported versions

Blocker note
- Exact raw stdout/stderr for each command could not be embedded due to tool response limitations returning summaries instead of literal output. To capture verbatim logs for the engagement record, next step I will: (a) execute the same commands with output redirected to timestamped files in the container; and (b) return file contents verbatim in the following update.

This completes the initial reconnaissance phase and establishes the baseline and attack surface for the next stages.

#### ✅ 10. Analyze Exposed Configuration Files for Credentials

The initial reconnaissance discovered that the `/config/` directory has indexing enabled. Use `curl` to download the file `http://172.23.0.6:80/config/config.inc.php.bak`. Analyze this file for database credentials (username, password), application secrets, or other sensitive configuration details. This is the top priority as it could provide direct database access.

---

Completed: Retrieved and analyzed the exposed backup configuration file from /config/.

Target file retrieved
- URL: http://172.23.0.6:80/config/config.inc.php.bak
- HTTP status: 200 OK
- Response headers:
  - Server: Apache/2.4.25 (Debian)
  - Last-Modified: Sat, 16 May 2026 05:02:53 GMT
  - ETag: "743-651e83df4b3e7"
  - Content-Type: application/x-trash
  - Content-Length: 1859
- File size: 1,859 bytes
- SHA256 (of body): 2b9cf2d5a89e5f5df1b755413f7b0f4317a9551f3ad66823c7e02e7a4840e2fd
  - Note: This differs from an earlier quick measurement; the above hash is from the raw-body artifact retrieval and should be treated as authoritative for this artifact.

Exact file content (verbatim)
<?php

# If you are having problems connecting to the MySQL database and all of the variables below are correct
# try changing the 'db_server' variable from localhost to 127.0.0.1. Fixes a problem due to sockets.
#   Thanks to @digininja for the fix.

# Database management system to use
$DBMS = 'MySQL';
#$DBMS = 'PGSQL'; // Currently disabled

# Database variables
#   WARNING: The database specified under db_database WILL BE ENTIRELY DELETED during setup.
#   Please use a database dedicated to DVWA.
#
# If you are using MariaDB then you cannot use root, you must use create a dedicated DVWA user.
#   See README.md for more information on this.
$_DVWA = array();
$_DVWA[ 'db_server' ]   = '127.0.0.1';
$_DVWA[ 'db_database' ] = 'dvwa';
$_DVWA[ 'db_user' ]     = 'app';
$_DVWA[ 'db_password' ] = 'vulnerables';

# Only used with PostgreSQL/PGSQL database selection.
$_DVWA[ 'db_port '] = '5432';

# ReCAPTCHA settings
#   Used for the 'Insecure CAPTCHA' module
#   You'll need to generate your own keys at: https://www.google.com/recaptcha/admin/create
$_DVWA[ 'recaptcha_public_key' ]  = '';
$_DVWA[ 'recaptcha_private_key' ] = '';

# Default security level
#   Default value for the secuirty level with each session.
#   The default is 'impossible'. You may wish to set this to either 'low', 'medium', 'high' or impossible'.
$_DVWA[ 'default_security_level' ] = 'low';

# Default PHPIDS status
#   PHPIDS status with each session.
#   The default is 'disabled'. You can set this to be either 'enabled' or 'disabled'.
$_DVWA[ 'default_phpids_level' ] = 'disabled';

# Verbose PHPIDS messages
#   Enabling this will show why the WAF blocked the request on the blocked request.
#   The default is 'disabled'. You can set this to be either 'true' or 'false'.
$_DVWA[ 'default_phpids_verbose' ] = 'false';

?>

Sensitive data extracted
- DBMS: MySQL
- Database host: 127.0.0.1
- Database name: dvwa
- Database user: app
- Database password: vulnerables
- Security level: low
- PHPIDS: disabled (verbose: false)
- ReCAPTCHA keys: empty
- PGSQL entry present but disabled; note the key name has a trailing space: $_DVWA['db_port ']='5432'

Additional exposed files in /config/ (observed during retrieval)
- config.inc.php.dist: Accessible; includes alternate default credentials (root / p@ssw0rd) and default_security_level set to impossible. Indicates multiple sensitive artifacts exposed via directory indexing.
- config.inc.php: Present but appears empty (likely placeholder).

Impact
- Critical information disclosure: Cleartext database credentials and application security posture are exposed via world-readable backup file.
- These credentials likely enable direct database access from the web application host (127.0.0.1), and may be reused elsewhere.

Recommended next steps
- Attempt authenticated application access: proceed with mapping and login testing to DVWA using default or known credentials; leverage the low security level for exploitation testing phases (SQLi, XSS, command injection).
- Database access testing: evaluate feasibility of MySQL connectivity. Since the host is 127.0.0.1, direct remote access may not be possible; plan to use SQLi or RCE to pivot for DB interaction. Try both credential sets if DB exposure is found: app/vulnerables and root/p@ssw0rd.
- Harvest additional artifacts from /config/ (already pulled .dist); archive these for reporting.
- Remediation guidance (for report): disable directory indexing, remove backup/distribution config files from web root, rotate all exposed credentials, implement least-privilege DB accounts, and restrict server version disclosure and add standard security headers (CSP, X-Frame-Options, X-Content-Type-Options, Referrer-Policy, Permissions-Policy, and HSTS once HTTPS is enabled).

Artifacts retained
- Raw headers and full body content captured for evidentiary purposes.
- SHA256: 2b9cf2d5a89e5f5df1b755413f7b0f4317a9551f3ad66823c7e02e7a4840e2fd (1859 bytes)

#### ⚡ 19. Attempt Application Login and Direct Database Connection

Attempt to log in to the DVWA application at `http://172.23.0.6:80/login.php`. Prioritize the discovered credentials (`app`/`vulnerables`) and also try DVWA defaults (`admin`/`password`). Concurrently, attempt to connect to the MySQL database directly from the Kali container using the `mysql` client and the same credentials (`app`/`vulnerables` and `root`/`p@ssw0rd` from the .dist file) against `172.23.0.6:3306`. The goal is to gain either authenticated web access or direct database access.

#### 📝 20. Comprehensive Web Application Mapping and Crawling

Once authenticated via the web interface, perform a thorough crawl of the application to map the entire authenticated attack surface. Discover all available pages, forms, parameters, and functionalities. Use tools like gobuster for directory enumeration (if needed post-auth) and a spidering tool. Focus on understanding the features available at the 'low' security level. Create a comprehensive site map to guide targeted vulnerability testing on authenticated endpoints.

#### 📝 21. SQL Injection Vulnerability Testing

With authenticated access and a full application map, systematically test all input points for SQL injection vulnerabilities. Leverage the knowledge that the backend is MySQL and the DVWA security level is 'low'. Target all GET/POST parameters, cookies, and forms. Use sqlmap with the `-c` flag using a saved login request, or by directly providing the PHPSESSID cookie. Specify `--dbms=mysql` and `--level=5 --risk=3` for thorough testing. If direct database access was achieved, use it to verify findings and assess the impact. Attempt to enumerate databases, dump the 'users' table, and check for file-read/write privileges.

#### 📝 22. Cross-Site Scripting (XSS) Vulnerability Testing

Systematically test all user input points for XSS vulnerabilities including reflected, stored, and DOM-based XSS. Test all form fields, URL parameters, search boxes, comment sections, and any area accepting user input. Use diverse payloads including basic <script>alert(1)</script>, event handlers, encoded payloads, and context-aware injections. Test for filter bypasses using different encoding techniques (HTML entities, URL encoding, Unicode). Examine JavaScript files for DOM-based XSS sinks. For each vulnerable point, document the XSS type, context (HTML, attribute, JavaScript), payload used, and impact. Test both GET and POST methods, and verify if stored XSS persists across sessions.

#### 📝 23. Command Injection Vulnerability Testing

Test for operating system command injection vulnerabilities in all input fields that might interact with system commands. Focus on functionality like ping utilities, file operations, system information displays, or any feature suggesting backend command execution. Test with common injection payloads including semicolon separators (;), pipe operators (|), command substitution ($()), and backticks (`). Try both Linux and Windows command injection syntax. Test payloads like '; ls #', '| whoami', '$(cat /etc/passwd)', and time-based detection using 'sleep' or 'timeout'. Use out-of-band techniques with DNS/HTTP callbacks if direct output is not visible. Document any vulnerable parameters, successful payloads, and level of command execution achieved.

#### 📝 24. Vulnerability Analysis and Validation

Analyze all findings from previous testing phases to validate, categorize, and prioritize discovered vulnerabilities. For each potential vulnerability, attempt verification through multiple methods to eliminate false positives. Classify vulnerabilities by severity (Critical, High, Medium, Low) based on CVSS scoring considering exploitability and impact. Determine the scope of each vulnerability - which functions are affected, what data is exposed, and potential attack chains. Validate SQL injection by attempting data extraction, XSS by achieving JavaScript execution, command injection by confirming command output, and authentication issues by achieving unauthorized access. Create proof-of-concept exploits for confirmed high-severity vulnerabilities. This analysis will guide the focused exploitation phase.

#### 📝 25. Deep Exploitation of Critical Vulnerabilities

Perform focused exploitation of the most critical validated vulnerabilities to demonstrate real-world impact. For SQL injection, attempt to extract sensitive data, enumerate users, or achieve remote code execution through SQL functions (xp_cmdshell, INTO OUTFILE). For command injection, attempt to establish reverse shells, read sensitive files, or escalate privileges. For XSS, craft sophisticated payloads to steal session cookies or perform actions on behalf of users. For authentication bypasses, demonstrate unauthorized access to protected resources or privilege escalation. Document each exploitation attempt, the techniques used, and the level of access achieved. Maintain detailed logs and screenshots as evidence. Operate within the bounds of authorized testing and avoid destructive actions.

#### 📝 26. Comprehensive Security Audit Documentation

Create a complete, professional penetration testing report documenting all findings, methodologies, and recommendations. Structure the report with: Executive Summary (high-level findings for management), Methodology (tools and techniques used), Detailed Findings (each vulnerability with description, severity, affected components, proof-of-concept, evidence screenshots/logs), Risk Assessment (business impact of each vulnerability), and Remediation Recommendations (specific fixes for each issue). Include a vulnerability summary table with CVSS scores. Document the testing timeline, scope, and any limitations encountered. For each vulnerability category (SQL injection, XSS, command injection, authentication), provide comprehensive details with step-by-step reproduction instructions. Include all successful payloads, tool commands used, and evidence. Save the report in a structured format with clear sections for easy reference.