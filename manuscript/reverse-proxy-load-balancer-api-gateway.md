# Reverse Proxy, Load Balancer, and API Gateway  
*A Systems Architecture Deep Dive*

---

## Table of Contents

1. Introduction  
2. Forward Proxy vs Reverse Proxy  
3. Reverse Proxy  
4. Load Balancer  
5. API Gateway  
6. Capability Spectrum  
7. Real-World Architecture  
8. Decision Framework  
9. Summary  

---

# 1. Introduction

Modern distributed systems rely heavily on intermediary infrastructure components that sit between clients and backend services.

Three commonly confused components are:

- **Reverse Proxy**
- **Load Balancer**
- **API Gateway**

They all:

- Sit between clients and servers  
- Forward requests  
- Return responses  

However, they solve different architectural problems.

Understanding the distinction is essential for designing scalable, secure, and maintainable systems.

---

# 2. Forward Proxy vs Reverse Proxy

Before understanding reverse proxies, we must clarify forward proxies.

## 2.1 Forward Proxy

A **forward proxy** sits in front of clients.

```
Client → Forward Proxy → Internet → Server
```

### Purpose

- Mask client identity (IP hiding)  
- Filter access (corporate restrictions)  
- Add anonymity  

Examples:
- Corporate web filtering systems  
- Privacy proxy services  

---

## 2.2 Reverse Proxy

A **reverse proxy** sits in front of servers.

```
Client → Reverse Proxy → Backend Server(s)
```

When accessing large platforms like Netflix, Stripe, or GitHub, users are not communicating directly with their application servers. Instead, they are interacting with a reverse proxy.

---

# 3. Reverse Proxy

A reverse proxy is a **general-purpose traffic forwarder** positioned in front of backend servers.

## 3.1 Core Responsibilities

### 1. SSL Termination

HTTPS encryption is CPU-intensive.

Instead of every backend service handling encryption:

```
Client → (HTTPS)
         Reverse Proxy → (HTTP)
                       Backend Services
```

The proxy:

- Handles TLS handshake  
- Decrypts traffic  
- Forwards plain HTTP internally  

---

### 2. Caching

If thousands of users request the same resource:

```
Client → Reverse Proxy (Cache Hit) → Response
```

The backend is never contacted.

Benefits:

- Reduced latency  
- Lower server load  
- Reduced infrastructure cost  

---

### 3. Security Isolation

Clients see only the proxy’s public IP address.  
Backend servers remain hidden from direct exposure.

---

### 4. Compression

The proxy can compress responses (e.g., GZIP) before sending them to clients.

---

## 3.2 Common Reverse Proxy Tools

- Nginx  
- HAProxy  
- Caddy  
- Apache HTTP Server  

A reverse proxy does not inherently care whether it is forwarding a website, API, or WebSocket traffic. It simply forwards requests based on configured rules.

---

# 4. Load Balancer

As traffic increases, a single server becomes insufficient.

```
Client → Load Balancer → Server A
                         Server B
                         Server C
```

A **load balancer is a specialized reverse proxy** whose primary purpose is distributing traffic across multiple backend servers.

## 4.1 Goals

- Scalability  
- High availability  

---

## 4.2 Load Balancing Algorithms

### Round Robin

```
Request 1 → A  
Request 2 → B  
Request 3 → C  
Request 4 → A  
```

Works well when servers are identical.

---

### Least Connections

Sends requests to the server with the fewest active connections.

Useful when request durations vary.

---

### IP Hash

Same client IP always routes to the same backend.

Used for session stickiness.

---

### Weighted Distribution

Some servers handle more traffic than others.

Useful when backend servers have different capacities.

---

## 4.3 Layer 4 vs Layer 7 Load Balancing

### Layer 4 (Transport Layer)

- Operates at TCP/UDP level  
- Sees IP addresses and ports  
- Does not inspect HTTP content  
- Faster but less intelligent  

---

### Layer 7 (Application Layer)

- Understands HTTP  
- Can inspect URLs, headers, cookies  
- Enables content-based routing  

Example:

```
/api/users   → Service Pool A  
/api/orders  → Service Pool B  
```

---

# 5. API Gateway

An API Gateway goes beyond forwarding and distributing traffic.

It is an **API-aware edge management layer**.

## 5.1 Why It Exists

In microservices architecture, multiple services may require:

- Authentication  
- Rate limiting  
- Logging  
- Monitoring  

Implementing this logic in every service leads to duplication and inconsistency.

An API Gateway centralizes these cross-cutting concerns.

---

## 5.2 Core Responsibilities

### 1. Authentication and Authorization

Validate tokens at the edge.  
Reject invalid requests before they reach backend services.

---

### 2. Rate Limiting

Example:

- Free tier → 100 requests per minute  
- Paid tier → 1000 requests per minute  

Prevents backend overload.

---

### 3. Request and Response Transformation

Example:

- Client sends JSON  
- Legacy backend expects XML  
- Gateway transforms format  

Or removes sensitive fields before returning responses.

---

### 4. API Versioning

```
/v1/users → Old Service  
/v2/users → New Service  
```

Enables gradual migrations without breaking clients.

---

### 5. Analytics and Monitoring

Since the gateway sees all traffic, it can measure:

- Endpoint usage  
- Latency percentiles  
- Error rates  

This data supports debugging and capacity planning.

---

## 5.3 Common API Gateway Tools

- Kong  
- AWS API Gateway  
- Apigee  
- Azure API Management  
- Tyk  

---

# 6. Capability Spectrum

These components exist on a spectrum of capabilities.

```
Reverse Proxy
    ↓
Load Balancer
    ↓
API Gateway
```

| Capability | Reverse Proxy | Load Balancer | API Gateway |
|------------|--------------|---------------|-------------|
| SSL Termination | Yes | Yes | Yes |
| Caching | Yes | Sometimes | Sometimes |
| Traffic Distribution | Limited | Yes | Yes |
| Health Checks | Basic | Yes | Yes |
| Authentication | No | No | Yes |
| Rate Limiting | Limited | Limited | Yes |
| API Analytics | No | No | Yes |

---

# 7. Real-World Architecture

Modern systems often combine these layers.

```
User
  ↓
CDN
  ↓
API Gateway
  ↓
Load Balancer
  ↓
Service Instances
```

## Layer Responsibilities

**CDN**
- Global caching  
- SSL termination close to users  
- Traffic spike absorption  

**API Gateway**
- Authentication  
- Rate limiting  
- API routing  

**Load Balancer**
- Traffic distribution  
- Health checks  
- Failover  

**Service-Level Reverse Proxy**
- Internal SSL  
- Static file serving  

Each layer has a distinct responsibility.

---

# 8. Decision Framework

Use this quick guide:

Multiple instances of a service?  
→ Use a Load Balancer  

Exposing APIs to external developers?  
→ Use an API Gateway  

Need authentication or rate limiting before backend?  
→ API Gateway  

Only need SSL termination and caching?  
→ Reverse Proxy  

Global distribution required?  
→ Add a CDN  

---

# 9. Summary

| Component | Core Purpose |
|------------|-------------|
| Reverse Proxy | General-purpose request forwarding |
| Load Balancer | Traffic distribution for scale and availability |
| API Gateway | API management (authentication, rate limiting, analytics) |

They are:

- Not competitors  
- Not interchangeable  
- Often layered together  

Design architecture based on actual system requirements, not trends.

Clear architectural understanding separates average implementations from production-grade systems.
