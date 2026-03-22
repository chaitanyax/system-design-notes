# Foundations of System Design

### Definition of System Design
System design is the process of defining the architecture, components, and data flow of a software system to meet specific functional and non-functional requirements, while balancing competing factors such as scalability, reliability, cost, and maintainability.

The discipline extends beyond merely drafting diagrams. It involves making strategic technical decisions under real-world constraints. Effective system design requires a deep understanding of how an application behaves under substantial load, how gracefully it recovers from localized failures, its extensibility, and how it interfaces within a broader ecosystem of microservices and organizational structures.

---

## 1. Requirement Gathering

### Functional Requirements
Functional requirements define the explicit behaviors, features, and business logic that a system must provide. They address the operational capabilities of the software.

**Key Questions Answered:**
- What operations should the system execute in response to user input?
- What specific business-driven actions must be supported?

**Examples:**
- End-users must be able to register and authenticate using standard protocols.
- The platform must support the uploading and processing of multimedia files (images and videos).
- The user service API must return the complete profile metadata given a valid user identifier.
- The order management module must trigger an asynchronous email notification upon successful transaction processing.

### Non-Functional Requirements (NFRs)
Non-functional requirements describe the quality attributes, systemic constraints, and performance expectations of the system rather than its specific features. They dictate how effectively the system performs its designated functions.

**Key Questions Answered:**
- What are the acceptable latency bounds for the system?
- What levels of security, scalability, and reliability are mandated?

**Examples:**
- The system must return responses to client requests within an average of 200 milliseconds.
- The architecture must be capable of supporting one million concurrent client sessions.
- Core services must maintain an uptime of 99.99% (four nines of availability).
- Personally Identifiable Information (PII) must be encrypted both in transit and at rest in compliance with regulatory standards.

**A Practical Analogy:**
In automotive engineering, a functional requirement dictates that a vehicle must be able to start, accelerate, brake, and steer. A non-functional requirement specifies that the vehicle must accelerate from 0 to 60 mph in under five seconds and maintain an interior noise level below 70 decibels.

### Case Study Framework: A Large-Scale Photo Sharing Application

To concretely illustrate these concepts, consider the architecture of a global photo-sharing application analogous to major market leaders.

**Functional Requirements:**
- Account creation and authentication mechanisms.
- Media upload, deletion, and chronological feed generation.
- Social interactions including likes, comments, following, and sharing.
- Ephemeral content generation (e.g., stories that expire after 24 hours).
- Direct peer-to-peer messaging and event-driven notifications.

**Non-Functional Requirements:**
- Media must render for edge clients within one second under typical cellular network conditions.
- The system must seamlessly handle one million concurrent connections without noticeable latency degradation.
- Read operations (feeds, profiles) must exhibit 99.99% availability.
- The infrastructure must scale horizontally to absorb immense, localized spikes in traffic.

---

## 2. Core Architectural Qualities

### Scalability
Scalability is a system's capacity to handle an increasing volume of work—such as higher request rates, concurrent users, or data volume—by integrating additional computational resources.

Systems rarely maintain consistent traffic patterns; they must grow from initial deployment parameters to handling potentially exponential increases in load. Proper architectural foresight ensures that growth does not necessitate a complete system rewrite.

**Two Models of Scaling:**

#### Vertical Scaling (Scaling Up)
Vertical scaling involves upgrading the hardware capacity (CPU, RAM, or generic storage) of a single node or server.

- **Advantages:** Minimal architectural complexity. The application codebase rarely requires modification, and debugging remains contained to a single environment.
- **Disadvantages:** Physical hardware has finite ceilings for upgrades. Furthermore, reliance on a single, powerful machine introduces a single point of failure and often incurs prohibitive financial scaling costs at the upper limits.

#### Horizontal Scaling (Scaling Out)
Horizontal scaling expands system capacity by distributing the computational load across an increasing number of discrete machines or nodes.

- **Advantages:** Theoretically limitless scale. It inherently supports fault tolerance; if one node fails, load balancers can intuitively redirect traffic to healthy nodes.
- **Disadvantages:** Introduces significant complexity regarding state management, data consistency across nodes, and distributed logging.

*Architectural Heuristic: Vertical scaling is effective for rapid prototyping and monolithic systems with moderate load ceilings, whereas horizontal scaling is a prerequisite for highly available, globally distributed applications.*

### Consistency
Consistency ensures that all clients accessing the system simultaneously observe the exact same data state, regardless of which physical node serves their request.

**Implications:**
In a distributed environment, ensuring strong consistency guarantees that operations occur in a strict sequence globally. However, enforcing strong consistency often demands synchronous data replication across nodes.

According to the CAP Theorem, systems cannot simultaneously guarantee perfect Consistency, Availability, and Partition Tolerance. Therefore, system designers must often choose between:
- **Strong Consistency:** Guarantees absolute data accuracy but frequently incurs higher latency. (e.g., Financial transactions).
- **Eventual Consistency:** Acceptable in high-availability scenarios where temporary data divergence is tolerable. The system state will eventually converge. (e.g., Social media like counts).

### Availability
Availability refers to the operational uptime of a system and its ability to respond to requests continuously, even in the event of partial hardware or network failures.

**Implementation Strategies:**
- Implementing redundancy across multiple Availability Zones (AZs) or geographical regions.
- Utilizing robust load balancing to detect failing instances and intelligently route traffic.
- Employing automated health checks and auto-healing infrastructure mechanisms (such as container orchestration).

**Trade-offs:** 
Increasing availability inevitably necessitates redundant infrastructure, fundamentally increasing operational costs. Additionally, prioritizing absolute availability in geographically distributed systems typically requires sacrificing strong data consistency.

### Reliability
Reliability describes the probability that a system will perform its intended function predictably and accurately over a specified period. A reliable system does not silently drop data or return erroneous responses under stress.

**Implementation Strategies:**
- Incorporating exponential backoff and retry logic for transient failures.
- Designing graceful degradation states (e.g., displaying cached approximations if a live database becomes unresponsive).
- Utilizing circuit breakers to prevent localized failures from causing cascading, system-wide outages.

### Low Latency
Latency measures the temporal delay between a client making a request and the system completing its response. Low latency is crucial for modern user experiences, where delays measured even in milliseconds can negatively impact user retention.

**Implementation Strategies:**
- Deploying Content Delivery Networks (CDNs) to cache and serve static assets geographically closer to end-users.
- Utilizing in-memory caching solutions (e.g., Redis or Memcached) to bypass heavy database computations for frequently accessed data.
- Maintaining stateless application tiers to expedite request routing and execution.

---

## 3. Key Engagement Metrics

When evaluating the required scale for system design, architects frequently rely on product engagement metrics to mathematically estimate throughput and storage requirements.

### Daily Active Users (DAU)
The count of unique individuals who interact with the system within a given 24-hour period. DAU dictates the baseline concurrent load a system must endure during peak daily operations and is critical for sizing the stateless application tier.

### Monthly Active Users (MAU)
The count of unique individuals who interact with the product within a 30-day window. MAU serves primarily as a metric for tracking total addressable traffic and informs capacity planning regarding long-term data storage allocations and asynchronous processing systems.

*Metric Usage:* The ratio of DAU to MAU corresponds to "user stickiness," providing insights into whether the infrastructure must primarily optimize for bursty, sporadic traffic or consistent, heavy daily usage.
