# 03 - Clean Code Principles

> "Any fool can write code that a computer can understand. Good programmers write code that humans can understand." - Martin Fowler

## Core Principles

### 1. Meaningful Names

Names should reveal intent without requiring comments.

```javascript
// BAD
const d; // elapsed time in days
function getData() { }
const list1;

// GOOD
const elapsedTimeInDays;
function getUserAccountDetails() { }
const activeCustomers;
```

**Rules:**
- Use pronounceable names: `generationTimestamp` not `genymdhms`
- Use searchable names: `MAX_STUDENTS_PER_CLASS` not `7`
- Avoid mental mapping: Don't use `i, j, k` if the loop is complex
- Class names: Nouns (`Customer`, `Account`)
- Function names: Verbs (`save`, `delete`, `calculateTotal`)

**This week:** Rename 5 poorly named variables/functions in your code.

### 2. Functions Should Do One Thing

**Single Responsibility Principle applied to functions.**

```javascript
// BAD: Does three things
function saveUserAndSendEmail(user) {
  validateUser(user);
  database.save(user);
  email.send(user.email, "Welcome!");
}

// GOOD: Each does one thing
function saveUser(user) {
  validateUser(user);
  database.save(user);
}

function sendWelcomeEmail(user) {
  email.send(user.email, "Welcome!");
}

function registerUser(user) {
  saveUser(user);
  sendWelcomeEmail(user);
}
```

**How to know if it does one thing:**
- Can you extract another function with a name that's not a restatement?
- Does the function do steps at multiple levels of abstraction?

### 3. Keep Functions Small

**Target: 5-15 lines**
**Max: 20-30 lines**

Small functions are:
- Easier to understand
- Easier to test
- Easier to reuse
- Easier to name

### 4. DRY (Don't Repeat Yourself)

Duplication is the root of evil in software.

**Every piece of knowledge should have a single, unambiguous representation.**

```javascript
// BAD: Duplicate validation logic
function createUser(data) {
  if (!data.email || !data.email.includes('@')) {
    throw new Error('Invalid email');
  }
  // ...
}

function updateUser(data) {
  if (!data.email || !data.email.includes('@')) {
    throw new Error('Invalid email');
  }
  // ...
}

// GOOD: Single source of truth
function validateEmail(email) {
  if (!email || !email.includes('@')) {
    throw new Error('Invalid email');
  }
}

function createUser(data) {
  validateEmail(data.email);
  // ...
}

function updateUser(data) {
  validateEmail(data.email);
  // ...
}
```

### 5. Proper Abstraction Levels

**Each function should operate at one level of abstraction.**

```javascript
// BAD: Mixing high and low level operations
function generateReport(data) {
  const report = [];
  // High level
  const summary = calculateSummary(data);

  // Low level - suddenly dealing with HTML
  report.push('<div class="header">');
  report.push('<h1>' + summary.title + '</h1>');
  report.push('</div>');

  return report.join('');
}

// GOOD: Consistent abstraction level
function generateReport(data) {
  const summary = calculateSummary(data);
  const header = createHeader(summary);
  const body = createBody(data);
  const footer = createFooter(summary);

  return assembleReport(header, body, footer);
}
```

### 6. Error Handling is One Thing

```javascript
// BAD: Mixed concerns
function processPayment(payment) {
  try {
    validatePayment(payment);
    const result = chargeCard(payment);
    updateDatabase(result);
    sendReceipt(payment);
  } catch (error) {
    log.error(error);
    // business logic mixed with error handling
  }
}

// GOOD: Separate error handling
function processPayment(payment) {
  validatePayment(payment);
  const result = chargeCard(payment);
  updateDatabase(result);
  sendReceipt(payment);
}

// Caller handles errors
try {
  processPayment(payment);
} catch (error) {
  handlePaymentError(error);
}
```

### 7. Minimize Dependencies

**Depend on abstractions, not concretions.**

```javascript
// BAD: Tightly coupled to specific email service
class UserService {
  registerUser(user) {
    // ...
    const sendgrid = new SendGridAPI();
    sendgrid.send(user.email, "Welcome");
  }
}

// GOOD: Depend on interface
class UserService {
  constructor(emailService) {
    this.emailService = emailService;
  }

  registerUser(user) {
    // ...
    this.emailService.send(user.email, "Welcome");
  }
}
```

### 8. Use Positive Conditionals

```javascript
// BAD: Double negatives are hard to read
if (!notLoggedIn) {
  // ...
}

// GOOD: Clear and positive
if (isLoggedIn) {
  // ...
}
```

### 9. Guard Clauses Over Nested IFs

```javascript
// BAD
function processOrder(order) {
  if (order) {
    if (order.items.length > 0) {
      if (order.customer) {
        // finally do work
      }
    }
  }
}

// GOOD
function processOrder(order) {
  if (!order) return;
  if (order.items.length === 0) return;
  if (!order.customer) return;

  // do work at base level
}
```

### 10. Comments Should Explain Why, Not What

```javascript
// BAD: States the obvious
// Check if age is greater than 18
if (age > 18) {
  // ...
}

// GOOD: Explains the reason
// Legal drinking age in the US
const LEGAL_DRINKING_AGE = 21;
if (age > LEGAL_DRINKING_AGE) {
  // ...
}

// GOOD: Explains non-obvious decision
// Using binary search instead of hash map because
// the data set is small (<100 items) and we need
// to maintain sort order for display
const result = binarySearch(data, target);
```

## The Boy Scout Rule

> "Leave the code better than you found it."

Every time you touch code:
- Fix one small thing
- Improve one name
- Extract one function
- Remove one piece of duplication

## Clean Code Checklist

Before submitting code, check:
- [ ] All names are clear and reveal intent
- [ ] Functions are small (<20 lines ideally)
- [ ] Each function does one thing
- [ ] No duplicate code
- [ ] Consistent abstraction levels
- [ ] Error handling is clean
- [ ] No magic numbers/strings
- [ ] Comments explain why, not what
- [ ] Positive conditionals used
- [ ] Guard clauses for early returns

## This Week's Practice

### Day 1: Names
- [ ] Find 10 unclear names in your code
- [ ] Rename them with clear, intention-revealing names
- [ ] Ensure they're pronounceable and searchable

### Day 2: Functions
- [ ] Find 3 functions longer than 30 lines
- [ ] Break them into smaller functions
- [ ] Ensure each does one thing

### Day 3: DRY
- [ ] Find duplicate code in your project
- [ ] Extract it into a shared function
- [ ] Reuse in all places

### Day 4: Abstraction
- [ ] Find one function mixing abstraction levels
- [ ] Refactor to maintain consistent level

### Day 5: Review
- [ ] Review a PR using this checklist
- [ ] Leave feedback on any violations
- [ ] Explain the principle behind your feedback

## Recommended Reading

- **Clean Code** by Robert C. Martin (Chapters 1-3)
- **The Art of Readable Code** by Boswell & Foucher

---

**Previous:** [[02 - Code Smells & Anti-Patterns]]
**Next:** [[04 - SOLID Principles]]
**Related:** [[Code Quality Checklist]]
