# Featured Writeup: SQL Injection (SQLi) Fundamentals & Exploitation

# 1. Reconnaissance & Enumeration

SQL Injection (SQLi) occurs when an application incorporates user-supplied input directly into a database query without proper sanitization or parameterization, allowing user input to be interpreted as executable SQL code.

Identifying Injection Points: Common testing vectors include URL parameters (e.g., id=1), search bars, form fields (such as login panels), cookies, and HTTP headers (like the Referer header).

## Detection Methodology:

Injecting special characters like a single quote (') or double quote (") to check if the application returns a raw database error or syntax leak.

Testing boolean logic (e.g., appending OR 1=1) to see if application output or record counts change.

Probing for time delays using functions like SLEEP() to identify blind vectors where no visual feedback is provided.

## 2. Exploitation

Depending on how the application processes input and returns feedback, exploitation branches into three primary categories:

## In-Band (Error-Based & Union-Based):

## Error-Based: Exploits verbose error handling to leak database versions, table structures, or data directly via syntax errors.

## Union-Based: Leverages the UNION operator to append custom SELECT statements to the original query. The workflow requires determining the correct number of columns (incrementing values until the error stops) and identifying renderable columns to extract data such as database names, table definitions from information_schema, and sensitive records using functions like group_concat().

## Blind (Authentication Bypass, Boolean-Based, and Time-Based):

## Authentication Bypass: Injects payloads like ' OR 1=1;-- into login fields to comment out password validations and force authentication success.

## Boolean/Time-Based: When no direct output is returned, yes/no questions are asked character-by-character either by reading binary page responses (e.g., API states) or monitoring response latency via conditional time delays (SLEEP(5)).

## Out-of-Band (OOB): Used when all in-band and blind channels are closed, forcing the database engine (via functions like LOAD_FILE() or xp_dirtree) to trigger external DNS or HTTP requests to an attacker-controlled listener.

# 2.1 Practical Lab Walkthrough

To put these core concepts into practice, I completed a structured 4-level target lab environment where each level isolates a different injection technique. The interface provided real-time feedback via a live SQL Query box.

## Level 1: Union-Based SQLi (In-Band)

Objective: Exploit a vulnerable blog article parameter (id) to extract internal records.

## Step 1 - Finding Column Count: Tested incrementing column counts via the URL parameter:

1 UNION SELECT 1 ➔ Error (wrong count)

1 UNION SELECT 1,2 ➔ Error

1 UNION SELECT 1,2,3 ➔ Success! The target table contains 3 columns.

## Step 2 - Making Output Visible: Set the initial ID to 0 so the original query returns an empty set, ensuring only our injected row renders:

0 UNION SELECT 1,2,3 ➔ The number 3 rendered successfully in the page content area, confirming the extraction column position.

## Step 3 - Extracting Database Name: Injected the MySQL function database() into the target column position:

0 UNION SELECT 1,2,database() ➔ Revealed the active database name: sqli_one.

## Step 4 & 5 - Enumerating Schema: Quarantined metadata via information_schema:

Listed tables: 0 UNION SELECT 1,2,group_concat(table_name) FROM information_schema.tables WHERE table_schema = 'sqli_one' (revealed staff_users).

Listed columns: 0 UNION SELECT 1,2,group_concat(column_name) FROM information_schema.columns WHERE table_name = 'staff_users' (revealed id, username, and password).

## Step 6 - Extracting Credentials: Pulled out records using group_concat() and retrieved Martin's password to clear Level 1.

## Level 2: Authentication Bypass (Blind)

Objective: Bypass a standard login form where execution results are hidden (success/failure response only).

Exploitation: Injected ' OR 1=1;-- into the Username field with arbitrary text in the Password field.

Mechanics: The resulting backend query became:
select * from users where username='' OR 1=1;--' and password='anything' LIMIT 1;
The OR 1=1 clause evaluated true globally, while the semicolon and double-dash (--) commented out the entire password verification block, successfully logging me in as the primary user account to capture the second flag.

## Level 3: Boolean-Based Blind SQLi

Objective: Extract data byte-by-byte from an API endpoint (/checkuser?username=) that returns binary JSON indicators ({"taken":true} vs {"taken":false}) with no page content output.

## Step 1 - Confirming Injection: Verified logical evaluation using wildcards:
admin123' UNION SELECT 1,2,3 where database() like '%';-- ➔ Returned {"taken":true}.

## Step 2 to 4 - Character-by-Character Enumeration: Systematically guessed database names, tables (information_schema.tables), and columns (information_schema.columns) by cycling through character sets using like 'a%', like 'b%', etc., uncovering the database name sqli_three and table structure users.

## Step 5 & 6 - Extracting Target Data: Narrowed down the target username (admin) and enumerated the password string character-by-character until resolving the complete password (3845).

## Step 7: Authenticated with the recovered credentials to clear Level 3.

## Level 4: Time-Based Blind SQLi

Objective: Extract data from an environment where both page content and boolean response signals are entirely suppressed, relying solely on network latency.

## Step 1 - Verifying Injection & Columns: Tested column count with conditional time delays:
admin123' UNION SELECT SLEEP(5),2;-- ➔ Resulted in a 5-second pause, confirming both column structure and timing logic execution.

## Step 2 & 3 - Character Enumeration via Clocks: Tested database names and schema details iteratively. A 5-second delay confirmed a correct character match (true), while an immediate response signified a mismatch (false), ultimately yielding the database name sqli_four.

## Step 4 - Extracting Admin Password: Iteratively scanned the admin password characters using conditional sleep increments (e.g., matching prefixes like 4, 49, 496, leading to the password 4961).

## Step 5: Successfully logged in using the timing-derived credentials, validating how blind data exfiltration can succeed entirely through time observation when all other communication channels are blocked.

# 3. Remediation

Preventing SQL Injection requires separating user-supplied data from the application's query logic:

Prepared Statements (Parameterised Queries): The gold standard defense. By defining query structures with placeholders (e.g., ? or %s), the database engine treats input exclusively as literal data rather than executable code.

Input Validation & Allowlisting: Enforcing strict type and format checks on incoming parameters (e.g., verifying an ID is strictly numeric) before it reaches the data layer.

Principle of Least Privilege: Restricting the database account used by the web application to the absolute minimum required permissions (e.g., granting only SELECT access where applicable) to limit potential blast radius.

Defense-in-Depth: Utilizing Web Application Firewalls (WAFs) as an auxiliary protective layer, while prioritizing secure coding over signature-based filtering.

Disclaimer: All writeups and technical notes contained in this repository are performed on authorized, isolated lab environments (such as TryHackMe and PortSwigger academies) strictly for educational purposes.
