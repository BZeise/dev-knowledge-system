# Microservices Architecture

A comprehensive guide to designing, building, and operating microservices systems. This covers core concepts, design patterns, implementation strategies, and real-world trade-offs.

---

## Table of Contents

1. [Fundamentals](#fundamentals)
2. [Why Microservices?](#why-microservices)
3. [Core Concepts](#core-concepts)
4. [Design Patterns](#design-patterns)
5. [Implementation Strategies](#implementation-strategies)
6. [Data Management](#data-management)
7. [Communication Patterns](#communication-patterns)
8. [Operational Challenges](#operational-challenges)
9. [When NOT to Use Microservices](#when-not-to-use-microservices)
10. [Real-World Examples](#real-world-examples)
11. [Resources & Further Reading](#resources--further-reading)

---

## Fundamentals

### What Are Microservices?

**Definition:** Microservices is an architectural style that structures an application as a collection of loosely coupled, independently deployable services, each owning a single business capability.

**Key Characteristics:**
- **Small & Focused:** Each service implements one business capability (e.g., "User Management," "Order Processing," "Payment")
- **Independently Deployable:** Update one service without redeploying others
- **Loosely Coupled:** Services communicate through well-defined interfaces (APIs, events), not shared data stores
- **Polyglot:** Each service can use different technology stacks, languages, and databases
- **Owned by Teams:** A small team (typically 5–10 people) owns a service end-to-end

**Contrast with Monolithic Architecture:**

| Aspect | Monolith | Microservices |
|--------|----------|---------------|
| **Structure** | Single codebase, shared database | Multiple codebases, separate databases |
| **Deployment** | Deploy entire app | Deploy services independently |
| **Scaling** | Scale entire application | Scale individual services |
| **Technology** | One technology stack | Mix of technologies per service |
| **Failure** | One failure affects entire app | Isolated failures (with proper design) |
| **Team Coordination** | High (shared code) | Low (autonomous teams) |
| **Complexity** | Lower (local) | Higher (distributed) |

### Historical Context

Microservices emerged as a response to limitations of monolithic architectures at scale:

- **Pre-2010:** Monolithic architecture dominates (simpler to build, deploy, and operate)
- **2010s:** Netflix, Amazon, and others pioneer microservices to enable independent scaling and rapid iteration
- **2014:** Martin Fowler publishes the [canonical microservices definition](https://martinfowler.com/articles/microservices.html)
- **2015+:** Containerization (Docker) and orchestration (Kubernetes) make microservices operationally feasible at scale
- **2020s:** Microservices become mainstream; ecosystem matures with observability, service mesh, serverless alternatives

---

## Why Microservices?

### Problems They Solve

1. **Team Scaling**
   - **Problem:** Large teams working on a monolith experience slowdowns—communication overhead, merge conflicts, deployment bottlenecks
   - **Solution:** Small, autonomous teams own services, ship independently
   - **Example:** 50-person engineering team becomes 5 teams of 10, each owning 2–3 services

2. **Technology Flexibility**
   - **Problem:** Monolith locked into one language/framework; hard to adopt new technologies
   - **Solution:** Each service chooses the best tool for its problem
   - **Example:** Payment service uses Rust for performance; search service uses Python with ML libraries

3. **Independent Scaling**
   - **Problem:** One slow component (e.g., search) forces scaling the entire app, wasting resources
   - **Solution:** Scale only services under load
   - **Example:** During peak search traffic, scale search service 10x while keeping user service at 2x

4. **Fault Isolation**
   - **Problem:** Bug in one feature cascades, bringing down entire system
   - **Solution:** Services fail in isolation; circuit breakers prevent cascades
   - **Example:** Payment service fails; checkout still works with a graceful "payment temporarily unavailable" message

5. **Rapid Iteration**
   - **Problem:** Feature releases require coordinating across teams; integration and regression testing slow everything down
   - **Solution:** Teams deploy independently; only their own service is tested
   - **Example:** Notification service ships a feature in 2 days without waiting for other teams

### Trade-Offs: What You Pay For

Microservices are not free. Every benefit comes with a cost:

| Benefit | Cost |
|---------|------|
| Team autonomy | Increased operational complexity; must manage many services |
| Independent scaling | Distributed systems challenges (eventual consistency, network latency) |
| Technology flexibility | No standardization; harder to share knowledge across teams |
| Fault isolation | Debugging across services is much harder |
| Rapid iteration | Data consistency becomes complex; transactions span services |

---

## Core Concepts

### 1. Service Boundaries

**Problem:** How do you decide where to split services?

**Approach: Domain-Driven Design (DDD)**

Define services around business domains, not technical layers.

**❌ Wrong:**
```
Service 1: Data Access Layer
Service 2: Business Logic Layer
Service 3: API Layer
```
(Technical layers—tight coupling when business logic changes)

**✅ Right:**
```
Service 1: User Service (manages user profiles, authentication)
Service 2: Order Service (manages orders, state machine)
Service 3: Payment Service (handles payments, integrates with providers)
Service 4: Notification Service (sends emails, SMS, push notifications)
```
(Business capabilities—each can be owned by a team, scales independently)

**Key Concept: Bounded Context**
- A bounded context is a natural division within your business
- Each service should own one bounded context
- Example: E-commerce system has contexts: Catalog (products), Orders, Payments, Shipping
- Each context owns its data and APIs

**Guidance from Martin Fowler:**
> "The microservice community favours an alternative approach: smart endpoints and dumb pipes. The microservices application uses these mechanisms to move simple data around. The application forms this data into more sensible forms. The microservices do real work here."

### 2. Data Ownership

**Principle: Database per Service**

Each service owns its data; no shared databases.

**Why?**
- Enables schema independence (service A evolves schema without affecting service B)
- Prevents tight coupling (service B can't directly query service A's tables)
- Allows polyglot persistence (service A uses SQL; service B uses NoSQL)

**What It Looks Like:**

```
┌─────────────────┐        ┌──────────────────┐
│  User Service   │        │  Order Service   │
├─────────────────┤        ├──────────────────┤
│ Users DB        │        │ Orders DB        │
│ (SQL)           │        │ (SQL)            │
│ - users         │        │ - orders         │
│ - profiles      │        │ - line_items     │
└─────────────────┘        │ - order_status   │
                           └──────────────────┘

User Service owns user data.
Order Service owns order data.
Neither can directly query the other's database.
```

**Challenge: Distributed Transactions**

When Order Service needs user email (in User Service), and Order Service needs payment info (in Payment Service), how do we ensure consistency?

**Answer:** You don't, in the ACID sense. Instead, embrace eventual consistency.

### 3. Eventual Consistency

**Concept:** Data is temporarily inconsistent across services; consistency is reached eventually.

**Example Scenario:**
1. User signs up (User Service creates user)
2. Welcome email sent (Notification Service subscribes to "UserCreated" event)
3. Email might arrive 100ms later; temporarily inconsistent state is acceptable

**When It's Acceptable:**
- Email arrives 1 second late ✅ (user can wait)
- Student grade not immediately visible in transcript ✅ (eventual consistency acceptable)

**When It's NOT Acceptable:**
- Money deducted from account but credit never applied ❌ (requires compensation, not waiting)

**Pattern: Saga**

When you need to coordinate across services, use the Saga pattern (compensating transactions).

**Example: Transfer $100 from Account A to Account B**

```
Step 1: Account Service debits Account A ($100)
Step 2: Account Service credits Account B ($100)

If Step 2 fails:
  Compensation: Account Service credits Account A ($100) — undo the debit
```

This is NOT atomic/ACID. But it's reliable: either the transfer succeeds or both accounts are in original state.

---

## Design Patterns

### 1. API Gateway

**Problem:** Clients need to know about dozens of internal services; scales poorly.

**Solution:** Single entry point (API Gateway) routes requests to services.

**Responsibilities:**
- Request routing (determine which service handles this request)
- Authentication/Authorization (verify client identity and permissions)
- Rate limiting (prevent abuse)
- Logging/Monitoring (see all requests)
- Request/Response transformation (e.g., caching, compression)

**Architecture:**

```
Client → API Gateway → Route to appropriate service
  (1 entry point)    ↓
                  User Service
                  Order Service
                  Payment Service
```

**Technology Examples:**
- **Azure:** Azure API Management, Application Gateway
- **AWS:** API Gateway, ALB
- **Open Source:** Kong, Nginx, Tyk
- **Service Mesh:** Envoy, Istio (advanced)

### 2. Service Discovery

**Problem:** Services need to find each other's network locations; IPs/ports change dynamically.

**Solutions:**

**Option A: Client-Side Discovery**
- Service registers itself (e.g., "Order Service at 10.0.1.5:3000")
- Client queries registry to find service location
- Tools: Consul, Eureka, etcd

**Option B: Server-Side Discovery**
- Service registers itself with load balancer
- Client calls load balancer; load balancer routes to available service
- Tools: Kubernetes Service, AWS ELB, Azure Load Balancer

**Kubernetes Example:**
```csharp
// In order-service pod:
// Call user-service by its DNS name (Kubernetes handles discovery)
var response = await _httpClient.GetAsync("http://user-service:3000/users/123");
```

Kubernetes Service provides:
- DNS name resolution (user-service → current IP)
- Load balancing across replicas
- Automatic scaling (add/remove pods; load balancer knows)

### 3. Circuit Breaker

**Problem:** If Order Service calls Payment Service, and Payment Service is slow/down, requests pile up and Order Service also becomes slow.

**Solution:** Circuit Breaker pattern stops calling a failing service.

**Three States:**

```
CLOSED (normal)
  ↓ (service seems healthy)
  │
  └→ OPEN (service failing)
      ↓ (after N failures or timeout)
      │
      └→ HALF-OPEN (test recovery)
          ↓ (successful request)
          │
          └→ CLOSED (service recovered!)
```

**Example Implementation in C#:**

```csharp
// Using Polly library
var policy = Policy
    .Handle<HttpRequestException>()
    .Or<TimeoutException>()
    .CircuitBreaker(
        handledEventsAllowedBeforeBreaking: 5,
        durationOfBreak: TimeSpan.FromSeconds(30)
    );

try
{
    await policy.ExecuteAsync(async () =>
    {
        return await _httpClient.GetAsync("http://payment-service/process");
    });
}
catch (BrokenCircuitException)
{
    // Circuit is open; fail fast instead of waiting
    return new PaymentResponse { Status = "Temporarily unavailable" };
}
```

**Benefits:**
- Fail fast (don't wait for timeout)
- Give failed service time to recover
- Prevent cascading failures

### 4. Service Mesh

**Advanced Pattern:** Handle cross-cutting concerns (retries, circuit breakers, mTLS, observability) in infrastructure, not application code.

**How It Works:**
- Deploy a sidecar proxy alongside each service
- All inter-service traffic flows through proxy
- Proxy handles resilience, security, observability

**Tools:** Istio, Linkerd, Consul

**Benefit:** Services don't need to implement resilience logic; proxy handles it.

**Trade-off:** Adds operational complexity; requires Kubernetes or similar orchestration.

---

## Implementation Strategies

### Containerization (Docker)

**Why:** Ensures consistency across dev, test, staging, production.

**Simple Dockerfile Example:**

```dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:8.0

WORKDIR /app
COPY bin/Release/net8.0/publish ./

ENTRYPOINT ["dotnet", "OrderService.dll"]
```

**Benefit:** Same container runs everywhere; no "works on my machine" problems.

### Orchestration (Kubernetes)

**Why:** Manage containers at scale—deploy, scale, self-heal, rolling updates.

**Kubernetes Abstractions:**

| Concept | Meaning |
|---------|---------|
| **Pod** | Smallest unit; one or more containers. Usually one container per pod. |
| **Service** | Exposes pods to network; provides load balancing and DNS. |
| **Deployment** | Manage replica set; handles scaling, rolling updates. |
| **StatefulSet** | For services needing stable identity (e.g., databases). |
| **Namespace** | Logical isolation; separate envs (dev, staging, prod). |

**Simple Deployment Example:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: order-service
spec:
  replicas: 3
  selector:
    matchLabels:
      app: order-service
  template:
    metadata:
      labels:
        app: order-service
    spec:
      containers:
      - name: order-service
        image: myregistry.azurecr.io/order-service:1.0
        ports:
        - containerPort: 3000
---
apiVersion: v1
kind: Service
metadata:
  name: order-service
spec:
  selector:
    app: order-service
  ports:
  - protocol: TCP
    port: 80
    targetPort: 3000
  type: LoadBalancer
```

**What This Does:**
- Deploys 3 replicas of order-service
- Creates a Service that load-balances across replicas
- Automatic scaling, self-healing, rolling updates included

---

## Data Management

### Polyglot Persistence

Different services can use different database technologies based on their needs.

**Examples:**

| Service | Database | Why |
|---------|----------|-----|
| User Service | PostgreSQL (relational) | User data has many relationships; complex queries |
| Product Catalog | Elasticsearch | Full-text search; analytical queries |
| Session Cache | Redis | Fast reads; ephemeral data |
| Analytics | Data Warehouse (BigQuery, Snowflake) | Analytical workloads; OLAP not OLTP |

**Principle:** Choose the best tool for the problem; don't force all data into one database.

### Event Sourcing (Optional)

**Concept:** Store every state change as an immutable event; rebuild state by replaying events.

**Example:**
```csharp
// Instead of: UPDATE orders SET status = 'shipped' WHERE id = 123

// Event source: Store immutable event
public record OrderShippedEvent(int OrderId, DateTime ShippedAt, string TrackingNumber);

// Rebuild state: Replay all events for order 123
var events = await _eventStore.GetEventsAsync(orderId: 123);
var order = events.Aggregate(new Order(), (o, e) =>
{
    return e switch
    {
        OrderCreatedEvent ce => new Order { Id = ce.OrderId, Status = "created" },
        OrderShippedEvent se => o with { Status = "shipped", TrackingNumber = se.TrackingNumber },
        _ => o
    };
});
```

**Advantages:**
- Complete audit trail (know every state change)
- Temporal queries ("what was the state on March 5?")
- Replay for recovery/testing

**Disadvantages:**
- Complex to implement correctly
- Storage overhead (every event forever)
- Performance (rebuild state from events)

**When to Use:** When audit trail is critical (financial systems, healthcare) or temporal queries are needed.

---

## Communication Patterns

### Synchronous (Request-Response)

**How It Works:**
```
Service A calls Service B via HTTP/gRPC
Service A waits for response
Service B processes and returns response
```

**Example:**
```csharp
// Order Service calling User Service
var userResponse = await _httpClient.GetAsync($"http://user-service/users/{userId}");
var user = await userResponse.Content.ReadAsAsync<User>();
// Continue only after user data arrives
```

**Advantages:**
- Simple mental model; direct causality
- Immediate feedback

**Disadvantages:**
- Tight coupling (if User Service is down, Order Service blocks)
- Cascading failures (slow User Service makes Order Service slow)
- Hard to scale (synchronous calls serialize)

### Asynchronous (Event-Driven)

**How It Works:**
```
Service A publishes an event
Message broker routes to subscribers
Service B receives event and reacts
Service A doesn't wait for B's response
```

**Example:**
```csharp
// Order Service publishes event (doesn't wait for responses)
public async Task CreateOrder(Order order)
{
    _context.Orders.Add(order);
    await _context.SaveChangesAsync();
    
    // Publish event; return immediately
    var @event = new OrderCreatedEvent(order.Id, order.CustomerId, order.CreatedAt);
    await _messageBroker.PublishAsync(@event);
    
    return order; // Client gets response before subscribers process event
}

// Notification Service (subscriber) receives event independently
[MessageHandler]
public async Task OnOrderCreated(OrderCreatedEvent @event)
{
    var customer = await _customerService.GetAsync(@event.CustomerId);
    await _emailService.SendAsync(
        customer.Email,
        subject: "Order Confirmed",
        body: $"Order {event.OrderId} created"
    );
}
```

**Advantages:**
- Loose coupling; Order Service doesn't know about Notification Service
- Non-blocking; Order Service returns immediately
- Scales well; subscribers scale independently
- Resilience; if Notification Service is down, Order Service still works

**Disadvantages:**
- Eventual consistency (email might arrive 1 second late)
- Harder to debug (asynchronous call chains are complex)
- Requires careful handling of idempotency (what if event is processed twice?)

### Message Brokers

**Tools:** Azure Service Bus, RabbitMQ, Apache Kafka, AWS SNS/SQS

**Comparison:**

| Feature | Service Bus | RabbitMQ | Kafka |
|---------|-------------|----------|-------|
| **Durability** | Messages persisted | Configurable | Messages persisted in log |
| **Ordering** | FIFO in sessions | FIFO | FIFO within partition |
| **Throughput** | Medium | Medium | Very high |
| **Complexity** | Simple | Simple-Medium | Complex but powerful |
| **Use Case** | Enterprise messaging | Flexible routing | High-volume event streaming |

---

## Operational Challenges

### 1. Distributed Tracing

**Problem:** User clicks button → request flows through 5 services → something fails. Where did it break?

**Solution:** Trace the request across all services.

**How It Works:**
```
Client Request includes: TraceId = "abc123"
  ↓
Service 1 logs: TraceId=abc123, Action=CreateOrder, Duration=50ms
  ↓
Service 2 logs: TraceId=abc123, Action=ReserveInventory, Duration=30ms
  ↓
Service 3 logs: TraceId=abc123, Action=ProcessPayment, Duration=500ms (slow!)
```

**Tools:**
- **OpenTelemetry:** Standardized tracing API
- **Azure Application Insights:** Azure-native observability
- **Jaeger:** Open-source distributed tracing
- **DataDog, New Relic, Elastic:** Commercial APM tools

### 2. Centralized Logging

**Problem:** Logs scattered across 20 services; hard to debug issues.

**Solution:** Aggregate logs in one place.

**Example with Application Insights:**
```csharp
// Every service logs with TraceId
logger.LogInformation("Order {OrderId} created", orderId, new Dictionary<string, object>
{
    { "TraceId", Activity.Current?.Id },
    { "UserId", userId },
    { "Amount", order.Total }
});

// Query all logs for this TraceId
// See the full request flow across all services
```

### 3. Monitoring & Alerting

**Metrics to Watch:**
- **Request latency:** P50, P95, P99 percentiles
- **Error rate:** % of requests failing
- **Throughput:** Requests per second
- **Saturation:** CPU, memory, disk usage

**Alerting Strategy:**
- Alert on error rate > 1% (not every spike, but sustained problems)
- Alert on P99 latency degradation (users are slow)
- Alert on service dependency failure (downstream service down)

**Tools:** Prometheus + Grafana, Azure Monitor, DataDog

### 4. Testing Microservices

**Challenge:** Testing is harder with distributed systems.

**Strategies:**

**Unit Tests:** Test individual service in isolation
```csharp
[Test]
public async Task CreateOrder_ValidInput_ReturnsOrder()
{
    var service = new OrderService(_mockRepository, _mockMessageBroker);
    var order = await service.CreateOrderAsync(new CreateOrderRequest { ... });
    
    Assert.That(order.Id, Is.GreaterThan(0));
    _mockMessageBroker.Verify(x => x.PublishAsync(It.IsAny<OrderCreatedEvent>()));
}
```

**Integration Tests:** Test service + dependencies
```csharp
[Test]
public async Task CreateOrder_WithRealDatabase_PersistsOrder()
{
    using var context = new TestOrderDbContext();
    var service = new OrderService(context, _mockMessageBroker);
    
    var order = await service.CreateOrderAsync(...);
    
    var persisted = await context.Orders.FindAsync(order.Id);
    Assert.NotNull(persisted);
}
```

**Contract Tests:** Verify service API matches what consumers expect
```csharp
// Provider test: Order Service
[Test]
public async Task OrderCreatedEvent_MatchesExpectedSchema()
{
    var @event = new OrderCreatedEvent(123, 456, DateTime.UtcNow);
    var json = JsonConvert.SerializeObject(@event);
    
    // Verify it contains expected fields
    var obj = JObject.Parse(json);
    Assert.That(obj["orderId"], Is.EqualTo(123));
}
```

**E2E Tests:** Test full workflow (expensive, use sparingly)
```csharp
[Test]
public async Task OrderWorkflow_CreateOrder_NotificationSent()
{
    // Start all services
    await StartOrderService();
    await StartNotificationService();
    
    // Create order
    var order = await CreateOrderViaApi(...);
    
    // Verify notification sent (poll for ~5 seconds)
    var notification = await WaitForNotificationAsync(order.Id, timeout: 5000);
    Assert.NotNull(notification);
}
```

**Testing Pyramid:**
```
        E2E Tests (few, slow, comprehensive)
       /                                    \
      /   Integration Tests (moderate)       \
     /                                        \
    /   Unit Tests (many, fast, focused)      \
   /________________________________________\
```

---

## When NOT to Use Microservices

Microservices are not appropriate for every project.

### ❌ Don't Use If:

1. **Small Codebase** (<100k lines of code)
   - Monolith is simpler; shared database is fine
   - Microservices overhead isn't justified

2. **Single Team**
   - One team working on one service is a monolith with extra complexity
   - Microservices shine when multiple teams work independently

3. **Low Scalability Requirements**
   - No need for independent scaling; monolith scales fine
   - Operational complexity not worth it

4. **Tightly Coupled Domain**
   - If services must coordinate constantly, you've just created a distributed monolith
   - Go back to monolith; share database if needed

5. **Weak Operations/DevOps**
   - Microservices require mature CI/CD, monitoring, incident response
   - Without that, they become a nightmare to operate

6. **No Organizational Structure for Autonomy**
   - Microservices require teams to own services end-to-end
   - If you have organizational dependencies (need permission to deploy, etc.), monolith is better

### ✅ Good Candidates:

- 10+ engineers working on same system
- Independent scaling needs (some services get more traffic than others)
- Team autonomy is a goal (different teams, different release cycles)
- Operational maturity (monitoring, CI/CD, incident response in place)
- Domain naturally decomposes into services (e.g., e-commerce: catalog, orders, payments, shipping)

---

## Real-World Examples

### Netflix

**Why:** Started as a monolith; couldn't scale to millions of users and regions.

**Solution:** Broke into 500+ microservices.

**Key Learnings:**
- Circuit Breaker pattern (preventing cascading failures)
- Chaos engineering (intentionally breaking things to test resilience)
- Polyglot (different teams use different technologies)

**Tools:** Spring Boot, Eureka (service discovery), Hystrix (circuit breaker)

### Amazon

**Coined "Service-Oriented Architecture" (SOA), precursor to microservices.**

**Mandate (2002):** All teams must expose functionality via service interfaces.

**Result:** Enables independent scaling, organizational autonomy, AWS business model.

### Clinician Nexus (Your Interview)

**Likely Pattern:**
- User Service (manages students, teachers)
- Schedule Service (manages courses, scheduling)
- Notification Service (emails, SMS, push)
- Analytics Service (tracks engagement, reporting)
- Payment Service (tuition, payment processing)

**Design Questions You Might Face:**
- "How would you handle a database failure in the payment service?"
- "A notification is failing to send to 10% of users. How would you debug?"
- "We need to add a new feature that requires data from 3 services. How would you design it?"

---

## Resources & Further Reading

### Primary Sources (Verified Current)

1. **Martin Fowler's Microservices Article**
   - URL: https://martinfowler.com/articles/microservices.html
   - **What:** Canonical definition and original thinking on microservices
   - **Best For:** Understanding core concepts and principles
   - **Length:** 30–40 minutes to read
   - **Note:** Published 2014; consider also his 2015 article on [Microservice Trade-Offs](https://martinfowler.com/articles/microservice-trade-offs.html)

2. **Microsoft Azure Architecture Center — Microservices**
   - URL: https://learn.microsoft.com/en-us/azure/architecture/guide/architecture-styles/microservices
   - **What:** Practical guide to designing, building, and operating microservices on Azure
   - **Best For:** Implementation patterns, Azure-specific guidance, antipatterns
   - **Length:** 20–30 minutes
   - **Bonus:** Includes links to reference architectures (AKS microservices, CI/CD, etc.)

3. **Chris Richardson's Microservices.io**
   - URL: https://microservices.io
   - **What:** Comprehensive pattern language for microservices
   - **Best For:** Deep dives on specific patterns (saga, CQRS, event sourcing)
   - **Interactive:** Pattern browser, discussion forum, training resources
   - **Latest Updates:** Actively maintained (updated April 2026)

### Books

4. **Sam Newman — "Building Microservices" (2nd Edition)**
   - **What:** Authoritative practical guide to microservices design and operation
   - **Best For:** Architecture decisions, organizational patterns, operational considerations
   - **Length:** ~400 pages
   - **Writing Style:** Very accessible; real-world examples
   - **Note:** First edition (2015) is also valuable if you find it; second edition is current

### Technical Guides

5. **NGINX — Microservices Architecture Series**
   - **Topics:** Introduction, service discovery, API gateways, load balancing
   - **Best For:** Understanding how services communicate and scale
   - **Hands-On:** Good diagrams and real-world scenarios

6. **Azure Architecture Center — Microservices Design Patterns**
   - URL: https://learn.microsoft.com/en-us/azure/architecture/microservices/design/
   - **What:** Detailed patterns: Service Discovery, API Gateway, Event Sourcing, CQRS, Saga
   - **Best For:** Implementation guidance for specific problems

### Tools & Frameworks

7. **OpenTelemetry Documentation**
   - **What:** Standardized observability API for tracing, metrics, logs
   - **Best For:** Understanding distributed tracing and observability

8. **Kubernetes Official Documentation**
   - **What:** Complete Kubernetes reference
   - **Best For:** Learning orchestration and deployment

9. **Chris Richardson's Microservices Patterns in Java / Spring Boot**
   - **What:** Code examples of patterns (Saga, CQRS, Event Sourcing)
   - **Available:** GitHub repository with working examples

---

## Quick Reference

### Key Patterns at a Glance

| Pattern | Problem | Solution |
|---------|---------|----------|
| **API Gateway** | Many clients, many services | Single entry point |
| **Service Discovery** | Finding services dynamically | Registry or load balancer |
| **Circuit Breaker** | Cascading failures | Stop calling failed services |
| **Saga** | Distributed transactions | Compensating transactions |
| **Event Sourcing** | Audit trail, temporal queries | Store events, rebuild state |
| **CQRS** | Complex queries, separate read/write | Separate read and write models |

### Decision Tree: Should I Use Microservices?

```
Do you have 10+ engineers? NO → Use monolith
                           YES ↓
Do you need independent scaling? NO → Use monolith
                                  YES ↓
Can you decompose domain into services? NO → Use monolith
                                        YES ↓
Do you have mature DevOps/Monitoring? NO → Build it first
                                      YES ↓
Is org structured for autonomy? NO → Restructure first
                                YES ↓
→ Microservices is appropriate
```

---

**Last Updated:** 2026-05-11  
**Source:** Martin Fowler, Microsoft Azure Architecture Center, Chris Richardson's Microservices.io, Sam Newman's "Building Microservices"
