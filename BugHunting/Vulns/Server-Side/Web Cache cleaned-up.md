# Web Cache Overview
- **Sits between user and origin server**: Acts as an intermediary layer, intercepting requests from clients (e.g., browsers) before they reach the backend server. Practical explanation: In a real-world scenario, when a user requests a webpage like `example.com/home` , the cache checks if it has a stored version; if not, it forwards the request to the origin server, reducing load on the server for repeated requests.

- **Stores static resources for faster delivery**: Caches files like images, CSS, or JS that don't change often. Practical explanation: For a site like an e-commerce platform, static assets such as product images are cached, so subsequent visitors load them quickly from the cache instead of fetching from the server each time, improving page load times and reducing bandwidth costs.

- **Dynamic content is not cached, as it often contains sensitive information**: Personalized data like user profiles or session-specific info is bypassed. Practical explanation: A logged-in user's dashboard showing their email or balance isn't cached to prevent one user's data from being served to another, avoiding privacy breaches in applications like banking sites.

## Cache Workflow
- **Cache Miss**: Request → Origin Server → Response → Cache → User (cached if rules allow). Practical explanation: If a user requests a new image /images/logo.png not yet in cache, the cache forwards it to the server, gets the response, stores it (if it's static and cacheable), and sends it to the user—subsequent requests for the same image hit the cache directly.

- **Cache Hit**: Cache serves stored copy instantly. Practical explanation: For a popular CSS file /styles/main.css already cached, the cache immediately returns it to the user without contacting the origin server, speeding up load times during high traffic, like on a news site during breaking news.

## Key Players
- **CDNs**: Global networks of caches. Practical explanation: Services like Cloudflare or Akamai distribute caches worldwide; for a global audience, a user in Europe gets content from a nearby European cache node instead of a US-based server, minimizing latency.

- **Benefit**: Serve content from nearest edge node for lower latency. Practical explanation: In video streaming like Netflix, CDNs cache movies regionally— a viewer in Asia streams from an Asian node, reducing buffering and improving experience compared to fetching from a distant central server.

## Cache Deception Attack
- **Exploit**: Trick cache into storing dynamic/private pages as if static. Practical explanation: An attacker crafts a URL that looks static to the cache (e.g., appending .js) but dynamic to the server, causing private user data like API keys to be cached and accessible to others, as seen in web security labs where victim data is exposed.

- **Focus on endpoints supporting `GET`, `HEAD`, or `OPTIONS` methods, as state-altering requests are generally not cached**: Targets read-only requests. Practical explanation: Attackers probe GET /user/profile instead of POST /update-profile, since caches ignore POSTs (which change data) but may store GET responses, enabling deception on read endpoints like account views.

- **Identification**: Find discrepancies in how cache and origin server parse URL paths. Practical explanation: Test URLs to see if the server ignores extra path segments (e.g., /my-account/foo.js serves /my-account content) while the cache treats it as a unique static file, revealing vulnerabilities through tools like Burp Suite.

  - **Map URLs to Resources**:
    ```
    <html>
      <body>
        <script>document.location="https://0a200029049ff1ff802fc6660049002d.web-security-academy.net/my-account/tracking.js"</script>
      </body>
    </html>
    ```
    Practical explanation: This HTML redirects a victim to a deceptive path /my-account/tracking.js; the server maps it to /my-account (dynamic page with sensitive data), but the cache stores it as static JS, allowing attackers to retrieve cached victim data later.

  - **Process Delimiter Characters**:
    ```
    document.location="https://0adf006e035d5bb380b1cb460008009b.web-security-academy.net/my-account/?p.js"
    ```
    Practical explanation: Uses ? as a delimiter; the server might truncate at ? and serve /my-account, ignoring ?p.js as query params, while the cache sees the full URL as cacheable, enabling storage of dynamic content in deception attacks.

  - **Normalize Paths**: Convert URL paths to standardized format.
    - **Exploitable discrepancy requires decoding characters in path traversal sequences and resolving dot-segments by cache or origin server**: Practical explanation: If the cache decodes %2f to / and resolves ../, but the server doesn't (or vice versa), attackers can craft paths like /static/../private to access and cache restricted areas.
    - **Test by encoding only the second slash in dot-segment (e.g., important for CDNs matching slashes after static directory prefix)**: Practical explanation: For /assets/js/../secret.js, encode as /assets/js/..%2fsecret.js; if CDN matches /assets/js/ prefix but resolves traversal differently, it might cache sensitive /secret.js as static.
    - **Try encoding full path traversal sequence or a dot instead of slash to affect parser decoding**: Practical explanation: Encode ../ as %2e%2e%2f; varying encoding can bypass one parser's normalization while triggering the other's, allowing paths like /public%2e%2e%2fprivate.js to be cached incorrectly.
    - **Add path traversal after directory prefix (e.g., `/assets/js/stockCheck.js` → `/assets/..%2fjs/stockCheck.js`)**:
      ```
      <script>
        document.location="https://0a230047045993b880faa8f8000d00d0.web-security-academy.net/resources/..%2fmy-account?abc"
      </script>
      ```
      Practical explanation: Redirects to /resources/../my-account; if the server normalizes to /my-account (dynamic) but cache sees /resources/..%2fmy-account as under static /resources/, it caches private data, exposing it via the normalized path.

- **Craft Malicious URL**: Use discrepancy to store dynamic response in cache. Practical explanation: Combine findings, e.g., append ;foo.js to a dynamic path if ; is a server delimiter but not for cache, forcing caching of private pages when a victim loads it.

- **Result**: Sensitive data (e.g., profiles, tokens) served publicly to next visitor. Practical explanation: After deception, anyone requesting the crafted URL gets the cached victim data, like an API key, leading to account compromise in vulnerable apps.

## Caching Rules
- **Static File Extension**: Matches extensions like `.css` for stylesheets or `.js` for JavaScript. Practical explanation: Caches apply rules to store .jpg files long-term; attackers exploit by appending .js to dynamic URLs, tricking rules into caching personalized content as script.

- **Static Directory**: Matches paths starting with prefixes like `/static` or `/assets` for static resources. Practical explanation: All files under /images/ are cached; deception might involve path traversal to make dynamic paths appear under /static/, caching them indefinitely.

- **File Name**: Matches specific files like `robots.txt` or `favicon.ico` that rarely change. Practical explanation: These are always cached for efficiency; rarely exploited, but if a dynamic endpoint mimics robots.txt, it could lead to unintended caching of sensitive info.

## Cache Headers
- **`X-Cache: hit`**: Indicates response from cache. Practical explanation: On second request to a cached URL, this header confirms storage; attackers check it to verify deception success before retrieving data.

- **`X-Cache: miss`**: Response fetched from origin. Practical explanation: First request to a new deceptive URL shows this; subsequent ones switch to hit if cached, signaling the attack worked.

- **`X-Cache: dynamic`**: Bypassed cache for dynamic content. Practical explanation: Seen on personalized pages; deception aims to avoid this by making dynamic URLs appear static, forcing a hit instead.

- **`X-Cache: refresh`**: Cache updating stale content. Practical explanation: If max-age expired, cache refreshes from server; attackers time exploits within cache lifetime to access fresh poisoned data.

## Path Mapping Discrepancies
- **Traditional URL**: Fixed paths like /about.html. Practical explanation: Servers map exactly; discrepancies arise if cache keys on full URL but server ignores extras, e.g., /about/foo.html serves /about but caches as unique.

- **REST-style URL**: Parametric like /users/{id}. Practical explanation: Servers use regex to map /users/123/details to user resource; cache might not, treating /users/123/foo.js as separate static file, enabling caching of user-specific data.