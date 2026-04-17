# 02 - Code Smells & Anti-Patterns

## What is a Code Smell?

A code smell is a surface indication that suggests a deeper problem in the code. It's not a bug, but a sign that the code might be hard to maintain, extend, or understand.

## Top Code Smells to Recognize

### 1. 🔴 Long Methods/Functions
**Smell:** Function longer than ~20-30 lines

**Why it's bad:**
- Hard to understand
- Hard to test
- Probably doing too many things

**Example:**
```javascript
// BAD: 100+ lines doing everything
function processUserOrder(order) {
  // validate order (20 lines)
  // check inventory (30 lines)
  // calculate pricing (25 lines)
  // process payment (40 lines)
  // send emails (20 lines)
  // update database (30 lines)
}

// GOOD: Single responsibility
function processUserOrder(order) {
  validateOrder(order);
  checkInventory(order);
  const total = calculatePricing(order);
  processPayment(order, total);
  sendConfirmationEmail(order);
  updateOrderDatabase(order);
}
```

**Practice:** Find one long function in your codebase this week and refactor it.

### 2. 🔴 Duplicate Code
**Smell:** Same code in multiple places

**Why it's bad:**
- Fix bugs in multiple places
- Inconsistent behavior
- Maintenance nightmare

**Refactoring:** Extract to a shared function/class

### 3. 🔴 Magic Numbers & Strings
**Smell:** Hardcoded values without explanation

```javascript
// BAD
if (user.age > 18 && status === 2) {
  discount = price * 0.15;
}

// GOOD
const LEGAL_AGE = 18;
const STATUS_PREMIUM = 2;
const PREMIUM_DISCOUNT_RATE = 0.15;

if (user.age > LEGAL_AGE && status === STATUS_PREMIUM) {
  discount = price * PREMIUM_DISCOUNT_RATE;
}
```

### 4. 🔴 Deep Nesting
**Smell:** More than 3 levels of indentation

```javascript
// BAD
if (user) {
  if (user.isActive) {
    if (user.hasPermission('write')) {
      if (document.isEditable) {
        // finally do something
      }
    }
  }
}

// GOOD: Guard clauses
if (!user) return;
if (!user.isActive) return;
if (!user.hasPermission('write')) return;
if (!document.isEditable) return;

// do something
```

### 5. 🔴 Large Classes (God Objects)
**Smell:** Class with too many responsibilities

**Signs:**
- Hundreds of lines
- Many dependencies
- Unclear purpose
- Hard to name clearly

**Solution:** Split into smaller, focused classes

### 6. 🔴 Long Parameter Lists
**Smell:** Function with 4+ parameters

```javascript
// BAD
function createUser(name, email, age, address, phone, role, department) {
  // ...
}

// GOOD: Use an object
function createUser({ name, email, age, address, phone, role, department }) {
  // ...
}

// BETTER: Use a User class/builder
const user = new UserBuilder()
  .withName(name)
  .withEmail(email)
  .withRole(role)
  .build();
```

### 7. 🔴 Primitive Obsession
**Smell:** Using primitives instead of small objects

```javascript
// BAD
function sendEmail(emailString) {
  if (!emailString.includes('@')) throw new Error('Invalid');
  // ...
}

// GOOD
class Email {
  constructor(value) {
    if (!value.includes('@')) throw new Error('Invalid email');
    this.value = value;
  }
}

function sendEmail(email: Email) {
  // email is guaranteed valid
}
```

### 8. 🔴 Feature Envy
**Smell:** Method uses another object's data more than its own

```javascript
// BAD
class Invoice {
  calculateTotal(customer) {
    return customer.getAddress().getCountry() === 'US'
      ? this.amount * 1.1
      : this.amount * 1.2;
  }
}

// GOOD: Behavior should live with data
class Customer {
  getTaxRate() {
    return this.address.country === 'US' ? 1.1 : 1.2;
  }
}

class Invoice {
  calculateTotal(customer) {
    return this.amount * customer.getTaxRate();
  }
}
```

### 9. 🔴 Shotgun Surgery
**Smell:** One change requires editing many classes

**Cause:** Related behavior scattered across codebase
**Solution:** Group related behavior together

### 10. 🔴 Comments That Explain What (Not Why)

```javascript
// BAD: Comment explains WHAT (code already shows this)
// Increment i by 1
i++;

// GOOD: Comment explains WHY
// Skip the first element as it's always the header
for (let i = 1; i < items.length; i++) {
  // ...
}
```

## Common Anti-Patterns

### 1. Copy-Paste Programming
Copying code without understanding it

### 2. Golden Hammer
Using the same solution for every problem ("I know X, so everything is an X problem")

### 3. Premature Optimization
Optimizing before you know it's a bottleneck

### 4. Not Invented Here (NIH)
Rejecting external solutions and rebuilding everything

### 5. Big Ball of Mud
No clear architecture, everything depends on everything

## Your Weekly Practice

### Day 1-2: Recognition
- [ ] Read through 3 files in your codebase
- [ ] Identify 3 code smells
- [ ] Understand why they're problematic

### Day 3-4: Small Refactors
- [ ] Fix magic numbers in one file
- [ ] Extract one long function
- [ ] Simplify one deeply nested block

### Day 5-7: Code Review Application
- [ ] Spot 2 code smells in PRs you review
- [ ] Suggest specific refactorings
- [ ] Explain the "why" in your comments

## Code Smell Checklist for Reviews

When reviewing code, ask:
- [ ] Any function longer than 30 lines?
- [ ] Any duplicate code I can spot?
- [ ] Any magic numbers/strings?
- [ ] Nesting deeper than 3 levels?
- [ ] Class doing too many things?
- [ ] Unclear naming?
- [ ] Comments explaining what instead of why?

---

**Previous:** [[01 - Code Review Fundamentals]]
**Next:** [[03 - Clean Code Principles]]
**Quick Reference:** [[Common Code Smells Quick Reference]]
