# 11 - Architecture Patterns

Architecture decisions are the hardest to change. Senior engineers recognize common patterns and know when to apply them.

## What is Software Architecture?

> "Architecture is about the important stuff. Whatever that is." - Ralph Johnson

**Architecture decisions:**
- Are hard to change
- Affect the entire system
- Have long-term consequences
- Require trade-off analysis

**Not architecture:**
- Which CSS framework to use
- Naming conventions
- Code formatting
- Local refactoring

## Layered Architecture

### The Classic N-Tier Architecture

```
┌─────────────────────┐
│  Presentation Layer │  (UI, Controllers, API)
├─────────────────────┤
│   Business Layer    │  (Domain Logic, Services)
├─────────────────────┤
│    Data Layer       │  (Database, Repositories)
└─────────────────────┘
```

**Rules:**
- Each layer only talks to the layer below
- Presentation ← Business ← Data
- Never skip layers

**Example:**
```javascript
// ❌ BAD: Controller talks directly to database
class UserController {
  async getUser(req, res) {
    const user = await db.query('SELECT * FROM users WHERE id = ?', [req.params.id]);
    res.json(user);
  }
}

// ✅ GOOD: Layers separated
// Presentation Layer
class UserController {
  constructor(userService) {
    this.userService = userService;
  }

  async getUser(req, res) {
    const user = await this.userService.getUserById(req.params.id);
    res.json(user);
  }
}

// Business Layer
class UserService {
  constructor(userRepository) {
    this.userRepository = userRepository;
  }

  async getUserById(id) {
    const user = await this.userRepository.findById(id);
    if (!user) {
      throw new Error('User not found');
    }
    return user;
  }
}

// Data Layer
class UserRepository {
  async findById(id) {
    return await db.query('SELECT * FROM users WHERE id = ?', [id]);
  }
}
```

**Benefits:**
- ✅ Clear separation of concerns
- ✅ Easy to test each layer
- ✅ Can swap implementations

**Drawbacks:**
- ❌ Can be over-engineered for simple apps
- ❌ Performance overhead from layers

## Microservices Architecture

### Monolith vs Microservices

**Monolith:**
```
┌──────────────────────────┐
│                          │
│  ┌────┐  ┌────┐  ┌────┐ │
│  │ UI │  │API │  │ DB │ │
│  └────┘  └────┘  └────┘ │
│                          │
│    Single Deployment     │
└──────────────────────────┘
```

**Microservices:**
```
┌─────────┐  ┌─────────┐  ┌─────────┐
│ User    │  │ Order   │  │ Payment │
│ Service │  │ Service │  │ Service │
│ + DB    │  │ + DB    │  │ + DB    │
└─────────┘  └─────────┘  └─────────┘
     │            │            │
     └────────────┴────────────┘
           API Gateway
```

**When to use Microservices:**
- ✅ Large team (>20 people)
- ✅ Need independent scaling
- ✅ Different technology requirements
- ✅ Clear bounded contexts
- ✅ Strong DevOps capability

**When to use Monolith:**
- ✅ Small team (<10 people)
- ✅ Early stage / MVP
- ✅ Unclear domain boundaries
- ✅ Simple deployment preferred

**The Rule:** Start with a monolith, extract services when pain is real.

### Microservices Communication

**Synchronous (REST/gRPC):**
```javascript
// User Service calls Order Service
class UserService {
  async getUserOrders(userId) {
    const user = await this.userRepo.findById(userId);

    // HTTP call to Order Service
    const orders = await fetch(`http://order-service/orders?userId=${userId}`);

    return {
      user,
      orders
    };
  }
}
```

**Pros:** Simple, immediate response
**Cons:** Coupling, cascading failures

**Asynchronous (Message Queue):**
```javascript
// User Service publishes event
class UserService {
  async createUser(userData) {
    const user = await this.userRepo.save(userData);

    // Publish event
    await messageQueue.publish('user.created', {
      userId: user.id,
      email: user.email
    });

    return user;
  }
}

// Order Service subscribes to event
messageQueue.subscribe('user.created', async (data) => {
  await orderService.initializeUserAccount(data.userId);
});

// Email Service subscribes to same event
messageQueue.subscribe('user.created', async (data) => {
  await emailService.sendWelcome(data.email);
});
```

**Pros:** Decoupled, resilient
**Cons:** Eventual consistency, complexity

## Event-Driven Architecture

### Event Sourcing

Instead of storing current state, store events that led to that state.

**Traditional (State-based):**
```javascript
// Database
{
  userId: 123,
  balance: 950  // Current state only
}
```

**Event Sourcing:**
```javascript
// Events
[
  { type: 'AccountCreated', userId: 123, initialBalance: 1000 },
  { type: 'MoneyWithdrawn', userId: 123, amount: 50 },
  { type: 'MoneyDeposited', userId: 123, amount: 100 },
  { type: 'MoneyWithdrawn', userId: 123, amount: 100 }
]

// Rebuild state by replaying events
function getBalance(events) {
  return events.reduce((balance, event) => {
    switch(event.type) {
      case 'AccountCreated':
        return event.initialBalance;
      case 'MoneyDeposited':
        return balance + event.amount;
      case 'MoneyWithdrawn':
        return balance - event.amount;
      default:
        return balance;
    }
  }, 0);
}
// Result: 950
```

**Benefits:**
- ✅ Complete audit trail
- ✅ Can replay events
- ✅ Time travel debugging
- ✅ Event-driven workflows

**Drawbacks:**
- ❌ Complex queries
- ❌ Event schema evolution
- ❌ Storage overhead

## CQRS (Command Query Responsibility Segregation)

Separate reads and writes.

**Traditional:**
```javascript
class UserService {
  // Same model for reads and writes
  async getUser(id) {
    return await this.userRepo.findById(id);
  }

  async updateUser(id, data) {
    return await this.userRepo.update(id, data);
  }
}
```

**CQRS:**
```javascript
// Write Model (Commands)
class UserCommandService {
  async createUser(command) {
    const user = new User(command);
    await this.writeRepo.save(user);

    // Publish event
    await this.eventBus.publish('user.created', user);
  }

  async updateUser(command) {
    const user = await this.writeRepo.findById(command.id);
    user.update(command);
    await this.writeRepo.save(user);

    await this.eventBus.publish('user.updated', user);
  }
}

// Read Model (Queries)
class UserQueryService {
  async getUserById(id) {
    // Optimized read model
    return await this.readRepo.findById(id);
  }

  async searchUsers(criteria) {
    // Denormalized, optimized for searching
    return await this.searchIndex.search(criteria);
  }
}

// Event handler keeps read model in sync
eventBus.on('user.created', async (user) => {
  await readRepo.save(user);
  await searchIndex.index(user);
});
```

**Benefits:**
- ✅ Optimize reads separately from writes
- ✅ Different data models for different needs
- ✅ Scalability

**Drawbacks:**
- ❌ Complexity
- ❌ Eventual consistency

## Hexagonal Architecture (Ports & Adapters)

**Goal:** Business logic independent of external concerns

```
┌─────────────────────────────────┐
│                                 │
│   ┌─────────────────────┐       │
│   │  Business Logic     │       │
│   │   (Core Domain)     │       │
│   └─────────────────────┘       │
│            │                    │
│   ┌────────┴────────┐           │
│   │  Ports (Interfaces)         │
│   └────────┬────────┘           │
│            │                    │
└────────────┼────────────────────┘
             │
   ┌─────────┴─────────┐
   │  Adapters         │
   ├───────────────────┤
   │ - REST API        │
   │ - Database        │
   │ - Message Queue   │
   │ - Email Service   │
   └───────────────────┘
```

**Example:**
```javascript
// Port (Interface)
class UserRepository {
  async save(user) {
    throw new Error('Must implement');
  }
  async findById(id) {
    throw new Error('Must implement');
  }
}

// Core Domain (Business Logic)
class UserService {
  constructor(userRepository) {
    this.userRepository = userRepository;  // Depends on interface
  }

  async registerUser(userData) {
    // Business logic
    const user = new User(userData);
    user.validate();

    await this.userRepository.save(user);
    return user;
  }
}

// Adapter 1: SQL Implementation
class SQLUserRepository extends UserRepository {
  async save(user) {
    return await db.query('INSERT INTO users ...', user);
  }
  async findById(id) {
    return await db.query('SELECT * FROM users WHERE id = ?', [id]);
  }
}

// Adapter 2: MongoDB Implementation
class MongoUserRepository extends UserRepository {
  async save(user) {
    return await mongodb.users.insertOne(user);
  }
  async findById(id) {
    return await mongodb.users.findOne({ _id: id });
  }
}

// Adapter 3: In-Memory (for testing)
class InMemoryUserRepository extends UserRepository {
  constructor() {
    super();
    this.users = new Map();
  }
  async save(user) {
    this.users.set(user.id, user);
  }
  async findById(id) {
    return this.users.get(id);
  }
}

// Use any adapter
const service = new UserService(new SQLUserRepository());
// or
const service = new UserService(new MongoUserRepository());
// or (for testing)
const service = new UserService(new InMemoryUserRepository());
```

**Benefits:**
- ✅ Business logic is testable
- ✅ Easy to swap implementations
- ✅ Technology agnostic core

## API Gateway Pattern

Single entry point for microservices.

```
Client
  │
  ├─ API Gateway ───┬─ User Service
  │                 ├─ Order Service
  │                 ├─ Payment Service
  │                 └─ Notification Service
```

**Responsibilities:**
- Routing
- Authentication
- Rate limiting
- Response aggregation
- Protocol translation

```javascript
// API Gateway
app.get('/api/user/:id/dashboard', async (req, res) => {
  // Aggregate data from multiple services
  const [user, orders, notifications] = await Promise.all([
    userService.getUser(req.params.id),
    orderService.getUserOrders(req.params.id),
    notificationService.getUserNotifications(req.params.id)
  ]);

  res.json({
    user,
    recentOrders: orders.slice(0, 5),
    unreadNotifications: notifications.filter(n => !n.read)
  });
});
```

## BFF (Backend For Frontend) Pattern

Different backends for different clients.

```
Mobile App ──── Mobile BFF ─────┬
                                │
Web App ─────── Web BFF ────────┼─── Microservices
                                │
Admin Panel ─── Admin BFF ──────┘
```

**Why:** Different clients need different data shapes.

```javascript
// Web BFF - Returns rich data
app.get('/api/products', async (req, res) => {
  const products = await productService.getProducts();
  res.json({
    products: products.map(p => ({
      id: p.id,
      name: p.name,
      description: p.description,  // Full description
      price: p.price,
      images: p.images,  // All images
      reviews: p.reviews  // All reviews
    }))
  });
});

// Mobile BFF - Returns minimal data
app.get('/api/products', async (req, res) => {
  const products = await productService.getProducts();
  res.json({
    products: products.map(p => ({
      id: p.id,
      name: p.name,
      price: p.price,
      thumbnail: p.images[0]  // Just first image
      // No description, reviews
    }))
  });
});
```

## Strangler Fig Pattern

Gradually replace legacy system.

```
Phase 1:              Phase 2:              Phase 3:
┌────────┐           ┌────────┐           ┌────────┐
│Legacy  │           │New ┌───┤           │  New   │
│System  │    →      │    │Leg│    →      │ System │
│        │           │    │acy│           │        │
└────────┘           └────┴───┘           └────────┘
```

**Strategy:**
1. Proxy routes to new system when implemented
2. Fall back to legacy for non-migrated features
3. Gradually migrate all features
4. Decomission legacy

```javascript
// Router/Proxy
app.use('/api/users', (req, res, next) => {
  if (isUserFeatureMigrated(req.path)) {
    // Route to new system
    proxy.web(req, res, { target: 'http://new-system' });
  } else {
    // Route to legacy system
    proxy.web(req, res, { target: 'http://legacy-system' });
  }
});
```

## Architecture Decision Records (ADRs)

Document important decisions.

**Template:**
```markdown
# ADR-001: Use PostgreSQL for Primary Database

## Status
Accepted

## Context
We need a database for user data, transactions, and reporting.
Requirements:
- ACID guarantees for transactions
- Complex querying for reports
- Open source
- Team expertise

## Decision
Use PostgreSQL as our primary database.

## Consequences

Positive:
- Strong consistency guarantees
- Excellent query capabilities (JSON, full-text search)
- Great tooling and community
- Team already knows SQL

Negative:
- Harder to scale horizontally (but not an immediate concern)
- More complex setup than managed NoSQL

## Alternatives Considered
- MongoDB: Good scalability but weaker consistency
- DynamoDB: Great performance but vendor lock-in
- MySQL: Similar to PostgreSQL but less features
```

## This Week's Practice

### Day 1-2: Understand
- [ ] Identify your current architecture
- [ ] Diagram the key components
- [ ] Document dependencies

### Day 3-4: Analyze
- [ ] Find pain points in current architecture
- [ ] Identify coupling between components
- [ ] Consider alternatives

### Day 5-7: Review
- [ ] Review PRs for architectural impact
- [ ] Ask about dependencies being added
- [ ] Suggest better patterns when appropriate

## Remember

> "Architecture is a hypothesis that needs to be validated through implementation and measurement."

**Key principles:**
- Start simple, evolve as needed
- Document decisions (ADRs)
- Consider trade-offs
- Optimize for change
- Don't over-engineer

---

**Previous:** [[10 - Performance & Optimization]]
**Next:** [[12 - Technical Debt Management]]
**Related:** [[09 - System Design Basics]], [[15 - Trade-offs & Decision Making]]
