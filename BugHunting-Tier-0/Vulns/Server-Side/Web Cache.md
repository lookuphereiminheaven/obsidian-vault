- Sits between user and origin server
- Stores static resources for faster delivery
- Dynamic content is not cached as it's more likely to contain sensitive information
#### Cache Workflow
- **Cache Miss** → Request → Origin Server → Response → Cache → User (cached if rules allow)
- **Cache Hit** → Cache serves stored copy instantly
#### Key Players
- **CDNs**: Global networks of caches
- **Benefit**: Serve content from nearest edge node → lower latency
#### Cache Deception (Attack)
- **Exploit**: Trick cache into storing dynamic/private pages as if static
- Focus on endpoints that support the `GET`, `HEAD`, or `OPTIONS` methods as requests that alter the origin server's state are generally not cached
- Identify a discrepancy in how the cache and origin server parse the URL path
    - Map URLs to resources.
      - ```
          ```<html>
          <body
          <script>document.location="https://0a200029049ff1ff802fc6660049002d.web-security-academy.net/my-account/tracking.js"
          </script>
          </body>
          </html>
        ```
    - Process delimiter characters.
      - ``` document.location=https://0adf006e035d5bb380b1cb460008009b.web-security-academy.net/my-account/?p.js```
    - Normalize paths.
      - converting various representations of URL paths into a standardized format
      - an exploitable normalization discrepancy requires that either the cache or origin server decodes characters in the path traversal sequence as well as resolving dot-segments
      - When testing for normalization, start by encoding only the second slash in the dot-segment. This is important because some CDNs match the slash following the static directory prefix
      - You can also try encoding the full path traversal sequence, or encoding a dot instead of the slash. This can sometimes impact whether the parser decodes the sequence
      - You can also add a path traversal sequence after the directory prefix. For example, modify `/assets/js/stockCheck.js` to `/assets/..%2fjs/stockCheck.js`
```
      - <script>
      document.location="https://0a230047045993b880faa8f8000d00d0.web-security-academy.net/resources/..%2fmy-account?abc"
    </script>
```
 - Craft a malicious URL that uses the discrepancy to trick the cache into storing a dynamic response
- **Result**: Sensitive data (profile, tokens) served publicly to next visitor
##### Rules
- Static file extension rules
  - These rules match the file extension of the requested resource, for example `.css` for stylesheets or `.js` for JavaScript files.
- Static directory rules
  - These rules match all URL paths that start with a specific prefix
  - These are often used to target specific directories that contain only static resources, for example `/static` or `/assets`.
- File name rules
  - These rules match specific file names to target files that are universally required for web operations and change rarely, such as `robots.txt` and `favicon.ico`
  #### Headers
  - `X-Cache: hit`
  - `X-Cache: miss`
  - `X-Cache: dynamic`
  - `X-Cache: refresh`
#### Path mapping discrepancies
- Traditional URL
- REST-style URL