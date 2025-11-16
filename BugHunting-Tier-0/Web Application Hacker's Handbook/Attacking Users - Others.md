### I. Core Model: Abusing Browser Trust

These attacks exploit the fundamental difference between authentication (who you are) and authorization (what you can do). By targeting the client's security context, the attacker bypasses server-side business logic and security controls that rely on user interaction. The objective is to use the victim's authenticated session to perform actions against the target application without the victim's consent or knowledge.

- **The Trust Failure:** The browser automatically sends session-identifying cookies with every request to the originating domain. This automatic behavior forms the basis of **Cross-Site Request Forgery (CSRF)**, the primary flaw exploited in this category.
- **The Attack Vector:** The vulnerability is typically delivered via a malicious external web site or a crafted link, tricking the victim into initiating a request that appears legitimate to the target application.

---

### II. Flaw Taxonomy: Weaponizing Client Automation

The major flaws in this chapter weaponize browser functionality—session cookies, redirection behavior, and screen rendering.

#### A. Cross-Site Request Forgery (CSRF)

CSRF forces an authenticated user to execute state-changing actions on a trusted web application without their intent.

- **Mechanism:** An attacker embeds a malicious request (e.g., change password, transfer funds) on a site they control. When the victim views the malicious site, their browser automatically includes the target domain’s session cookies, authenticating the forged request.
- **Defense Failure:** The application checks authentication (the session cookie) but fails to check authorization proof (an unpredictable anti-CSRF token) that proves the request originated from the official, authorized application interface.

#### B. Open Redirect

This flaw uses a trusted domain to redirect the user to an arbitrary, malicious destination.

- **Mechanism:** The application accepts user input (via a parameter like `redirect_url` or `next`) and uses it without strict validation to send the user to another page (302/301 status code).
- **Weaponization:** Open Redirects are often chained with phishing attacks (the trusted domain hides the malicious destination) or, critically, **OAuth Token Theft**, where the redirect is leveraged to steal temporary authorization codes in SSO/SPA flows.

#### C. HTTP Header Injection (HHI) / Response Splitting (HRS)

These flaws target parameters reflected into HTTP response headers, allowing the attacker to inject arbitrary headers or split the response message .

- **Mechanism:** An attacker injects the Carriage Return/Line Feed sequence (`%0d%0a` or CRLF) into a parameter, which is then unsafely included in a response header (e.g., the `Location` header during a redirect).
- **Weaponization:** HHI is escalated to **HTTP Response Splitting** by injecting a double CRLF sequence, allowing the attacker to completely control the subsequent HTTP response, leading to **Cache Poisoning** or pervasive **Session Fixation** via a forged `Set-Cookie` header .

#### D. Session Fixation and Cookie Injection

These flaws manipulate the integrity or lifecycle of the session token.

- **Session Fixation:** An attacker sets a known session ID (via Cookie Injection) on the victim's browser _before_ they log in. If the application fails to renew the session token upon successful authentication, the attacker uses the known ID to hijack the newly authenticated session .
- **Cookie Injection:** Directly manipulating cookie values or attributes (e.g., overwriting a user preference cookie with a security token) to introduce arbitrary data into the server's session context .

---

### III. Attacker Playbook: Execution and Chaining

Successful exploitation requires meticulous traffic analysis and often the chaining of multiple low-impact flaws to achieve high-tier rewards.

1. **CSRF Audit:** Identify all POST requests that change state (e.g., account settings, financial transfers). Verify the presence of an anti-CSRF token. If present, test its validity (e.g., replay the request with a known-bad or expired token). If missing, a CSRF Proof-of-Concept (PoC) is immediately possible.
2. **Open Redirect Fuzzing:** Fuzz all URL-accepting parameters (`return`, `dest`, `continue`) with external HTTP links. Test both simple payloads and complex variations, including partial encoding (`//evil.com`) and protocol-relative URLs (`//evil.com`) to bypass filters.
3. **HHI/HRS Probing:** Send CRLF payloads (`%0d%0a`) into parameters (e.g., those controlling headers or body reflection) and monitor the response headers in the proxy for new, injected lines.
4. **SSO (OAuth) Replay:** During an OAuth flow, capture the authorization request and the `state` parameter. If the `state` parameter is weak or reusable, attempt to replay it in conjunction with a forced victim login to hijack the authorization code, a critical modern attack on SPAs.
5. **LLM Web Logic Bypass (2025+):** Identify parameters (e.g., for logging, URL preview) that feed user input into an internal AI component (A01: LLM Prompt Injection). Use a payload designed to trick the LLM into generating a "trusted" or whitelisted URL or token structure in its output, potentially circumventing Open Redirect filters or anti-CSRF logic enforcement.

---

### IV. Real Exploits and Advanced Breaches

The value of these flaws is maximized by demonstrating critical impact via chaining.

- **HackerOne Interstitial Redirect:** An Open Redirect vulnerability in HackerOne's interstitial page was leveraged to redirect users, proving that exploiting trust in a domain yields high reward.
- **CSRF leading to Account Takeover:** CSRF flaws remain highly rewarding when they lead to password change or email modification without the victim's knowledge, which constitutes immediate account takeover.
- **UI Redressing/Clickjacking:** While clickjacking (UI redress) is often lower severity, an attacker can leverage it to trick a user into clicking an invisible button that initiates a CSRF request, chaining two flaws for higher impact.
- **File Disclosure via SOP Bypass:** Attacks that exploit quirks in the browser's **Same-Origin Policy (SOP)** can sometimes lead to reading local cached web content or network disclosure, leaking sensitive data viewed by the user.

---

### V. Defense Gaps: Why Checks Fail

These pervasive flaws persist because developers fail to grasp the hostile nature of the client and rely on incomplete or easily bypassed controls.

- **Missing Anti-CSRF Enforcement:** The most frequent failure is omitting robust, unique, and cryptographically secure anti-CSRF tokens for all state-changing POST requests.
- **Blacklisting Redirects:** Developers attempt to secure Open Redirects by filtering known bad domains, a method easily defeated by obfuscation, encoding, or using protocol-relative/partial URL payloads. **Strict whitelisting** is the only defense.
- **Server-Side State Confusion:** Session Fixation succeeds because the server fails to manage session state transitions atomically; it updates authorization but keeps the old, attacker-known session identifier .
- **Header Trust:** Failure to strictly sanitize input reflected into headers allows CRLF injection to execute, often due to poor use of functions like `header()` in vulnerable ways.

---

### VI. One-Liner

Use `ffuf` to quickly test common Open Redirect parameters, identifying responses that redirect to a custom, non-whitelisted target (e.g., your collaborator link):

```
ffuf -w /path/to/redirect_params.txt -u https://target.com/login?FUZZ=http://attacker.com -mc 301,302,307 -H "Cookie: Session=[TOKEN]"
```

_Purpose: Fuzzes a wordlist of common redirect parameters (`redirect`, `next`, `url`) and monitors for status codes (3xx) indicating a redirect. This confirms if the application accepts an arbitrary, unauthorized external URL, validating the Open Redirect flaw._