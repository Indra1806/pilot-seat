> **Mode:** Book
> **Pilot-Seat Standard**

# Introduction
WebSockets and Real-time Frontend development refer to the technologies and methods used to build web applications that update instantly when new data is available on a server, without requiring the user to refresh the page or manually click a reload button. It establishes a **persistent, two-way (bidirectional) connection** between the user's browser and the backend server.

# Why It Exists
HTTP, the standard protocol of the web, was designed for a request-response model. The browser requests a document, the server sends it back, and the connection closes immediately. If you were building a live chat app, the browser had no way to know if another user sent a message. The only solution was **polling**: having the browser silently ask the server every 2 seconds: *"Do I have new messages? How about now? How about now?"* This was highly inefficient, wasting server CPU power and bandwidth. Engineers created WebSockets in 2011 to keep a single connection channel open indefinitely so the server can push updates to the browser the millisecond they occur.

# Problem It Solves
WebSockets solve the delay and connection overhead problem in real-time updates.

### Before WebSockets (Polling):
- High latency: Updates only appeared as fast as your polling interval (e.g. every 3 seconds).
- High overhead: Every request sent massive, repetitive HTTP headers, wasting network bandwidth.
- Servers crashed under the load of thousands of clients asking for updates every second.

### After WebSockets (Persistent Streaming):
- True real-time: Messages, prices, or movements appear instantly (millisecond latency).
- Low overhead: Once connected, data packets are tiny and require no heavy headers.
- Servers can manage tens of thousands of active connections simultaneously.

# Core Concepts
To build real-time frontends, you must understand three core concepts:

1. **The Handshake:** The initial connection. A WebSocket starts as a standard HTTP request. The browser sends a special "Upgrade" header asking the server to switch to the WebSocket protocol. If the server agrees, the connection switches from `http://` to `ws://` (or `wss://` for secure connections).
2. **Persistent Connection:** The connection remains open and active. Both the client and the server can send messages at any time.
3. **Heartbeats (Ping/Pong):** Small, silent signals sent back and forth between browser and server to check if the connection is still alive. If a client goes through a tunnel or loses service, the heartbeat fails, and the system knows to try to reconnect.

# Architecture / Components
The WebSocket model establishes a direct, continuous stream between client and server.

```text
       [ Browser Client ]                  [ Backend Server ]
               │                                   │
               │ ─── 1. HTTP Upgrade Handshake ──> │
               │ <─── 2. Handshake Approved ────── │
               │                                   │
               │ ===== 3. Persistent Socket ====== │  <- Open connection
               │ ─── 4. Client Message (Data) ───> │
               │ <─── 5. Server Push (Data) ────── │
               │                                   │
               │ ─── 6. Connection Closed ──────── │
```

- **WebSocket API:** The built-in browser interface (`new WebSocket()`) that manages connections and events.
- **WSS Protocol:** WebSocket Secure (uses SSL/TLS encryption, similar to how HTTPS secures HTTP) to prevent attackers from snooping on the data stream.
- **Event Handlers:** Browser hooks like `onopen`, `onmessage`, `onerror`, and `onclose` that trigger JavaScript actions when the connection state changes.

# Workflow
The lifecycle of a real-time connection:

```text
Step 1: The browser sends an HTTP request with an "Upgrade: websocket" header.
                             ↓
Step 2: The server responds with status code 101, agreeing to switch protocols.
                             ↓
Step 3: The TCP socket stays open, establishing a continuous pipeline.
                             ↓
Step 4: The server pushes new data to the browser (e.g. "User B is typing...").
                             ↓
Step 5: The browser's `onmessage` event fires, updating the UI dynamically.
                             ↓
Step 6: When the user closes the tab, the connection closes cleanly.
```

# Real World Examples
Think of WebSockets as a **telephone call** versus standard HTTP which is like **snail mail**.
- Traditional HTTP is like writing a letter and waiting for a reply. If you want to know if a package has arrived, you must write another letter asking for status.
- A WebSocket connection is like dialing a phone number and keeping the line open. Both you and the other person can speak whenever you want. You do not need to hang up and redial to say your next sentence. You can hear each other instantly, and the call lasts until someone physically hangs up.

# Implementation
Here is how you write simple real-time code in the browser using the native WebSocket API:

```javascript
// 1. Open a secure connection to a chat server
const socket = new WebSocket("wss://echo.websocket.org");

// 2. Handle the successful connection open
socket.addEventListener("open", (event) => {
  console.log("Connected to the real-time server!");
  
  // Send a message to the server
  socket.send("Hello Server!");
});

// 3. Listen for incoming messages pushed by the server
socket.addEventListener("message", (event) => {
  const incomingData = event.data;
  console.log("Message from server: ", incomingData);
  
  // Update the UI dynamically
  const chatBox = document.getElementById("chat-window");
  chatBox.innerHTML += `<p>${incomingData}</p>`;
});

// 4. Handle unexpected connection closures (e.g., loss of internet)
socket.addEventListener("close", (event) => {
  console.log("Connection closed. Attempting to reconnect in 5 seconds...");
  setTimeout(reconnectFunction, 5000);
});
```

# Best Practices
- **Always Use WSS:** Always use the secure `wss://` protocol instead of `ws://`. Standard firewalls and routers frequently block unencrypted WebSocket traffic because they do not recognize the custom protocol pattern.
- **Implement Reconnection Logic:** Mobile users constantly lose cellular signals. Always write code that automatically attempts to reconnect with an "exponential backoff" (waiting longer between each attempt so you do not crash your server with reconnection requests).
- **Clean Up Sockets:** Close connections when components unmount in single-page applications to prevent unused connections from draining browser memory.

# Industry Standards
In production applications, developers rarely write raw WebSockets. They use libraries like **Socket.io** (JavaScript) or **Centrifugo**, which handle automatic reconnections, fallbacks to polling if WebSockets are blocked by proxies, and rooms/channels grouping out of the box.

# Common Mistakes
- **Forgetting Reconnection Handling:** Assuming a WebSocket connection will stay open forever. A user locking their phone or walking out of Wi-Fi range will break the connection immediately.
- **Ignoring Scale Limitations:** Sockets require servers to keep a file descriptor open in memory for *every active user*. While HTTP servers handle requests and free up memory instantly, WebSocket servers must hold connections open, requiring specialized scaling infrastructure (like Redis adapters).

# Security & Performance Considerations
- **WebSocket Hijacking (CSWSH):** Unlike standard HTTP requests, WebSockets are not restricted by the browser's Same-Origin Policy. An attacker's site can open a connection to your WebSocket server using your user's session cookies. Always validate the `Origin` header on your backend server.
- **Denial of Service (DoS):** Because connections stay open, attackers can open thousands of sockets to exhaust your server's memory. Implement strict rate limiting on handshakes.

# Related Technologies
- **Server-Sent Events (SSE):** A simpler, one-way real-time protocol where only the server can push data to the client (great for stock tickers or sports scores).
- **WebRTC:** A protocol used for direct, peer-to-peer real-time video/audio streaming (skipping the server entirely).
- **Socket.io:** A popular library wrapping WebSockets with fallback features.

# Summary

## What We Learned
- WebSockets provide a persistent, bi-directional communication channel over a single socket connection.
- They replace inefficient HTTP polling with low-latency server pushes.
- Real-time applications must secure connections with WSS and build robust reconnection fallbacks.

## Key Takeaways
- Use WSS to guarantee delivery through firewalls and proxies.
- Plan for connection failures: implement heartbeats and reconnection logic.

# Keywords
- WebSockets
- Real-time
- Bidirectional
- Handshake
- Upgrading
- WSS
- Reconnection
- Heartbeat

# Glossary

| Term | Meaning |
|---|---|
| Handshake | The initial negotiation step where the browser and server agree to upgrade a connection to WebSockets. |
| Bidirectional | Two-way communication; both client and server can transmit data simultaneously. |
| Polling | An inefficient technique where the client repeatedly queries the server at fixed intervals to check for updates. |
| WSS | WebSocket Secure; the encrypted version of the WebSocket protocol. |
| Heartbeat | A periodic message exchanged between client and server to verify the connection is still open. |

## Next Recommended Chapters
- 02-Web-Development/02-HTTP-And-HTTPS.md
- 04-Backend/09-Background-Jobs-And-Workers.md
