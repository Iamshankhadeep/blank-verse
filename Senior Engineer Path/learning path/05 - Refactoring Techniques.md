# 05 - Refactoring Techniques

Refactoring is the art of improving code structure without changing its behavior. Senior engineers refactor continuously to keep code maintainable.

## What is Refactoring?

> "Refactoring is a disciplined technique for restructuring existing code, altering its internal structure without changing its external behavior." - Martin Fowler

**Refactoring is NOT:**
- ❌ Rewriting from scratch
- ❌ Adding features
- ❌ Fixing bugs
- ❌ "Cleaning up" without tests

**Refactoring IS:**
- ✅ Small, safe transformations
- ✅ Improving design
- ✅ Making code easier to understand
- ✅ Making future changes easier

## When to Refactor

### The Rule of Three

First time: Just do it
Second time: Notice duplication but let it be
Third time: Refactor

### Refactor When:

1. **Before adding a feature**
   - Make the change easy, then make the easy change

2. **During code review**
   - Clean up code you touch

3. **When fixing bugs**
   - Often reveals design issues

4. **When you're confused**
   - If code is hard to understand, it's hard to maintain

### DON'T Refactor When:

- ❌ Under tight deadline (schedule explicit time)
- ❌ Without tests
- ❌ Code that rarely changes
- ❌ Before understanding the domain

## Essential Refactoring Techniques

### 1. Extract Method/Function

**Problem:** Long function doing multiple things

**Before:**
```javascript
function displayOrderDetails(order) {
  console.log('===== Order Details =====');
  console.log(`Order ID: ${order.id}`);
  console.log(`Customer: ${order.customerName}`);
  console.log(`Email: ${order.email}`);
  console.log('');
  console.log('Items:');
  let total = 0;
  order.items.forEach(item => {
    const subtotal = item.price * item.quantity;
    total += subtotal;
    console.log(`  ${item.name} x${item.quantity}: $${subtotal}`);
  });
  console.log('');
  const tax = total * 0.08;
  const grandTotal = total + tax;
  console.log(`Subtotal: $${total}`);
  console.log(`Tax (8%): $${tax}`);
  console.log(`Total: $${grandTotal}`);
}
```

**After:**
```javascript
function displayOrderDetails(order) {
  displayOrderHeader(order);
  displayOrderItems(order);
  displayOrderTotal(order);
}

function displayOrderHeader(order) {
  console.log('===== Order Details =====');
  console.log(`Order ID: ${order.id}`);
  console.log(`Customer: ${order.customerName}`);
  console.log(`Email: ${order.email}`);
  console.log('');
}

function displayOrderItems(order) {
  console.log('Items:');
  order.items.forEach(item => {
    const subtotal = calculateItemSubtotal(item);
    console.log(`  ${item.name} x${item.quantity}: $${subtotal}`);
  });
  console.log('');
}

function displayOrderTotal(order) {
  const total = calculateOrderTotal(order);
  const tax = calculateTax(total);
  const grandTotal = total + tax;

  console.log(`Subtotal: $${total}`);
  console.log(`Tax (8%): $${tax}`);
  console.log(`Total: $${grandTotal}`);
}

function calculateItemSubtotal(item) {
  return item.price * item.quantity;
}

function calculateOrderTotal(order) {
  return order.items.reduce((sum, item) =>
    sum + calculateItemSubtotal(item), 0
  );
}

function calculateTax(amount) {
  const TAX_RATE = 0.08;
  return amount * TAX_RATE;
}
```

**Benefits:**
- Each function does one thing
- Easy to test
- Reusable pieces
- Self-documenting

### 2. Rename Variable/Function/Class

**Problem:** Unclear names

**Before:**
```javascript
function calc(d) {
  const r = d > 100 ? 0.2 : 0.15;
  return d * r;
}
```

**After:**
```javascript
function calculateDiscount(purchaseAmount) {
  const BULK_THRESHOLD = 100;
  const BULK_DISCOUNT_RATE = 0.2;
  const STANDARD_DISCOUNT_RATE = 0.15;

  const discountRate = purchaseAmount > BULK_THRESHOLD
    ? BULK_DISCOUNT_RATE
    : STANDARD_DISCOUNT_RATE;

  return purchaseAmount * discountRate;
}
```

### 3. Introduce Parameter Object

**Problem:** Long parameter list

**Before:**
```javascript
function createUser(
  firstName,
  lastName,
  email,
  phone,
  address,
  city,
  state,
  zipCode,
  country
) {
  // ...
}

createUser('John', 'Doe', 'john@example.com', '555-0100',
           '123 Main St', 'Springfield', 'IL', '62701', 'USA');
```

**After:**
```javascript
class User {
  constructor(personalInfo, contactInfo, address) {
    this.personalInfo = personalInfo;
    this.contactInfo = contactInfo;
    this.address = address;
  }
}

function createUser({ personalInfo, contactInfo, address }) {
  return new User(personalInfo, contactInfo, address);
}

createUser({
  personalInfo: { firstName: 'John', lastName: 'Doe' },
  contactInfo: { email: 'john@example.com', phone: '555-0100' },
  address: {
    street: '123 Main St',
    city: 'Springfield',
    state: 'IL',
    zipCode: '62701',
    country: 'USA'
  }
});
```

### 4. Replace Magic Number with Constant

**Before:**
```javascript
function calculateShipping(weight) {
  if (weight > 50) {
    return weight * 0.75;
  }
  return weight * 1.25;
}
```

**After:**
```javascript
const HEAVY_PACKAGE_THRESHOLD = 50;
const HEAVY_PACKAGE_RATE = 0.75;
const STANDARD_PACKAGE_RATE = 1.25;

function calculateShipping(weight) {
  if (weight > HEAVY_PACKAGE_THRESHOLD) {
    return weight * HEAVY_PACKAGE_RATE;
  }
  return weight * STANDARD_PACKAGE_RATE;
}
```

### 5. Replace Conditional with Polymorphism

**Problem:** Type checking with if/else

**Before:**
```javascript
class Bird {
  constructor(type) {
    this.type = type;
  }

  fly() {
    if (this.type === 'duck') {
      return 'Duck flying';
    } else if (this.type === 'eagle') {
      return 'Eagle soaring';
    } else if (this.type === 'penguin') {
      throw new Error('Penguins cannot fly');
    }
  }
}
```

**After:**
```javascript
class Bird {
  fly() {
    throw new Error('Must implement fly()');
  }
}

class Duck extends Bird {
  fly() {
    return 'Duck flying';
  }
}

class Eagle extends Bird {
  fly() {
    return 'Eagle soaring';
  }
}

class Penguin extends Bird {
  swim() {
    return 'Penguin swimming';
  }
}
```

### 6. Decompose Conditional

**Problem:** Complex conditionals

**Before:**
```javascript
if (date.before(SUMMER_START) || date.after(SUMMER_END)) {
  charge = quantity * winterRate + winterServiceCharge;
} else {
  charge = quantity * summerRate;
}
```

**After:**
```javascript
const charge = isWinter(date)
  ? calculateWinterCharge(quantity)
  : calculateSummerCharge(quantity);

function isWinter(date) {
  return date.before(SUMMER_START) || date.after(SUMMER_END);
}

function calculateWinterCharge(quantity) {
  return quantity * winterRate + winterServiceCharge;
}

function calculateSummerCharge(quantity) {
  return quantity * summerRate;
}
```

### 7. Consolidate Duplicate Conditional Fragments

**Before:**
```javascript
if (isSpecialDeal()) {
  total = price * 0.95;
  send();
} else {
  total = price * 0.98;
  send();
}
```

**After:**
```javascript
total = isSpecialDeal()
  ? price * 0.95
  : price * 0.98;
send();
```

### 8. Remove Dead Code

**Before:**
```javascript
function processOrder(order) {
  validateOrder(order);
  // calculateDiscount(order); // Not used anymore
  // const oldTotal = order.total; // Debugging leftover
  processPayment(order);
}
```

**After:**
```javascript
function processOrder(order) {
  validateOrder(order);
  processPayment(order);
}
```

### 9. Replace Nested Conditionals with Guard Clauses

**Before:**
```javascript
function getPaymentAmount(employee) {
  let result;
  if (employee.isSeparated) {
    result = { amount: 0, reason: 'separated' };
  } else {
    if (employee.isRetired) {
      result = { amount: 0, reason: 'retired' };
    } else {
      // Complex payment logic
      result = { amount: calculatePayment(employee), reason: 'active' };
    }
  }
  return result;
}
```

**After:**
```javascript
function getPaymentAmount(employee) {
  if (employee.isSeparated) {
    return { amount: 0, reason: 'separated' };
  }

  if (employee.isRetired) {
    return { amount: 0, reason: 'retired' };
  }

  return { amount: calculatePayment(employee), reason: 'active' };
}
```

### 10. Split Variable

**Problem:** Variable used for multiple purposes

**Before:**
```javascript
let temp = 2 * (height + width);
console.log(temp);
temp = height * width;
console.log(temp);
```

**After:**
```javascript
const perimeter = 2 * (height + width);
console.log(perimeter);

const area = height * width;
console.log(area);
```

## The Refactoring Process

### Step 1: Ensure Tests Exist
```javascript
// Write tests first if they don't exist
test('calculateDiscount returns correct amount', () => {
  expect(calculateDiscount(50)).toBe(7.5);
  expect(calculateDiscount(150)).toBe(30);
});
```

### Step 2: Make Small Changes
```javascript
// Change ONE thing at a time
// Extract one method
// Rename one variable
// Remove one piece of duplication
```

### Step 3: Run Tests
```bash
npm test
# Tests should still pass
```

### Step 4: Commit
```bash
git add .
git commit -m "refactor: extract calculateDiscount method"
```

### Step 5: Repeat
Small steps, always green tests.

## Refactoring Patterns

### Pattern: Extract Class

**When:** Class doing too much

**Before:**
```javascript
class Person {
  constructor(name, officeCode, officeNumber) {
    this.name = name;
    this.officeCode = officeCode;
    this.officeNumber = officeNumber;
  }

  getOfficePhone() {
    return `(${this.officeCode}) ${this.officeNumber}`;
  }
}
```

**After:**
```javascript
class TelephoneNumber {
  constructor(areaCode, number) {
    this.areaCode = areaCode;
    this.number = number;
  }

  toString() {
    return `(${this.areaCode}) ${this.number}`;
  }
}

class Person {
  constructor(name, officePhone) {
    this.name = name;
    this.officePhone = officePhone;
  }

  getOfficePhone() {
    return this.officePhone.toString();
  }
}
```

### Pattern: Inline Function

**When:** Function body is as clear as the name

**Before:**
```javascript
function getRating(driver) {
  return moreThanFiveLateDeliveries(driver) ? 2 : 1;
}

function moreThanFiveLateDeliveries(driver) {
  return driver.numberOfLateDeliveries > 5;
}
```

**After:**
```javascript
function getRating(driver) {
  return driver.numberOfLateDeliveries > 5 ? 2 : 1;
}
```

### Pattern: Replace Loop with Pipeline

**Before:**
```javascript
const names = [];
for (const person of people) {
  if (person.age >= 18) {
    names.push(person.name);
  }
}
```

**After:**
```javascript
const names = people
  .filter(person => person.age >= 18)
  .map(person => person.name);
```

## Code Smells → Refactoring Map

| Code Smell | Refactoring Technique |
|------------|----------------------|
| Long Method | Extract Method |
| Large Class | Extract Class |
| Long Parameter List | Introduce Parameter Object |
| Duplicate Code | Extract Method, Pull Up Method |
| Magic Numbers | Replace Magic Number with Constant |
| Unclear Names | Rename Variable/Method/Class |
| Deep Nesting | Replace Nested Conditional with Guard Clauses |
| Feature Envy | Move Method |
| Data Clumps | Introduce Parameter Object, Extract Class |
| Switch Statements | Replace Conditional with Polymorphism |

## Refactoring Safety Net

### Before Refactoring

- [ ] Tests exist and pass
- [ ] Understand the code
- [ ] One refactoring at a time
- [ ] Commit after each safe step

### During Refactoring

- [ ] Make small changes
- [ ] Run tests frequently
- [ ] Don't change behavior
- [ ] Commit working states

### After Refactoring

- [ ] All tests pass
- [ ] Code is clearer
- [ ] No new bugs introduced
- [ ] Team reviews changes

## This Week's Practice

### Day 1: Identify
- [ ] Find one long method in your codebase
- [ ] Identify what it's doing
- [ ] Plan how to extract methods

### Day 2: Extract
- [ ] Write tests if missing
- [ ] Extract 2-3 methods
- [ ] Run tests after each extraction

### Day 3: Names
- [ ] Find 5 poorly named things
- [ ] Rename them
- [ ] Use IDE refactoring tools

### Day 4: Conditionals
- [ ] Find nested conditionals
- [ ] Replace with guard clauses
- [ ] Extract complex conditions to functions

### Day 5: Review
- [ ] In PR reviews, suggest refactorings
- [ ] Point out code smells
- [ ] Suggest specific techniques

## Refactoring Tools

### IDE Support
- **VS Code:** F2 (rename), Extract Method
- **IntelliJ/WebStorm:** Ctrl+Alt+M (extract method), Shift+F6 (rename)
- **Eclipse:** Alt+Shift+R (rename), Alt+Shift+M (extract)

### Automated Refactoring
- Use IDE refactoring features (safer than manual)
- Run tests after each change
- Use version control

## Common Mistakes

### ❌ Refactoring Without Tests
Always have tests before refactoring.

### ❌ Refactoring and Adding Features Together
Separate commits: refactor first, then add feature.

### ❌ Big Bang Refactoring
Small, incremental changes are safer.

### ❌ Refactoring Because "It's Ugly"
Refactor to make future changes easier, not for aesthetics.

### ❌ Perfectionism
Good enough is good enough. Ship it.

## Remember

> "Make the change easy, then make the easy change." - Kent Beck

**Key principles:**
- Refactor in small steps
- Always keep tests green
- Commit frequently
- Make behavior-preserving changes
- Stop when good enough

---

**Previous:** [[04 - SOLID Principles]]
**Next:** [[06 - Design Patterns]]
**Related:** [[02 - Code Smells & Anti-Patterns]], [[03 - Clean Code Principles]]
