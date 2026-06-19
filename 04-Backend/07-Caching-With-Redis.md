> **Mode:** Book
> **Pilot-Seat Standard**

# Introduction
**Caching** is the practice of saving a temporary copy of frequently requested or expensive data in a high-speed storage layer so that subsequent requests for that data can be answered almost instantly. **Redis** (Remote Dictionary Server) is the industry-standard software tool used to build caches. It is an open-source, in-memory key-value database, meaning it stores all its data directly in the server's fast RAM (Random Access Memory) instead of writing to a slow hard drive.

# Why It Exists
Traditional databases (like PostgreSQL or MongoDB) store their data on hard drives. When a user requests data, the database engine must spin up a query, read files from the disk, parse relationships, and load it into memory. This disk reading takes milliseconds. If your website goes viral and 10,000 users visit your homepage at the same second, the database hard drive will choke under the flood of repeated disk-read requests, slowing the site to a crawl. Engineers created Redis to offload this burden by caching the results in ultra-fast RAM, avoiding the hard drive entirely for repeated queries.

# Problem It Solves
Caching and Redis solve slow database read speeds, server load bottlenecks, and network latency problems.

### Before Caching (Database-only access):
- The server ran the exact same expensive database query 10,000 times a minute to show the exact same static homepage text, wasting CPU power and disk health.
- Page load speeds were limited by hard drive speeds.
- High traffic events (like black Friday sales) regularly crashed backend databases.

### After Caching (Redis integration):
- The server queries the database once, stores the result in Redis, and serves subsequent visitors directly from memory.
- Responses are delivered in microseconds instead of milliseconds.
- Database load drops by up to 90%, ensuring the system stays stable during traffic spikes.

# Core Concepts
To write caching layers, you must master the cache-read lifecycle:

1. **Cache Hit vs. Cache Miss:**
   - **Cache Hit:** The server searches for the requested data in Redis, finds it, and returns it immediately.
   - **Cache Miss:** The server searches Redis but the data is not there. The server must fall back to query the main database, write the result into Redis so it is there next time, and return the data.
2. **TTL (Time to Live):** A timer set on a cached item (e.g. 5 minutes). When the timer expires, Redis automatically deletes the item, forcing the server to get fresh data from the main database next time. This prevents the cache from serving outdated (stale) data forever.
3. **Cache Invalidation:** The process of manually deleting a cached item when the underlying data changes (e.g., if an author edits a blog post, your code must immediately delete the old cached post from Redis so visitors see the new edits).

# Architecture / Components
The flow of traffic through a caching layer:

```text
  [ User Request ] ──> [ Backend Server ]
                              │
                    (Check Redis Cache)
                    ┌─────────┴─────────┐
                 Cache Hit           Cache Miss
                    ▼                   ▼
             [ Read RAM (Redis) ]  [ Read Disk (Postgres) ]
             (Delivered instantly)      │
                                        ▼
                                 [ Write to Redis ]
                                 (Save copy for next time)
```

- **Redis Server:** The standalone database process storing key-value pairs in RAM.
- **In-Memory Storage:** Storing active data in volatile RAM (extremely fast) rather than non-volatile solid-state drives (SSD) or hard drives.
- **Key-Value Store:** Storing data as simple label-to-content pairs (e.g., Key: `user:101:profile`, Value: `{"name":"Alice"}`).

# Workflow
The standard Cache-Aside workflow:

```text
Step 1: A user requests a product profile: `GET /products/77`.
                             ↓
Step 2: The server checks Redis for the key `product:77`.
                             ↓
Step 3: If Redis has it (Cache Hit), the server returns the JSON immediately, skipping steps 4-5.
                             ↓
Step 4: If Redis doesn't have it (Cache Miss), the server queries the PostgreSQL database.
                             ↓
Step 5: The server writes the query result into Redis with a 1-hour TTL: `SET product:77 data EX 3600`.
                             ↓
Step 6: The server returns the product data to the user.
```

# Real World Examples
Think of caching with Redis as **desk drawer filing**.
- A SQL database (like PostgreSQL) is like the **main library archive vault in the basement**. If a customer asks you for a book, you have to walk down the stairs, search the shelves, find the book, walk back up, and hand it over. This takes minutes (an expensive database query).
- Redis is like a **small desk drawer** right in front of you at the reception desk.
- The first time a customer asks for a book, you walk to the basement archive, get it, write a copy of the key pages, and slide the copy into your desk drawer (Cache Miss -> Database Fetch -> Cache Update).
- The next 50 times customers ask for that exact same book, you simply pull open your desk drawer and hand them the copy instantly (Cache Hit). It takes 2 seconds.
- You place a sticky note on the paper saying: *"Throw away after 1 hour"* (TTL) so you eventually go back to the basement to check for updates.

# Implementation
Here is how you write a Cache-Aside query wrapper in Node.js using the standard `redis` client:

```javascript
const { createClient } = require('redis');
const express = require('express');
const app = express();

// 1. Initialize the Redis client and connect to the local Redis server
const redisClient = createClient();
redisClient.connect().then(() => console.log("Connected to Redis!"));

// Mock Database fetch function
async function fetchUserFromDatabase(id) {
  // Simulates a slow database query taking 1.5 seconds
  return new Promise(resolve => {
    setTimeout(() => {
      resolve({ id, name: "Alice", email: "alice@email.com" });
    }, 1500);
  });
}

// 2. Setup the cached API route
app.get('/api/users/:id', async (req, res) => {
  const userId = req.params.id;
  const cacheKey = `user:${userId}`;

  try {
    // A. Check if the user exists in the Redis memory cache
    const cachedData = await redisClient.get(cacheKey);

    if (cachedData) {
      console.log("CACHE HIT! Returning data from Redis RAM...");
      return res.json(JSON.parse(cachedData)); // Return cached JSON immediately
    }

    // B. Cache Miss: Fetch from the slow database
    console.log("CACHE MISS! Fetching from slow database...");
    const user = await fetchUserFromDatabase(userId);

    // C. Write the data to Redis, set an expiration (TTL) of 300 seconds (5 minutes)
    await redisClient.set(cacheKey, JSON.stringify(user), {
      EX: 300
    });

    // D. Return the data to the user
    return res.json(user);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

app.listen(3000);
```

# Best Practices
- **Cache Only Read-Heavy, Slow Data:** Do not cache data that changes constantly (like stock tickers or user typing messages). Caching is best for data that is read frequently but updated rarely (like configuration settings, product catalogs, or blog posts).
- **Always Set a TTL (Time to Live):** Never cache data indefinitely. If you forget to set a TTL, your cache will grow until it consumes all the server's RAM and crashes, or it will continue serving stale data forever.
- **Implement Cache Eviction Policies:** Configure Redis to use **LRU (Least Recently Used)** eviction. This tells Redis that if it runs out of memory, it should automatically delete the oldest, least-read keys to make room for new ones.

# Industry Standards
Almost all modern high-traffic web applications use **Redis** or **Memcached** for caching. Redis is preferred because it supports complex data structures (like lists, hashes, and sorted sets) and can save data to disk periodically so it doesn't vanish if the server restarts.

# Common Mistakes
- **Assuming Cache is Always Fresh:** Forgetting to update or invalidate the cache when editing data. For example, a shop owner updates a product price in the database, but customers continue buying it at the old price for hours because the server is serving the stale cached page from Redis.
- **Cache Stampede (Thundering Herd):** When a highly popular cached key expires, and 5,000 requests try to fetch it at the exact same millisecond. Since the cache is empty, all 5,000 requests hit the backend SQL database simultaneously, crashing the database. Use mutex locks to ensure only one process queries the database to rebuild the cache.

# Security & Performance Considerations
- **Memory Limits:** Since Redis stores data in RAM, which is far more expensive than hard drive space, monitor your memory usage. If memory runs out, Redis will refuse new writes unless eviction policies are active.
- **Unauthorized Network Access:** By default, Redis is extremely fast because it does not require authentication. If you expose your Redis port (`6379`) to the public internet, anyone can connect and read all your cached user sessions. Always bind Redis to `127.0.0.1` (localhost) or protect it inside a private VPC network.

# Related Technologies
- **Memcached:** A simpler, high-performance in-memory caching tool.
- **Redis Sentinel / Cluster:** Redis tools used to duplicate data across multiple servers for high availability and automatic failover.

# Summary

## What We Learned
- Caching offloads database reads by saving temporary copies of data in high-speed storage.
- Redis is an in-memory key-value database storing data in RAM for microsecond retrieval.
- Setting TTLs and managing cache invalidation are critical to prevent stale data delivery.

## Key Takeaways
- Use the Cache-Aside pattern (Check cache -> Query database -> Write cache) for standard queries.
- Protect Redis networks from public access to avoid session hijacking.

# Keywords
- Caching
- Redis
- In-Memory
- RAM
- Cache Hit
- Cache Miss
- TTL
- Cache Invalidation

# Glossary

| Term | Meaning |
|---|---|
| In-Memory | Storing data in the computer's volatile RAM instead of writing to non-volatile hard drives or SSDs. |
| Cache Hit | A successful cache query where the requested data is found in memory. |
| Cache Miss | A cache query where the requested data is missing, requiring a database query. |
| TTL | Time to Live; the lifespan configuration indicating when a cache key should automatically expire. |
| Eviction Policy | The rules Redis follows to delete old data keys when it runs out of memory space. |

## Next Recommended Chapters
- 05-Databases-SQL-vs-NoSQL.md
- 04-Backend/09-Background-Jobs-And-Workers.md
