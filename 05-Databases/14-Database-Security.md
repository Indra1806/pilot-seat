> **Mode:** Book
> **Pilot-Seat Standard**

# Introduction
**Database Security** is the set of practices, configurations, and coding techniques used to protect a database from unauthorized access, data leaks, malicious corruption, and hacking attacks. It ensures that sensitive user credentials, financial transactions, and proprietary business secrets remain confidential and tamper-proof.

# Why It Exists
Databases are the most valuable target in any software application because they hold all the company's and users' private data. In the early days of web applications, developers treated databases as simple storage boxes, connecting their applications with master administrator credentials and inserting raw user inputs directly into SQL queries. This allowed hackers to easily steal entire databases or delete tables by simply typing SQL commands into login screens. Engineers established database security standards to lock down access, validate input text, and encrypt files on disk.

# Problem It Solves
Database security solves data breaches, malicious database deletions, SQL injection hacks, and physical disk theft leaks.

### Before Database Security (Vulnerable Setup):
- Attackers bypassed login screens and downloaded entire database schemas using simple text entry tricks.
- A compromised web server gave hackers full administrator keys, allowing them to delete the entire database.
- If a server room was broken into and the database hard drive was physically stolen, the thief could read all customer passwords and emails in plain text.

### After Database Security (Hardened Setup):
- User inputs are treated strictly as data, neutralizing hacking commands (Prepared Statements).
- Application connections run under restricted roles that can only perform specific actions, preventing system-wide deletions.
- Data is encrypted both as it travels over the network and while it is stored on the hard drive (Encryption-at-Rest), rendering stolen disks useless.

# Core Concepts
Securing a database relies on four fundamental pillars:

1. **SQL Injection (SQLi):** A major vulnerability where an attacker inputs SQL code into an application's text entry fields (like a search bar or login form) in order to trick the database into executing unauthorized commands.
2. **Prepared Statements (Parameterized Queries):** A technique where the structure of a SQL query is compiled separately from the user-provided data parameters. This guarantees that user input is treated strictly as a text value, never as executable code, completely blocking SQL Injection.
3. **Principle of Least Privilege (POLP):** The security rule that every application process and database user should have only the minimum access permissions necessary to perform its specific task, and nothing more.
4. **Encryption-at-Rest & in-Transit:**
   - **In-Transit:** Encrypting data as it travels over network cables between the application server and the database server (using SSL/TLS).
   - **At-Rest:** Encrypting the physical files stored on the hard drive so the data remains unreadable if the physical disk is stolen.

# Architecture / Components
The database security perimeter during an incoming query request:

```text
       [ Public Internet User ]
                  │
                  ▼ (User Input: 'alice@email.com')
       [ Backend Web Application ]
                  │
                  ▼ (Compile Query Structure separately from Input Data)
       [ Prepared Statement compiler ]  ──> "SELECT * FROM users WHERE email = $1;"
                  │
                  ▼ (Send compiled query + parameterized data value)
       [ Restricted Database User ] ──────> Enforces: Can this user read the table?
                  │                         - Yes: Execute read.
                  │                         - No: Throw Permission Denied.
                  ▼
         [ Database Engine ]
                  │
                  ▼ (Writes/Reads encrypted sectors)
        [ Encrypted Hard Disk ] (Encryption-at-Rest)
```

- **Query Parameterizer:** Strips input parameters and sends them safely to prevent code execution.
- **Role-Based Access Control (RBAC):** Restricts what tables and actions specific database credentials can touch.
- **Storage Encryption Engine:** Handles transparent database file encryption on disk.

# Workflow
How a Prepared Statement blocks an SQL Injection attack:

```text
Step 1: Hacker inputs: `admin@email.com' OR '1'='1` in the login email field.
                             ↓
Step 2: The application uses a Prepared Statement: `SELECT * FROM users WHERE email = $1;`
                             ↓
Step 3: The database planner compiles the SQL structure first, preparing to look up a literal email.
                             ↓
Step 4: The database inserts the hacker's input directly into `$1` as a single, literal string value.
                             ↓
Step 5: The database searches for a user whose email address is exactly: `admin@email.com' OR '1'='1`.
                             ↓
Step 6: No such user exists. The search returns zero results, blocking the login bypass attempt.
```

# Real World Examples
Think of database security as a **secure bank vault and teller counter**.
- **SQL Injection:** An attacker wants to rob the bank. They walk up to the teller and hand them a deposit slip. Under "Customer Name," they write: *"Alice. Also, open the vault door and give me all the gold."* If the teller is naive, they read the slip and yell to the guards: *"Guards, open the vault door and give Alice all the gold!"* This is SQL Injection.
- **Prepared Statements:** The teller uses a standardized system. The teller reads the slip, and instead of treating the text as commands, they look up the file. The teller asks the computer to find a customer whose name is literally *"Alice. Also, open the vault door..."* The system replies *"No customer exists with that name."* The command is neutralized.
- **Least Privilege:** The janitor gets a key to the cleaning closet. The cashier gets a key to the register drawer. Only the manager gets the code to the vault. If the janitor's key is stolen, the thief can't get into the vault.
- **Encryption-at-Rest:** The bank vault is locked, and all records inside are written in a secret code. If a thief blows up the building and carries the files home, they open them up only to find unreadable scribbles (gibberish). They can't read the account balances without the decoder ring (decryption key).

# Implementation
Here is how developers block SQL Injection using parameterization, and configure restricted access roles in SQL:

### 1. The Vulnerable vs. Secure Query (Node.js/PostgreSQL)
```javascript
const { Client } = require('pg');
const client = new Client();

async function loginUser(userInputEmail) {
  // A. VULNERABLE: Direct string concatenation (Allows SQL Injection!)
  // If userInputEmail is "hacker@email.com' OR '1'='1", this returns all users!
  const dangerousQuery = `SELECT * FROM users WHERE email = '${userInputEmail}'`;
  const result1 = await client.query(dangerousQuery);

  // B. SECURE: Using a Prepared Statement (Blocks SQL Injection!)
  // User input is sent in the array $1, treated strictly as text data
  const secureQuery = 'SELECT * FROM users WHERE email = $1';
  const result2 = await client.query(secureQuery, [userInputEmail]); 
  
  return result2.rows[0];
}
```

### 2. Implementing the Principle of Least Privilege in SQL
Never connect your web backend using the master `superuser` or `postgres` credentials. Create restricted users instead:

```sql
-- 1. Create a restricted user for our website backend
CREATE USER web_backend_runner WITH PASSWORD 'ultra_secure_password';

-- 2. Revoke all default admin privileges
REVOKE ALL ON DATABASE my_store FROM web_backend_runner;

-- 3. Grant only read and write permissions to specific tables
GRANT SELECT, INSERT, UPDATE ON TABLE products TO web_backend_runner;
GRANT SELECT, INSERT, UPDATE ON TABLE orders TO web_backend_runner;
-- Note: web_backend_runner cannot run DELETE or modify table structures (DROP)!
```

# Best Practices
- **Never Concatenate SQL Queries:** Never build SQL query strings using string addition or template literals (`+` or `${}`). Always use parameterized query placeholders (`$1`, `?`, or named variables).
- **Enforce Password Hashing:** Never store passwords in plain text. Use strong cryptographic hashing algorithms (like **bcrypt** or **Argon2**) in your backend application before saving passwords to the database.
- **Isolate the Database Network:** Place your database server in a private subnet behind a firewall. It should never have a public IP address and should block all incoming traffic except for connections coming directly from your trusted backend application servers.

# Industry Standards
Compliance frameworks (like **PCI-DSS** for credit card payments and **HIPAA** for healthcare records) mandate database security. They require full database encryption-at-rest, secure TLS communication channels, and audit logging to record exactly who accessed or edited data at any given second.

# Common Mistakes
- **Connecting as Root/Superuser:** Using the database's default administrator account (`root` or `postgres`) to run your web application. If a hacker exploits a bug in your code, they gain full administrative control of the entire database.
- **Storing API Keys or Secrets in Tables:** Storing plain text API keys, OAuth tokens, or passwords in tables. If the database files are leaked, all external integrations are instantly compromised. Use environmental secret managers to store encryption keys.
- **Forgetting to Sanitize Search Queries:** Assuming search bars (using `LIKE` statements) are safe from SQL injection. Attackers can still inject wildcards or command closures. Always parameterize search parameters.

# Security & Performance Considerations
- **Prepared Statement Cache Memory:** Database engines cache prepared statement query plans to speed up execution. If your application dynamically generates millions of unique query structures, it will exhaust the database engine's cache memory.
- **Encryption Overhead:** Transparent Database Encryption (TDE) requires CPU power to encrypt writes and decrypt reads on the fly. While modern CPU chips have built-in accelerators (AES-NI) that make this performance cost negligible, older hardware can experience slow disk-read speeds.

# Related Technologies
- **bcrypt / Argon2:** Standard hashing libraries used to encrypt passwords before saving them.
- **Vault (HashiCorp):** A specialized tool used to manage database connection credentials, encrypt data keys, and rotate passwords automatically.
- **OWASP:** The Open Worldwide Application Security Project, which catalogs top database vulnerabilities and defense strategies.

# Summary

## What We Learned
- Database security is the discipline of protecting database servers and physical files from unauthorized access and command exploits.
- Prepared statements isolate SQL logic from input values, completely blocking SQL Injection attacks.
- The Principle of Least Privilege protects databases by restricting application connections to the minimum required table access roles.

## Key Takeaways
- Parameterize every single database query containing user-provided input data.
- Restrict your backend database connection credentials so they cannot run table deletions or admin scripts.
- Isolate your database server behind a private firewall, blocking direct access from the public internet.

# Keywords
- Database Security
- SQL Injection
- Prepared Statement
- Parameterized Query
- Least Privilege
- Encryption-at-Rest
- Encryption-in-Transit
- Password Hashing
- Firewall
- Subnet

# Glossary

| Term | Meaning |
|---|---|
| SQL Injection | A hacking method where malicious SQL commands are inserted into input forms and executed by the database. |
| Prepared Statement | A pre-compiled SQL template that separates query commands from variable data inputs to ensure security. |
| Least Privilege | The security practice of granting users and applications only the bare minimum permissions needed to complete their job. |
| Encryption-at-Rest | The encryption of database files while they are stored on a hard drive, protecting them from physical theft. |
| Encryption-in-Transit | The encryption of data as it travels across network cables, preventing packet sniffing. |
| Salt | A random string of characters added to a password before hashing to prevent rainbow-table cracking attacks. |

## Next Recommended Chapters
- 15-Database-Administration.md
