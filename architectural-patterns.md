# Architectural Patterns

Choosing the right architectural pattern is a critical decision in system design. It defines how components interact, how data flows, and how the system scales over time. In this chapter, we will explore five fundamental architectural patterns in detail.

## 1. Layered (N-Tier) Architecture

The Layered (or N-Tier) architecture organizes components into horizontal logical layers. A request typically enters through the top layer and works its way down, with each layer having a specific responsibility. 

### Core Concepts
- **Presentation Layer**: Handles UI and user interaction.
- **Business Logic Layer**: Contains the core business rules and processing.
- **Data Access Layer**: Manages communication with the database or external storage.
- **Database Layer**: The actual data storage.

### Diagram

```mermaid
flowchart TD
    Client((Client Request)) --> Presentation[Presentation Layer\nUI / Controllers]
    Presentation --> Business[Business Logic Layer\nServices / Rules]
    Business --> DataAccess[Data Access Layer\nORMs / Repositories]
    DataAccess --> DB[(Database)]
    DB --> DataAccess
    DataAccess --> Business
    Business --> Presentation
    Presentation --> Client
```

### Pros and Cons
**Advantages**:
- **Separation of Concerns**: Each layer has a distinct responsibility.
- **Ease of testing**: Layers can be tested independently (e.g., mocking the data access layer).
- **Simplicity**: Very natural way to build many standard business applications.

**Disadvantages**:
- **Monolithic deployments**: Usually deployed as a single unit.
- **Performance**: A user request must pass through multiple layers, which can add overhead.

---

## 2. Event-Driven Architecture

In this model, components are highly decoupled. Instead of calling each other directly (synchronously), they communicate by producing and consuming events through a central mediator or event bus.

### Core Concepts
- **Event Producers**: Components that generate and publish events when a state changes.
- **Event Bus / Broker**: The intermediary message router (e.g., Kafka, RabbitMQ) that accepts events and routes them to interested parties.
- **Event Consumers**: Components that listen for specific events and react accordingly.

### Diagram

```mermaid
flowchart LR
    P1[Producer: Checkout Service] -->|OrderPlaced Event| Bus{Event Bus / Broker}
    P2[Producer: User Service] -->|UserCreated Event| Bus
    
    Bus -->|OrderPlaced Event| C1[Consumer: Inventory Service]
    Bus -->|OrderPlaced Event| C2[Consumer: Notification Service]
    Bus -->|UserCreated Event| C3[Consumer: Email Marketing]
```

### Pros and Cons
**Advantages**:
- **High Decoupling**: Producers and consumers don't need to know about each other.
- **Scalability**: Can easily scale consumers independently to handle high event loads.
- **Extensibility**: New consumers can be added without modifying producers.

**Disadvantages**:
- **Complexity**: Harder to reason about system flow and track down errors.
- **Eventual Consistency**: Data might not be updated instantaneously across all services.
- **Error Handling**: Requires complex mechanisms like dead-letter queues for failed events.

---

## 3. Microkernel (Plug-in) Architecture

This pattern consists of a "Core System" that provides minimal functionality required to run the application, along with independent "Plug-in modules" that add specific, specialized features.

### Core Concepts
- **Core System**: Manages the application lifecycle, plugin registration, and basic operations.
- **Plug-in Modules**: Standalone, independent components that provide extended functionality.
- **Plugin Registry**: How the core system knows which plugins are available and how to execute them.

*Commonly seen in applications like VS Code, Eclipse, or web browsers.*

### Diagram

```mermaid
flowchart TD
    subgraph Core System
        Registry[Plugin Registry]
        Lifecycle[Lifecycle Manager]
        ExtPoints[Extension Points]
    end
    
    PluginA[Plugin A\nTheme Support] -.-> ExtPoints
    PluginB[Plugin B\nCode Analyzer] -.-> ExtPoints
    PluginC[Plugin C\nGit Integration] -.-> ExtPoints
    
    Registry --> PluginA
    Registry --> PluginB
    Registry --> PluginC
```

### Pros and Cons
**Advantages**:
- **Extensibility**: Extremely easy to add new features without changing the core system.
- **Customization**: Users can tailor the application by installing only the plugins they need.
- **Isolation**: Plugins can be developed and updated independently by different teams or third parties.

**Disadvantages**:
- **Complexity in Core Design**: The core system must be carefully designed to support flexible extension points.
- **Performance**: Heavy reliance on plugins can slow down the core system (e.g., too many browser extensions).

---

## 4. Microservices Architecture

Unlike a monolith, the microservices pattern treats the application as a suite of small, independently deployable services. Each service typically runs in its own process, achieves a specific business goal, and owns its own database to ensure loose coupling.

### Core Concepts
- **Independent Services**: Modeled around business domains (e.g., Orders, Users, Payments).
- **API Gateway**: A single entry point that routes client requests to the appropriate microservices.
- **Decentralized Data**: Each service manages its own database schema.

### Diagram

```mermaid
flowchart TD
    Client([Client App]) --> API[API Gateway]
    
    API --> S1[User Service]
    API --> S2[Order Service]
    API --> S3[Payment Service]
    
    S1 --> DB1[(User DB)]
    S2 --> DB2[(Order DB)]
    S3 --> DB3[(Payment DB)]
    
    S2 -.->|Async Event| S3
```

### Pros and Cons
**Advantages**:
- **Independent Deployments**: Teams can update small parts of the system without redeploying everything.
- **Technology Diversity**: Different services can be written in different languages (Polyglot).
- **Fault Isolation**: A crash in one service generally doesn't bring down the entire system.

**Disadvantages**:
- **Operational Complexity**: Requires robust orchestration (e.g., Kubernetes), logging, and monitoring.
- **Distributed Data**: Makes maintaining data consistency and distributed transactions very challenging.
- **Network Latency**: Inter-service communication relies on the network, leading to potential latency and failure points.

---

## 5. Monolithic vs. Modular Monolith

While microservices are popular, monoliths still have a strong place. The key distinction to understand is between a traditional (sometimes messy) monolith and a cleanly structured **Modular Monolith**.

### Core Concepts
- **Standard Monolith**: A single deployment unit where code might easily become intertwined. A change in one area can inadvertently affect another because boundaries are not enforced.
- **Modular Monolith**: Still a single deployment unit (one process, one repository), but the code is strictly separated into independent modules. Modules communicate through well-defined interfaces, much like microservices, but without the network overhead.

### Diagram

```mermaid
flowchart LR
    subgraph Standard Monolith Spaghetti
        UI1[UI] <--> BizA[Business Auth]
        BizA <--> BizO[Business Orders]
        BizO <--> DA[Data Access]
        BizA <--> DA
        UI1 <--> DA
    end
    
    subgraph Modular Monolith Clean Boundaries
        UI2[API / UI] --> ModAuth[Auth Module]
        UI2 --> ModOrd[Order Module]
        
        ModAuth -- Interface --> ModOrd
        
        ModAuth --> DBAuth[\Auth Schema/]
        ModOrd --> DBOrd[\Order Schema/]
    end
```

### Differences at a Glance

| Feature | Standard Monolith | Modular Monolith |
|---------|-------------------|------------------|
| **Boundaries** | Weak or non-existent (Spaghetti code) | Strict, enforced by rules or packaging |
| **Inter-module Calls**| Direct class/method references everywhere | Through defined public interfaces (APIs) |
| **Data Ownership** | Unrestricted access to any table | Each module manages its own tables/schema|
| **Refactoring to Microservices**| Very difficult | Relatively straightforward |

**Why Modular Monoliths?**
They offer the best of both worlds for many mid-sized projects: you get the simplicity of a single deployment and simple local method calls, but you retain the clean separation of concerns required to scale the team and eventually split out microservices if truly necessary.
