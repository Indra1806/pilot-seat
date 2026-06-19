> **Mode:** Book
> **Pilot-Seat Standard**

# Introduction
A **Web Server** is a software program running on a server computer that sits listening for incoming requests from users over the internet, processes those requests, and returns the appropriate files or data. **HTTP** (HyperText Transfer Protocol) is the standardized set of communication rules that browsers and web servers use to format, send, and understand those requests and responses.

# Why It Exists
In the early days of networking, there was no standard protocol for requesting files. If a client computer wanted to fetch a document from a server, they had to establish raw network connections and use custom commands, which was incredibly complex. If the server was busy, or the file was missing, there was no standardized way to explain what went wrong. Engineers created HTTP as a simple, text-based language that any computer could use to request resources, and built Web Servers as specialized software dedicated to listening for and answering those requests.

# Problem It Solves
Web servers and HTTP solve connection handling and protocol standardization.

### Before Web Servers & HTTP:
- Accessing files on other computers required custom, low-level networking code.
- If a connection failed, there were no standard error indicators, leaving developers in the dark.
- Computer programs could not talk to each other across different operating systems.

### After Web Servers & HTTP:
- Any web browser on any device can instantly download a page from any server globally.
- Standardized status numbers (like 200 for success, 404 for missing files) tell the client exactly what happened.
- Web servers can efficiently handle thousands of users asking for files at the same time.

# Core Concepts
To understand how backend systems handle traffic, you must master three basic concepts:

1. **Ports (The Listening Channels):** A server computer runs many programs simultaneously. To keep traffic organized, the computer uses numbered virtual channels called ports. Web servers listen for insecure web traffic on **Port 80** and secure traffic on **Port 443**.
2. **HTTP Request Methods (The Actions):** Every request specifies what action it wants to take. The most common methods are:
   - `GET`: Retrieve a resource (like reading a webpage).
   - `POST`: Send new data (like submitting a login form).
   - `PUT`/`PATCH`: Update existing data (like editing a profile).
   - `DELETE`: Remove a resource.
3. **HTTP Status Codes (The Receipts):** A three-digit number sent back by the server explaining the result of the request:
   - `2xx` (e.g. 200 OK): Success!
   - `3xx` (e.g. 301 Redirect): The resource moved elsewhere.
   - `4xx` (e.g. 404 Not Found): The user made an error (requested a missing file).
   - `5xx` (e.g. 500 Server Error): The server crashed or encountered a bug.

# Architecture / Components
A web server acts as the entry gateway for incoming requests:

```text
  [ User Browser ] ─── (HTTP Request) ───> [ Web Server (Port 80/443) ]
          ▲                                       │
          │                                 (Static file?)
          │                                ┌──────┴──────┐
          │                               Yes            No
          │                               ▼              ▼
  [ Returns Response ] <──────────────── [ File ]   [ Backend App ]
                                         (HTML/JS)   (Node/Go/Python)
```

- **HTTP Request:** The package sent by the browser containing a Method, a URL path, Headers (browser metadata), and an optional Body (user input).
- **Web Server Software:** (e.g. Nginx, Apache, or Node.js HTTP module) The process that receives raw socket connections, parses HTTP text, and routes requests.
- **HTTP Response:** The package sent back containing a Status Code, Headers (server metadata), and a Body (the HTML page or data payload).

# Workflow
The request-response lifecycle:

```text
Step 1: The user clicks a link to `mysite.com/images/logo.png`.
                             ↓
Step 2: The browser sends an HTTP `GET /images/logo.png` request to the server on Port 443.
                             ↓
Step 3: The Web Server receives the request and parses the path.
                             ↓
Step 4: The server locates the `logo.png` file on its hard drive.
                             ↓
Step 5: The server wraps the image in an HTTP response packet with a `200 OK` status.
                             ↓
Step 6: The browser receives the response packet and draws the image on screen.
```

# Real World Examples
Think of a Web Server as a **fast food drive-thru window** and HTTP as the **ordering language**.
- The server computer is the physical restaurant building.
- The Web Server software is the **employee standing at the drive-thru window**.
- The **Port** is the physical window opening (Port 80). The employee stands there listening for cars.
- When a car drives up, they place an order (the Request) using a strict menu structure (the HTTP Protocol):
  - They choose an action: "GET me a cheeseburger" (GET method).
  - They add specifications: "No pickles, extra sauce" (HTTP Headers).
  - They pay and submit details (HTTP Body).
- The employee checks the kitchen (Server storage), packs the order in a bag (Response Body), and hands it over with a receipt showing the status (Status Code: `200 OK` for a complete meal, or `404 Out of Stock`).

# Implementation
Here is how you write a basic Web Server in JavaScript using Node.js that listens for requests and returns a message:

```javascript
// 1. Import Node's built-in HTTP server module
const http = require('http');

// 2. Define the port we want to listen to
const PORT = 3000;

// 3. Create the server process
const server = http.createServer((request, response) => {
  console.log(`Received request: ${request.method} ${request.url}`);

  // Check the request path
  if (request.url === '/') {
    // Write HTTP response headers
    response.writeHead(200, { 'Content-Type': 'text/plain' });
    // Write response content (body)
    response.end('Welcome to the Homepage!');
  } else if (request.url === '/about') {
    response.writeHead(200, { 'Content-Type': 'text/plain' });
    response.end('About Us Section.');
  } else {
    // Return a 404 error if path doesn't exist
    response.writeHead(404, { 'Content-Type': 'text/plain' });
    response.end('Page Not Found');
  }
});

// 4. Start the server, telling it to listen on our port
server.listen(PORT, () => {
  console.log(`Web server is active and listening on http://localhost:${PORT}`);
});
```

# Best Practices
- **Always Enforce HTTPS:** Secure your traffic using SSL/TLS certificates (on Port 443). Plain HTTP (Port 80) transmits all data in unencrypted text, allowing hackers on the same network to steal login cookies and passwords.
- **Use Nginx as a Reverse Proxy:** Instead of exposing your Node.js or Python application directly to the internet, run a highly optimized web server like **Nginx** in front of it. Nginx is extremely fast at serving static images and routing incoming traffic, shielding your main application.
- **Set Accurate Status Codes:** Never return `200 OK` with a text body saying "Error: page not found". Use correct codes like `404` or `500` so frontend applications and search engines know what actually occurred.

# Industry Standards
Almost all modern web infrastructure runs on Linux servers. The standard software tools used to manage web servers at scale are **Nginx** (incredibly fast for routing and static assets) and **Apache HTTP Server**. In modern cloud systems, servers are often automatically configured using hosting platforms (like AWS or Cloudflare) which wrap web server software in a user-friendly interface.

# Common Mistakes
- **Listening on the Wrong Ports in Production:** Trying to run a public web server on non-standard ports (like `Port 8080` or `Port 3000`) without setting up a router port forwarding rule. Firewalls and corporate networks block all ports except `80` and `443` by default.
- **Blocking the Server Loop:** Running CPU-intensive math calculations directly in the request handler, which blocks the single-threaded web server process from answering requests from other users, causing their browsers to time out.

# Security & Performance Considerations
- **Slowloris Attacks (DoS):** An attack where thousands of bots open connections to your web server and send HTTP headers extremely slowly, holding the server's connection slots open and preventing legitimate users from accessing the site. Implement strict connection timeouts.
- **HTTP/2 and HTTP/3 Multiplexing:** Modern versions of HTTP (HTTP/2 and HTTP/3) allow browsers to download multiple files simultaneously over a single network connection, drastically speeding up page load times compared to older HTTP/1.1 connections.

# Related Technologies
- **Nginx / Apache:** The classic web server software programs.
- **Express.js:** A popular Node.js framework that simplifies writing web server routing logic.
- **SSL/TLS Certificates:** Cryptographic keys (often provided by Let's Encrypt) used to turn insecure HTTP into secure HTTPS.

# Summary

## What We Learned
- Web servers listen on numbered channels called ports to handle HTTP request-response cycles.
- HTTP defines the communication standards (methods, headers, status codes) used between browsers and servers.
- Securing traffic with HTTPS (Port 443) and using reverse proxies like Nginx are mandatory production steps.

## Key Takeaways
- Use correct HTTP status codes to communicate execution outcomes to frontends.
- Enforce secure connections using SSL certificates.

# Keywords
- Web Server
- HTTP
- Port
- GET
- POST
- Status Code
- Headers
- HTTPS

# Glossary

| Term | Meaning |
|---|---|
| Port | A virtual network channel used by computers to separate different types of traffic (e.g. web vs email). |
| Web Server | Software that listens for incoming HTTP requests and returns web assets or data. |
| HTTP Method | The verb in an HTTP request indicating what action the client wishes to perform (GET, POST, etc.). |
| Headers | Metadata key-value pairs sent along with HTTP requests and responses containing details like content type or cookie credentials. |
| Reverse Proxy | A server gateway that sits in front of backend applications to distribute traffic, cache resources, and improve security. |

## Next Recommended Chapters
- 02-Web-Development/02-HTTP-And-HTTPS.md
- 04-Backend/02-REST-APIs-And-GraphQL.md
