> **Mode:** Book
> **Pilot-Seat Standard**

# Introduction
**Authentication** is the process of verifying *who* a user is (confirming their identity). In web applications, this typically starts when a user inputs their username and password. **Session Management** is the system the backend server uses to remember this verified identity across subsequent HTTP requests so the user does not have to log in again with every click.

# Why It Exists
HTTP is a stateless protocol. This means the server treats every single request as a completely independent transaction from a total stranger. If you log in on the homepage, and then click to view your profile page, the server has already forgotten who you are. Without session management, you would have to type your password again for every page you navigate to or every item you add to your shopping cart. Engineers created stateful sessions and stateless tokens to give users a persistent, logged-in identity in a stateless web.

# Problem It Solves
Authentication and sessions solve stateless identity tracking and credential storage security problems.

### Before Session Management:
- Users had to log in repeatedly for every page interaction.
- Storing raw user passwords in plain text in database tables meant anyone hacking the database stole all user passwords immediately.

### After Stateful Sessions:
- The server checks credentials once, creates a temporary "Session ID," and stores it in memory.
- The browser saves the Session ID as a cookie and transmits it automatically with every subsequent request.

### After Stateless Tokens (JWT):
- The server encodes user details into a signed token package (JWT) and hands it to the browser.
- The server does not need to store anything in its memory, making the system highly scalable across multiple servers.

# Core Concepts
To secure user identities, you must understand three core models:

1. **Stateful Sessions:** The traditional way. The server creates a session file in its memory or database and writes the Session ID to a browser cookie. The server must check its database for every incoming request to see if the session is still active.
2. **Stateless Tokens (JWT):** The modern way. JSON Web Tokens (JWT) are self-contained credentials. The server packs user details into a string, signs it cryptographically, and hands it to the client. The server can verify the token is valid simply by checking the cryptographic signature, without querying a database.
3. **Password Hashing & Salting:** Passwords must never be saved in plain text.
   - **Hashing:** Running a password through a one-way math formula (like bcrypt) to generate a unique string of characters (a hash) that cannot be reversed.
   - **Salting:** Adding a random string of characters (a salt) to the password *before* hashing it, preventing hackers from guessing passwords using lookup lists of pre-calculated hashes (rainbow tables).

# Architecture / Components
Comparing the architecture of stateful sessions versus stateless tokens:

### Stateful Session Architecture
```text
  [ Client ] ── 1. Send Cookie (Session ID: 99) ──> [ Web Server ] ── 2. Read DB ──> [ Session DB ]
                                                                                      (Checks if active)
```

### Stateless Token Architecture (JWT)
```text
  [ Client ] ── 1. Send JWT Token in Header ──────> [ Web Server ] ── (Verifies Signature in memory)
                                                    - No database query needed!
```

- **JWT Signature:** The cryptographic stamp created using a secret key on the server, guaranteeing the token content has not been altered by the client.
- **Session Database:** The fast in-memory database (like Redis) used to store session objects for quick lookups.
- **Hashing Library:** (e.g. bcrypt, Argon2) The backend library used to securely salt and hash user passwords during registration.

# Workflow
How a stateless token (JWT) authentication flow operates:

```text
Step 1: User submits their username and password via a HTTPS POST request.
                             ↓
Step 2: The server hashes the incoming password and compares it to the hash in the database.
                             ↓
Step 3: If they match, the server generates a JWT containing user details (e.g. `{ username: "Alice", role: "User" }`).
                             ↓
Step 4: The server signs the JWT using a private secret key and sends it to the browser.
                             ↓
Step 5: The browser stores the JWT and sends it in the Authorization header for subsequent API calls.
                             ↓
Step 6: The server reads the token, verifies the cryptographic signature in memory, and immediately serves the profile.
```

# Real World Examples
Think of authentication as **checking in at a hotel**.
- **Stateful Sessions** are like a **traditional metal key**. The receptionist checks your ID (Authentication) and writes your room number down in their central registration logbook (the Session Database). They hand you a metal key stamped with a number. Every time you try to enter the pool, restaurant, or room, the security guard inspects the key, then calls the front desk to look at the logbook to see if that key is still valid.
- **Stateless Tokens (JWT)** are like a **printed card with a holographic stamp**. The receptionist prints a card containing: *"Room 304, Access to Pool, Expires tomorrow"* and stamps it with an unforgeable, shiny holographic signature. When you walk up to the pool, the gate scanner checks the holographic stamp. If it is genuine, you are let in immediately. The gate does not need to call the receptionist or check a computer database because the ticket itself holds the proof of validity.

# Implementation
Here is how you register a user with password hashing, and verify them using JWTs in Node.js:

### 1. Registration: Hashing a Password (using bcrypt)
```javascript
const bcrypt = require('bcrypt');

async function registerUser(username, plainPassword) {
  const saltRounds = 10;
  // Generate salt and hash the password in one step
  const hashedPassword = await bcrypt.hash(plainPassword, saltRounds);
  
  // Save user with the HASHED password to the database
  const newUser = { username, passwordHash: hashedPassword };
  await saveToDatabase(newUser);
}
```

### 2. Login: Verifying and Generating a JWT (using jsonwebtoken)
```javascript
const jwt = require('jsonwebtoken');
const SECRET_KEY = "my_super_secret_key";

async function loginUser(username, inputPassword) {
  // Fetch user from DB
  const user = await findUserInDatabase(username);
  
  // Compare the input password hash with the stored hash
  const isMatch = await bcrypt.compare(inputPassword, user.passwordHash);
  
  if (isMatch) {
    // Generate a token signed with our secret key (expires in 1 hour)
    const token = jwt.sign({ id: user.id, name: user.name }, SECRET_KEY, { expiresIn: '1h' });
    return { success: true, token };
  }
  
  return { success: false, message: "Invalid credentials" };
}
```

# Best Practices
- **Never Store Passwords in Plain Text:** Always use a modern hashing algorithm like **bcrypt** or **Argon2** with a salt factor of 10 or higher.
- **Use Secure Cookie Storage for JWTs:** Do not store JWTs in `localStorage` where XSS scripts can steal them. Send them in cookies marked `HttpOnly`, `Secure`, and `SameSite` to prevent script reading and CSRF attacks.
- **Keep Secret Keys Safe:** The security of your JWT tokens depends entirely on your server's secret key. If an attacker steals your secret key, they can generate fake tokens and log into any account. Store this key in protected environment variables, never in git repositories.

# Industry Standards
Almost all modern single-page applications (React, Angular) and microservices use **JWT (JSON Web Tokens)** for communication because they are stateless, lightweight, and scale easily. Traditional monolithic systems (like standard WordPress, Django, or Rails apps) still commonly use **Stateful Session Cookies** because they are simple and allow instant session revocation.

# Common Mistakes
- **Using Weak Hashing Algorithms:** Saving passwords using outdated algorithms like MD5 or SHA-1. Modern computers can crack MD5 hashed passwords in milliseconds using brute-force search databases.
- **Putting Sensitive Data in JWTs:** Putting a user's password or social security number inside a JWT. While a JWT cannot be altered without breaking the signature, the data inside the token is only base64-encoded, meaning *anyone* who intercepts the token can read the data.

# Security & Performance Considerations
- **Token Revocation (The JWT Downside):** Since JWTs are stateless, the server does not check a database to verify them. This means that if a user loses their phone and wants to "Log out of all devices," you cannot easily invalidate an active JWT until its expiration time passes. The industry standard is to use short-lived Access Tokens (e.g. 15 minutes) paired with database-tracked Refresh Tokens.
- **Session Database Memory Overhead:** If you use stateful sessions, storing millions of active sessions directly in your server's RAM can crash your server. Use dedicated caching databases like Redis to offload session memory.

# Related Technologies
- **JSON Web Token (JWT):** The stateless credential exchange standard.
- **bcrypt / Argon2:** The standard modern hashing libraries.
- **OAuth 2.0 / OpenID Connect:** The protocols used for "Social Sign-in" (e.g. Log In with Google).

# Summary

## What We Learned
- Authentication validates user credentials, while session management tracks active logins.
- Stateful sessions store session IDs on servers, whereas JWT tokens store signed state on clients.
- Passwords must be salted and hashed using secure algorithms (like bcrypt) before database storage.

## Key Takeaways
- Always store passwords hashed, never in plain text.
- Protect JWT tokens from script theft by serving them in `HttpOnly` cookies.

# Keywords
- Authentication
- Session
- JWT
- Password Hashing
- Salting
- bcrypt
- Cookie
- Stateless

# Glossary

| Term | Meaning |
|---|---|
| Hashing | A one-way mathematical function that converts text data into a fixed-length string of characters that cannot be reversed. |
| Salting | Adding random characters to a password before hashing it to prevent dictionary lookup attacks. |
| JWT | JSON Web Token; a self-contained string containing encoded user details signed with a server secret. |
| Stateful | An architecture where the server stores active session states in its own memory database. |
| Stateless | An architecture where the server stores no session states in memory, relying on signed tokens sent by the client. |

## Next Recommended Chapters
- 01-Web-Servers-And-HTTP.md
- 04-Authorization-And-RBAC.md
