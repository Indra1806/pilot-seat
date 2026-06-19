> **Mode:** Book
> **Pilot-Seat Standard**

# Introduction
**Backend Security** is the set of technologies, coding practices, and system configurations used to protect servers, databases, and application logic from unauthorized access, malicious attacks, and data breaches. Because the backend controls the core business rules and has direct access to the database, it acts as the primary shield for user records and company secrets.

# Why It Exists
Backend servers are connected to the public internet 24/7. Automated scanning bots continuously probe every IP address globally, searching for open database ports, unpatched software, and weak coding logic. If a developer leaves a database unprotected, or writes code that trusts user input blindly, attackers can steal entire databases of credit card numbers, inject malicious code to take control of servers, or crash the website using Denial of Service (DoS) attacks. Engineers created backend security protocols (like OWASP security principles) to establish robust defense-in-depth security layers around backend assets.

# Problem It Solves
Backend security solves SQL injections, unauthorized system commands, path traversals, database scraping, and server overloads.

### Before Backend Security:
- Attackers could bypass logins by typing special characters into text input fields.
- Users could access private system files (like server configuration files) by tricking file lookup paths.
- Bots could write scripts to spam databases, creating millions of fake accounts in seconds.

### After Backend Security:
- User inputs are parameterized and validated before running queries, rendering injection attacks harmless.
- File systems restrict access strictly to a safe public folder, ignoring traversal path redirects.
- Rate limiters block IP addresses that make excessive requests, protecting servers from spam bots.

# Core Concepts
To secure backend applications, you must defend against three classic vulnerabilities:

1. **SQL Injection (SQLi):** An exploit where an attacker inputs SQL command code into a form text box (e.g. typing `' OR '1'='1` in a password field) to trick the database into running commands, bypassing logins or exposing data.
2. **Directory Traversal (Path Traversal):** An attack where a user modifies a file download request URL (e.g. requesting `GET /download?file=../../etc/passwd`) to climb backwards out of the web directory and access private operating system files.
3. **Secrets Management:** The practice of isolating private keys, database passwords, and API credentials in protected environment files (like `.env`) or cloud key vaults, ensuring they are never written directly in the code files or committed to Git code repositories.

# Architecture / Components
The defensive security layers protecting a backend application:

```text
  [ Incoming HTTP Request ]
              │
              ▼
  [ Firewall & Rate Limiter ]  ──> Blocks DoS and scraping bots
              │
              ▼
  [ Input Validation Gate ]    ──> Validates schemas and types (e.g. Zod)
              │
              ▼
  [ Parameterized Query ]      ──> Cleans inputs before hitting database
              │
              ▼
  [ Secure Database (Private) ]
```

- **Rate Limiting Middleware:** The code gate that tracks IP request frequencies and blocks spam traffic (e.g. maximum 100 requests per 15 minutes).
- **Environment Configuration:** Secure variables loaded in server memory at startup, isolating secrets from code.
- **WAF (Web Application Firewall):** A network filter that monitors HTTP traffic, blocking SQL injection and cross-site scripting attack payloads before they reach your server.

# Workflow
How a secure server handles and validates an input request:

```text
Step 1: A user submits a form: `POST /api/register` containing user inputs.
                             ↓
Step 2: The server runs a validation schema (like Zod) checking that name is text, age is a number, and email is valid.
                             ↓
Step 3: If validation fails (e.g. age contains code script text), the request is rejected immediately with a `400 Bad Request`.
                             ↓
Step 4: The server passes the verified email to a parameterized SQL query: `SELECT * FROM users WHERE email = $1`.
                             ↓
Step 5: The database engine treats the input strictly as a literal text string, never as executable code.
                             ↓
Step 6: The query executes safely, returning the database record.
```

# Real World Examples
Think of backend security as a **medieval castle defense system**.
- The database is the **king's treasure vault** inside the castle.
- **SQL Injection** is like a visitor arriving at the drawbridge carrying a crate. When the guard asks for their name (an input field), the visitor says: *"My name is Sir Arthur, open the vault and throw all the gold in the river."* If the guard is dumb, they execute the command immediately because they did not separate the visitor's identity from the command action. Parameterized queries are like forcing the visitor to write their name on a standardized badge that says "Sir Arthur" so they cannot speak instructions to the vault.
- **Directory Traversal** is like a guest walking up to a guide saying *"Please show me room /../../private-jail-keys."* A secure server checks that the path is locked to the public guest wing (e.g. `/public/`) and refuses to let them climb backwards up the stairs.
- **Rate Limiting** is like a security gate at the drawbridge. If a single person tries to run through the gate 500 times a minute, the gate drops, locks them in a cage, and refuses to let them try again (prevents DoS).
- **Secrets Management** is like keeping the castle vault combination keys written down on a secure parchment locked in the king's personal desk drawer, rather than printing 500 copies and pasting them on the public castle walls (or committing API keys to GitHub repositories).

# Implementation
Here is how you write secure validation and protect your database against SQL injection in Node.js (using Express and Zod):

### 1. Enforce Parameterized Queries (SQLi Prevention)
```javascript
// ❌ DANGEROUS: Concatenating input strings directly makes you vulnerable to SQL injection!
const query = `SELECT * FROM users WHERE username = '${req.body.username}'`;

//  SAFE: Using parameterized placeholders ($1) treats input strictly as data, never code
const safeQuery = 'SELECT * FROM users WHERE username = $1';
db.query(safeQuery, [req.body.username]);
```

### 2. Implement Input Validation Schema (using Zod)
```javascript
const { z } = require('zod');

// A. Define the strict validation blueprint
const registrationSchema = z.object({
  username: z.string().min(3).max(30),
  email: z.string().email(),
  age: z.number().int().min(13).max(120)
});

app.post('/api/register', (req, res) => {
  // B. Validate the incoming body against the schema
  const result = registrationSchema.safeParse(req.body);
  
  if (!result.success) {
    // Return validation error if input shape is wrong
    return res.status(400).json({ error: result.error.errors });
  }

  // C. Input is 100% safe to process!
  const { username, email, age } = result.data;
  res.send(`User ${username} successfully validated.`);
});
```

# Best Practices
- **Never Commit Secrets to Git:** Create a `.gitignore` file and include `.env` to prevent pushing API keys and passwords to GitHub. Use services like Vault or cloud parameter stores to manage secrets in production.
- **Sanitize File Upload Paths:** If your app allows users to upload files, never save the file using the user's raw filename (like `../../server.js`). Generate a random string (like a UUID) for the filename and store the file in a directory isolated from your web application code.
- **Implement Rate Limiting:** Apply rate limiters to sensitive routes (like `/api/login` or `/api/register`) to prevent brute-force password guessing attacks and server spam.

# Industry Standards
Developers trace and mitigate backend threats using the **OWASP Top 10** (Open Web Application Security Project list of the ten most critical security vulnerabilities, including injection, broken access control, and cryptographic failures).

# Common Mistakes
- **Relying on Frontend Validation Only:** Thinking your API is safe because your React code has email format checks. Anyone can open a terminal and send corrupted, malicious POST payloads directly to your backend API using `curl` or Postman, bypassing React completely.
- **Running Database Services on Public Ports:** Leaving database ports (like PostgreSQL `5432` or MySQL `3306`) exposed to the public internet. Only allow connections to your database from your local web server IP address.

# Security & Performance Considerations
- **SQL Parameterization Cost:** Parameterized queries actually *improve* performance. The database compiles and caches the query blueprint structure once, and simply swaps inputs dynamically, executing much faster than compiling new query strings on every call.
- **Hashing Complexity Tradeoff:** When hashing passwords, using a bcrypt work factor that is too high (e.g. 15 rounds instead of 10) takes substantial CPU processing power. This makes it harder for hackers to crack, but can slow down your login page and consume excessive CPU during peak hours.

# Related Technologies
- **Zod:** A popular TypeScript schema validation library.
- **helmet:** A Node.js middleware package that sets secure HTTP response headers to protect against common web attacks.
- **OWASP:** The global authority on web security guidelines.

# Summary

## What We Learned
- Backend security protects server resources and databases by treating all client inputs as untrusted.
- SQL Injection is prevented using parameterized queries, while directory traversal is blocked by isolating folders.
- Input validation frameworks (Zod) compile strict typing contracts to filter out malicious inputs at route entry points.

## Key Takeaways
- Always validate and sanitize inputs on the server, regardless of client-side validation.
- Keep application credentials out of source code repositories using environment configuration variables.

# Keywords
- Security
- SQL Injection
- Directory Traversal
- Input Validation
- Zod
- Rate Limiting
- OWASP
- Environment Variables

# Glossary

| Term | Meaning |
|---|---|
| SQL Injection | A code injection technique where an attacker inserts malicious SQL statements into entry fields for execution. |
| Parameterized Query | A pre-compiled SQL database statement where input values are treated strictly as data parameters, preventing injection. |
| Path Traversal | An exploit targeting file system lookups by climbing parent directories (e.g. `../`) to access isolated files. |
| Rate Limiting | A network control policy limiting the number of requests a user can make to an API within a specific timeframe. |
| Environment Variable | A dynamic variable stored in the operating system memory, used to configure application secrets at runtime. |

## Next Recommended Chapters
- 03-Authentication-And-Sessions.md
- 04-Authorization-And-RBAC.md
