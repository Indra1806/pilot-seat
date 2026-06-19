> **Mode:** Book
> **Pilot-Seat Standard**

# Introduction
**System Design Case Studies** analyze real-world application designs, demonstrating how fundamental system components (load balancers, web servers, caches, and databases) are combined to build scalable, reliable production systems. This chapter explores the step-by-step architecture of two classic system designs: a **URL Shortener** (TinyURL) and a **Pastebin** text-sharing service.

# Why It Exists
Learning individual system design concepts (like sharding or load balancing) is like studying individual engine parts. To build a functioning car, you must see how these parts connect, interact, and balance each other's limitations. System design case studies provide complete end-to-end blueprints that teach developers how to analyze business requirements, calculate resource capacity, design database schemas, and scale infrastructure logically.

# Problem It Solves
System design case studies solve architectural blind spots, helping engineers avoid over-engineering simple features and under-engineering high-scale bottlenecks.

### Before Case Study Analysis (Ad-hoc Design):
- Systems were built without mathematical estimation, leading to servers running out of disk space in weeks.
- Databases were overloaded with read queries that could have easily been resolved using simple caches.
- Large files (like raw code pastes) were stored directly in database cells, slowing down the database.

### After Case Study Analysis (Structured Blueprints):
- Capacity estimations are calculated first, ensuring the storage and network capacity are sized correctly for the next 5 years.
- Read-heavy systems are optimized using Redis caches, protecting primary databases.
- Heavy data payloads are separated, storing metadata in relational databases and large files in object storage.

# Core Concepts
To master case studies, you must learn to apply structured design steps:

1. **Functional vs. Non-Functional Requirements:**
   - **Functional:** What the system *does* (e.g., user inputs a long URL, system returns a short code).
   - **Non-Functional:** How the system *behaves* (e.g., redirect lookups must complete in under 50ms, data must be highly available).
2. **Back-of-the-Envelope Math:** High-level estimations of write volume, read queries per second (RPS), storage growth, and bandwidth capacity.
3. **Metadata vs. Payload Separation:** The database design practice of storing small query indices (metadata) in fast databases, while storing large raw payloads (like files or text blocks) in cheap, dedicated cloud storage.

# Architecture / Components
The high-level architecture of a URL Shortener designed to resolve redirects instantly using a cache-aside layer:

```text
  [ User clicks tinyurl.com/a8f23b ]
                 │
                 ▼
         [ Load Balancer ]
                 │
                 ▼ (Routes request)
       [ Web Server Node ]
                 │
        ┌────────┴────────┐
        ▼ (Check Cache)   ▼ (On Cache Miss query)
     [ Redis Cache ]   [ PostgreSQL DB ] (Stores: short_code -> long_url)
     - Fast lookup     - Slow lookup
     - Returns 1ms     - Returns 45ms
```

- **TinyURL Redirection:** Reads are 100 times more frequent than writes, requiring heavy caching.
- **Pastebin Storage:** Writes have heavy payloads, requiring object storage integration.

# Workflow
The workflow of creating a new Pastebin entry:

```text
Step 1: User pastes a large 50KB code snippet and clicks "Save".
                             ↓
Step 2: The Web Server receives the request and generates a unique ID (e.g. `pst_87a32`).
                             ↓
Step 3: The Web Server saves the raw 50KB text file directly to Cloud Object Storage (S3).
                             ↓
Step 4: The Web Server writes a record to the Database: `{"paste_id": "pst_87a32", "s3_url": "s3://bucket/pst_87a32.txt", "expire_at": "2026-06-26"}`.
                             ↓
Step 5: The Web Server returns the shareable link to the user: `example.com/paste/pst_87a32`.
```

# Real World Examples
Think of these case studies as **post office sorting designs**.
- **Case Study 1: URL Shortener (TinyURL)**
  - Think of a URL shortener as a **luggage tag system**. Instead of writing your full address, email, phone number, and flight history directly on every single bag handle (which takes up too much space), the counter agent stamps your bag with a 6-character tag code (**Short Code**).
  - At the airport, when handlers scan the tag code, they check a registry list (**Database Cache**) that maps the tag code to your full address.
  - **Base62 Encoding Math:** To make codes short, we use letters `a-z`, `A-Z`, and numbers `0-9`. This gives us 62 characters to choose from. A 7-character code gives us \(62^7 = 3.5\text{ trillion}\) unique luggage tags—more than enough for everyone on Earth.
- **Case Study 2: Pastebin**
  - Think of Pastebin as a **hotel luggage locker service**.
  - If a guest wants to store three giant suitcases (large text pastes), the receptionist (**Web Server**) doesn't try to squeeze the suitcases into the tiny reception desk drawer (**Relational Database**). 
  - Instead, the receptionist takes the suitcases, places them in a large basement locker (**Object Storage**), locks the door, and writes the locker room number (*Locker #104*) on a tiny paper receipt card (**Database Metadata record**) which they file in the desk drawer. 
  - When the guest returns with their receipt, the receptionist reads the locker number and retrieves the suitcases from the basement.

# Implementation
Here is how to implement the core redirection logic for a URL Shortener in Node.js, showing how HTTP redirection statuses are returned:

### URL Shortener Redirect Server (Express)
```javascript
const express = require('express');
const { createClient } = require('redis');
const { Pool } = require('pg');

const app = express();
const db = new Pool();
const cache = createClient();
cache.connect().catch(console.error);

app.get('/:shortCode', async (req, res) => {
  const shortCode = req.params.shortCode;
  const cacheKey = `short:${shortCode}`;

  try {
    // 1. Check Redis Cache first
    const cachedUrl = await cache.get(cacheKey);
    if (cachedUrl) {
      console.log("Cache Hit! Redirecting...");
      // Use 302 Found (Temporary Redirect) so browsers don't cache redirects permanently,
      // allowing us to collect click analytics every time.
      return res.redirect(302, cachedUrl);
    }

    // 2. Cache Miss: Query SQL Database
    console.log("Cache Miss. Querying database...");
    const dbResult = await db.query('SELECT long_url FROM urls WHERE short_code = $1', [shortCode]);
    
    if (dbResult.rows.length === 0) {
      return res.status(404).send("URL not found");
    }

    const longUrl = dbResult.rows[0].long_url;

    // 3. Save to Cache for future visitors
    await cache.set(cacheKey, longUrl, { EX: 86400 }); // Cache for 24 hours

    res.redirect(302, longUrl);
  } catch (error) {
    console.error("Redirect failed:", error.message);
    res.status(500).send("Server Error");
  }
});
```

# Best Practices
- **Define Requirements First:** Never start drawing an architecture diagram during a system design review until you have spent 5 minutes clarifying the requirements (e.g. read/write ratios, file sizes, and user counts).
- **Separate Large Payloads:** Do not store large blocks of text or raw media files (like images, videos, or documents) inside standard SQL or NoSQL database tables. Keep your databases lightweight by storing files in object storage and storing only file paths in the database.
- **Use 302 Redirects for Analytics:** When redirecting users in a URL shortener, use `302 Found` (temporary redirect) instead of `301 Moved Permanently`. A 301 redirect causes the user's browser to cache the destination, meaning subsequent clicks will bypass your servers completely, preventing you from counting clicks.

# Industry Standards
URL shorteners and text sharing platforms operate under extreme read-heavy loads. To handle this, systems like Bitly cache 99% of redirects at edge CDN locations, meaning client redirection requests are resolved at local city servers without ever hitting the primary databases in the origin datacenter.

# Common Mistakes
- **Using UUIDs for Short Links:** Using standard UUIDs (e.g., `123e4567-e89b-12d3-a456-426614174000`) as the short link code. This results in incredibly long URLs, defeating the purpose of a URL shortener. Use Base62 hashing algorithms.
- **Database Bloat in Pastebin:** Trying to store raw text pastes in relational database `TEXT` columns. As millions of users paste large code blocks, the database backup sizes and index memory requirements will explode, resulting in massive storage costs and database slowdowns.
- **No Expiration Cleaning:** Allowing temporary links and paste files to sit on S3 storage forever. Systems must run automated background cron-jobs to clean up expired files daily.

# Security & Performance Considerations
- **URL Scraping and Brute Forcing:** Because short codes are short (e.g. `tinyurl.com/a8f2b`), malicious scrapers can write scripts to query every alphabetical combination sequentially, downloading the entire database of long URLs. Teams must configure aggressive rate-limiters on redirect endpoints.
- **Object Storage Retrieval Latency:** Fetching large text files from cloud object storage (S3) takes longer than reading database memory. To optimize Pastebin loading speeds, developers cache the most recently viewed pastes directly in Redis.

# Related Technologies
- **Base62 Encoding:** An algorithm that converts numbers into a sequence of alphanumeric characters to generate short link codes.
- **Amazon S3 / Google Cloud Storage:** Cloud object storage systems used to store large text payloads for Pastebin services.
- **Crontab / Celery Beat:** Scheduling tools used to trigger daily database cleanup scripts.

# Summary

## What We Learned
- System design case studies demonstrate how to combine fundamental components logically to meet scaling requirements.
- URL shorteners are read-heavy architectures that rely heavily on Redis caching and Base62 encoding.
- Pastebin services use payload separation: saving metadata in databases and large text payloads in cloud object storage.

## Key Takeaways
- Always split metadata (stored in SQL) from large raw payloads (stored in S3) to protect database performance.
- Use temporary HTTP `302` redirects to ensure client clicks hit your servers, allowing accurate analytics tracking.
- Apply Base62 encoding to numerical keys to generate short, user-friendly URLs.

# Keywords
- Case Study
- TinyURL
- Pastebin
- Base62 Encoding
- Payload Separation
- Object Storage
- Temporary Redirect (302)
- Metadata
- Rate Limiting
- Cron Job

# Glossary

| Term | Meaning |
|---|---|
| Base62 Encoding | A counting system using 62 symbols (0-9, a-z, A-Z) to compress large numbers into short text codes. |
| Metadata | Structural indexing data (like timestamps, sizes, or URLs) that describes a larger data payload. |
| Object Storage | A flat file storage service (like AWS S3) optimized for storing unstructured data files. |
| 302 Found | An HTTP redirection status code indicating that the resource resides temporarily under a different URL. |
| 301 Moved Permanently | An HTTP status code that instructs browsers to cache the redirection destination forever. |
| Cache Miss | A state where a lookup key is not found in cache memory, forcing a disk database read. |

## Next Recommended Chapters
- 14-Large-Scale-Architectures.md
