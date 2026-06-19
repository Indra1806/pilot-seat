> **Mode:** Book
> **Pilot-Seat Standard**

# Introduction
**Caching** is the architectural practice of storing temporary copies of frequently accessed data in a high-speed, temporary storage layer (usually RAM) located closer to the processing unit or user. By resolving read queries from this fast cache layer, systems bypass slow disk operations and heavy calculations, dramatically improving response speeds.

# Why It Exists
In computer architecture, data storage speed is inversely proportional to capacity and cost. Hard drives (SSDs/HDDs) can store terabytes of data cheaply, but reading from them is relatively slow, taking milliseconds. Random Access Memory (RAM) is incredibly fast (nanoseconds) but expensive and limited in capacity. Caching exists to bridge this gap, keeping a small, highly relevant subset of data in expensive, ultra-fast RAM, while keeping the main, massive dataset on cheaper, slower disks.

# Problem It Solves
Caching solves slow page load times, database read overload bottlenecks, and redundant computational calculations.

### Before Caching (Direct Disk Queries):
- Every user landing on a news homepage triggered a slow SQL query to pull the exact same articles from disk, causing database crashes.
- Users located in London had to wait seconds for images to travel across undersea cables from a primary server located in San Francisco.
- Computing a user's monthly expense report took several seconds of heavy CPU calculation every single time they opened the dashboard.

### After Caching (Multi-Tier Caching):
- The news homepage is loaded from memory in under a millisecond, completely shielding the database.
- Static images are cached in the user's browser or at a local city server (CDN), loading instantly.
- The expense report is calculated once, saved in the cache, and retrieved instantly on subsequent views.

# Core Concepts
To design cached systems, you must understand cache tiers, invalidation strategies, and eviction policies:

1. **Caching Tiers:** Caches can be placed at multiple levels in a system:
   - **Browser Cache:** Storing files locally on the user's phone or computer.
   - **CDN Cache:** Edge servers located around the globe to serve static files locally.
   - **Application Cache:** In-memory databases (like Redis) running next to application servers.
   - **Database Buffer:** Internally cached disk blocks in database RAM.
2. **Cache Invalidation:** The process of updating or deleting cached data when the source data changes.
3. **Cache Eviction:** The method used to choose which data to delete when the cache runs out of memory.
   - **LRU (Least Recently Used):** Deletes the items that have not been read for the longest time.
   - **FIFO (First In, First Out):** Deletes the oldest cached items first, regardless of usage.

# Architecture / Components
The flow of write data through different **Cache Invalidation Policies**:

```text
  Write-Through Policy (Consistent, Slower Writes)
  [ Backend App ] ─── (Write Update) ───> [ Cache ] ─── (Write Update) ───> [ Database ]
  * Write is only complete when both structures save.

  Write-Back Policy (Fast Writes, Risk of Data Loss)
  [ Backend App ] ─── (Write Update) ───> [ Cache ] (Write Complete!)
                                             │
                                             ▼ (Delayed background batch write)
                                        [ Database ]
```

- **Cache-Aside (Lazy Loading):** The application reads from the cache; on a cache miss, it reads from the database and manually updates the cache.
- **Write-Through:** Data is written to the cache and database simultaneously.
- **Write-Back (Write-Behind):** Data is written to the cache instantly; the cache updates the database asynchronously later.

# Workflow
The standard retrieval loop under a Cache-Aside strategy:

```text
Step 1: Application requests product details for ID #50.
                             ↓
Step 2: Check Cache: `GET product:50`.
        - If found (Cache Hit): Return data to client. Done.
                             ↓
Step 3: If not found (Cache Miss): Query SQL Database `SELECT * FROM products WHERE id = 50`.
                             ↓
Step 4: Database reads disk and returns row data.
                             ↓
Step 5: Write data to Cache with an expiration: `SET product:50 [data] EX 600`.
                             ↓
Step 6: Return product details to client.
```

# Real World Examples
Think of caching as **managing study notes for a research paper**.
- **Without Caching:** Every time you need to write a sentence about a historical date, you stand up, walk to the public library, search the catalog, find the book, write down the date, walk home, and sit down. This is a **Cache Miss / Disk Query** and takes 2 hours.
- **With Caching:** The first time you go to the library, you copy the 20 most important historical dates onto a small index card (**Cache**) and lay it on your desk. The next time you need a date, you glance at the card. It takes 1 second (**Cache Hit**).
- **Tiers of Caching:**
  - **Browser Cache:** The index card is in your shirt pocket. You don't even need to look at your desk.
  - **CDN Cache:** The book is kept at a mini-library on your block, rather than the city central archive.
  - **Application Cache:** The index card on your desk.
- **Write-Through Invalidation:** Every time you change your research hypothesis, you write it on your index card and in your final paper draft simultaneously.
- **Write-Back Invalidation:** You scribble ideas on your scratchpad throughout the day, and copy them into your final paper once a week. If you spill coffee on your scratchpad, you lose all draft updates.
- **Eviction (LRU):** Your desk has limited space. When it is full and you need to add a new index card, you throw away the card you haven't looked at for the longest time.

# Implementation
Here is how to implement the standard Cache-Aside pattern in Javascript using Express and Redis:

### Caching Middleware implementation (Node.js)
```javascript
const express = require('express');
const { createClient } = require('redis');

const app = express();
const redisClient = createClient();
redisClient.connect().catch(console.error);

// Middleware to check cache before hit backend routing
async function cacheMiddleware(req, res, next) {
  const cacheKey = `url:${req.originalUrl}`;
  
  try {
    const cachedData = await redisClient.get(cacheKey);
    if (cachedData) {
      console.log("Cache Hit!");
      return res.json(JSON.parse(cachedData)); // Return cached value immediately
    }
    
    // If cache miss, override res.send to save data into cache before sending
    res.sendResponse = res.json;
    res.json = (body) => {
      // Save data to redis with 5 minute expiration (300 seconds)
      redisClient.set(cacheKey, JSON.stringify(body), { EX: 300 });
      res.sendResponse(body);
    };
    
    next();
  } catch (err) {
    console.error("Cache error, bypassing to database:", err);
    next(); // Bypass cache on error to ensure app doesn't crash
  }
}

// Route using the cache middleware
app.get('/api/articles', cacheMiddleware, async (req, res) => {
  // Simulate slow database read query (2 seconds)
  await new Promise(resolve => setTimeout(resolve, 2000));
  
  const articles = [
    { id: 1, title: 'System Design 101' },
    { id: 2, title: 'Mastering Caching' }
  ];
  
  res.json(articles); // Trigger modified res.json to save cache
});
```

# Best Practices
- **Configure Time-To-Live (TTL):** Never store cached data without an expiration timer. If you cache a user's settings profile forever, they will never see updates they make to their profile unless you manually delete the cache key.
- **Set Up Memory Alert Limits:** Ensure your cache server is configured with a maximum memory limit and a Least Recently Used (`LRU`) eviction policy. If Redis runs out of memory, it should delete old keys rather than crashing the system.
- **Design for Cache Miss Latency:** Always assume the cache will eventually go down or expire. Design your backend databases to handle the query traffic in the event of a cache outage.

# Industry Standards
Modern high-performance web infrastructures utilize a layered caching approach. Static images and videos are cached at the edge using CDNs (like Cloudflare or Akamai), static HTML structures are cached in Varnish reverse proxies, and internal business calculations are cached in Redis clusters.

# Common Mistakes
- **Caching Highly Dynamic Data:** Attempting to cache data that changes every second (like real-time stock prices or GPS locations). The cost of constantly invalidating and rewriting the cache is higher than simply reading directly from the source.
- **Cache Invalidation Desynchronization:** Updating a user's address in the SQL database but forgetting to clear the corresponding cached key in Redis. The user will see their old address until the TTL expires, leading to delivery errors.
- **Failing to Cache Empty Results:** If a user searches for an item that doesn't exist, the database returns `null`. If you don't cache this empty result, and attackers query for non-existent items millions of times, they will bypass the cache entirely and overload the database. This is called **Cache Penetration**.

# Security & Performance Considerations
- **Cache Serialization Overhead:** Converting complex programming objects into JSON strings (serialization) to save in Redis, and parsing them back into objects (deserialization) on reads. For massive payloads, the CPU cost of parsing JSON can become slower than the database query itself. Keep cached objects small.
- **Sensitive Data Caching:** Avoid caching unencrypted sensitive data (like user passwords or credit card tokens) in Redis. If an attacker gains access to the Redis server memory, they can read all sensitive user data in plain text.

# Related Technologies
- **Memcached:** A simple, high-performance in-memory key-value cache system.
- **Varnish Cache:** A specialized reverse proxy cache designed to store and serve static HTML pages.
- **Cloudflare:** A global Content Delivery Network (CDN) providing edge caching services.

# Summary

## What We Learned
- Caching speeds up systems by keeping copies of active data in fast RAM, bypassing slow disk lookups.
- Cache-Aside is the default lazy-loading strategy; Write-Through maintains consistency; Write-Back optimizes write speeds.
- Expiration rules (TTL) and eviction policies (LRU) prevent cache servers from serving stale data or running out of memory.

## Key Takeaways
- Never cache highly volatile data; focus caching on read-heavy, slow-changing records.
- Set strict LRU eviction policies to prevent out-of-memory crashes on cache nodes.
- Cache empty query results (nulls) to protect backend databases from cache penetration attacks.

# Keywords
- Caching
- Cache Hit
- Cache Miss
- Write-Through
- Write-Back
- Cache-Aside
- TTL (Time-To-Live)
- LRU (Least Recently Used)
- Eviction
- Cache Penetration

# Glossary

| Term | Meaning |
|---|---|
| Caching | The practice of storing copies of active data in temporary high-speed RAM to speed up retrieval. |
| Cache Hit | A state where requested data is found inside the cache layer, avoiding a database lookup. |
| Cache Miss | A state where requested data is missing from the cache, forcing a read from the slower database. |
| Cache Invalidation | The process of updating or clearing cache keys when the source data changes. |
| Cache Eviction | The automatic deletion of cached keys to reclaim memory space when limits are reached. |
| Cache Penetration | A performance bottleneck where queries for non-existent keys bypass the cache and hit the database. |

## Next Recommended Chapters
- 05-Database-Scaling.md
