### I. Fundamental Message Structure and Flow

An HTTP transaction is fundamentally composed of a request command and a corresponding response result, facilitated by structured data blocks known as HTTP messages.

#### A. Message Anatomy

Every HTTP message, whether a request or a response, is composed of three logically distinct sections:

1. **Start Line:** Describes the message, indicating the action requested or the outcome achieved.
2. **Headers:** A block of zero or more lines containing attributes or metadata about the message, its sender, or the payload.
3. **Entity Body (Body):** An optional component containing the actual data payload.

HTTP messages are inherently textual, consisting of simple, line-oriented sequences of characters, often utilizing plain ASCII text. The termination of each line is mandated by the two-character sequence, carriage return and line feed (CRLF).

#### B. Message Directionality

The flow of messages is defined by two pairs of descriptive terms:

- **Inbound and Outbound:** Messages travel **inbound** toward the origin server and subsequently travel **outbound** back toward the user agent (client) once the transaction's work is concluded.
- **Downstream and Upstream:** All messages flow **downstream**. The fundamental orientation dictates that the sender of any message resides **upstream** of the receiver.

### II. The Start Line: Command and Status

The initial line of the message sets the functional context for the transaction.

#### A. Request Message Start Line (Request Line)

The request message begins with the request line, which contains three crucial elements separated by whitespace:

1. **Method:** The specific action the client wants the server to perform.
2. **Request-URL:** A complete URL or the absolute path component naming the targeted resource.
3. **Version:** The version of the HTTP protocol being utilized by the client (e.g., `HTTP/1.1`).

#### B. Response Message Start Line (Response Line)

The response message begins with the response line, which carries status information back to the client. It contains three elements separated by whitespace:

1. **Version:** The HTTP version used by the response.
2. **Status Code:** A three-digit numeric code detailing the outcome of the request.
3. **Reason Phrase:** A textual explanation intended for descriptive purposes, accompanying the numeric status code (e.g., "OK").

### III. HTTP Methods: Defining the Action

HTTP methods are the commands sent in the request message that instruct the server on the desired operation.

#### A. Common Methods (Highlighted Parts)

The core specification defines a mandatory set of operational methods:

- **GET:** The most common method, used to request a server to send a resource to the client.
- **HEAD:** Behaves exactly like `GET`, but the server returns _only_ the headers, with no entity body. This allows a client to inspect metadata (e.g., content type or modification status) without transferring the entire object. **Highlight:** HTTP/1.1 compliance necessitates implementation of both the `GET` and `HEAD` methods.
- **POST:** Used to send input data to the server for processing, commonly supporting HTML forms.
- **PUT:** Designed to write documents or store data onto a named server resource.
- **DELETE:** Requests the server to remove the specified resource.
- **TRACE:** Initiates a "loopback" diagnostic to see how the request message looks upon reaching the destination server, capturing any modifications made by intervening proxies.
- **OPTIONS:** Asks the server to report its supported capabilities and methods, either generally or for a specific resource.

#### B. Extension Methods

HTTP is designed to be extensible, allowing servers to implement custom request methods that extend the protocol beyond the standard specification.

### IV. Status Codes: Categorizing the Result

The status code provides an immediate, systematic way for clients to comprehend the result of their transactions. Status codes are segmented into five classes based on their initial digit:

|Range|Category|Meaning|
|:-:|:-:|:--|
|**100–199**|Informational|Used for provisional responses and ongoing transaction information. **Highlight:** This class, introduced in HTTP/1.1, includes `100 Continue`, intended to optimize scenarios where a client sends a large entity body, allowing it to check server acceptance before transmitting the full payload.|
|**200–299**|Successful|Indicates that the client's request was successfully received, understood, and accepted (e.g., `200 OK`).|
|**300–399**|Redirection|Indicates that further action is required to complete the request, often directing the client to a new location (e.g., `302 Redirect`, `304 Not Modified`).|
|**400–499**|Client Error|Indicates an error arising from the client (e.g., `404 Not Found`, `401 Unauthorized`).|
|**500–599**|Server Error|Indicates a failure or inability to perform the request due to server fault.|

**Highlight:** If an application encounters an unrecognized status code, robust practice dictates treating it as a generic member of its numerical class.

### V. Header Fields: Supplementary Metadata

HTTP header fields are name-value pairs that supply additional metadata and configuration information to request and response messages. They are separated from the start line and the body by blank lines.

Headers are broadly categorized into four standard classes:

1. **General Headers:** Provide basic information applicable to both request and response messages (e.g., `Date`, `Connection`, `Transfer-Encoding`).
2. **Request Headers:** Supply more context about the request, the client's preferences, and its capabilities.
    - **Accept Headers:** Inform the server about the media types, character sets, languages, and encodings the client can handle or prefers (e.g., `Accept-Language`, `Accept-Encoding`).
    - **Conditional Headers:** Restrict the conditions under which the request should be fulfilled (e.g., `If-Modified-Since`, `If-None-Match`).
3. **Response Headers:** Offer supplementary information about the response or the server itself (e.g., `Server`, `Retry-After`).
4. **Entity Headers:** Describe the entity body being transported, including its dimensions and content attributes (e.g., `Content-Type`, `Content-Length`, `ETag`).
5. **Extension Headers:** Custom headers not defined in the official specification.

### VI. Entity Body: The Data Payload

The entity body is the payload section of the HTTP message, which is optional. It follows the headers, separated by a blank CRLF line. Unlike the highly structured and textual start line and headers, the entity body contains the raw cargo—arbitrary data that can be textual or binary, such as images, videos, or HTML content. Entity headers (Section V) are crucial for describing how the client should interpret this raw data.