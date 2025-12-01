Directory traversal
- Client → HTTP Request → Web Server → App Code → File System Read → Response → Client. The vuln is in the "App Code" trusting bad input
- Read arbitrary files on the server that is running an application
- Application code and data
- Credentials for back-end systems
- Sensitive operating system files
- `../ or ..\ | GET /image?filename=../../../etc/passwd`
Access Control
- **Authentication**
- **Session management** identifies which subsequent HTTP requests are being made by that same user
- **Access control** determines whether the user is allowed to carry out the action that they are attempting to perform
- Robots.txt
- Parameter-based
  - A hidden field
  - A cookie
  - A preset query string parameter
- Nested traversal sequences, such as `....//` or `....\/`