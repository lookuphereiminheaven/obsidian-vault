### I. Core Model: Code Injection into Interpreted Contexts

The database is the ultimate prize, holding all sensitive application information. **Injection** occurs when the server fails to maintain a strict separation between user-supplied data and the application's executable code.

*   **The Trust Failure:** The application trusts that incoming parameters are only data, but an attacker submits syntax.
*   **The Goal:** Transform a harmless input parameter (e.g., a username or ID) into a database command (e.g., `UNION SELECT`, `DROP TABLE`).
*   **Target Scope:** While SQL Injection (SQLi) is paramount, this chapter extends to NoSQL (MongoDB), XPath, and LDAP, all of which embed user input into interpreted query languages.

---

### II. Flaw Taxonomy: Exploiting Query Syntax

Injection flaws are categorized by the target language and the attack vector (in-band, blind, or out-of-band).

#### A. SQL Injection (SQLi)
The most common and highest impact data store vulnerability.

*   **Logic Bypass:** Trivial SQLi can bypass login functions by manipulating a `WHERE` clause to always return `TRUE`, often with a simple payload like `' OR 1=1--`.
*   **Data Exfiltration (UNION Attack):** Used when the query returns results directly to the application. The attacker injects a `UNION SELECT` statement to combine the malicious query results with the legitimate output, dumping arbitrary data from other tables (e.g., user credentials).
*   **Blind SQLi:** Used when the application suppresses errors and does not directly reflect query output. Attackers exploit subtle differences in server responses:
    *   **Boolean-based:** Uses conditional logic (e.g., `AND 1=1`) to ask the database true/false questions, determining output character by character.
    *   **Time-based:** Uses the `SLEEP()` function within the conditional logic. A successful guess results in a measurable delay, allowing data extraction even without direct output.
*   **Second-Order SQLi:** Input is stored securely but retrieved and processed unsafely later in a different function, violating the assumption that previously validated data remains safe.

#### B. Other Data Store Injections
The underlying mechanism (unsafe embedding of input) applies broadly.

*   **NoSQL Injection (MongoDB):** Exploits query structures that use JSON or JavaScript, allowing attackers to manipulate the JSON structure or inject JavaScript code to retrieve unauthorized data or bypass authentication.
*   **XPath Injection:** Attacks applications using XML data stores by manipulating XPath queries to subvert logic or disclose content (e.g., revealing an entire XML document structure).
*   **LDAP Injection:** Exploits applications using Lightweight Directory Access Protocol (LDAP) directories, typically for authentication or user search. Injecting filter syntax can bypass authentication or perform unauthorized searches.

---

### III. Attacker Playbook: Maximizing Data Theft

The methodology moves from detection (fuzzing) to specialized exploitation (data harvesting).

1.  **Input Fuzzing for Errors:** Fuzz every parameter (URL, POST, Cookie, Header) with common injection syntax (`'`, `"`, `--`) to force the application to reveal **verbose error messages**. These error messages often contain sensitive details, like database structure or query structure, aiding subsequent exploitation.
2.  **Fingerprinting and Payload Selection:** Use error messages or time delays to identify the database type (MySQL, Oracle, MS-SQL). Adjust the exploitation payload (e.g., comment syntax: `--` for MySQL, `;%00` for others).
3.  **Weaponizing Data Exfiltration (UNION):** Once the number of columns in the query is determined, use `UNION SELECT` combined with schema-revealing tables (e.g., `information_schema` in MySQL) to dump critical tables, like those containing user credentials or sensitive PII.
4.  **Blind Exploitation:** If no direct output is possible, automate character-by-character extraction using time-based payloads (e.g., `IF(1=1, SLEEP(5), 0)`) via Burp Intruder, configuring the tool to monitor time delays for inference.
5.  **Out-of-Band Channels:** When direct output is restricted, look for ways to force the database to initiate an external request (e.g., using DNS or HTTP requests triggered by the database engine). This is often used for retrieving credentials or confirming successful exploitation without relying on reflected content.

---

### IV. Real Exploits and Advanced Breaches

SQLi remains a significant threat, requiring customized tools and techniques.

*   **Injection in Interpreted Contexts:** Injection flaws (A05) are highly valued in bug bounty programs. Successful exploitation often results in high payouts because of the ability to extract sensitive data or achieve full system compromise.
*   **Chaining for RCE:** If the database privileges are high enough, SQLi can be leveraged to write files to the server's filesystem, often leading to **Remote Code Execution (RCE)** by writing a web shell.
*   **LLM Database Interaction (2025+):** Applications utilizing AI services (like LangChain) to query internal databases based on user input (e.g., "Find customer X") are severely exposed. An attacker submits an **LLM prompt injection** payload disguised as a legitimate query, which causes the internal AI wrapper to execute arbitrary SQL or NoSQL commands on the backend data store, effectively bypassing traditional sanitization layers and achieving SQLi through the AI supply chain.
*   **Race Conditions on Database Updates:** Application logic flaws, such as the **Starbucks race condition**, often rely on timing exploits during transaction logic (WAHH 11.5). The flaw exists because the database transaction is not atomic, allowing two simultaneous requests to confirm an action (like applying a discount or withdrawing funds) based on an outdated, vulnerable state read from the database.

---

### V. Defense Gaps: The Failure of Input Filters

Injection flaws persist because developers rely on inadequate filtering instead of implementing fundamental architectural defense.

*   **Blacklisting Failures:** Developers commonly use "reject known bad" blacklists to filter dangerous keywords (`UNION`, `SELECT`). Attackers bypass these easily using:
    *   **Encoding Tricks:** Using URL encoding, double-encoding, or special characters (NULL bytes) to obfuscate payloads that bypass filters but are correctly interpreted by the database.
    *   **Contextual Manipulation:** Exploiting language-specific syntax (e.g., comments, string manipulation) to break out of the interpreted context.
*   **Inadequate Parameterization:** The single most effective defense, **parameterized queries**, which ensures user input is always treated as data, is often neglected or incorrectly applied, especially in dynamic or legacy code.
*   **Verbose Errors:** Failure to disable detailed debugging and error messages in production allows attackers to harvest critical database information needed for complex blind and union attacks.

---

### VI. One-Liner

Use `ffuf` to quickly test every parameter for classic SQL injection errors, looking for anomalous HTTP status codes or content length changes indicative of a successful syntax error:

```bash
ffuf -w custom_sql_fuzz.txt -u https://target.com/api/product?id=FUZZ -recursion -e .html,.php -mc 500,404,403 -ms 32,5400 -H "Cookie: SessionID=[TOKEN]"
```
*Purpose: Fuzzes the 'id' parameter with a custom list of SQL payloads (e.g., ' and 1=1-- , ' and 1=0-- ) and filters for unexpected HTTP status codes (200 OK) or anomalous content lengths, indicating a successful injection that altered the query logic.*