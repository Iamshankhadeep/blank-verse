# 04 - SOLID Principles

The SOLID principles are the foundation of maintainable object-oriented design. Understanding these deeply separates mid-level from senior engineers.

## S - Single Responsibility Principle (SRP)

> "A class should have only one reason to change."

**Meaning:** Each class should have one job, one responsibility.

### ❌ Violation
```javascript
class User {
  constructor(name, email) {
    this.name = name;
    this.email = email;
  }

  // Responsibility 1: User data
  getName() { return this.name; }

  // Responsibility 2: Database operations
  save() {
    database.save(this);
  }

  // Responsibility 3: Email operations
  sendEmail(subject, body) {
    emailService.send(this.email, subject, body);
  }

  // Responsibility 4: Validation
  validate() {
    if (!this.email.includes('@')) throw new Error();
  }
}
```

**Problems:**
- Changes to email logic affect User class
- Changes to database affect User class
- Hard to test
- Hard to reuse

### ✅ Better
```javascript
// Single responsibility: Hold user data
class User {
  constructor(name, email) {
    this.name = name;
    this.email = email;
  }

  getName() { return this.name; }
  getEmail() { return this.email; }
}

// Single responsibility: Validate users
class UserValidator {
  validate(user) {
    if (!user.getEmail().includes('@')) {
      throw new Error('Invalid email');
    }
  }
}

// Single responsibility: Persist users
class UserRepository {
  save(user) {
    database.save(user);
  }
}

// Single responsibility: Email operations
class EmailService {
  sendWelcomeEmail(user) {
    this.send(user.getEmail(), "Welcome", "...");
  }
}
```

**Benefits:**
- Each class has one reason to change
- Easier to test in isolation
- Easier to reuse
- Clearer responsibilities

## O - Open/Closed Principle (OCP)

> "Software entities should be open for extension, but closed for modification."

**Meaning:** Add new features by extending code, not by modifying existing code.

### ❌ Violation
```javascript
class PaymentProcessor {
  processPayment(payment, type) {
    if (type === 'credit_card') {
      // process credit card
      chargeCreditCard(payment);
    } else if (type === 'paypal') {
      // process paypal
      chargePayPal(payment);
    } else if (type === 'crypto') {
      // process crypto
      chargeCrypto(payment);
    }
    // Adding new payment method requires modifying this class!
  }
}
```

**Problem:** Every new payment method requires modifying existing code.

### ✅ Better
```javascript
// Base interface (or abstract class)
class PaymentMethod {
  process(payment) {
    throw new Error('Must implement process()');
  }
}

// Extension 1
class CreditCardPayment extends PaymentMethod {
  process(payment) {
    chargeCreditCard(payment);
  }
}

// Extension 2
class PayPalPayment extends PaymentMethod {
  process(payment) {
    chargePayPal(payment);
  }
}

// Extension 3 - NEW, doesn't modify existing code
class CryptoPayment extends PaymentMethod {
  process(payment) {
    chargeCrypto(payment);
  }
}

// Processor doesn't need to change
class PaymentProcessor {
  processPayment(payment, paymentMethod) {
    paymentMethod.process(payment);
  }
}
```

**Benefits:**
- Add new payment methods without touching existing code
- Reduces risk of breaking existing functionality
- Follows "closed for modification, open for extension"

## L - Liskov Substitution Principle (LSP)

> "Objects should be replaceable with instances of their subtypes without altering program correctness."

**Meaning:** If B is a subtype of A, you should be able to use B anywhere you use A without breaking anything.

### ❌ Violation
```javascript
class Bird {
  fly() {
    console.log('Flying...');
  }
}

class Duck extends Bird {
  fly() {
    console.log('Duck flying');
  }
}

class Penguin extends Bird {
  fly() {
    throw new Error('Penguins cannot fly!');
  }
}

// This breaks LSP!
function makeBirdFly(bird) {
  bird.fly(); // Will throw error if bird is a Penguin
}
```

### ✅ Better
```javascript
class Bird {
  // Common bird behavior
}

class FlyingBird extends Bird {
  fly() {
    console.log('Flying...');
  }
}

class Duck extends FlyingBird {
  fly() {
    console.log('Duck flying');
  }
}

class Penguin extends Bird {
  swim() {
    console.log('Penguin swimming');
  }
}

// Now we can safely assume all FlyingBirds can fly
function makeBirdFly(bird) {
  if (bird instanceof FlyingBird) {
    bird.fly();
  }
}
```

**Real-world example:**
```javascript
// Violation
class Rectangle {
  setWidth(width) { this.width = width; }
  setHeight(height) { this.height = height; }
  getArea() { return this.width * this.height; }
}

class Square extends Rectangle {
  setWidth(width) {
    this.width = width;
    this.height = width; // Must keep square!
  }
  setHeight(height) {
    this.width = height;
    this.height = height;
  }
}

// This breaks!
function testRectangle(rect) {
  rect.setWidth(5);
  rect.setHeight(4);
  expect(rect.getArea()).toBe(20); // Fails for Square!
}
```

## I - Interface Segregation Principle (ISP)

> "No client should be forced to depend on methods it does not use."

**Meaning:** Many specific interfaces are better than one general-purpose interface.

### ❌ Violation
```javascript
class Worker {
  work() { }
  eat() { }
  sleep() { }
  getPaid() { }
}

class HumanWorker extends Worker {
  work() { console.log('Working'); }
  eat() { console.log('Eating'); }
  sleep() { console.log('Sleeping'); }
  getPaid() { console.log('Getting paid'); }
}

class RobotWorker extends Worker {
  work() { console.log('Working'); }
  eat() { throw new Error('Robots do not eat!'); }
  sleep() { throw new Error('Robots do not sleep!'); }
  getPaid() { throw new Error('Robots do not get paid!'); }
}
```

### ✅ Better
```javascript
// Split into specific interfaces
class Workable {
  work() { throw new Error('Must implement'); }
}

class Eatable {
  eat() { throw new Error('Must implement'); }
}

class Sleepable {
  sleep() { throw new Error('Must implement'); }
}

class Payable {
  getPaid() { throw new Error('Must implement'); }
}

// Implement only what's needed
class HumanWorker extends Workable, Eatable, Sleepable, Payable {
  work() { console.log('Working'); }
  eat() { console.log('Eating'); }
  sleep() { console.log('Sleeping'); }
  getPaid() { console.log('Getting paid'); }
}

class RobotWorker extends Workable {
  work() { console.log('Working'); }
  // Only implements what it needs!
}
```

**Benefits:**
- Classes only depend on methods they actually use
- More flexible
- Easier to understand

## D - Dependency Inversion Principle (DIP)

> "Depend on abstractions, not on concretions."

**Meaning:** High-level modules shouldn't depend on low-level modules. Both should depend on abstractions.

### ❌ Violation
```javascript
// Low-level module
class MySQLDatabase {
  save(data) {
    console.log('Saving to MySQL');
  }
}

// High-level module depends on concrete class
class UserService {
  constructor() {
    this.database = new MySQLDatabase(); // Tight coupling!
  }

  saveUser(user) {
    this.database.save(user);
  }
}

// Can't easily switch to PostgreSQL or MongoDB!
```

### ✅ Better
```javascript
// Abstraction
class Database {
  save(data) {
    throw new Error('Must implement');
  }
}

// Low-level implementations
class MySQLDatabase extends Database {
  save(data) {
    console.log('Saving to MySQL');
  }
}

class PostgreSQLDatabase extends Database {
  save(data) {
    console.log('Saving to PostgreSQL');
  }
}

class MongoDatabase extends Database {
  save(data) {
    console.log('Saving to MongoDB');
  }
}

// High-level module depends on abstraction
class UserService {
  constructor(database) {
    this.database = database; // Depends on interface!
  }

  saveUser(user) {
    this.database.save(user);
  }
}

// Easy to swap implementations
const service1 = new UserService(new MySQLDatabase());
const service2 = new UserService(new PostgreSQLDatabase());
const service3 = new UserService(new MongoDatabase());
```

**Benefits:**
- Easy to swap implementations
- Easy to test (inject mocks)
- Reduces coupling
- More flexible

## SOLID in Code Reviews

When reviewing code, ask:

### SRP Check
- [ ] Does this class have multiple reasons to change?
- [ ] Could I extract responsibilities into separate classes?

### OCP Check
- [ ] Will adding new features require modifying this code?
- [ ] Could we use polymorphism/strategy pattern instead of if/else?

### LSP Check
- [ ] Can I substitute subtypes without breaking behavior?
- [ ] Are child classes throwing unexpected errors?
- [ ] Do child classes properly extend parent behavior?

### ISP Check
- [ ] Are we forcing classes to implement unused methods?
- [ ] Should we split this interface into smaller ones?

### DIP Check
- [ ] Is this class depending on concrete implementations?
- [ ] Should we inject dependencies instead?
- [ ] Are we using `new` to create dependencies? (Code smell!)

## This Week's Practice

### Day 1-2: SRP
- [ ] Find one class doing multiple things
- [ ] Split it into focused classes
- [ ] Discuss: What were the responsibilities?

### Day 3: OCP
- [ ] Find one if/else chain for types
- [ ] Refactor using polymorphism
- [ ] Add a new type without modifying existing code

### Day 4: LSP
- [ ] Review your class hierarchies
- [ ] Check if subtypes can truly replace parents
- [ ] Fix any violations

### Day 5: ISP & DIP
- [ ] Find classes with unused interface methods
- [ ] Identify concrete dependencies
- [ ] Refactor to depend on abstractions

### Day 6-7: Review
- [ ] Review 3 PRs using SOLID principles
- [ ] Identify violations
- [ ] Suggest improvements with explanations

## Common Questions

**Q: Should I always follow SOLID strictly?**
A: No. SOLID are guidelines, not laws. Over-engineering simple code is worse than slightly violating SOLID. Use judgment.

**Q: When is it okay to violate SOLID?**
A:
- Prototypes and throwaway code
- Very simple applications
- When the cost of abstraction exceeds the benefit
- When following SOLID makes code more complex, not less

**Q: How do I know if I'm over-engineering?**
A: If you have interfaces with only one implementation and no plan for more, you might be over-engineering.

---

**Previous:** [[03 - Clean Code Principles]]
**Next:** [[05 - Refactoring Techniques]]
**Related:** [[06 - Design Patterns]]
