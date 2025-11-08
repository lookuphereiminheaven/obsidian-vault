- Declarative language for managing and querying relational databases, based on set theory, relational algebra, and Boolean conditions
###### Parsing and Execution Pipeline
  - **Lexical Analysis:** Breaks the statement into tokens (e.g., keywords like SELECT, operators like =, literals like 'admin'
  - **Syntax Parsing:** Builds a parse tree ensuring grammatical correctness (e.g., WHERE clause must follow FROM
  - **Semantic Analysis:** Checks meaning (e.g., table/column existence, type compatibility)
  - **Optimization:** Rewrites for efficiency (e.g., index usage
  - **Execution**: Scans tables, evaluates conditions, and returns results
  - **Boolean** **Logic**: Conditions use AND (both true), OR (either true), NOT (invert). Precedence: NOT > AND > OR. Parentheses override
  - **Short-Circuit Evaluation:** DBMS may skip parts if outcome is determined (e.g., in false AND something, "something" isn't evaluated)
###### Data Types and Literals
- Strings in quotes ('text' or "text"), numbers unquoted. Mismatches (e.g., unclosed quotes) cause syntax errors
###### Subqueries and Functions
- Nested queries (e.g., SELECT (SELECT ... )) or built-ins (e.g., CONCAT(), SUBSTRING()) add complexity
###### Statements
- SELECT
  - **Structure:** `SELECT [columns] FROM [table] [WHERE condition] [GROUP BY] [ORDER BY] [LIMIT]`
  - **Logic:** Filters rows based on Boolean conditions in WHERE. Joins multiple tables if needed
  - Injections in WHERE can bypass filters or union extra data
- INSERT/UPDATE/DELETE
  - **Structure:** `INSERT INTO table (cols) VALUES (values); UPDATE table SET col=value WHERE condition`
  - **Logic**: Applies changes atomically (or in transactions). Conditions determine affected rows
  - Similar to SELECT, but modifies data. Can chain with subqueries
- Other
  - DDL (e.g., CREATE TABLE, DROP TABLE): Alters schema; high-privilege
  - Functions/Operators: Arithmetic (+), string (CONCAT), conditional (IF, CASE), system (e.g., DATABASE() for metadata)