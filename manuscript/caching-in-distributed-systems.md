# Caching in Distributed Systems: Architecture, Internals, and Production Design

## 1. Introduction

Caching is one of the most important performance optimization techniques in modern distributed systems. At internet scale, databases, storage systems, and external APIs are too slow and expensive to serve every request directly. Caching reduces latency, decreases infrastructure cost, improves throughput, and protects backend systems from overload.

Fundamentally, caching is a trade-off between:

- Freshness
- Consistency
- Performance
- Memory utilization

A well-designed caching layer can reduce response times from hundreds of milliseconds to microseconds.

---

## 2. Core Philosophy of Caching

Caching is based on the observation that data access patterns are rarely random.

Most real-world workloads exhibit:

### 2.1 Temporal Locality

Recently accessed data is likely to be accessed again soon.

Example:
- Trending products
- Viral videos
- Active user sessions

---

### 2.2 Spatial Locality

Data near recently accessed data is likely to be requested soon.

Example:
- Sequential database pages
- Nearby memory addresses
- Adjacent records

---

## 3. Why Caching Exists

Without caching:

- Every request hits the database
- Disk I/O becomes bottleneck
- Network latency accumulates
- CPU usage increases dramatically

At scale, databases are typically incapable of serving:
- millions of QPS
- real-time analytics
- global low-latency traffic

Caching shifts workload from:
- expensive storage
- slower computation
- remote systems

to:
- fast memory
- localized access

---

## 4. Latency Hierarchy (Critical System Insight)

Understanding cache value requires understanding latency differences.

| Resource | Approximate Latency |
|---|---|
| CPU Register | <1 ns |
| L1 Cache | ~1 ns |
| RAM | ~100 ns |
| SSD | ~100 µs |
| HDD | ~10 ms |
| Network Call | ~50–200 ms |

Key insight:

> Memory is thousands to millions of times faster than remote systems.

Most system architecture decisions are fundamentally latency optimization problems.

---

## 5. Cache Hit and Cache Miss

### Cache Hit
Requested data exists in cache.

Result:
- Fast response
- No backend call

---

### Cache Miss
Requested data absent from cache.

Result:
- Backend fetch required
- Increased latency

---

### Cache Hit Ratio

Formula:

    Hit Ratio = Hits / (Hits + Misses)

**Example:**

- Hits = 950
- Misses = 50

Result:

    950 / 1000 = 95%

Even small improvements in hit ratio produce massive infrastructure savings at scale.

---

## 6. Internal Mechanics of a Cache

A cache internally consists of:

- Key-value lookup structure
- Memory allocator
- Eviction policy
- Expiration manager
- Replication subsystem
- Persistence layer (optional)

---

## 7. Key Design Principle: Hot Data

In real systems:
- Small percentage of data receives majority of traffic

This follows:
- Zipfian distribution
- Power-law access patterns

Example:
- 1% of videos may generate 90% of traffic

Caching optimizes for:
> “Keep hot data closest to computation.”

---

## 8. Types of Caching

### 8.1 CPU Cache

Hardware-level caching:
- L1
- L2
- L3

Managed automatically by processors.

---

### 8.2 Application Cache

Stored inside application memory.

Example:
- Java HashMap
- Guava Cache

Advantages:
- Extremely fast
- No network overhead

Disadvantages:
- Limited scalability
- Cache duplication across servers

---

### 8.3 Distributed Cache

External shared cache cluster.

Examples:
- Redis
- Memcached

Advantages:
- Shared state
- Horizontal scalability
- Centralized invalidation

---

### 8.4 CDN (Edge Cache)

Caches content near users globally.

Examples:
- Cloudflare
- Akamai

Caches:
- Images
- Videos
- CSS
- JavaScript

---

## 9. Cache Read Strategies

### 9.1 Cache-Aside (Lazy Loading)

Most common strategy.

#### Workflow

1. Check cache
2. If miss → query DB
3. Store in cache
4. Return result

#### Advantages

- Simple
- Resilient
- Cache populated only when needed

#### Problems

- First request suffers latency
- Potential cache inconsistency

---

### 9.2 Read-Through

Application requests data from cache directly.

Cache itself:
- fetches data
- updates storage

Application is unaware of backend DB.

---

## 10. Cache Write Strategies

### 10.1 Write-Through

Write to:
- cache
- database

simultaneously.

#### Advantages

- Strong consistency
- Cache always updated

#### Disadvantages

- Higher write latency
- Increased write amplification

---

### 10.2 Write-Around

Writes bypass cache entirely.

Used when:
- write-heavy systems
- low re-read probability

---

### 10.3 Write-Back (Write-Behind)

Data written to cache first.

Database updated asynchronously later.

#### Advantages

- Extremely fast writes
- Reduced DB pressure

#### Risks

- Potential data loss
- Complex durability guarantees

---

## 11. Eviction Policies

Cache memory is finite.

When full:
- entries must be removed

### 11.1 LRU (Least Recently Used)

Evicts least recently accessed entry.

Internally implemented using:
- hash map
- doubly linked list

Complexity:
- O(1) access
- O(1) eviction

---

### 11.2 LFU (Least Frequently Used)

Tracks frequency count.

Useful for:
- stable hot datasets

Problem:
- Old popular data may persist forever

---

### 11.3 FIFO

Evicts oldest inserted entry.

Simple but inefficient.

---

### 11.4 TTL (Time-To-Live)

Expiration timestamp attached to key.

Critical for:
- stale data prevention
- eventual consistency

---

## 12. Distributed Cache Architecture

Modern systems use distributed caches between:
- application
- database

Architecture:

Application → Cache → Database

---

## 13. Cache Partitioning (Sharding)

Large caches distribute data across nodes.

### Consistent Hashing

Used to minimize key redistribution.

Benefits:
- scalable
- fault tolerant
- balanced distribution

Used heavily in:
- Distributed cache clusters (e.g., Redis, Memcached)
- CDNs

---

## 14. Replication and High Availability

Caches are often replicated.

Primary node:
- accepts writes

Replica nodes:
- serve reads

### Problem

Replication lag may cause:
- stale reads
- inconsistency

---

## 15. Redis Internals (Architect-Level Understanding)

Redis is not “just a cache.”

It is:
- single-threaded event-driven engine
- in-memory data structure server

### Why Single Thread?

Avoids:
- locks
- context switching
- synchronization overhead

Performance achieved via:
- non-blocking I/O
- memory locality

### Redis Persistence

#### RDB
Snapshot-based persistence.

#### AOF
Append-only log.

Trade-off:
- durability vs performance

### Redis Data Structures

- String
- Hash
- Set
- Sorted Set
- Bitmap
- HyperLogLog

These enable:
- leaderboards
- counters
- rate limiting
- queues

---

## 16. Memcached Internals

Memcached focuses purely on:
- simple key-value caching

Characteristics:
- multithreaded
- no persistence
- slab allocator for memory efficiency

Very fast for:
- simple object caching

---

## 17. Cache Invalidation (The Hardest Problem)

Famous quote:

> “There are only two hard things in Computer Science: cache invalidation and naming things.”

### Why It’s Hard

Data changes in origin DB.

Cache may still contain:
- stale value
- outdated object

---

## 18. Invalidation Strategies

### 18.1 TTL-Based Expiration

Simplest approach.

Problem:
- stale window exists

---

### 18.2 Write Invalidation

On DB update:
- delete cache key immediately

Most common strategy.

---

### 18.3 Event-Driven Invalidation

Changes publish events:
- Kafka
- message broker

Consumers invalidate related keys.

Used in:
- microservices
- distributed systems

---

## 19. Cache Stampede (Thundering Herd)

Occurs when:
- popular key expires
- thousands of requests miss simultaneously

Result:
- database overload
- cascading failure

---

## 20. Stampede Prevention Techniques

### 20.1 Request Coalescing

Only one request fetches data.
Others wait.

---

### 20.2 Soft Expiration

Serve stale data temporarily while refresh occurs asynchronously.

---

### 20.3 Jittered TTL

Randomize expiration times.

Prevents simultaneous expiration.

---

## 21. Cache Penetration

Requests target non-existent keys repeatedly.

Result:
- cache bypass
- DB overload

### Solutions

#### Null Caching
Store “not found” result.

#### Bloom Filters

Probabilistic structure:
- quickly checks possible existence

Very memory efficient.

---

## 22. Cache Avalanche

Large number of keys expire simultaneously.

Causes:
- sudden DB spike

Mitigation:
- staggered TTL
- layered caching

---

## 23. Multi-Level Caching

Modern systems use cache hierarchy.

Example:

1. CPU cache
2. Application memory cache
3. Redis distributed cache
4. CDN edge cache

Each layer:
- reduces latency further
- protects deeper layers

---

## 24. CDN and Edge Caching

Global systems cache content near users.

### Anycast Routing

User routed to nearest edge node.

Benefits:
- lower latency
- reduced backbone traffic

### Origin Shielding

Intermediate cache protects origin server.

Critical during:
- traffic spikes
- viral content

---

## 25. Real-World Example: Large E-Commerce System

### Layer 1: CDN

Caches:
- images
- product pages

### Layer 2: Redis

Caches:
- user sessions
- carts
- inventory counters

### Layer 3: Application Cache

Stores:
- trending products
- recommendation results

### Layer 4: Database Buffer Pool

Database caches:
- index pages
- frequently accessed rows

---

## 26. Observability and Metrics

Key metrics:

- Hit ratio
- Eviction rate
- Memory fragmentation
- Replication lag
- Cache latency

---

## 27. Common Architectural Mistakes

- Treating cache as source of truth
- No invalidation strategy
- Huge object caching
- Over-caching cold data
- Ignoring memory fragmentation

---

## 28. Advanced Insight: Cache is a Distributed Systems Problem

At scale, caching is no longer:
> “store object in memory”

It becomes:
- consistency management
- replication management
- failure coordination
- distributed synchronization

Large-scale caching systems are effectively:
> distributed databases optimized for speed.

---

## 29. Final Architectural Principle

A senior architect does not ask:
> “Should we use caching?”

They ask:
- What should be cached?
- Where should it be cached?
- How long should it remain cached?
- What consistency guarantees are acceptable?
- What happens when cache fails?

Those answers define the scalability characteristics of the entire system.
