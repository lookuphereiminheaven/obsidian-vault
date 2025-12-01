### I. Core Model: Abusing Implicit Trust Boundaries

The fundamental vulnerability arises from the application's failure to sanitize user input before passing it to an execution context _outside_ the application runtime, such as the shell interpreter or a file system API. The core failure is the breach of the internal trust boundary: the application trusts that external components (OS, kernel) will receive only safe data, not executable commands.

- **OS Command Injection:** Input bypasses the web application's language and executes directly on the server's operating system.
- **Path Traversal/File Inclusion:** Input manipulates file path logic to access or execute arbitrary files on the local or remote filesystem.
- **SSRF:** Input manipulates server-side networking calls, causing the application to attack other internal or external systems.

---

### II. Flaw Taxonomy: Exploiting System Interaction

Attack vectors are diverse, targeting specific execution mechanisms and protocols used for internal or external communication.

#### A. Operating System (OS) Command Injection

This attack is the result of using functions that execute system commands and embedding unsanitized user input directly within the command string.

- **Mechanism:** An attacker inserts shell metacharacters (e.g., `&`, `|`, `;`, `` ` ``) to terminate the intended command and append an arbitrary new command.
- **Defense Failure:** Applications rely on blacklisting filters, which are easily bypassed by encoding, using different metacharacters, or abusing spaces to inject parameters.
- **Weaponization:** Executes utilities like `whoami`, `ls`, or `sleep 5` for confirmation, escalating to data exfiltration or RCE via common Linux/Windows tools.

#### B. File Path Traversal and Inclusion (LFI/RFI)

These flaws target file handling functions where the application constructs a file path based on user input (e.g., retrieving a document or template).

- **Path Traversal (LFI):** Attacker submits directory traversal sequences (`../../`) to break out of the intended directory and access sensitive local files (e.g., `/etc/passwd` or `/windows/win.ini`).
- **Remote File Inclusion (RFI):** If the application dynamically includes a file based on a URL parameter and the server configuration permits it, an attacker forces the inclusion and execution of a malicious script hosted on a remote server.
- **Weaponization:** File access can lead to information disclosure (A21). Successful LFI is often leveraged to achieve RCE by including a file containing attacker-controlled data (e.g., a manipulated session file or log file containing malicious PHP code).

#### C. XML External Entity (XXE) Injection

This is a modern injection flaw (often related to Injection A05) targeting applications that process XML documents from user input.

- **Mechanism:** XML parsers, when configured to process external entities (DTDs), allow attackers to define internal or external entities that the parser then processes (e.g., reading local files or making internal network requests).
- **Weaponization:** XXE is primarily used for **Local File Read** (exfiltrating sensitive files like `/etc/passwd`) or **Blind SSRF** (forcing the XML parser to attack internal services).

#### D. Server-Side Request Forgery (SSRF)

This vulnerability forces the server to make arbitrary requests to resources it can reach, but the client cannot (e.g., internal network interfaces or cloud metadata services).

- **Mechanism:** Attacker controls a URL input parameter that the server uses internally to fetch data (e.g., fetching a remote image or processing a webhook URL).
- **Weaponization:** Used to scan internal network ports, bypass firewall restrictions, or steal credentials from cloud service metadata endpoints (e.g., AWS EC2 metadata).
- **Modern Relevance:** SSRF is often facilitated by XML processing flaws (XXE) or applications utilizing web services that accept URLs as input.

---

### III. Attacker Playbook: Testing Internal Trust

The methodology focuses on identifying functions that interact with the local operating environment or network services.

1. **Fuzzing for OS/System Interaction:** Fuzz all parameters with shell metacharacters (`|`, `&`, `;`, `$()`) and basic commands (`sleep 5`, `ping -c 1 127.0.0.1`). Success is confirmed via a measurable **time delay** or successful **out-of-band (OOB) communication** (e.g., a DNS lookup to Burp Collaborator).
2. **Path Traversal/LFI Probing:** Identify file handling parameters (e.g., `file`, `template`, `page`) and submit exhaustive traversal payloads (`../../`) followed by known world-readable files (`/etc/passwd`). Test common encoding variations (URL, double-URL) to bypass filters.
3. **SSRF Discovery:** Locate parameters that accept URLs, web service endpoints, or file inclusion paths. Submit requests targeting internal infrastructure (e.g., `http://127.0.0.1/`) or cloud metadata endpoints (`http://169.254.169.254/latest/meta-data/`).
4. **XML Payload Testing (XXE):** Fuzz applications accepting XML or SOAP/Web Service input. Inject malicious DOCTYPE definitions designed to read local files or trigger internal network requests.
5. **Second-Order Injection:** Input that is safely validated and stored (e.g., a user profile field) might be passed unsafely to a command executor or file handler when accessed by a different function (e.g., generating a PDF report).

---

### IV. Real Exploits and Advanced Breaches

High-severity vulnerabilities, particularly RCE, stem directly from these back-end flaws and are highly prized in bug bounty hunting.

- **OS Command Injection:** Successful execution of the `id` command after exploiting a flawed component is a standard Proof-of-Concept for OS Command Injection.
- **AI Supply Chain RCE (2025+):** If the application uses an AI framework (like LangChain) and allows user input to specify a file or network operation, an attacker can use a **LLM prompt injection** to trick the model into executing dangerous operating system or network calls via its internal tools API. This bypasses web input filters entirely by executing the command from a trusted, internal AI context.
- **Insecure File Upload/LFI Chain:** Exploiting client-side upload bypasses (Chapter 5) to place a malicious file (e.g., a PHP web shell) on the server, then leveraging an LFI vulnerability to execute that file, resulting in full RCE.
- **SSRF against Internal Services:** Attackers exploit SSRF to access internal endpoints that may expose sensitive admin interfaces or metadata that is normally firewalled off, a common vector against cloud infrastructure.

---

### V. Defense Gaps: Why Protection Fails

These injection flaws persist due to flawed input sanitation and a failure to enforce strict architectural controls.

- **Blacklisting Metacharacters:** Developers fail by attempting to filter only common shell metacharacters (`&`, `|`, `;`), a method easily defeated by obfuscation, encoding, or using lesser-known shell syntax.
- **Flawed Path Canonicalization:** File path checks fail if the application doesn't correctly normalize input (e.g., resolving `../` sequences) or if NULL bytes are used to prematurely terminate the string before validation checks execute.
- **Allowing Remote Includes:** Server configurations that enable dynamic remote file inclusion (e.g., `allow_url_include` in PHP) fundamentally expose the system to RFI.
- **Default Parser Trust:** XML and other parser defaults are often too permissive, enabling features like DTD processing that allow external entities, thereby creating XXE vulnerabilities.

---

### VI. One-Liner

Use `ffuf` to test common LFI/Path Traversal vectors against a suspected file inclusion parameter, looking for a unique HTTP status or content length that indicates a successful file read (e.g., finding `/etc/passwd`):

```
ffuf -w /path/to/lfi_payloads.txt -u https://target.com/api/get_doc?filename=FUZZ -mc 200,403 -fs 0,4321
```

_Purpose: Fuzzes the 'filename' parameter with payloads containing traversal sequences (`../../`), filtering for successful status codes (200 OK) and suppressing common error page lengths (e.g., suppressing the length of the 404 page). A successful file read (e.g., of `/etc/passwd`) often results in a unique, non-error content length._