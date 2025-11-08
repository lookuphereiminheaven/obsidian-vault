Configuration Obfuscation (The .htaccess Attack)

This is the multi-step attack you were working on, which uses a file upload to change the server's execution rules.

    Step 1: Upload a file named .htaccess (often blocked, but sometimes not) with the content: AddType application/x-httpd-php .l33t.

    Step 2: Upload your web shell payload with the filename shell.l33t.

How to Implement in Repeater:

    Perform the first POST request with filename=".htaccess" and the Apache directive as the payload.

    Perform the second POST request with filename="shell.l33t" and your PHP code as the payload.

This is the most powerful method, as it allows you to choose an arbitrary, un-blacklisted extension (.l33t) and force the server to execute it as code.