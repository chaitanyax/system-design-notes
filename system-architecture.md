  # Part 2: System Architectures
Choosing the right architecture is like choosing the right foundation for your house — it decides how fast you can build, how easy it is to scale, and how well it handles change over time.

Here are the most common architectures you'll encounter, with real-world relevance and trade-offs.

### Client-Server Architecture
Client-server architecture is a fundamental computing model that structures systems as service requesters (clients) and service providers (servers). This model has become the backbone of modern distributed computing, powering everything from web applications to enterprise systems.

#### Core Components
#### 1. Client
> Definition: A client is a device or software application that requests services or resources from a server

#### Characteristics:

- Initiates requests to servers
- Typically has a user interface
- May perform some local processing
- Examples: Web browsers, mobile apps, desktop applications

#### 2. Server
> Definition: A server is a powerful computer or software system that provides services, resources, or data to clients

#### Characteristics:

- Listens for client requests
- Processes requests and returns responses
- Typically more powerful than clients
- Examples: Web servers, database servers, file servers

#### Communication Protocols
- Client-server systems rely on standardized protocols for communication:
- HTTP/HTTPS: For web-based applications
- TCP/IP: Foundation for most internet communications
- WebSockets: For real-time, bidirectional communication
- gRPC: For high-performance RPC (Remote Procedure Calls)
- GraphQL: For flexible data querying

#### Architectural Variations
#### 1. Two-Tier Architecture
- Simple model with direct client-server interaction
- Client handles presentation logic
- Server handles business logic and data storage
- Common in early web applications

#### 2. Three-Tier Architecture
- Presentation tier: User interface (client)
- Application tier: Business logic (server)
- Data tier: Database/storage (separate server)
- Provides better scalability and separation of concerns

#### 3. N-Tier Architecture
- Further divides application into additional specialized layers
- May include separate tiers for caching, API gateways, etc.
- Enables microservices patterns