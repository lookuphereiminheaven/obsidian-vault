### I. The Nature and Role of HTTP

HTTP, the Hypertext Transfer Protocol, functions fundamentally as the Internet’s Multimedia Courier. Its primary function is to move vast quantities of diverse data—including HTML pages, JPEG images, audio files, and software applets—quickly, conveniently, and reliably across the Internet from servers to clients.

#### A. Web Clients and Servers

The fundamental architecture of the World Wide Web consists of HTTP clients and HTTP servers.

1. **Web Servers:** These entities, often referred to as HTTP servers, store the Internet's data and communicate using the HTTP protocol. They fulfill requests sent by clients.
2. **Web Clients:** The most common client is a web browser (e.g., Microsoft Internet Explorer or Netscape Navigator), which sends HTTP requests to servers and displays the retrieved objects. A client sends a request for an object (e.g., `/index.html`), and the server responds with the requested data, along with metadata such as the object's type and length.

### II. Resource Identification and Typing

HTTP facilitates the exchange of _resources_, which are defined broadly as any kind of content source. Resources can be static (like an image file or a spreadsheet) or dynamic (like an Internet search engine or a web gateway to a database).

#### A. Uniform Resource Identifiers (URIs)

Every web server resource is assigned a name called a uniform resource identifier (URI). URIs function analogously to postal addresses for the Internet, serving to uniquely identify and locate resources globally. The common format used in HTTP applications is the URL (Uniform Resource Locator), a subset of the URI concept. A URL specifies three crucial elements: the protocol (`scheme`), the physical location (`host`), and the specific item on that location (`path`). For instance, a URL often dictates the protocol used for access (e.g., `http`). Although technically distinct, URIs and URLs are used interchangeably throughout the discussion in the source material.

#### B. Media Types (MIME)

To ensure proper processing of content, HTTP employs a data format label called a MIME type (Multipurpose Internet Mail Extensions) to tag every object being transported. MIME types permit the client to ascertain how to handle the received object—for example, whether to display it as an image, format it as HTML, or launch a plug-in. A MIME type is a textual label structured as a primary object type and a specific subtype, separated by a slash (e.g., `text/html`, `image/gif`, or `application/vnd.ms-powerpoint`).

### III. HTTP Transactions and Messages

The communication cycle between client and server is termed an HTTP transaction, which is fundamentally composed of a request command and a corresponding response result, facilitated by formatted blocks of data called HTTP messages.

#### A. HTTP Methods and Status Codes

1. **Methods:** These are the request commands that specify the action the server should perform, such as fetching a page or deleting a file. Examples of common methods include `GET` (send resource to client), `PUT` (store data into a named resource), `DELETE` (remove the named resource), `POST` (send data to a gateway application), and `HEAD` (send only the headers for a resource).
2. **Status Codes:** Every HTTP response carries a three-digit numeric status code, along with an explanatory textual "reason phrase" (e.g., "OK"). The numeric code is used for processing and informs the client whether the request succeeded or if further action is necessary (e.g., `200 OK`, `302 Redirect`, `404 Not Found`).

#### B. Message Structure

HTTP messages are textual, line-oriented sequences of characters, often utilizing plain ASCII text. They come in two varieties: request messages (client to server) and response messages (server to client). Each message is comprised of three distinct parts:

1. **Start Line:** The initial line of the message, indicating the action requested (for requests) or the outcome (for responses).
2. **Header Fields:** Zero or more lines following the start line, composed of name-value pairs separated by a colon (`:`). These headers conclude with a blank line, demarcating the transition to the body.
3. **Body:** An optional message body that follows the blank line. Unlike the structured and textual start line and headers, the body contains the actual data payload, which can be arbitrary binary data (images, videos) or text.

### IV. Network Transport Layer

HTTP functions as an application layer protocol, abstracting away the intricacies of network communication by relying on TCP/IP (Transmission Control Protocol/Internet Protocol).

#### A. TCP/IP Reliability

TCP/IP is the reliable Internet transport protocol that ensures data transportation is error-free, preventing loss, damage, or out-of-order reception of bytes flowing between client and server.

#### B. Addressing and Connection

To initiate a transaction, the client must identify the server's IP address and port number from the URL. Hostnames (textual domain names) are converted to IP addresses via the Domain Name Service (DNS). By default, HTTP uses port 80. The transaction sequence involves the browser establishing a TCP connection using the determined IP address and port, sending the HTTP request, receiving the response, and then closing the connection.

### V. Evolution of the Protocol and Architecture

The HTTP protocol has undergone significant evolution, leading to enhanced versions and a complex web architecture relying on specialized components.

#### A. Protocol Versions

- **HTTP/0.9 (1991):** The initial minimalist protocol.
- **HTTP/1.0 (Foundation):** Describes the modern foundation for HTTP.
- **HTTP/1.1 (Current):** Focused on correcting architectural flaws, introducing performance optimizations, and removing "mis-features". This is the current version of HTTP.
- **HTTP-NG (HTTP/2.0):** A prototype proposal focused on significant performance improvements and a more robust framework for remote server logic execution; research concluded in 1998.

#### B. Architectural Components

The functioning Web relies on several crucial architectural components beyond basic clients and servers:

1. **Proxies:** HTTP intermediaries that sit between clients and servers, relaying and potentially modifying requests. They are used for security (acting as trusted intermediaries) and filtering.
2. **Caches:** Specialized HTTP proxy servers (caching proxies) that store local copies of frequently requested documents. Caches improve performance and reduce network traffic by serving copies from local storage.
3. **Gateways:** Special servers that act as intermediaries to other servers, often converting HTTP traffic to a different protocol (e.g., an HTTP/FTP gateway). A gateway appears to the client as the origin server for the resource.
4. **Tunnels:** Special proxies that are designed to blindly forward HTTP communications. They are typically used to convey non-HTTP traffic, such as encrypted Secure Sockets Layer (SSL) protocols, through HTTP connections.
5. **Agents (Robots):** Semi-intelligent web clients that execute automated HTTP requests, such as crawlers and spiders used by search engines.