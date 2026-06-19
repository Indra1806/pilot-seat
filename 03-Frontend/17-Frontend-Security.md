> **Mode:** Book
> **Pilot-Seat Standard**

# Introduction
Frontend Security refers to the practices, coding standards, and browser features used to protect the client-side of a web application from malicious attacks. Since the frontend of a website runs directly inside a user's browser, developers must assume that the environment is completely public and vulnerable to exploitation, requiring strict measures to protect user data and session credentials.

# Why It Exists
Unlike backend servers, which reside in locked, private data centers, a frontend application is delivered directly to the user's device. Anyone can right-click a webpage, open Developer Tools, view the source code, intercept network requests, and edit variables in memory. This public nature makes frontends a prime target for hackers. If an attacker can inject malicious code into a website, they can steal credit card numbers, hijack user accounts, or redirect users to scam pages. Engineers created browser security standards and coding best practices to establish safety guardrails inside this public environment.

# Problem It Solves
Frontend security solves script injections, unauthorized cross-site requests, and sensitive data leakage.

### Before Frontend Security:
- Attackers could easily inject scripts into comments boxes that stole everyone else's session passwords.
- A malicious email link could trigger your browser to secretly transfer money from your bank account website in the background because the bank site trusted browser requests blindly.
- Storing secret keys in browser memory allowed any script to read and transmit them.

### After Frontend Security:
- Browsers refuse to run scripts that do not match a strict whitelist (Content Security Policy).
- Requests originating from external sites are blocked or ignored unless explicitly authorized (CORS & SameSite cookies).
- Crucial keys are stored in secure compartments that client-side scripts cannot access.

# Core Concepts
To secure frontends, you must understand the three most common attack vectors:

1. **Cross-Site Scripting (XSS):** An attack where a hacker successfully injects malicious JavaScript code into a trusted website, which then runs inside the browsers of other visitors.
2. **Cross-Site Request Forgery (CSRF):** An attack where a malicious website tricks a user's browser into performing an action on a trusted site where the user is currently logged in (like secretly hitting "Delete Account" on a social media site).
3. **The Same-Origin Policy (SOP):** A fundamental browser safety rule that prevents scripts on one website (e.g. `malicious.com`) from reading or editing data on another website (e.g. `mybank.com`).

# Architecture / Components
Browsers implement security barriers to isolate websites and control resource execution:

```text
  [ Web Browser Sandbox ]
  ┌────────────────────────────────────────────────────────┐
  │  [ Site A: mybank.com ]     [ Site B: malicious.com ]  │
  │  - Session Cookie           - Tries to read A's data   │
  │    (HttpOnly, SameSite)       (Blocked by browser SOP) │
  │                                                        │
  │                    [ Browser Shield ]                  │
  │                    - CSP Headers                       │
  │                    - CORS Gates                        │
  └────────────────────────────────────────────────────────┘
```

- **Same-Origin Policy:** The internal browser barrier ensuring Site A cannot spy on Site B.
- **Content Security Policy (CSP):** An HTTP response header sent by the server listing exactly which domains the browser is allowed to download scripts, styles, and images from.
- **Secure Cookie Compartments:** Restricting cookies using the `HttpOnly` tag so that JavaScript code cannot read them, preventing XSS scripts from stealing session tokens.

# Workflow
How a secure cookie prevents session theft:

```text
   [ Unsecured Session Flow (Vulnerable to XSS) ]
   User Logs In ──> Server sends token in JS-readable Cookie ──> XSS Script runs `document.cookie` ──> Token stolen!

   [ Secured Session Flow (Protected) ]
   User Logs In ──> Server sends token in `HttpOnly` Cookie ──> XSS Script runs `document.cookie` ──> Returns blank! (Protected)
```

# Real World Examples
Think of frontend security as the **lobby of a high-security bank**.
- The frontend is the public lobby. Anyone can walk in off the street, look at the layout, and talk to people.
- Because it is public, you would never store gold bars or the bank vault key out on a lobby table (no database passwords or private API keys in JS code).
- **XSS** is like a thief pasting a fake sign over the deposit slip table saying: *"Slip your credit card details under the floorboards."* (Injecting malicious script tags).
- **CSRF** is like a thief tricking a customer's personal courier (the browser's automatic cookie delivery) into delivering a forged transaction slip, simply because the courier is recognized by the teller.
- To prevent this, you hire a guard at the door who checks IDs and packages (CSP), scan incoming slips for hidden pens or explosives (input sanitizing), and place the vault key in a locked safe that only the vault manager can open, which is completely out of reach of anyone in the lobby (`HttpOnly` cookies).

# Implementation
Here is how you configure security headers and write clean code to prevent script injections:

### 1. The Safe Way to Render User Input (Prevents XSS)
```javascript
const userInput = "<script>stealData()</script>"; // Malicious input

// ❌ Vulnerable: Runs the script in the user's browser!
document.getElementById("output").innerHTML = userInput;

//  Safe: Treats the input strictly as harmless text characters
document.getElementById("output").textContent = userInput;
```

### 2. Configure a Content Security Policy (CSP) Header
This header (configured on your server) tells the browser to only run scripts originating from your own domain:
```http
Content-Security-Policy: default-src 'self'; script-src 'self' https://trustedscripts.com;
```

### 3. Setting a Secure Session Cookie (Server-side header definition)
When generating session tokens, always set these attributes to lock the cookie down:
```http
Set-Cookie: session_token=xyz123; Secure; HttpOnly; SameSite=Strict;
```
- `Secure`: Cookie is only sent over encrypted HTTPS connections.
- `HttpOnly`: Client-side JavaScript cannot read the cookie.
- `SameSite=Strict`: The cookie is never sent along with requests originating from external websites (stops CSRF).

# Best Practices
- **Never Store Sensitive Tokens in LocalStorage:** `localStorage` has no security boundaries; any JavaScript running on your page (including third-party analytics scripts) can read it. Store login tokens in `HttpOnly`, `SameSite` cookies instead.
- **Sanitize and Escape All User Input:** If you must render HTML written by users (like a rich-text blog post editor), use a trusted sanitizing library like **DOMPurify** to strip out script tags before inserting it into the DOM.
- **Implement a Strict CSP:** Treat Content Security Policies as a mandatory checklist item. A strong CSP acts as a final safety net that disables inline scripts and unrecognized domain downloads.

# Industry Standards
Modern browsers strictly enforce the Same-Origin Policy and CORS gates. Applications must explicitly handle cross-origin sharing using secure servers, and must compile code regularly checking for outdated, vulnerable packages (`npm audit`).

# Common Mistakes
- **Storing Private API Keys in Client Code:** Placing your private Stripe or OpenAI API key inside your React source code. Since the code is bundled and sent to the browser, any user can inspect the file and steal your key, billing charges to your account. Private keys belong on backend servers.
- **Allowing CORS Wildcards in Production:** Setting the header `Access-Control-Allow-Origin: *` (allow everyone) on a backend server holding private user data. This allows any malicious website to read your database responses via AJAX.

# Security & Performance Considerations
- **Third-Party Dependencies (Supply Chain Attacks):** If an attacker hacks a minor NPM package that you use in your build chain, they can inject malicious code that gets bundled into your site. Lock your dependency versions and run security scanners.
- **HTTPS Only:** Running a frontend site over plain HTTP allows hackers on public Wi-Fi networks to inject advertisements or malicious scripts into your pages as they travel through the air. Always enforce HTTPS.

# Related Technologies
- **DOMPurify:** A highly optimized library used to clean HTML strings and prevent script injection.
- **OWASP:** The Open Web Application Security Project, a global non-profit organization tracking the top web security threats.
- **CORS:** Cross-Origin Resource Sharing, the server gateway protocol that manages cross-site data sharing.

# Summary

## What We Learned
- The frontend is a public execution environment that must be treated as inherently untrusted.
- XSS injects malicious code; CSRF hijacks user credentials; clickjacking overlays transparent triggers.
- Defenses include secure cookie tags (`HttpOnly`, `SameSite`), input sanitizing (DOMPurify), and Content Security Policies.

## Key Takeaways
- Never store secrets or private keys on the client-side.
- Always use `HttpOnly` cookies for authentication tokens to isolate them from JavaScript access.

# Keywords
- Security
- XSS
- CSRF
- Same-Origin Policy
- CSP
- HttpOnly
- CORS
- Sanitization

# Glossary

| Term | Meaning |
|---|---|
| Cross-Site Scripting | An attack where a hacker successfully runs unauthorized JavaScript code inside a victim's browser session. |
| Cross-Site Request Forgery | An exploit where an external site tricks a browser into sending authorized requests to a site the user is logged into. |
| Same-Origin Policy | A core browser security mechanism restricting how a document or script loaded from one origin interacts with resources from another. |
| Content Security Policy | An HTTP security header listing approved sources of assets and scripts that the browser is permitted to run. |
| HttpOnly | A cookie attribute that prevents client-side scripts from reading cookie values, protecting them from theft. |

## Next Recommended Chapters
- 02-Web-Development/02-HTTP-And-HTTPS.md
- 04-Backend/03-Authentication-And-Sessions.md
