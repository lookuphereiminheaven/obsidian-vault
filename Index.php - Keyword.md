Credentials and Secrets
- **Database Credentials:** `mysqli_connect()`, `$db_user`, `$password`, `$host`, or connection strings (e.g., `mysql://user:pass@host:port`
- API Keys: AWS, Stripe, Google Maps
- Symmetric Keys/Salts: `$secret_key` or `$salt`
- Default Passwords: `if (user == 'admin') { password = 'password123'; }`).
Hidden Paths and Files
- Includes/Requires: `include()`, `require()`, `include_once()`, or `require_once()`. The files referenced (e.g., `config.php`, `admin/panel.php`, `utility/db.inc`) are new, undiscovered targets.
- **URL Endpoints:** Check for logic that handles different pages or parameters (e.g., `if (isset($_GET['page']))`), which often reveals hidden pages like `/backup`, `/dev`, or `/test`
- Log Files:   /var/log/application.log
Security Vulnerabilities
- **File Inclusion (LFI/RFI):** Check functions that allow the user to specify a file path, such as `include($_GET['file'])`. This is a major vulnerability.
- **Command Injection:** Look for functions that execute system commands, especially if user input is involved: `system($_POST['cmd'])`, `exec()`, `shell_exec()`
- **SQL Injection (SQLi):** Code that directly inserts user input into a database query without using prepared statements (e.g., `SELECT * FROM users WHERE username = '{$_POST['user']}'`
Technology and Comments
- **Comments:** Developers often use comments (`//` or `/* */`) to leave notes, to-do lists, or explanations, which can be the easiest way to find passwords or future plans.
- **Outdated Functions:** Look for outdated or deprecated functions, as they might indicate the application is old and vulnerable
- **Version Numbers:** Any references to the application framework's version or specific library versions (e.g., "Running CMS version 1.2") can be used to search for known public exploits.
**RCE (Remote Command Execution):** Look for functions like `system()`, `exec()`, or `shell_exec()` tied to **`$_GET[]`** or **`$_POST[]`** user input.
#### **Key Keywords***
- RCE (Remote Command Execution): Look for functions like system(), exec(), or shell_exec() tied to $_GET[] or $_POST[] user input.
- LFI/RFI (File Inclusion): Look for include() or require() where the filename is controlled by a user parameter like $_GET['file'].
- SQLi (SQL Injection): Look for database queries using SELECT or UPDATE where user input is directly concatenated without using prepared statements.