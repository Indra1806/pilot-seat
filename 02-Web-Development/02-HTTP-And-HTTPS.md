> **Mode:** Book
> **Pilot-Seat Standard**

# Introduction
HTTP (HyperText Transfer Protocol) is the foundational language of the World Wide Web. It is the agreed-upon set of rules that allows web browsers (clients) to talk to web servers and exchange data like HTML documents, images, and videos.

# Why It Exists
In 1989, Tim Berners-Lee wanted a way for scientists to share documents across the internet easily. While the internet existed to move raw data, there was no standard way to format requests for "documents" or to link them together. He invented HTTP so that if a client asked for a document in a very specific format, the server would understand and reply in a specific format.

# Problem It Solves
It solves the problem of standardized communication between vastly different systems.

**Before:**
A Windows computer running custom software couldn't easily ask a Linux server for a specific file unless both were programmed with the exact same proprietary software.

**After:**
As long as both computers speak "HTTP", a Mac can request a webpage from a Linux server, and the Linux server knows exactly how to respond.

# Core Concepts
1. **Request / Response Cycle:** The entire web is based on this. The Client sends an HTTP Request, and the Server sends back an HTTP Response. The server never speaks first.
2. **Statelessness:** HTTP has no memory. Every single request is treated as a brand new, isolated event. The server does not remember your previous request unless you use clever workarounds (like Cookies).
3. **Methods (Verbs):** The type of action the client wants to perform. Common ones are `GET` (give me data), `POST` (save this new data), `PUT` (update data), and `DELETE` (remove data).
4. **Status Codes:** A 3-digit number the server sends back to summarize what happened. (e.g., `200 OK`, `404 Not Found`).

# Architecture / Components
```text
+-----------------+                                 +-----------------+
|     CLIENT      |                                 |     SERVER      |
|  (Web Browser)  |                                 | (Database/Host) |
+-----------------+                                 +-----------------+
        |                                                   |
        |  --- HTTP REQUEST ---------------------------->   |
        |      GET /index.html                              |
        |      Host: pilot-seat.com                         |
        |                                                   |
        |  <-- HTTP RESPONSE ----------------------------   |
        |      200 OK                                       |
        |      Content-Type: text/html                      |
        |      <html><body>Hello</body></html>              |
```

# Workflow
1. You type `pilot-seat.com` into your browser.
2. The browser generates an **HTTP GET Request**.
3. The request is sent over the internet to the server.
4. The server receives the text of the request, processes it, and finds `index.html`.
5. The server generates an **HTTP Response**, attaches a `200 OK` status code, and pastes the HTML into the body.
6. The browser receives the response and paints the HTML on your screen.

# Real World Examples
Think of HTTP like ordering at a **Fast Food Drive-Thru**:
* **The Request:** You say, "I want a burger" (`GET /burger`).
* **The Server:** The cashier processes the order and hands you the food.
* **Statelessness:** If you drive back around 5 minutes later and say, "I want another one," the cashier treats you like a brand new customer. They don't remember you were just there unless you show them a receipt (a Cookie).
* **Status Codes:** If they have the burger, they give it to you (`200 OK`). If they are out of beef, they say "We don't have that" (`404 Not Found`). If the grill is broken, they say "Our kitchen is down" (`500 Internal Server Error`).

# Implementation
As a web developer, you will write APIs that handle HTTP requests.
Example in Node.js (Express):
```javascript
app.get('/users', (req, res) => {
   // Client made a GET request
   res.status(200).send("Here are the users"); // Server sends Response
});
```

# Best Practices
* **Use the right Verbs:** Don't use a `GET` request to delete a user. `GET` requests should *only* retrieve data and never alter the database. Use `DELETE` instead.
* **Use proper Status Codes:** If a user tries to log in with a bad password, don't return `200 OK` with an error message in the text. Return `401 Unauthorized`.

# Industry Standards
* **HTTPS (HyperText Transfer Protocol Secure):** This is just HTTP, but the entire conversation is encrypted using TLS/SSL. In modern web development, HTTPS is absolutely mandatory. Without it, anyone on your WiFi network can read the plain text of your passwords as they travel to the server.

# Common Mistakes
* **Ignoring CORS (Cross-Origin Resource Sharing):** Browsers block HTTP requests made via JavaScript to a different domain for security reasons unless the server explicitly allows it. Beginners often struggle with "CORS errors" when trying to connect their frontend to their backend.

# Security & Performance Considerations
* **Man-in-the-Middle Attacks:** If you use plain HTTP, hackers can intercept the traffic and inject their own malicious code into the website before it reaches your browser. HTTPS prevents this.
* **Caching:** Because HTTP is stateless, fetching the same massive image on every page load is slow. HTTP Headers allow the server to tell the browser "Cache this image for 30 days" so it doesn't have to be downloaded again.

# Related Technologies
* 03. How Browsers Work
* REST APIs

# Summary
## What We Learned
- HTTP is a request-response protocol used to communicate on the Web.
- It is stateless, meaning it relies on things like Cookies to remember users.
- Requests use Verbs (GET, POST), and Responses use Status Codes (200, 404).

## Key Takeaways
- Always use HTTPS to encrypt data.
- Following HTTP conventions (using the right verbs and status codes) makes your APIs predictable and robust.

# Keywords
- HTTP / HTTPS
- Request / Response
- Stateless
- Status Code
- GET / POST

# Glossary
| Term | Meaning |
|---|---|
| HTTP | The protocol used to transmit web pages. |
| HTTPS | The encrypted, secure version of HTTP. |
| Stateless | A system that does not retain memory of previous interactions. |
| Status Code | A 3-digit number summarizing the result of a request. |

## Next Recommended Chapters
- 03. How Browsers Work
