# 09 - System Design Basics

Senior engineers think about the system, not just the code. Understanding system design is crucial for architectural reviews and technical leadership.

## Why System Design Matters

Mid-level engineers: "This code works."
Senior engineers: "This system scales, is maintainable, and handles failure gracefully."

## Key Concepts

### 1. Scalability

**Vertical Scaling (Scale Up)**
- Add more power to existing machine (CPU, RAM)
- ✅ Simple
- ❌ Limited by hardware
- ❌ Single point of failure

**Horizontal Scaling (Scale Out)**
- Add more machines
- ✅ No hardware limits
- ✅ Better fault tolerance
- ❌ More complex (load balancing, data consistency)

**When reviewing code, ask:**
- Will this work with 1000x the current load?
- Are we assuming single-server architecture?
- Can we add more servers to handle this?

### 2. Load Balancing

Distributes traffic across multiple servers.

**Types:**
- **Round Robin**: Distribute evenly
- **Least Connections**: Send to least busy server
- **IP Hash**: Same client → same server

**Code review implications:**
- Session management (stateless > stateful)
- Database connections (use connection pooling)
- Caching strategies

### 3. Caching

**Cache Levels:**
```
Browser Cache → CDN → Application Cache → Database Cache
```

**Strategies:**
- **Cache-Aside**: App checks cache, then DB
- **Write-Through**: Write to cache and DB together
- **Write-Behind**: Write to cache, async to DB

**When reviewing:**
- Are we caching frequently accessed data?
- What's the cache invalidation strategy?
- Are we caching user-specific data correctly?

```javascript
// ❌ Without caching - DB hit every time
async function getUser(id) {
  return await db.query('SELECT * FROM users WHERE id = ?', [id]);
}

// ✅ With caching
async function getUser(id) {
  const cached = await cache.get(`user:${id}`);
  if (cached) return cached;

  const user = await db.query('SELECT * FROM users WHERE id = ?', [id]);
  await cache.set(`user:${id}`, user, { ttl: 3600 });
  return user;
}
```

### 4. Database Design

**Relational (SQL)**
- ✅ ACID guarantees
- ✅ Complex queries
- ✅ Relationships
- ❌ Harder to scale horizontally

**Non-Relational (NoSQL)**
- ✅ Horizontal scaling
- ✅ Flexible schema
- ✅ Fast for simple queries
- ❌ Limited complex queries

**When to use each:**
- SQL: Financial transactions, complex reporting
- NoSQL: Real-time analytics, user profiles, logs

**Database Patterns:**

**Read Replicas:**
```
Write → Primary DB
Read → Replica 1, 2, 3...
```

**Sharding:**
```
Users 1-1M → Shard 1
Users 1M-2M → Shard 2
```

### 5. Message Queues

Decouple services and handle async operations.

```javascript
// ❌ Synchronous - Slow for user
app.post('/register', async (req, res) => {
  await createUser(req.body);
  await sendWelcomeEmail(req.body.email); // Blocks response
  await generateAnalytics(req.body);
  res.json({ success: true });
});

// ✅ Asynchronous - Fast response
app.post('/register', async (req, res) => {
  await createUser(req.body);

  // Queue background tasks
  await queue.publish('user.registered', {
    userId: user.id,
    email: user.email
  });

  res.json({ success: true }); // Immediate response
});

// Worker processes queue
queue.subscribe('user.registered', async (data) => {
  await sendWelcomeEmail(data.email);
  await generateAnalytics(data);
});
```

**Popular queues:**
- RabbitMQ
- Redis (pub/sub)
- AWS SQS
- Kafka (high throughput)

### 6. Microservices vs Monolith

**Monolith:**
- ✅ Simpler to develop initially
- ✅ Easier to debug
- ✅ Simpler deployment
- ❌ Hard to scale specific parts
- ❌ All-or-nothing deployment

**Microservices:**
- ✅ Scale services independently
- ✅ Technology flexibility
- ✅ Team autonomy
- ❌ Complex infrastructure
- ❌ Network overhead
- ❌ Distributed system challenges

**Rule of thumb:**
- Start with monolith
- Extract microservices when:
  - Clear boundaries emerge
  - Specific scaling needs
  - Team size > 20
  - You have DevOps expertise

### 7. API Design

**RESTful Principles:**
```
GET    /users        → List users
GET    /users/:id    → Get user
POST   /users        → Create user
PUT    /users/:id    → Update user
DELETE /users/:id    → Delete user
```

**Key principles:**
- Use HTTP methods correctly
- Return appropriate status codes
- Version your API (`/api/v1/users`)
- Use pagination for lists
- Include rate limiting

**GraphQL Alternative:**
- Client specifies exactly what data needed
- Single endpoint
- Reduces over-fetching

## System Design Trade-offs

Every decision is a trade-off. Senior engineers understand and communicate these.

### Consistency vs Availability (CAP Theorem)

**You can have 2 of 3:**
- **C**onsistency: All nodes see same data
- **A**vailability: System always responds
- **P**artition Tolerance: Works despite network issues

**Examples:**
- Banking (CP): Consistency > Availability
- Social media (AP): Availability > Consistency

### Latency vs Throughput

- **Latency**: Time for one request
- **Throughput**: Requests per second

Sometimes optimizing one hurts the other.

### Strong vs Eventual Consistency

**Strong:**
- Read always gets latest write
- ✅ Guarantees accuracy
- ❌ Slower, less available

**Eventual:**
- Reads might be stale temporarily
- ✅ Faster, more available
- ❌ Temporary inconsistency

## Code Review for System Design

### Ask These Questions:

#### Performance
- [ ] What's the time complexity? (O(n)? O(n²)?)
- [ ] Will this scale to 1M records?
- [ ] Are we doing N+1 queries?
- [ ] Should this be cached?
- [ ] Should this be async/background job?

#### Reliability
- [ ] What happens if this service is down?
- [ ] What happens if the database is unavailable?
- [ ] Are we handling timeouts?
- [ ] What if this operation is called twice? (Idempotency)

#### Data
- [ ] What's the data retention policy?
- [ ] Do we need historical data?
- [ ] What about GDPR/data privacy?
- [ ] Is this the right database for this data?

#### Architecture
- [ ] Does this fit our architecture?
- [ ] Are we introducing new dependencies?
- [ ] Can this be deployed independently?
- [ ] Is this creating coupling?

## Common Patterns to Recognize

### Circuit Breaker
Stop calling failing service to prevent cascading failures.

```javascript
class CircuitBreaker {
  constructor(threshold = 5) {
    this.failures = 0;
    this.threshold = threshold;
    this.state = 'CLOSED'; // CLOSED, OPEN, HALF_OPEN
  }

  async call(fn) {
    if (this.state === 'OPEN') {
      throw new Error('Circuit breaker is OPEN');
    }

    try {
      const result = await fn();
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }

  onSuccess() {
    this.failures = 0;
    this.state = 'CLOSED';
  }

  onFailure() {
    this.failures++;
    if (this.failures >= this.threshold) {
      this.state = 'OPEN';
      setTimeout(() => this.state = 'HALF_OPEN', 60000);
    }
  }
}
```

### Retry with Exponential Backoff
```javascript
async function retryWithBackoff(fn, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;

      const delay = Math.pow(2, i) * 1000; // 1s, 2s, 4s...
      await sleep(delay);
    }
  }
}
```

### Rate Limiting
```javascript
class RateLimiter {
  constructor(maxRequests, windowMs) {
    this.maxRequests = maxRequests;
    this.windowMs = windowMs;
    this.requests = new Map();
  }

  isAllowed(userId) {
    const now = Date.now();
    const userRequests = this.requests.get(userId) || [];

    // Remove old requests
    const validRequests = userRequests.filter(
      time => now - time < this.windowMs
    );

    if (validRequests.length >= this.maxRequests) {
      return false;
    }

    validRequests.push(now);
    this.requests.set(userId, validRequests);
    return true;
  }
}
```

## This Week's Practice

### Day 1-2: Learn
- [ ] Study one system design pattern
- [ ] Understand trade-offs
- [ ] Draw a diagram

### Day 3-4: Review
- [ ] Review PRs with system design lens
- [ ] Ask scalability questions
- [ ] Consider failure scenarios

### Day 5-7: Design
- [ ] Diagram your current system
- [ ] Identify bottlenecks
- [ ] Propose improvements

## Resources

**Books:**
- Designing Data-Intensive Applications (Kleppmann)
- System Design Interview Vol 1 & 2 (Alex Xu)

**Online:**
- System Design Primer (GitHub)
- ByteByteGo blog

**Practice:**
- Design Twitter
- Design URL shortener
- Design Netflix
- Design Uber

---

**Previous:** [[08 - Security in Code Reviews]]
**Next:** [[10 - Performance & Optimization]]
**Related:** [[15 - Trade-offs & Decision Making]]
