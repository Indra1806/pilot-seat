> **Mode:** Book
> **Pilot-Seat Standard**

# Introduction
A **Content Delivery Network (CDN)** is a globally distributed network of proxy servers (referred to as **Edge Servers** or Points of Presence - PoPs) that work together to deliver internet content—such as images, video files, stylesheet scripts, and HTML pages—to users rapidly based on their physical location.

# Why It Exists
Network data packets travel through fiber-optic cables at the speed of light. However, physical distance still introduces delay (network latency). If your application's primary server (the **Origin**) is located in New York, and a user in Sydney, Australia attempts to load your website, the request packets must travel 10,000 miles across undersea cables, taking hundreds of milliseconds. If the page contains 50 images, the user will experience significant lag. Engineers created CDNs to store copies of static website files in edge servers located in major cities around the world, reducing the physical distance data must travel.

# Problem It Solves
CDNs solve slow page load times caused by geographical distance, origin server traffic overload, and network bandwidth costs.

### Before CDNs (Single Location Origin):
- Users located far away from the server experienced slow, laggy page load times.
- Sudden traffic spikes (like viral video launches) overloaded the single origin server, causing crashes.
- The company paid high network bandwidth bills because all data was sent from a single central server.

### After CDNs (Distributed Edge Delivery):
- Web pages load in milliseconds because content is served from an edge server in the user's local city.
- The origin server is shielded from 80% to 90% of incoming read traffic, protecting it from crashing.
- Bandwidth costs drop because global CDN networks cache and serve the bulk of files locally.

# Core Concepts
To design CDN integrations, you must understand edge caching, push/pull models, and origin servers:

1. **Origin Server:** The primary host database and application server where the source of truth code and database records live.
2. **Edge Server (Point of Presence - PoP):** CDN servers located at the edges of the internet (close to users' local internet service providers) designed to cache and serve static files.
3. **Pull Cache Model:** The CDN edge server is empty at first. When a user requests a file, the edge server pulls it from the origin server, caches a copy, and returns it. Subsequent local users receive the cached copy directly.
4. **Push Cache Model:** The origin server proactively uploads (pushes) static files to the CDN edge servers whenever content is created or updated, ensuring files are pre-loaded before any user requests them.

# Architecture / Components
The geographical routing flow of client requests being intercepted and served by local CDN edge nodes:

```text
                  [ User in Tokyo ]                      [ User in London ]
                          │                                      │
                          ▼ (Requests image.png)                 ▼ (Requests image.png)
                  [ Tokyo Edge Server ]                  [ London Edge Server ]
                   - Returns cached copy                  - Returns cached copy
                   - Time: 15ms                           - Time: 12ms
                          │                                      │
                          └───────────────────┬──────────────────┘
                                              ▼ (Cache Miss routes to)
                                    [ Origin Server (New York) ]
```

- **Anycast Routing:** A network addressing method that routes client requests automatically to the physically nearest CDN edge server sharing the same IP address.
- **Cache Hit Ratio:** The percentage of request queries resolved by edge servers without needing to contact the origin server.

# Workflow
The retrieval workflow under a Pull-based CDN configuration:

```text
Step 1: A user in London requests `example.com/logo.png`.
                             ↓
Step 2: Anycast routing detects the user's location and routes the request to the London CDN Edge Server.
                             ↓
Step 3: The edge server checks its memory: "Do I have `logo.png`?"
        - If YES (Cache Hit): Return the file. Done (12ms).
                             ↓
Step 4: If NO (Cache Miss): The edge server queries the Origin Server in New York over a fast dedicated network.
                             ↓
Step 5: The Origin Server returns the file to the London Edge Server.
                             ↓
Step 6: The London Edge Server saves a copy locally on its disk and delivers it to the user.
```

# Real World Examples
Think of a CDN as a **global book publisher's distribution warehouse network**.
- **Without a CDN (Origin only):** You write and print books in New York. If a reader in Tokyo wants a copy, you must package the book and mail it across the Pacific Ocean. It takes 2 weeks to arrive (**High latency**).
- **With a CDN:** You rent small storage warehouses (**Edge Servers**) in London, Tokyo, Paris, and Sydney.
- **Pull Model:** The Tokyo warehouse starts empty. When a Tokyo reader orders a book, the Tokyo warehouse calls New York, orders a shipping crate, keeps 10 copies on its local shelf, and hands 1 copy to the reader. When the next Tokyo reader orders, the warehouse pulls it off the local shelf instantly.
- **Push Model:** The moment you print a new book in New York, you proactively ship 100 copies to Tokyo, London, and Paris. The books are waiting on the shelves before anyone even orders.
- **Static vs. Dynamic:** You can cache printed books (static files like images). You cannot cache custom, handwritten letters containing a user's bank statement (dynamic data like user profiles).

# Implementation
Here is how frontend developers point static website assets (like images or stylesheets) to a CDN rather than loading them directly from their local backend servers:

### HTML Asset Routing with CDN URLs
Instead of requesting assets relative to the backend server, developers rewrite URLs to point to the CDN domain:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>My E-Commerce Site</title>
  
  <!-- 1. BAD: Loading stylesheets directly from the origin server -->
  <!-- <link rel="stylesheet" href="/css/styles.css"> -->

  <!-- 2. GOOD: Loading stylesheets from a global CDN edge server -->
  <link rel="stylesheet" href="https://cdn.mywebsite.com/css/styles.css">
</head>
<body>
  <h1>Welcome to the Store</h1>
  
  <!-- Loading product images from the CDN -->
  <img src="https://cdn.mywebsite.com/images/products/phone_4k.jpg" alt="Smartphone">
</body>
</html>
```

# Best Practices
- **Cache Static Assets Indefinitely:** Configure static assets (images, fonts, compiled CSS/JS) with long HTTP cache headers (e.g. `Cache-Control: max-age=31536000` - 1 year).
- **Use Cache Busting (Versioning):** Since static files are cached at edge servers for up to a year, if you edit a CSS file, users won't see changes. Use cache-busting filenames containing hashes (e.g., rename `styles.css` to `styles.a8f23b.css`) to force the CDN to pull the new file.
- **Minimize Dynamic Logic at the Edge:** Keep the CDN focused on static assets. While modern CDNs allow running code at the edge (Edge Workers), overusing edge computing increases complexity and cost.

# Industry Standards
Almost all modern high-traffic websites utilize CDNs (like Cloudflare, Fastly, or AWS CloudFront). They configure CDNs as a **Reverse Proxy** shield, pointing their main domain's DNS records directly to the CDN. This protects the origin server by absorbing and filtering all traffic at the edge.

# Common Mistakes
- **Caching Private User Pages:** Accidentally configuring the CDN to cache private pages (like `example.com/dashboard` or billing screens). The next visitor to hit that edge server will see the previous user's private financial data, resulting in a severe security leak.
- **Forgetting to Purge on Critical Updates:** Updating a homepage image file directly (replacing `banner.jpg` without changing its name) and forgetting to send a "Purge" command to the CDN API. Users will continue to see the old cached banner image for weeks.
- **Exposing the Origin Server IP:** Failing to block public traffic from accessing your origin server directly. If hackers find your origin server's direct IP address, they can bypass the CDN completely and run DDoS attacks to crash your site. Secure the origin by only accepting traffic coming from the CDN's IP range.

# Security & Performance Considerations
- **DDoS Mitigation (Edge Filtering):** CDNs have massive network capacity (hundreds of terabits per second). When a Distributed Denial of Service (DDoS) attack occurs, the CDN absorbs the traffic across thousands of edge servers, filtering out malicious bot requests before they reach your origin.
- **SSL Handshake Latency:** Handshaking TLS keys across the globe takes time. Setting up SSL termination at the edge allows the handshake to complete locally at the nearest edge server, reducing latency for secure connections.

# Related Technologies
- **AWS CloudFront:** Amazon's managed cloud Content Delivery Network.
- **Cloudflare Workers:** A serverless platform that allows running javascript code directly on CDN edge servers.
- **Anycast DNS:** A DNS routing system that resolves domain names to the nearest physical IP node.

# Summary

## What We Learned
- CDNs decrease latency by caching and serving static web content from edge servers close to the user's physical location.
- Pull caches load data lazily on first query; Push caches load data proactively to prevent first-query lag.
- CDNs shield origin servers from bulk read traffic and absorb DDoS attacks at the network perimeter.

## Key Takeaways
- Use versioned hashes in filenames (cache busting) to ensure immediate deployment updates for cached files.
- Restrict your origin server firewall to only accept connections originating from your CDN provider's IP range.
- Never allow public CDNs to cache pages that contain private, session-specific user details.

# Keywords
- Content Delivery Network (CDN)
- Edge Server
- Point of Presence (PoP)
- Origin Server
- Cache Hit Ratio
- Cache Busting
- Anycast
- Pull Model
- Push Model
- DDoS Protection

# Glossary

| Term | Meaning |
|---|---|
| Edge Server | A CDN server located close to the user network designed to cache and serve local file copies. |
| Origin Server | The primary server hosting the source database and core application logic. |
| Cache Busting | Changing the URL/filename of static assets (using hashes) to force a cache reload. |
| Anycast | A network routing system where multiple servers share the same IP address, directing traffic to the nearest node. |
| Point of Presence (PoP) | A physical location where a CDN provider houses edge servers to interface with local networks. |
| TTL | Time-To-Live; the cache header specifying how long an edge server can store a file before querying the origin. |

## Next Recommended Chapters
- 07-Message-Queues.md
