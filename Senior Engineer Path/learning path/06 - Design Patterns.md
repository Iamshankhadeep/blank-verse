# 06 - Design Patterns

Design patterns are proven solutions to common problems. Senior engineers recognize when to use them and, more importantly, when NOT to use them.

## What Are Design Patterns?

> "Each pattern describes a problem which occurs over and over again in our environment, and then describes the core of the solution to that problem." - Christopher Alexander

**Design patterns are:**
- ✅ Reusable solutions to common problems
- ✅ Communication tools (shared vocabulary)
- ✅ Proven best practices

**Design patterns are NOT:**
- ❌ Code to copy-paste
- ❌ Always the right solution
- ❌ Substitutes for thinking

## When to Use Patterns

### ✅ Use a pattern when:
- Problem fits the pattern exactly
- Team understands the pattern
- Simplifies the solution
- Improves maintainability

### ❌ DON'T use a pattern when:
- Forcing it to fit
- Over-engineering simple code
- Team doesn't understand it
- Simpler solution exists

> "Design patterns should not be applied indiscriminately. Often they achieve flexibility and variability by introducing additional levels of indirection, and that can complicate a design." - Gang of Four

## Essential Patterns for Senior Engineers

### 1. Strategy Pattern

**Problem:** Different algorithms/behaviors based on context

**When to use:**
- Multiple ways to do the same thing
- Want to switch behavior at runtime
- Avoid long if/else chains

**Example: Payment Processing**

```javascript
// ❌ Without Strategy Pattern
class PaymentProcessor {
  processPayment(amount, type) {
    if (type === 'credit_card') {
      // Credit card logic
      console.log(`Processing $${amount} via Credit Card`);
    } else if (type === 'paypal') {
      // PayPal logic
      console.log(`Processing $${amount} via PayPal`);
    } else if (type === 'crypto') {
      // Crypto logic
      console.log(`Processing $${amount} via Crypto`);
    }
  }
}

// Adding new payment method = modifying this class (violates OCP)
```

```javascript
// ✅ With Strategy Pattern
class PaymentStrategy {
  pay(amount) {
    throw new Error('Must implement pay()');
  }
}

class CreditCardPayment extends PaymentStrategy {
  pay(amount) {
    console.log(`Processing $${amount} via Credit Card`);
    // Credit card specific logic
  }
}

class PayPalPayment extends PaymentStrategy {
  pay(amount) {
    console.log(`Processing $${amount} via PayPal`);
    // PayPal specific logic
  }
}

class CryptoPayment extends PaymentStrategy {
  pay(amount) {
    console.log(`Processing $${amount} via Crypto`);
    // Crypto specific logic
  }
}

class PaymentProcessor {
  constructor(strategy) {
    this.strategy = strategy;
  }

  setStrategy(strategy) {
    this.strategy = strategy;
  }

  processPayment(amount) {
    this.strategy.pay(amount);
  }
}

// Usage
const processor = new PaymentProcessor(new CreditCardPayment());
processor.processPayment(100);

processor.setStrategy(new PayPalPayment());
processor.processPayment(200);
```

**Benefits:**
- Easy to add new payment methods (just create new class)
- Each strategy is independently testable
- Follows Open/Closed Principle

### 2. Factory Pattern

**Problem:** Complex object creation

**When to use:**
- Object creation is complex
- Want to hide creation logic
- Create different types based on input

**Example: User Creation**

```javascript
// ❌ Without Factory
const user1 = new AdminUser('John', 'john@example.com');
user1.permissions = ['read', 'write', 'delete'];
user1.dashboard = 'admin';

const user2 = new RegularUser('Jane', 'jane@example.com');
user2.permissions = ['read'];
user2.dashboard = 'user';
```

```javascript
// ✅ With Factory Pattern
class User {
  constructor(name, email, type) {
    this.name = name;
    this.email = email;
    this.type = type;
  }
}

class UserFactory {
  static createUser(name, email, type) {
    switch (type) {
      case 'admin':
        return this.createAdmin(name, email);
      case 'moderator':
        return this.createModerator(name, email);
      default:
        return this.createRegularUser(name, email);
    }
  }

  static createAdmin(name, email) {
    const user = new User(name, email, 'admin');
    user.permissions = ['read', 'write', 'delete', 'manage'];
    user.dashboard = 'admin';
    return user;
  }

  static createModerator(name, email) {
    const user = new User(name, email, 'moderator');
    user.permissions = ['read', 'write', 'moderate'];
    user.dashboard = 'moderator';
    return user;
  }

  static createRegularUser(name, email) {
    const user = new User(name, email, 'regular');
    user.permissions = ['read'];
    user.dashboard = 'user';
    return user;
  }
}

// Usage
const admin = UserFactory.createUser('John', 'john@example.com', 'admin');
const regular = UserFactory.createUser('Jane', 'jane@example.com', 'regular');
```

### 3. Observer Pattern (Pub/Sub)

**Problem:** One object needs to notify many objects of changes

**When to use:**
- Loose coupling between components
- One-to-many relationships
- Event-driven architecture

**Example: Event System**

```javascript
class EventEmitter {
  constructor() {
    this.events = {};
  }

  on(event, listener) {
    if (!this.events[event]) {
      this.events[event] = [];
    }
    this.events[event].push(listener);
  }

  off(event, listenerToRemove) {
    if (!this.events[event]) return;

    this.events[event] = this.events[event].filter(
      listener => listener !== listenerToRemove
    );
  }

  emit(event, data) {
    if (!this.events[event]) return;

    this.events[event].forEach(listener => {
      listener(data);
    });
  }
}

// Usage
const userEvents = new EventEmitter();

// Email service subscribes
userEvents.on('user:registered', (user) => {
  console.log(`Sending welcome email to ${user.email}`);
});

// Analytics service subscribes
userEvents.on('user:registered', (user) => {
  console.log(`Tracking registration for ${user.email}`);
});

// Notification service subscribes
userEvents.on('user:registered', (user) => {
  console.log(`Sending push notification to ${user.email}`);
});

// When user registers
const newUser = { email: 'john@example.com', name: 'John' };
userEvents.emit('user:registered', newUser);
// All subscribers are notified
```

### 4. Singleton Pattern

**Problem:** Need exactly one instance of a class

**When to use:**
- Database connection pool
- Configuration manager
- Logger
- Cache

**⚠️ Warning:** Often overused. Consider if you really need it.

```javascript
class DatabaseConnection {
  constructor() {
    if (DatabaseConnection.instance) {
      return DatabaseConnection.instance;
    }

    this.connection = this.createConnection();
    DatabaseConnection.instance = this;
  }

  createConnection() {
    console.log('Creating database connection');
    return { connected: true };
  }

  query(sql) {
    console.log(`Executing: ${sql}`);
    return this.connection;
  }
}

// Usage
const db1 = new DatabaseConnection();
const db2 = new DatabaseConnection();

console.log(db1 === db2); // true - same instance
```

**Better alternative (Dependency Injection):**
```javascript
// Create once
const dbConnection = new DatabaseConnection();

// Inject where needed
class UserRepository {
  constructor(db) {
    this.db = db;
  }

  findUser(id) {
    return this.db.query(`SELECT * FROM users WHERE id = ${id}`);
  }
}

const userRepo = new UserRepository(dbConnection);
```

### 5. Decorator Pattern

**Problem:** Add behavior to objects without modifying their class

**When to use:**
- Add responsibilities dynamically
- Avoid subclass explosion
- Keep classes focused

**Example: Logging & Authentication**

```javascript
// Base component
class DataService {
  getData() {
    return { data: 'some data' };
  }
}

// Decorator 1: Logging
class LoggingDecorator {
  constructor(service) {
    this.service = service;
  }

  getData() {
    console.log('Fetching data...');
    const result = this.service.getData();
    console.log('Data fetched:', result);
    return result;
  }
}

// Decorator 2: Caching
class CachingDecorator {
  constructor(service) {
    this.service = service;
    this.cache = null;
  }

  getData() {
    if (this.cache) {
      console.log('Returning cached data');
      return this.cache;
    }

    const result = this.service.getData();
    this.cache = result;
    return result;
  }
}

// Decorator 3: Authentication
class AuthDecorator {
  constructor(service, user) {
    this.service = service;
    this.user = user;
  }

  getData() {
    if (!this.user.isAuthenticated) {
      throw new Error('Not authenticated');
    }
    return this.service.getData();
  }
}

// Usage: Stack decorators
let service = new DataService();
service = new AuthDecorator(service, { isAuthenticated: true });
service = new CachingDecorator(service);
service = new LoggingDecorator(service);

service.getData(); // Logs, checks cache, checks auth, gets data
```

### 6. Adapter Pattern

**Problem:** Make incompatible interfaces work together

**When to use:**
- Integrating third-party libraries
- Legacy code integration
- API versioning

**Example: Payment Gateway Adapter**

```javascript
// Old payment system
class LegacyPaymentProcessor {
  makePayment(accountNumber, amount) {
    console.log(`Legacy: Processing $${amount} from account ${accountNumber}`);
  }
}

// New payment system interface
class ModernPaymentGateway {
  processTransaction(paymentDetails) {
    console.log(`Modern: Processing transaction`, paymentDetails);
  }
}

// Adapter to make legacy system work with new interface
class PaymentAdapter {
  constructor(legacyProcessor) {
    this.legacyProcessor = legacyProcessor;
  }

  processTransaction(paymentDetails) {
    // Convert modern format to legacy format
    this.legacyProcessor.makePayment(
      paymentDetails.accountNumber,
      paymentDetails.amount
    );
  }
}

// Usage
const legacySystem = new LegacyPaymentProcessor();
const adapter = new PaymentAdapter(legacySystem);

// Can now use legacy system with modern interface
adapter.processTransaction({
  accountNumber: '12345',
  amount: 100,
  currency: 'USD'
});
```

### 7. Repository Pattern

**Problem:** Separate data access logic from business logic

**When to use:**
- Want to abstract database operations
- Need to swap data sources
- Improve testability

**Example: User Repository**

```javascript
// Repository interface
class UserRepository {
  findById(id) {
    throw new Error('Must implement');
  }

  findAll() {
    throw new Error('Must implement');
  }

  save(user) {
    throw new Error('Must implement');
  }

  delete(id) {
    throw new Error('Must implement');
  }
}

// SQL implementation
class SQLUserRepository extends UserRepository {
  constructor(database) {
    super();
    this.db = database;
  }

  findById(id) {
    return this.db.query('SELECT * FROM users WHERE id = ?', [id]);
  }

  findAll() {
    return this.db.query('SELECT * FROM users');
  }

  save(user) {
    return this.db.query(
      'INSERT INTO users (name, email) VALUES (?, ?)',
      [user.name, user.email]
    );
  }

  delete(id) {
    return this.db.query('DELETE FROM users WHERE id = ?', [id]);
  }
}

// MongoDB implementation
class MongoUserRepository extends UserRepository {
  constructor(collection) {
    super();
    this.collection = collection;
  }

  async findById(id) {
    return await this.collection.findOne({ _id: id });
  }

  async findAll() {
    return await this.collection.find({}).toArray();
  }

  async save(user) {
    return await this.collection.insertOne(user);
  }

  async delete(id) {
    return await this.collection.deleteOne({ _id: id });
  }
}

// Business logic doesn't care about database
class UserService {
  constructor(userRepository) {
    this.userRepository = userRepository;
  }

  async getUser(id) {
    return await this.userRepository.findById(id);
  }

  async createUser(userData) {
    // Business logic here
    return await this.userRepository.save(userData);
  }
}

// Easy to swap implementations
const userService = new UserService(new SQLUserRepository(db));
// or
const userService2 = new UserService(new MongoUserRepository(collection));
```

## Pattern Combinations

Patterns often work together:

```javascript
// Factory + Strategy + Repository
class PaymentProcessorFactory {
  static create(type, repository) {
    const strategy = this.createStrategy(type);
    return new PaymentProcessor(strategy, repository);
  }

  static createStrategy(type) {
    switch (type) {
      case 'card':
        return new CreditCardStrategy();
      case 'paypal':
        return new PayPalStrategy();
      default:
        throw new Error('Unknown payment type');
    }
  }
}

class PaymentProcessor {
  constructor(strategy, repository) {
    this.strategy = strategy;
    this.repository = repository;
  }

  async process(amount) {
    const result = await this.strategy.pay(amount);
    await this.repository.save(result);
    return result;
  }
}
```

## Anti-Patterns to Avoid

### 1. God Object
One class that does everything
```javascript
// ❌ BAD
class Application {
  handleRequest() { }
  renderUI() { }
  connectDatabase() { }
  sendEmail() { }
  processPayment() { }
  generateReport() { }
}
```

### 2. Golden Hammer
Using the same pattern for everything
```javascript
// ❌ Using Factory for simple objects
const user = UserFactory.create('John'); // Overkill

// ✅ Just create it directly
const user = new User('John');
```

### 3. Overengineering
Too many patterns for simple problems
```javascript
// ❌ For a simple calculator
class CalculatorFactory {
  create() {
    return new CalculatorBuilder()
      .withStrategy(new AdditionStrategy())
      .withDecorator(new LoggingDecorator())
      .build();
  }
}

// ✅ Just do it simply
function add(a, b) {
  return a + b;
}
```

## Pattern Selection Guide

| Problem | Pattern |
|---------|---------|
| Multiple algorithms/behaviors | Strategy |
| Complex object creation | Factory |
| Notify multiple objects | Observer |
| Need exactly one instance | Singleton (use sparingly) |
| Add behavior dynamically | Decorator |
| Incompatible interfaces | Adapter |
| Abstract data access | Repository |
| Simplify complex interface | Facade |
| Share expensive objects | Flyweight |

## When Reviewing Code

### Look for:
- [ ] Repeated if/else for types → Strategy
- [ ] Complex object creation → Factory
- [ ] Tight coupling → Dependency Injection
- [ ] Data access mixed with logic → Repository
- [ ] Notification needs → Observer

### Ask:
- Is this the right pattern?
- Is a pattern needed at all?
- Is this over-engineered?
- Would a simpler solution work?

## This Week's Practice

### Day 1-2: Recognition
- [ ] Find 3 places where a pattern could help
- [ ] Identify which pattern fits
- [ ] Consider if it's worth the complexity

### Day 3-4: Implementation
- [ ] Refactor one area using appropriate pattern
- [ ] Ensure tests pass
- [ ] Measure: Is it actually better?

### Day 5-7: Review
- [ ] In PR reviews, recognize patterns
- [ ] Suggest patterns when appropriate
- [ ] Question over-engineering

## Resources

**Books:**
- Head First Design Patterns (easiest)
- Design Patterns: Elements of Reusable Object-Oriented Software (GoF)

**Online:**
- Refactoring Guru: https://refactoring.guru/design-patterns
- Source Making: https://sourcemaking.com/design_patterns

## Remember

> "Don't use a pattern just because you can. Use it because it solves a problem better than a simpler solution would."

Patterns are tools, not goals. Simple code > clever patterns.

---

**Previous:** [[05 - Refactoring Techniques]]
**Next:** [[07 - Testing Strategies]]
**Related:** [[04 - SOLID Principles]]
