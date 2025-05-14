# Part 1: Foundations

### Definition of System Design
> System design is the process of defining the architecture, components, and data flow of a software system to meet specific functional and non-functional requirements, while balancing scalability, reliability, cost, and maintainability.

It’s not just about drawing diagrams—it's about making smart decisions in the face of real-world constraints. Good system design considers how your application behaves under load, how it recovers from failure, how easy it is to extend or debug, and how it fits into a larger ecosystem of services and teams.

---

### Functional Requirements
> These define what the system should do — the features, behaviors, and business logic of the system.

#### They answer:
- ✅ What should the system do when a user interacts with it?
- ✅ What actions should be supported?

#### Examples:
- A user should be able to sign up and log in.
- A file upload feature should support images and videos.
- An API should return the user profile when given a user ID.
- The system should send an email when an order is placed.

---

### Non-Functional Requirements (NFRs)

> These define how the system should perform, rather than what it does. They're about quality attributes and constraints.

#### They answer:
- ✅ How fast should it be?
- ✅ How secure, scalable, or reliable should it be?

#### Examples:
- The system should respond to user actions within 200ms.
- It should handle 1 million concurrent users.
- It must have 99.99% uptime.
- All user data must be encrypted in transit and at rest.
- The system should scale horizontally as load increases.

#### 🎯 Simple Analogy:
Think of building a car:

> Functional requirements: The car must be able to start, accelerate, brake, and steer.

> Non-functional requirements: The car should go from 0 to 60 mph in under 5 seconds, consume less than 6L/100km, and have a noise level below 70 dB.

### 📱 Instagram – Functional vs Non-Functional Requirements
#### 🔹 Functional Requirements (What it should do)
- ✅ Users can create an account and log in.
- ✅ Users can upload and delete photos and videos.
- ✅ Users can like, comment, and share posts.
- ✅ Users can follow and unfollow other users.
- ✅ Stories disappear after 24 hours.
- ✅ Users can send direct messages to other users.
- ✅ Users get notifications when someone likes or comments on their post.

> 📌 These are features that define how Instagram behaves and what it allows users to do.

---

### 🔹 Non-Functional Requirements (How it should perform)
- ⚡ Photos and videos should load within 1 second on a 4G connection.
- 📈 The system should handle 1 million concurrent users without degradation.
- 🕒 Posts and stories must be available 99.99% of the time (high availability).
- 🔐 All user data must be encrypted and follow GDPR compliance.
- 🌍 Content Delivery Network (CDN) should be used for fast media delivery worldwide.
- 🧱 The system should scale horizontally to handle spikes (e.g., during celebrity livestreams).
- 🛠️ The system should log all errors and have real-time monitoring for failures.
- 📌 These ensure performance, security, scalability, and user experience—even if users never see them directly.

#### Pro Tip for Readers:
> Functional requirements define what users expect.
Non-functional requirements ensure those expectations are met—even at scale.

---

### 🔹 1. Scalability
> Definition: Scalability is the ability of a system to handle increased load (more users, more data, more requests) by adding more resources.

#### 🧠 Why It Matters:
- Startups begin with 100 users. Successful ones scale to millions.
- Poorly designed systems crash or slow down under pressure.
- Scalability ensures your app grows without breaking.

#### 📱 Instagram Example:
When a celebrity posts a photo, millions of followers refresh their feed. The system must scale to handle that sudden spike in traffic.

#### 🛠️ How It’s Achieved:
- Horizontal scaling: Adding more servers (e.g., load balancing across web servers)
- Database sharding: Splitting data by user or geography
- Caching: Serving repeat data quickly (e.g., Redis or CDN for media)
- Stateless services: So new instances can spin up instantly

### ⚖️ Trade-offs:
- More servers = more complexity (e.g., session management, sync issues)
- Cache inconsistency risks
- Costs grow with scale

---

### 🔸 1. Vertical Scaling (Scaling Up)
> Definition: Adding more power (CPU, RAM, disk) to a single machine.

#### 🧠 Think of it like:
Upgrading your laptop’s RAM from 8GB to 32GB so it can handle more tabs in Chrome.

#### 📱 Instagram Example:
Instead of multiple servers, Instagram runs everything on one giant server and keeps upgrading it whenever load increases.

This works... until it doesn’t.

#### ✅ Pros:
- Simple to implement
- No need to rewrite your architecture
- Easier to debug (one machine, one log)

#### ❌ Cons:
- There’s a limit to how powerful one machine can get
- Downtime during upgrades
- Becomes a single point of failure
- Can get very expensive

---

### 🔸 2. Horizontal Scaling (Scaling Out)
> Definition: Adding more machines to handle load by distributing work.

#### 🧠 Think of it like:
Instead of buying one supercomputer, you buy 10 average laptops and split the work among them.

#### 📱 Instagram Example:
- When you upload a photo, it’s handled by one server.
- When I view a story, it's handled by another.
- Multiple web servers, multiple databases, CDNs — all scaled horizontally to support billions of requests.

#### ✅ Pros:
- No theoretical limit to scale (just add more machines)
- Fault-tolerant: if one machine dies, others take over
- Better for distributed global systems

#### ❌ Cons:
- Requires load balancing
- State management is tricky (sessions, cache consistency)
- Debugging is harder across many nodes
- Needs stateless services or smart data partitioning

#### ⚖️ When to Use What?

| Use Case                           | Vertical Scaling              | Horizontal Scaling             |
|------------------------------------|-------------------------------|--------------------------------|
| Early-stage MVP                    | ✅ Easier and faster          | ❌ Overkill for small systems  |
| Production app with growth         | ❌ Hits limits quickly        | ✅ Best for long-term scale    |
| Real-time global app (e.g., Instagram) | ❌ Risky (one machine can't serve all) | ✅ Industry standard          |
| Simpler codebase & infra           | ✅ Less complexity            | ❌ Needs orchestration (e.g., Kubernetes) |

#### 🔧 TL;DR for Developers:
- Scale up when you're small and need to move fast.
- Scale out when you're growing and need to survive.

---
### 🔹 2. Consistency
> Definition: Consistency means all users see the same data at the same time, no matter which server they hit.

#### 🧠 Why It Matters:
- If I like a photo, I should immediately see the like count updated—same for others.
- Inconsistent data breaks user trust and can be dangerous (especially in finance, e-commerce).

#### 📱 Instagram Example:
You comment on a post, but the comment doesn't show up for others for 5 minutes. That’s a consistency failure.

#### 🛠️ How It’s Achieved:
- Use strong consistency with databases like PostgreSQL (ACID)
- For distributed systems, ensure synchronous replication or consensus protocols like Raft/Paxos
- Use eventual consistency when availability is more critical than immediate accuracy

#### ⚖️ Trade-offs:
CAP Theorem: You can’t have perfect Consistency, Availability, and Partition Tolerance at the same time

##### Strong consistency adds latency

##### Eventual consistency can cause weird user experiences (e.g., delayed likes or comments)

---

### 🔹 3. Availability
> Definition: Availability is the ability of a system to be up and responsive—even during failures or high load.

#### 🧠 Why It Matters:
- Downtime = loss of trust, revenue, users.
- For global apps, availability expectations are 24/7.

#### 📱 Instagram Example:
If the service goes down during a global event (e.g., Met Gala), millions of users are affected. You lose both engagement and reputation.

#### 🛠️ How It’s Achieved:
- Redundancy: Multiple servers in different regions (multi-AZ, multi-region)
- Failover systems: If one database/server dies, switch to a backup
- Load balancing: Distribute traffic smartly
- Health checks & auto-restarts with container orchestration (e.g., Kubernetes)

#### ⚖️ Trade-offs:
- More redundancy = higher cost
- Higher availability often reduces consistency (again, CAP Theorem)
- Requires strong monitoring and alerting

---

### 🔹 4. Reliability
> Definition: Reliability means the system consistently works as expected over time—it doesn’t randomly fail or produce wrong results.

#### 🧠 Why It Matters:
- Users expect your system to be predictable.
- Reliability builds trust.

#### 📱 Instagram Example:
- If you upload a video and it occasionally fails or goes missing—that’s a reliability issue.
- If DMs are randomly delayed or lost, people stop using them.

#### 🛠️ How It’s Achieved:
- Retry logic for transient failures (e.g., uploading)
- Graceful degradation (e.g., show cached data when live DB is down)
- Circuit breakers and rate limiters to avoid cascading failures
- Monitoring, logging, alerting to detect issues early

#### ⚖️ Trade-offs:
- Reliable systems are often more complex to build and test.
- Trade off speed for error handling and retries.
- May increase latency to ensure completeness (e.g., retries, verification)

---
### ✨ Summary Table:
| NFR          | Definition                                 | Instagram Example                         | Key Trade-Offs                                     |
|--------------|--------------------------------------------|--------------------------------------------|----------------------------------------------------|
| Scalability  | Handles more load with more resources      | Viral post spikes                          | Cost, complexity                                   |
| Consistency  | All users see the same data                | Comment appears instantly to all           | May reduce availability, increase latency          |
| Availability | System is always up and responsive         | No downtime during big events              | Redundancy cost, affects consistency               |
| Reliability  | System behaves predictably and accurately  | Uploads never
---

### 🔹 Low Latency
> Definition: Latency is the time it takes for a system to respond to a request.

> Low latency means the system responds quickly, ideally within milliseconds.

#### 🧠 Why It Matters:
- Users expect instant results. Even a 1-second delay feels slow.
- Low latency improves user experience, retention, and engagement.

#### 📱 Instagram Example:
- When you tap on a story, it loads instantly — that’s low latency.
- If it takes 3–4 seconds, users drop off or get frustrated.

#### 🛠️ How to Achieve Low Latency:
- Use Content Delivery Networks (CDNs) to serve images/videos from locations close to the user.
- Optimize backend queries and use indexes properly.
- Add caching layers (e.g., Redis, Memcached) for frequently accessed data.
- Design systems to be stateless, allowing faster routing and processing.

#### ⚖️ Trade-offs:
- Caching can lead to stale data
- Optimization may increase infrastructure cost

##### 📌 Goal: Keep user-facing latency under 100–200ms for most interactions.

---

### 🔹 Daily Active Users (DAU)
> Definition: The number of unique users who use your product in a single day.

#### 🧠 Why It Matters:
- DAU measures daily engagement.
- High DAU means users find value and come back often.
- It’s a key health metric for consumer apps.

#### 📱 Instagram Example:
- If 500 million people opened Instagram today and interacted with it, that’s your DAU.

#### 🛠️ How It's Used in Design:
- Helps decide how much backend load to expect on a normal day.
- Drives rate limiting, resource allocation, and system tuning.

##### 📌 Companies often compare DAU to MAU to calculate stickiness:
`Stickiness = (DAU / MAU) × 100%`

---

### 🔹 Monthly Active Users (MAU)
> Definition: The number of unique users who use your product in a 30-day window.

#### 🧠 Why It Matters:
- MAU shows reach: how many people are aware of and occasionally using your product.
- Good for tracking growth over time.

#### 📱 Instagram Example:
- If 1.5 billion users opened the app at least once in a month, that’s the MAU.

#### 🛠️ How It's Used in Design:
- Helps with capacity planning: storage, user history, notifications, etc.
- MAU drives business metrics (like ad revenue, user base growth).

---
# 🌐 How the Internet Works - Deep Dive for System Design Interviews

This document is a complete guide to understanding how the internet works, including protocols and internals. Ideal for system design interviews.

---

## 📌 Table of Contents

1. [What Happens When You Type a URL](#1-what-happens-when-you-type-a-url)
2. [TCP/IP Stack and OSI Model](#2-tcpip-stack-and-osi-model)
3. [Protocol Internals](#3-protocol-internals)
    - [IP](#ip-internet-protocol)
    - [TCP](#tcp-transmission-control-protocol)
    - [UDP](#udp-user-datagram-protocol)
    - [TLS](#tls-transport-layer-security)
    - [HTTP Versions](#http-versions)
4. [DNS - Domain Name System](#4-dns---domain-name-system)
5. [Resources & References](#5-resources--references)

---

## 1. What Happens When You Type a URL

Typing `https://example.com` in the browser triggers these steps:

### 🔹 DNS Resolution
- Resolves domain to IP address.
- Queries may include:
  - Browser cache
  - OS cache
  - Recursive resolver (ISP or Google DNS)
  - Root → TLD → Authoritative name servers

**Protocol:** `DNS` over `UDP` (port 53), fallback to `TCP`.

### 🔹 TCP Connection
- **3-Way Handshake:**
  1. SYN → 
  2. SYN-ACK ←
  3. ACK →

**Protocol:** `TCP` (Transmission Control Protocol)

### 🔹 TLS Handshake (if HTTPS)
- Ensures secure encrypted connection.
- Uses asymmetric crypto to exchange symmetric session key.

**Protocol:** `TLS` (runs over TCP)

### 🔹 HTTP Request/Response
- Browser sends HTTP request to server:
- Server returns HTML, JS, images, etc.

**Protocol:** `HTTP` or `HTTPS`

---

## 2. TCP/IP Stack and OSI Model

### 📦 TCP/IP Stack (Real-World Internet Model)

| Layer           | Protocols                            | Description |
|------------------|----------------------------------------|-------------|
| Application      | HTTP, DNS, FTP, SMTP, TLS              | End-user communication |
| Transport        | TCP, UDP                               | Reliable/unreliable delivery |
| Network          | IP (IPv4/IPv6), ICMP                   | Routing, addressing |
| Data Link        | Ethernet, ARP, Wi-Fi                   | Local delivery |
| Physical         | Fiber optics, copper, radio waves      | Transmission medium |

---

## 3. Protocol Internals

### IP (Internet Protocol)
- Stateless, unreliable
- Handles packet addressing and routing
- Includes TTL, source/destination IP

### TCP (Transmission Control Protocol)
- Reliable, ordered, connection-oriented
- Features:
- 3-Way Handshake
- Sequence numbers
- Acknowledgments
- Retransmission on loss
- Congestion control
- Flow control (sliding window)

### UDP (User Datagram Protocol)
- Connectionless, faster, unreliable
- Used in DNS, VoIP, streaming

### TLS (Transport Layer Security)
- Secures communication over insecure networks
- Uses:
- Asymmetric encryption (RSA, DH)
- Certificate exchange (X.509)
- Session key negotiation
- Used in HTTPS

### HTTP Versions
| Version   | Key Features                              |
|-----------|--------------------------------------------|
| HTTP/1.1  | Text-based, one request/connection (keep-alive) |
| HTTP/2    | Binary protocol, multiplexed streams        |
| HTTP/3    | Uses QUIC (over UDP), faster, secure by default |

---

## 4. DNS - Domain Name System

- Maps domain names to IP addresses
- Uses **recursive and authoritative** name servers
- Key components:
- Root servers
- TLD servers (.com, .net)
- Authoritative servers

**Protocol:** DNS over UDP (default), TCP fallback for large responses

---

## 5. Resources & References

### 📚 Books
- [Computer Networking: A Top-Down Approach]()
