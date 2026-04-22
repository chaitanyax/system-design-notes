# 🏛️ System Design: The Complete Guide to Modern Architecture

Welcome to the definitive guide on designing scalable, reliable, and high-performance software systems.

---

## 📖 Front Matter
* [**Preface**](manuscript/preface.md) ....................................................................................... *iv*

---

## 📑 Table of Contents

### Part 1: Core Concepts & Architectures
**1. [Foundations of System Design](manuscript/Introduction.md)** .............................................. *1*
   *Functional & Non-Functional Requirements, Scalability, Consistency*

**2. [How the Internet Works](manuscript/how-internet-works.md)** .................................................. *12*
   *DNS, TCP/IP Stack, Protocol Internals*

**3. [System Architectures](manuscript/system-architecture.md)** ..................................................... *21*
   *Monoliths, Microservices, and Migration Strategies*

**4. [Architectural Patterns](manuscript/architectural-patterns.md)** .................................................. *34*
   *Layered, Event-Driven, Microkernel, and Modular Monoliths*

**5. [Event-Driven Architecture Deep Dive](manuscript/event-driven-architecture.md)** .................... *42*
   *Producers, Consumers, Brokers, Pub/Sub, and CQRS*

**6. [Modern System Infrastructure Tools](manuscript/basic-tools-required.md)** .................................. *53*
   *Gateways, Container Orchestration, In-Memory Caches, and Event Streams*

### Part 2: Core Infrastructure & Deep Dives
**7. [Web Authentication Deep Dive](manuscript/session-vs-jwt-authentication.md)** ..................................... *65*
   *Session-based Auth, JWTs, and Distributed Token Management*

**8. [Reverse Proxy, Load Balancer, and API Gateway](manuscript/reverse-proxy-load-balancer-api-gateway.md)** ............ *80*
   *Forward vs Reverse Proxies, SSL Termination, & Gateway Patterns*

**9. [Load Balancing Algorithms & Tools](manuscript/load-balancing.md)** .................................................. *88*
   *Round Robin, Consistent Hashing, Nginx, HAProxy, Envoy*

**10. [SSL/TLS Certificates in Distributed Systems](manuscript/ssl-tls-certificates.md)** ................................. *97*
   *Handshake Process, PKI, Cipher Suites, and SSL Termination*

**11. [Data Streams and Streaming Architectures](manuscript/data-streams.md)** .......................... *107*
   *Event Time, State, Message Brokers, Lambda vs Kappa Architecture*

**12. [Consensus in Distributed Systems](manuscript/consensus-in-distributed-systems.md)** ................................. *115*
   *Safety vs Liveness, Paxos, Raft, Quorum, and Split-Brain*

**13. [CI/CD Pipeline in Modern Software Engineering](manuscript/ci-cd-pipeline.md)** ........................... *125*
   *Automation Tools, Deployment Strategies, DevSecOps, & GitOps Flow*

### Part 3: System Design Case Studies
**14. [URL Shortener System Design (Bitly)](manuscript/url-shortner.md)** .................................. *136*
   *High Read/Write Architectures, Base62 Encoding, Analytics Tracking*

**15. [Real-time Stock Broker Application](manuscript/stock-trading-app.md)** ........................................ *150*
   *Order Matching Engines, WebSockets, Circuit Breakers, Financial Scale*

---

## 🛠️ How this Book is Published

*(Developer Note: Page numbers in the Table of Contents above are simulated for the GitHub preview.)*

To convert this repository into a stunning, readable format, we can use an industry-standard static book generator. Some popular methods include:

1. **mdBook**: A highly popular command-line tool written in Rust (often used by official documentation). It rapidly converts Markdown files into a beautifully formatted, interactive static website that supports Mermaid diagrams and offline viewing out-of-the-box.
2. **GitBook**: A cloud-based platform that syncs directly with GitHub to turn markdown files into an aesthetically pleasing, searchable technical book.
3. **Pandoc + LaTeX**: For generating a classical, physical-grade PDF with dynamically generated hyperlinked tables of contents and actual page numbers.
4. **Leanpub**: For packaging your Markdown directly into ePub, PDF, and MOBI formats to sell on digital marketplaces.
