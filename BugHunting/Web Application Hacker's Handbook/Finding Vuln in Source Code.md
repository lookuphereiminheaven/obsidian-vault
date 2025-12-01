### I. Core Model: Code as the Ultimate Oracle

Source code review provides the attacker with an undeniable map of the application's internal logic, including variables, dependencies, and all hidden code paths. It bypasses the need for guesswork (fuzzing) and provides surgical precision in vulnerability identification.

- **Attack Objective:** Identify code execution paths where user-supplied data (parameters, cookies, headers) is processed without proper sanitization before being passed to a dangerous API or execution environment.
- **The Power of Discovery:** Source code exposes flaws like **backdoor passwords** that are nearly impossible to find through password guessing alone.
- **Complementation:** Code review is not a substitute for black-box testing, but a powerful complement; finding a vulnerability in code is followed by testing it on the running application for confirmation and exploitation.

---

### II. Flaw Taxonomy: Signatures of Vulnerability

The methodology focuses on searching the codebase for function calls that are known to be execution sinks, where user data is likely to be improperly handled.

#### A. Injection Signatures (A05)

These signatures mark code paths where input is merged unsafely with executable instructions.

- **SQL Injection (SQLi):** Look for string concatenation using request variables adjacent to database functions (e.g., `SELECT * FROM users WHERE name = ' . $input . '`). This is the fundamental failure of parameterization.
- **XSS/HTML Injection:** Search for functions that write user input directly to the HTTP output stream or the DOM (e.g., `document.write`, `print`, `echo`) without contextual encoding.
- **OS Command Injection:** Target functions that execute shell commands (e.g., `system()`, `exec()`, `passthru()`) where input is embedded in the command string. Look for shell metacharacters abuse.

#### B. Access Control and Logic Signatures (A01)

These flaws reveal defects in authorization checks and developer assumptions.

- **Backdoor Passwords:** Search for hardcoded credentials used for debugging or emergency access (e.g., `if (password == "gofaster")`).
- **Path Traversal/LFI:** Identify file system functions (e.g., `fopen()`, `include()`, `require()`) that accept file paths based on user input (e.g., URL parameters), which can lead to reading sensitive files like `/etc/passwd`.

#### C. Native Code and Serialization Signatures (RCE)

These target underlying memory safety issues or complex data processing routines.

- **Native Software Bugs (Buffer Overflow):** Identify low-level C/C++ functions handling input size without bounds checking (e.g., `strcpy`, `strcat`), a potential precursor to RCE.
- **Insecure Deserialization:** Locate functions that deserialize complex object structures from user input. This is a high-impact flaw (like exploiting ViewState deserialization via `YSoSerial.Net 2025` payloads) if type safety or integrity (MAC) checks are absent.

---

### III. Attacker Playbook: Tracing and Grepping

The audit is structured and iterative, maximizing coverage of the codebase quickly by relying on high-speed text processing.

1. **Trace Data Flow:** Identify all entry points for user data (request parameters, cookies, headers) and follow the variable path through the application's layers until it hits an execution sink (database query, file operation, output rendering).
2. **Signature Search (The Grep Attack):** Utilize regular expressions and code browsing tools (Source Insight, `grep`) to search the entire codebase for dangerous API calls:
    - Search for common database interaction functions (`query`, `execute`, `select`).
    - Search for system execution commands (`system`, `exec`, `shell_exec`).
    - Search for authentication bypass keywords (`admin=true`, `password`, `secret`).
3. **Review Session and Credential Logic:** Prioritize modules responsible for authentication, session management, and authorization (Chapter 6, 7, 8) to find flaws like predictable session tokens or hardcoded backdoors.
4. **Modern Code Auditing:**
    - **JWT Secrets:** Search configuration files and code constants for weak or hardcoded JWT signing keys, which could allow **JWT forgery** and full access if exploited (e.g., exploiting a weak key via an HS256 downgrade).
    - **AI Wrappers (2025+):** Review code that implements interfaces to internal AI systems (A01: LLM Prompt Injection). Identify the "tools" exposed to the LLM; if the code allows the LLM to call `read_file()` or `run_shell()`, this exposes severe **AI supply chain** RCE risks.

---

### IV. Real Exploits and Defense Gaps

Code review often reveals architectural flaws and hard-to-find logic defects that scanners completely miss.

- **Backdoor Passwords:** Simple backdoors are a failure of the secure software development lifecycle (SDLC) and are often the highest-impact finding from a white-box test.
- **Defense Failure (Incomplete Filtering):** Reviewing source code proves that developers attempt to use flawed **blacklisting** or client-side filtering (Chapter 5) instead of fundamental output encoding or parameterized queries. Code review confirms these filters are always insufficient.
- **Misconfiguration Disclosure (A02):** Code often exposes environment variables, internal server paths, or database connection strings, supporting the exploitation of architecture flaws (Chapter 17).

---

### V. One-Liner

Use `grep` recursively to locate the signature of a dangerous function call that executes a shell command with a variable derived from user input (potential OS Command Injection):

```
grep -rE '(system|exec|shell_exec)\s*\((.*request.*|.*get_param.*)\)' /path/to/codebase
```

_Purpose: Recursively searches application code for functions known to execute OS commands (`system`, `exec`, `shell_exec`) where the input parameter is clearly derived from an external user request variable._