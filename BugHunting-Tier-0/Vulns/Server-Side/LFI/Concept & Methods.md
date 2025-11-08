- Which aspect of the file the website fails to validate properly, whether that be its size, type, contents, and so on
- What restrictions are imposed on the file once it has been successfully uploaded
- Historically, websites consisted almost entirely of static files that would be served to users when requested. As a result, the path of each request could be mapped 1:1 with the hierarchy of directories and files on the server's filesystem. Nowadays, websites are increasingly dynamic and the path of a request often has no direct relationship to the filesystem at all. Nevertheless, web servers still deal with requests for some static files, including stylesheets, images, and so on.
- The `Content-Type` response header may provide clues as to what kind of file the server thinks it has served. If this header hasn't been explicitly set by the application code, it normally contains the result of the file extension/MIME type mapping
- `<?php echo file_get_contents('/path/to/target/file'); ?>`
- `<?php echo system($_GET['command']); ?>`
  `GET /example/exploit.php?command=id HTTP/1.1`
- before an Apache server will execute PHP files requested by a client, developers might have to add the following directives to their `/etc/apache2/apache2.conf` file
  `LoadModule php_module /usr/lib/apache2/modules/libphp.so AddType application/x-httpd-php .php`
Methods
- Exploiting unrestricted file uploads to deploy a web shell
- Exploiting flawed validation of file uploads
- Web shell upload via Content-Type restriction bypass
- Web shell upload via path traversal
- Insufficient blacklisting of dangerous file types
- Web shell upload via extension blacklist bypass
- Web shell upload via obfuscated file extension
- Remote code execution via polyglot web shell upload
- Exploiting file upload race conditions
- Uploading malicious client-side scripts
- Exploiting vulnerabilities in the parsing of uploaded files
- Uploading files using PUT