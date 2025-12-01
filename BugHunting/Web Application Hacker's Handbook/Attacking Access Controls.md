### I. Core Model: The Failed Authorization Check

Access control flaws are conceptually simple: the application trusts the session token (Authentication/Session Management) but fails to rigorously check *authorization* for the specific resource or function requested.

*   **Vertical Escalation:** Attacker gains higher privileges (e.g., standard user accesses admin panel).
*   **Horizontal Escalation (IDOR):** Attacker accesses resources belonging to another user at the same privilege level (e.g., viewing another user's profile or data).
*   **Trust Boundary Violation:** The core failure is when the application relies on client-supplied data (e.g., a URL parameter or hidden field) to determine authorization, instead of using the secure, server-side session context.

---

### II. Flaw Taxonomy: Weaponizing Access Defects

Attackers exploit the differences in how the application segregates users and controls access to data and functions.

#### A. Insecure Direct Object Reference (IDOR) / BOLA
This is the modern, scalable manifestation of horizontal privilege escalation.

*   **Mechanism:** The application retrieves a resource (user profile, invoice, document) using a user-controllable identifier (e.g., `user_id=123`) without verifying that the authenticated user owns that ID.
*   **Weaponization:** Harvest easily guessable identifiers (sequential integers, predictable UUIDs) during reconnaissance. Systematically substitute these harvested IDs into requests, escalating access across arbitrary user accounts.
*   **Modern Scale (GraphQL/BOLA):** In GraphQL APIs, once a sequential identifier pattern is confirmed, an attacker leverages **batching attacks** to submit hundreds of manipulated IDs simultaneously, mass-exfiltrating unauthorized data pools efficiently.

#### B. Parameter-Based Access Control
Developers often attempt to enforce authorization by checking client-submitted fields that determine the user's role or status.

*   **Mechanism:** The application trusts parameters like `role`, `privilege`, or Boolean flags (`admin=true`) submitted by the client.
*   **Weaponization:** Use an intercepting proxy to modify these values, switching a low-privilege flag (`admin=false`) to a high-privilege value (`admin=true`) in the URL query string or POST body. This must be tested on both GET and POST requests, as controls might be missing in one method.

#### C. Flawed Functional Segregation (Forced Browsing)
This targets controls that rely purely on obscurity or UI enforcement.

*   **Mechanism:** High-privilege functions or resources are merely unlinked from the main UI, assuming that users won't guess the URL.
*   **Weaponization:** Employ **forced browsing** by navigating directly to known or guessed sensitive URLs (e.g., `/admin/config.jsp`, `/api/v2/users/`) using a standard user session. This is often the quickest path to vertical privilege escalation.
*   **Defense Failure:** Controls are often **declarative** (UI layer) rather than **imperative** (server code validation), meaning the server doesn't re-check authorization upon direct access.

---

### III. Attacker Playbook: Rigorous Authorization Audit

Systematic access control testing relies on multi-user context and detailed traffic analysis (Chapter 4,).

1.  **Context Mapping:** Map the entire application using both a low-privileged user (e.g., standard user) and a high-privileged user (e.g., administrator). Record all discovered functionality for each role.
2.  **Horizontal Attack Chain:**
    *   Harvest all observable resource identifiers (IDs, usernames, tokens) from API responses and HTML content.
    *   Use the low-privileged session to replay requests, substituting the harvested ID with an ID belonging to another user. Look for 200/302 responses or unexpected data disclosure.
3.  **Vertical Attack Chain (Forced Browsing):**
    *   Compile a list of all administrative/sensitive URLs and parameters found during high-privilege mapping.
    *   Replay these sensitive requests using the low-privileged session, watching for unauthorized success (e.g., successful access to `/admin/logs` or modification of another user's password).
4.  **Parameter Tampering:** Identify parameters that seem to govern internal state or authorization (e.g., `userType`, `is_premium`) and use Burp Repeater to modify them to higher privilege values. Test removal of parameters (parameter skipping) to check for fail-open logic.
5.  **Multistage Flow Bypass:** Analyze multi-step processes (e.g., password reset, checkout) and attempt to skip steps or submit the same data multiple times to violate assumed sequences.

---

### IV. Real Exploits and Advanced Breaches

Authentication failures often result from complex logic oversights and are highly valued by bug bounty programs.

*   **GitLab 2FA Bypass:** A classic application logic flaw where the two-factor authentication flow was bypassed by exploiting a defective logic check in a peripheral function, granting access without credentials. This highlights the necessity of testing all related identity functions, not just the main login.
*   **Shopify Admin Authentication Bypass:** A logic flaw allowed an attacker to gain admin privileges by exploiting defective application logic and assumptions, illustrating the high value of identifying and attacking logical defects over simple syntax errors.
*   **AI Supply Chain Access Control Flaws (2025+):** If user-controlled input (e.g., a URL parameter) is passed to an internal AI component (like a LangChain wrapper or custom model), an attacker can use sophisticated **LLM Prompt Injection** techniques. This payload could bypass web logic, tricking the LLM into accessing and outputting sensitive data (e.g., PII or internal files) that the user is not authorized to see, effectively turning an LLM into an authorization oracle.
*   **JWT Token Forgery:** When applications use JWTs for authorization, a failure to validate the cryptographic signature allows forgery. Setting the `alg` parameter to `none` or successfully executing a **HS256 downgrade attack** allows the attacker to embed a high-privilege user ID in the token payload and bypass access controls entirely.

---

### V. Defense Gaps: Why Checks Fail

The persistent existence of BAC (A01) proves developers routinely fail to implement authorization checks effectively across all code paths.

*   **Incomplete Enforcement:** Developers often apply robust checks only to the main UI entry point, forgetting to replicate these checks in underlying APIs or secondary methods (like PUT/DELETE methods).
*   **Trusting the Client:** Authorization parameters (like the user role) are read directly from the client request (URL or hidden field) and trusted, rather than being retrieved exclusively from the secure, server-side session context. This is a fundamental security architecture failure.
*   **Reliance on Frameworks:** Declarative access controls provided by frameworks are sometimes insufficient or incorrectly configured, failing to stop determined attackers using unconventional methods or custom HTTP requests.

---

### VI. One-Liner

Use `ffuf` to enumerate sequential user identifiers in an API endpoint, looking for unauthorized content access (IDOR/BOLA):

```bash
ffuf -w wordlist/numbers:1-1000 -u https://target.com/api/v1/users/FUZZ/data -H "Authorization: Bearer [LOW_PRIV_TOKEN]" -fs 0,2100 -mc 200,403
```
*Purpose: Cycles through user IDs 1 to 1000, suppressing responses that match the size of common error pages (e.g., 403 Forbidden or "Not Found"). Successful access (200 OK) or unexpected lengths indicate an IDOR hit.*