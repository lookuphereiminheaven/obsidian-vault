### 1. Purpose: Why Mapping Is Essential Before Exploitation

Mapping is not merely a preliminary task; it is the highest-leverage activity in web testing because it defines the scope and vector of the entire attack.

- **Defines the Attack Surface:** The process thoroughly catalogs all content and functionality, identifying every point where user-supplied input can be processed by the server. This includes URLs, query string parameters, POST data, cookies, and other HTTP headers.
- **Enables Informed Triage:** A seasoned expert uses the map to quickly triage areas of functionality, prioritizing time investment into specific, high-risk functions where severe flaws are likely to reside. Sensitive features, such as those processing credit card data or managing user accounts, are prioritized to maximize the business impact of discovered vulnerabilities.
- **Facilitates Attack Formulation:** Mapping allows the attacker to infer server-side technologies, development assumptions, and expected workflows. This knowledge is vital for formulating customized attacks tailored to the application's specific behavior, which is necessary because every web application is different.
- **Primes Logic Flaw Discovery:** Understanding the logical relationships and dependencies between functions allows the attacker to predict and violate the developers' assumptions, leading to the discovery of application logic flaws, which are among the most valuable vulnerabilities.

---

### 2. Core Concepts

Effective mapping is driven by understanding how applications expose functionality, process parameters, and manage internal state.

- **Content and Functionality Enumeration:** The initial phase involves cataloging the application's content and functionality to understand what it does and how it behaves. This is achieved by walking through the visible content and actively searching for hidden resources.
- **Application Pages Versus Functional Paths:** While many functions are accessed via unique URLs, modern applications may use a single URL with parameters to determine the action. The attack focus should shift to mapping **functional paths**—the logical sequence and dependencies between actions (e.g., TransferFunds steps)—as these relationships expose developer assumptions and possible sequence violations.
- **Parameter Analysis:** Every query string parameter, item of POST data, cookie, and HTTP header constitutes an entry point for user input. Identifying these points is key to mapping the application's attack surface. Parameters can define functions (e.g., `/admin.jsp?action=editUser`) or control logic (e.g., `debug=true`).
- **Client/Server Behavior Inference:** Analyzing the traffic allows the inference of server-side technologies (e.g., scripting languages, databases, web server software) and client-side mechanisms (forms, scripts, cookies, ActiveX controls, and Flash). Session tokens, URL formats, and specific headers often provide clues about the underlying technologies.

---

### 3. Methods (Practical + Bug Bounty Focus)

The most effective mapping strategy combines disciplined manual testing with controlled automation, utilizing the proxy as the central tool for traffic analysis.

#### Manual Proxy-Based Mapping (User-Directed Spidering)

This sophisticated technique is generally preferable to fully automated spidering because the user retains control over the session and data inputs.

- The user browses the application normally, attempting to navigate all functionality, while an intercepting proxy (like Burp Suite or ZAP) records and parses all resulting traffic.
- The human operator ensures input validation requirements are met, allowing passage through multi-stage functions that automated tools often break.
- The proxy compiles the **Site Map** based on the URLs visited by the browser and passively extracts new links from the responses.

#### Automated Spidering (Limits)

While automated spidering can enumerate content by following links recursively, it has significant limitations in modern, stateful applications.

- **Failure Modes:** Automated spiders struggle with unusual navigation (JavaScript menus), complex input validation, and managing authentication state across multiple requests.
- **Session Breaks:** Spiders may invalidate their own session by requesting the `/logout` function, submitting invalid input to sensitive functions, or failing to handle per-page anti-CSRF tokens.
- **Danger:** Unrestricted automated spidering can be highly dangerous, potentially executing administrative functions (e.g., deleting users) or causing real-time damage if access controls are flawed.

#### Directory/Endpoint Fuzzing

Brute-force techniques are essential for discovering **hidden content**—functionality that is not linked from the main site.

- **Tools:** Tools like **Gobuster** and Burp Intruder are used to iterate through wordlists of common directories, filenames, and extensions.
- **Hit Detection:** Attackers cannot rely solely on the `200 OK` status code. Hits are identified by analyzing response characteristics, such as slight differences in response length or the absence of the application's customized "file not found" page.
- **Inference:** Observed filenames and directory structures are used to infer missing related names (e.g., finding `AddDocument.jsp` suggests testing for `EditDocument.jsp`).

#### Parameter Mining

Parameters are the primary data entry points.

- **Hidden Parameters:** Attackers look for parameters that control logic but are not explicitly surfaced. Burp Intruder can perform "cluster bomb" attacks, combining lists of common debug parameter names (`debug`, `test`, `source`) with values (`true`, `1`) to find hidden application logic.
- **Identifier Harvesting:** Parameters that specify resources (e.g., `id=123`) are harvested and enumerated to find subsequent IDOR vulnerabilities.

#### Client-Side Code Analysis

Reviewing client-side code provides valuable intelligence about the application's underlying mechanisms.

- **Functionality Clues:** JavaScript code, HTML comments, and hidden form fields may reveal function names, server-side references, and internal application structure.
- **Bypass Strategy:** Client-side input validation controls (e.g., JavaScript logic) are inherently untrustworthy and must be tested for bypass using the intercepting proxy.

#### State/Workflow Discovery (Cookies, Tokens, Hidden Fields)

State-tracking mechanisms are prime attack vectors as they are often improperly protected.

- **Cookie Review:** Session tokens and cookies are checked for predictability and information leakage. The presence or absence of security flags like `Secure` and `HttpOnly` is noted.
- **Hidden Fields:** Hidden form fields, which are client-generated data items sent back to the server, must be analyzed, their intended role guessed, and their values modified to test for logic flaws or unauthorized access.

---

### 4. Advanced Tactics

Expert-level mapping focuses on complex architectural features and logic boundaries that scanners fail to cover.

- **Mapping Multi-Step Flows:** Critical functionality often involves a defined sequence of requests. Mapping involves logging all requests in the sequence and testing for defects like logic bypass by accessing subsequent stages out of order. Hidden parameters used to track state across stages are targets for manipulation.
- **Access Control Boundaries:** Testing requires comprehensive mapping across different user contexts (e.g., administrator vs. basic user). **Burp Suite's "compare site maps"** feature is used to automatically identify differences between application views, highlighting unauthorized access to resources.
- **API/GraphQL Mapping:** Modern applications rely heavily on APIs. Mapping these involves specialized techniques:
    - Fuzzing tailored wordlists to discover hidden API endpoints and parameters.
    - Attempting GraphQL **introspection queries** to dump the entire schema, or using tools like ZAP's GraphQL add-on or Clairvoyance to gain insight when introspection is disabled.
    - Identifying authentication mechanisms (tokens, requirements) used by API endpoints.
- **Hidden/Unlinked Endpoints:** Discovery extends beyond simple brute force to leveraging public resources like the **Wayback Machine** or **Google Dorking** to retrieve historical or externally linked content not currently visible. Nikto and Wikto are also useful for locating default content and known misconfigurations.
- **Business Logic Surfaces:** Focus testing on logic flaws within applications. This involves analyzing functional paths and critical security functions (login, registration) for defective assumptions, such as logic that can be manipulated by submitting negative values or exploiting transaction boundaries.

---

### 5. Common Findings From Good Mapping

The outcome of disciplined mapping is the identification of high-impact vulnerabilities rooted in flawed logic and broken access controls.

- **IDOR / Broken Object Level Authorization (BOLA):** Mapping reveals predictable numerical or sequential identifiers, which, when enumerated, lead to unauthorized access to other users' resources (horizontal privilege escalation).
- **Authentication and Access Control Bypass:** Discovered by manipulating hidden parameters (e.g., `admin=true`), violating the logical sequence of multi-step processes (forced browsing), or exploiting flaws in password recovery functions.
- **Privilege Jumps (Vertical Privilege Escalation):** Found by comparing site maps or testing admin functionality exposed to low-privileged users.
- **Parameter Pollution (HPP):** Mapping identifies all parameters, which are then fuzzed by submitting duplicate parameters to confirm how the server-side code handles the unexpected input, often leading to unexpected results or data leaks.
- **Workflow Abuses (Logic Flaws):** Detected by observing how the application responds to attempts to submit incomplete input, remove parameters, or access functionality out of sequence.

---

### 6. Quick Checklist (Battle-Ready Bug Hunter Edition)

A concise, high-priority list for maximizing attack surface coverage.

- **Setup:** Configure Burp Suite/ZAP (the critical tool) with defined scope, ensuring the proxy intercepts all traffic (HTTP/S) and maintains authenticated state.
- **Manual Traversal (User-Directed Spidering):** Browse the entire visible application as a normal user, ensuring all forms are submitted with valid data and all multi-step functions are completed.
    - **Context:** Perform full traversal for _all_ user roles (anonymous, low-privilege, high-privilege).
    - **Passive Recon:** Monitor the proxy Site Map for passively discovered links (from scripts, comments) that were not manually visited.
- **Hidden Content Discovery (Active Recon):**
    - **Brute Force:** Use **Gobuster** or Burp Intruder/Content Discovery with comprehensive wordlists (SecLists) to enumerate directories, files, and common extensions (`.bak`, `.old`, `.inc`).
    - **Public Sources:** Query Wayback Machine and use Google Dorking to retrieve old/unlinked URLs, feeding them back into the spider/fuzzer.
- **Input & Parameter Hunting:**
    - **Catalog Entry Points:** Log every parameter (URL, POST, Cookie, Header).
    - **Hidden Logic:** Fuzz known high-priority pages (login, search) for hidden parameters (e.g., `debug=true`) using Burp Intruder's cluster bomb attack type.
- **API & IDOR Target Identification:**
    - Identify all API endpoints. Test for GraphQL Introspection or search for OpenAPI/Swagger documentation.
    - Extract sequential or predictable identifiers from observed parameters (e.g., `/user?id=123`) and reserve them for automated IDOR enumeration.
- **Logic Mapping:** Focus on multi-stage processes. Log the exact sequence of requests and prepare to test for sequence violation and parameter manipulation by removing or modifying critical hidden state parameters.

---

### 7. Condensed Summary + Mini Quiz

#### One-Page Summary

Mapping the application, as detailed in WAHH Chapter 4, is the critical first step in web security assessment, aiming to define the entire attack surface. This process relies on **user-directed spidering**—manually walking through all functionality while an intercepting proxy (like Burp Suite) passively records traffic and builds a detailed site map. This approach overcomes the inherent limitations of fully automated spiders, which often fail when encountering complex, stateful processes or authentication mechanisms.

Mapping involves two core activities: **Enumeration** and **Analysis**. Enumeration uses brute-force tools (Gobuster, Burp Intruder) to discover hidden or forgotten content, often inferring resource names based on observed naming conventions. Analysis focuses on identifying all **input entry points** (parameters, headers, cookies) and mapping the application's **functional paths**—the logical sequence of actions—rather than just the URLs, as this exposes the underlying business logic assumptions. Advanced mapping includes searching public archives (Wayback Machine), fuzzing API endpoints for hidden versions, and maintaining authenticated sessions across long automation tasks using proxy macros. The ultimate goal is to pinpoint high-yield attack vectors, particularly those related to access control (IDOR) and business logic flaws, which require human intelligence and methodical exploration.

#### Short Quiz (Expert Challenge)

1. According to WAHH, what is the key difference between **Application Pages** and **Functional Paths**, and why is the latter more useful to an attacker?
2. What specific Burp Suite feature is recommended to automate the process of handling ephemeral anti-CSRF tokens during long automated attacks?
3. Name two types of publicly available resources that can aid in discovering unlinked, hidden content.
4. Why are input validation checks performed on the client side (e.g., via JavaScript) considered insecure by the application mapping methodology?
5. What common attack technique does the identification and enumeration of sequential resource IDs (e.g., `id=1`, `id=2`) primarily support?

#### Quiz Answers

1. **Application Pages** are based on unique URLs (like static files), whereas **Functional Paths** map the logical sequence and dependencies between actions, regardless of the URL. The latter is more useful because it exposes developer assumptions and logic flaws.
2. **Session Handling Rules** and **Macros**.
3. Web archives (e.g., Wayback Machine) and search engines (e.g., Google Dorking).
4. Client-side controls are outside the application's control, and users can easily view and modify them (e.g., using an intercepting proxy).
5. **Insecure Direct Object Reference (IDOR)**.