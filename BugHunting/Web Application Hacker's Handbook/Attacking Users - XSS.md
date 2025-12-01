### I. Core Model: The Failed Context Separation

XSS occurs because the web application fails to maintain strict separation between user-supplied data (untrusted input) and the legitimate executable code (trusted output). When unsanitized data is inserted into an HTML page, the victim's browser interprets the data as executable script code.

- **The Trust Failure:** The vulnerability is fundamentally an **input validation error**. The application blindly assumes the input is harmless data suitable for display, rather than malicious code designed for execution.
- **Attack Objective:** Execute arbitrary client-side script (typically JavaScript) within the security context of the victim's browser, granting access to the victim's session cookies, local storage, and DOM.
- **Impact:** Ranging from cosmetic defacement to full account takeover (session hijacking) and malware distribution.

---

### II. Flaw Taxonomy: Pervasive Execution Vectors

XSS is categorized based on where the payload is introduced and where the execution occurs. Attackers must test all three varieties for comprehensive coverage.

#### A. Reflected XSS

The payload is delivered in a single HTTP request (e.g., in a URL parameter) and immediately executed in the resulting response.

- **Weaponization:** Requires social engineering; the attacker must trick the victim into clicking a crafted malicious link.

#### B. Stored XSS

The payload is permanently persisted on the server (e.g., stored in a database via a comment field, user profile) and delivered to all users who retrieve that data.

- **Weaponization:** Highest impact due to reliability and widespread compromise, often enabling mass data theft or virtual defacement .
- **Escalation:** Self-XSS (low impact, requiring the victim to inject the payload themselves) can often be escalated to Stored XSS by chaining it with a process that saves the input server-side, a key bug bounty tactic.

#### C. DOM-Based XSS

The vulnerability occurs entirely client-side. JavaScript reads user-controllable data from a "source" (e.g., `document.URL`) and writes it unsafely to an execution "sink" (e.g., `element.innerHTML`). The server never processes the payload, making traditional server-side scanners ineffective.

- **Weaponization:** Requires client-side code review to trace the flow of untrusted data from URL fragments or local storage directly into dangerous DOM sinks.

#### D. Advanced Delivery and Token Theft

Attackers prioritize delivery vectors that bypass standard payload filtering.

- **Header and Cookie Injection:** Exploit XSS via non-standard input vectors, such as user-controllable data reflected from `Referer` headers or custom cookies.
- **Cookie Theft:** A successful XSS attack will target the session token by reading `document.cookie` and exfiltrating it, but only if the session cookie lacks the **`HttpOnly`** flag.

---

### III. Attacker Playbook: Filter Evasion and Obfuscation

The XSS methodology moves beyond simple payloads (`<script>alert(1)`) to complex obfuscation and filter bypass techniques, leveraging encoding and creative syntax to confuse defensive blacklists.

1. **Input Fuzzing:** Fuzz every conceivable parameter for XSS, including obvious fields, hidden fields, cookies, and HTTP headers. Use HTTP proxies (Burp Suite/ZAP) to ignore and override client-side validation that attempts to block XSS input.
2. **Encoding and Variants:** Developers use weak blacklists, easily bypassed by experimenting with URL encoding, double-encoding, HTML encoding, or using non-standard tags/attributes (e.g., `svg`, `img`). Persistence and creativity are paramount for evasion.
3. **Contextual Analysis:** Observe exactly where the input is reflected. If it's inside an HTML tag attribute (e.g., `value="..."`), craft the payload to break out of that context first (`">XSS`). If it's inside a script block, escape the script context.
4. **Targeting Filter Quirks:** Exploit filter bugs by using recursive payloads (e.g., embedding a blocked string within itself) or null bytes to confuse canonicalization and validation logic.
5. **LLM Prompt Injection (2025+):** Identify inputs that feed directly into an internal AI component (A01: LLM Prompt Injection). If the LLM's response is then unsafely integrated into the victim's HTML (the execution sink), an attacker can inject malicious markup via the AI component, effectively bypassing traditional web application firewalls (WAFs) and filters designed for direct HTTP input.

---

### IV. Real Exploits and Advanced Breaches

XSS remains a highly valued vulnerability due to its ease of exploitation and high impact when chained with session integrity failures.

- **Bypassing WAFs via Obfuscation:** The history of XSS bounty reports confirms that filters are consistently bypassed through brief experimentation and clever encoding variations.
- **Chaining for Account Takeover:** XSS is frequently chained to steal session cookies that lack the `HttpOnly` flag. The theft of a valid token (Chapter 7) results in immediate, unauthorized account access.
- **Third-Party Integration Exploitation:** Attackers often find XSS in a weakly secured component (e.g., a video widget or a user badge) and exploit it against a high-value target (e.g., exploiting Wistia integration to attack Trello).

---

### V. Defense Gaps: The Input Validation Failure

XSS persists because developers fail to adopt the fundamental defense strategy: contextual output encoding, relying instead on flawed, permissive input validation.

- **Blacklisting Fails:** Filtering known bad strings is inherently inadequate; attackers simply find new, non-blacklisted syntax or encoding methods to achieve execution.
- **Lack of Contextual Encoding:** The definitive defense is output encoding: ensuring data is encoded relative to the context where it is displayed (e.g., HTML entity encoding, JavaScript encoding). Failure to do this across all code paths guarantees vulnerability.
- **Browser Filter Bypass:** Built-in browser filters (like XSS Auditors) provide weak defense, as their complex logic often creates new vectors for bypass, requiring the attacker to only be "creative".

---

### VI. One-Liner

Use `ffuf` to quickly test every query parameter on a page with a comprehensive, double-encoded XSS payload list, specifically looking for reflected input that alters the response length or triggers a unique status code (e.g., 200 OK):

```
ffuf -w /path/to/xss_fuzz_list.txt -u https://target.com/search?q=FUZZ&param2=FUZ2 -H "Cookie: session=[TOKEN]" -mc 200 -fs 0,1337
```

_Purpose: Fuzzes all parameters (here, 'q' and 'param2' simultaneously) with XSS payloads, suppressing content lengths typical of error pages, and looking for a 200 OK status, which indicates potential success or a successful filter bypass._