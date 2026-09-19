# Vulnerability Writeup: SQL Injection (SQLi)
# 1. Reconnaissance & Enumeration

  Discovery: During the web application mapping phase, the product search feature (/catalog/search?q=) was identified as a potential user-supplied input sink.

  Behavioral Analysis: Submitting a single quote (') to the search parameter caused a database error message to be reflected in the HTTP response (SQL syntax error near...), indicating that input was being concatenated directly into a backend database query.

  Vulnerability Hypothesis: The search function lacks input sanitization and parameterized queries, allowing raw user input to alter the structure of the underlying SQL query.

# 2. Exploitation

  Step 1: Intercepted the search request using an interception proxy and sent it to the repeater tool.

  Step 2: Injected a classic authentication/logic bypass payload into the parameter: ?q=test' OR 1=1 -- .

  Step 3: Forwarded the request and analyzed the application response, which returned the entire catalog of database items—including unreleased products—bypassing the intended filtering logic.

  Step 4: (Optional/Advanced) Utilized an automated tool like sqlmap (sqlmap -u "[https://target.com/catalog/search?q=test](https://target.com/catalog/search?q=test)" --current-db) to confirm database enumeration capabilities, successfully extracting database banner and table names.

# 3. Remediation

  Parameterized Queries (Prepared Statements): Ensure all database interactions utilize parameterized queries or prepared statements. This separates code from data, ensuring the database engine treats user input strictly as a literal value rather than executable code.

  Object-Relational Mapping (ORM): Utilize modern ORMs (like Hibernate, Entity Framework, or Sequelize) which naturally abstract raw SQL queries and implement safe input handling by default.

  Input Validation & Least Privilege: Implement strict allow-listing for expected input formats and configure the database user account used by the web application with the minimum necessary privileges (e.g., preventing DROP or administrative commands).
