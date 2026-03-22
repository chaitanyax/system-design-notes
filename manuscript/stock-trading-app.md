# System Design: Stock Broker Application

## 1. Problem Statement

Design a **Stock Broker Platform** (like Groww, Zerodha, Upstox) that:

- Allows users to:
  - View real-time stock prices
  - Place buy/sell orders (Market + Limit)
  - View order history
- Forwards orders to a Stock Exchange (NSE/BSE)
- Does NOT implement exchange-side order matching

Assume:
- Exchange APIs already exist
- Exchange handles order matching and fulfillment

---

# 2. Functional Requirements

## 2.1 Core Requirements

1. Users can:
   - View stock prices (real-time)
   - Place Buy/Sell orders
   - Support:
     - Market Order
     - Limit Order

2. Platform forwards order to exchange.

3. Platform stores order history.

4. Users can see:
   - Current price
   - Intraday price history

---

# 3. Non-Functional Requirements

## 3.1 Consistency vs Availability

| Component | Priority |
|-----------|----------|
| Order Placement | Strong Consistency |
| Price Viewing | High Availability |

### Reasoning

- Orders involve money → must be strongly consistent.
- Price data can tolerate small staleness (few seconds).

---

# 4. Capacity Planning

## Assumptions

- Total users: 100M
- Daily Active Users (DAU): 10% → 10M
- Each user:
  - Views 10 stocks
  - Checks 20 times/day

### Read Traffic (Price Views)

```
10M users × 10 stocks × 20 checks
= 2B requests/day
```

Peak traffic assumed 5x:

```
= 10B requests/day
```

Convert to QPS:

```
2B / 100,000 ≈ 20,000 QPS
```

### Write Traffic (Orders)

Assume 10% of traffic:

```
≈ 2,000 QPS
```

---

# 5. High-Level Architecture

```
Client
  ↓
API Gateway
  ↓
----------------------------------
|            SERVICES            |
----------------------------------
| Price Service                  |
| Order Management Service (OMS) |
| Exchange Gateway Processor     |
----------------------------------
  ↓
Databases + Streaming
  ↓
Stock Exchange (External)
```

---

# 6. Components

---

## 6.1 API Gateway

Responsibilities:

- Authentication
- Authorization
- Rate limiting
- Fraud checks
- Routing

---

# 7. Real-Time Price System Design

## 7.1 Key Observations

- Read-heavy system
- Must support:
  - Live updates
  - Intraday history

---

## 7.2 Exchange Integration

Exchange sends:

- Continuous stock ticks (Server-Sent Events)

### Why not Polling?

- Too expensive
- High load on exchange

### Why not WebSockets?

- Not true bidirectional need
- Overkill
- Too many connections on client

### Chosen Approach

Use **Server-Sent Events (SSE)**

- Unidirectional
- Lightweight
- Persistent connection
- Suitable for stock tick streaming

---

## 7.3 Architecture Flow

```
Exchange
   ↓ (SSE)
Exchange Gateway Processor
   ↓
Time-Series DB
   ↓
Price Service
   ↓ (SSE)
Client
```

---

## 7.4 Time-Series Database

Use:

- TimescaleDB (on PostgreSQL)

### Schema

```
price_history(
    symbol VARCHAR,
    timestamp TIMESTAMP,
    price DECIMAL
)
```

### Design Decisions

- Index on (symbol, timestamp)
- Shard by:
  - High-volume symbols (separate shards)
- Bucketing:
  - 1-hour
  - 1-day

---

## 7.5 Price API

### Fetch price + subscribe

```
GET /api/v1/stock-price?symbol=INFY&timeRange=1day
```

- Returns historical price
- Establishes SSE connection for live updates

---

# 8. Order Management System (OMS)

Orders require:

- Strong consistency
- Audit trail
- Reconciliation
- Low latency

---

## 8.1 Architecture Flow

```
Client
   ↓
API Gateway
   ↓
Order Management Service (OMS)
   ↓
Order DB
   ↓
Kafka
   ↓
Exchange Gateway Processor
   ↓
Exchange
   ↓ (Webhook callback)
Exchange Gateway Processor
   ↓
Order DB (Update status)
```

---

## 8.2 Why Store Orders Locally?

Even though exchange stores orders:

1. Reduce exchange API calls
2. Faster order history access
3. Analytics & OLAP
4. Reconciliation
5. Audit compliance

---

## 8.3 Order Database

Use:

- PostgreSQL / RDS
- ACID compliant

### Schema

```
orders(
    order_id UUID (PK),
    user_id UUID,
    symbol VARCHAR,
    order_side ENUM(BUY, SELL),
    order_type ENUM(MARKET, LIMIT),
    limit_price DECIMAL,
    status ENUM(PENDING, PLACED, FULFILLED, CANCELLED),
    exchange_order_id VARCHAR,
    created_at TIMESTAMP
)
```

---

## 8.4 Order Placement API

```
POST /api/v1/order
```

### Request Body

```
{
  "symbol": "INFY",
  "orderSide": "BUY",
  "orderType": "LIMIT",
  "limitPrice": 1450
}
```

---

# 9. Asynchronous Order Processing

## Why Not Direct Sync Call?

- Exchange overloaded
- Need backpressure handling
- Retry mechanism

---

## 9.1 Streaming Layer

Use:

- Apache Kafka

### Benefits

- Buffering
- Retry
- Backpressure
- Dead Letter Queue (DLQ)

---

## 9.2 Exchange Order Flow

1. OMS stores order
2. Push to Kafka
3. Exchange Gateway picks from Kafka
4. Sends to Exchange
5. Receives exchange_order_id
6. Updates DB

---

## 9.3 Order Status Update

Use:

- Webhook from Exchange

Reason:

- Order fulfillment may take long
- Avoid polling
- Avoid maintaining persistent connections

---

## 9.4 Failure Handling

If no exchange_order_id:

- Retry via Kafka
- After N retries → Dead Letter Queue
- Manual intervention

---

# 10. Scaling Strategy

---

## 10.1 Price System Scaling

Problem:

- Massive tick updates
- High-frequency changes

### Optimization

Instead of direct API updates:

Use:

- Redis Pub/Sub (push-based)
- Push model better than pull model

Flow:

```
Exchange Gateway
    ↓
Redis Pub/Sub
    ↓
Price Service
    ↓
SSE to Clients
```

---

## 10.2 Reduce Latency

- Deploy infra in same region as exchange
- Ideally in same data center
- Dedicated wired connection

Reason:

- Millisecond-level latency matters

---

## 10.3 Dedicated Servers

Separate clusters for:

- Retail users
- Hedge funds / institutional traders

Reason:

- High-volume traders generate bulk load

---

## 10.4 Hybrid Scaling

Not purely autoscaling.

Use:

- Pre-provisioned servers
- Symbol-based routing
- High-volume symbols → dedicated large instances

---

## 10.5 Order DB Scaling

Shard by:

- User ID

Index:

- (user_id, created_at)

So we can fetch:

- Last 30 orders quickly

---

# 11. Rate Limiting Strategy

High surge scenarios:

- Market open
- Market close

Solutions:

1. Rate limiting at API Gateway
2. Backpressure via Kafka
3. Limit Exchange Gateway throughput
4. Prioritize high-cap stocks
5. Accept orders asynchronously

---

# 12. CAP Considerations

| System Part | CAP Choice |
|-------------|------------|
| Orders | CP |
| Price Feed | AP |

---

# 13. Final Architecture Summary

### Core Services

- API Gateway
- Price Service
- Order Management Service
- Exchange Gateway Processor

### Storage

- Time-Series DB (Stock Prices)
- RDBMS (Orders)

### Streaming

- Kafka (Orders)
- Redis Pub/Sub (Price Updates)

### Communication

- SSE (Client Price Streaming)
- Webhook (Order Status Update)

---

# 14. Key Design Principles

- Separation of Concerns
- Single Responsibility
- Async Processing
- Backpressure handling
- Sharding strategy
- Low latency infra placement
- Hybrid scaling model

---

# 15. Interview Takeaways

- Separate read-heavy vs write-heavy paths
- Always justify consistency choices
- Use async systems for financial workloads
- Never overload external systems
- Consider reconciliation & audit
- Think about latency as a competitive advantage

---

End of Notes.
