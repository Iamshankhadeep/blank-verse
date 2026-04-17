# 07 - Testing Strategies

Senior engineers write tests that catch bugs, not just achieve coverage. They know what to test, how to test it, and when tests add value.

## The Testing Pyramid

```
        /\
       /  \        E2E Tests (Few)
      /____\       - Slow, brittle, expensive
     /      \      - Test critical user flows
    /________\
   /          \    Integration Tests (Some)
  /____________\   - Test component interactions
 /              \
/________________\ Unit Tests (Many)
                   - Fast, focused, cheap
                   - Test individual functions
```

**The Rule:** Lots of unit tests, some integration tests, few E2E tests.

## What Makes a Good Test?

Good tests are **FIRST**:

- **F**ast - Runs in milliseconds
- **I**ndependent - Doesn't depend on other tests
- **R**epeatable - Same result every time
- **S**elf-validating - Pass or fail, no manual checking
- **T**imely - Written before/with the code

## Unit Testing

### What to Test

✅ **DO test:**
- Business logic
- Edge cases
- Error handling
- Complex conditions
- Data transformations

❌ **DON'T test:**
- Trivial getters/setters
- Third-party libraries
- Framework code
- Database queries (use integration tests)

### Example: Good Unit Test

```javascript
// Function to test
function calculateShipping(weight, distance, isPremium) {
  if (weight <= 0) {
    throw new Error('Weight must be positive');
  }

  const baseRate = 5;
  const weightRate = 0.5;
  const distanceRate = 0.1;
  const premiumDiscount = 0.2;

  let cost = baseRate + (weight * weightRate) + (distance * distanceRate);

  if (isPremium) {
    cost *= (1 - premiumDiscount);
  }

  return parseFloat(cost.toFixed(2));
}

// Tests
describe('calculateShipping', () => {
  test('calculates basic shipping cost', () => {
    const cost = calculateShipping(10, 100, false);
    expect(cost).toBe(20); // 5 + (10 * 0.5) + (100 * 0.1)
  });

  test('applies premium discount', () => {
    const cost = calculateShipping(10, 100, true);
    expect(cost).toBe(16); // 20 * 0.8
  });

  test('throws error for zero weight', () => {
    expect(() => calculateShipping(0, 100, false))
      .toThrow('Weight must be positive');
  });

  test('throws error for negative weight', () => {
    expect(() => calculateShipping(-5, 100, false))
      .toThrow('Weight must be positive');
  });

  test('handles decimal weights', () => {
    const cost = calculateShipping(5.5, 100, false);
    expect(cost).toBe(17.75); // 5 + (5.5 * 0.5) + (100 * 0.1)
  });

  test('rounds to 2 decimal places', () => {
    const cost = calculateShipping(5, 101, false);
    expect(cost).toBe(17.60); // Not 17.599999
  });
});
```

### Test Structure: AAA Pattern

```javascript
test('description', () => {
  // Arrange - Set up test data
  const user = { name: 'John', age: 25 };
  const validator = new UserValidator();

  // Act - Execute the code being tested
  const result = validator.validate(user);

  // Assert - Verify the result
  expect(result.isValid).toBe(true);
  expect(result.errors).toHaveLength(0);
});
```

### Test Naming

**Bad names:**
```javascript
test('test1', () => { });
test('it works', () => { });
test('user', () => { });
```

**Good names:**
```javascript
test('returns true when email is valid', () => { });
test('throws error when password is less than 8 characters', () => { });
test('calculates discount correctly for premium users', () => { });
```

## Integration Testing

Test how components work together.

```javascript
// Integration test - Database + Service
describe('UserService Integration', () => {
  let database;
  let userService;

  beforeEach(async () => {
    // Set up test database
    database = await createTestDatabase();
    userService = new UserService(database);
  });

  afterEach(async () => {
    // Clean up
    await database.close();
  });

  test('creates user and stores in database', async () => {
    const userData = {
      name: 'John Doe',
      email: 'john@example.com'
    };

    const user = await userService.createUser(userData);

    // Verify user was created
    expect(user.id).toBeDefined();

    // Verify user is in database
    const stored = await database.users.findById(user.id);
    expect(stored.name).toBe('John Doe');
    expect(stored.email).toBe('john@example.com');
  });

  test('prevents duplicate emails', async () => {
    await userService.createUser({
      name: 'John',
      email: 'john@example.com'
    });

    await expect(
      userService.createUser({
        name: 'Jane',
        email: 'john@example.com'
      })
    ).rejects.toThrow('Email already exists');
  });
});
```

## E2E (End-to-End) Testing

Test complete user flows.

```javascript
// E2E test - Full user registration flow
describe('User Registration Flow', () => {
  test('user can register and login', async () => {
    // Navigate to registration page
    await page.goto('http://localhost:3000/register');

    // Fill in registration form
    await page.fill('[name="email"]', 'test@example.com');
    await page.fill('[name="password"]', 'SecurePass123');
    await page.fill('[name="confirmPassword"]', 'SecurePass123');
    await page.click('button[type="submit"]');

    // Should see success message
    await expect(page.locator('.success-message'))
      .toContainText('Registration successful');

    // Should redirect to login
    await expect(page).toHaveURL('http://localhost:3000/login');

    // Should be able to login
    await page.fill('[name="email"]', 'test@example.com');
    await page.fill('[name="password"]', 'SecurePass123');
    await page.click('button[type="submit"]');

    // Should see dashboard
    await expect(page).toHaveURL('http://localhost:3000/dashboard');
    await expect(page.locator('h1')).toContainText('Welcome');
  });
});
```

## Test Doubles

### Mocks vs Stubs vs Spies

**Stub:** Returns canned response
```javascript
const emailService = {
  send: () => true // Always returns true
};
```

**Mock:** Verifies it was called correctly
```javascript
const emailService = {
  send: jest.fn()
};

userService.register(user);

expect(emailService.send).toHaveBeenCalledWith(
  'john@example.com',
  'Welcome!'
);
```

**Spy:** Watches real implementation
```javascript
const emailService = new EmailService();
const spy = jest.spyOn(emailService, 'send');

userService.register(user);

expect(spy).toHaveBeenCalled();
spy.mockRestore();
```

### When to Use Each

**Stub:** Fast, predictable tests
```javascript
test('processes payment when gateway is available', () => {
  const gatewayStub = {
    charge: () => ({ success: true, transactionId: '123' })
  };

  const result = processPayment(100, gatewayStub);
  expect(result.success).toBe(true);
});
```

**Mock:** Verify interactions
```javascript
test('sends confirmation email after payment', () => {
  const emailMock = { send: jest.fn() };

  processPayment(100, paymentGateway, emailMock);

  expect(emailMock.send).toHaveBeenCalledWith(
    expect.objectContaining({
      subject: 'Payment Confirmation'
    })
  );
});
```

## Testing Best Practices

### 1. Test Behavior, Not Implementation

**❌ Bad (tests implementation):**
```javascript
test('uses bubble sort algorithm', () => {
  const spy = jest.spyOn(sorter, 'bubbleSort');
  sorter.sort([3, 1, 2]);
  expect(spy).toHaveBeenCalled();
});
```

**✅ Good (tests behavior):**
```javascript
test('sorts array in ascending order', () => {
  const result = sorter.sort([3, 1, 2]);
  expect(result).toEqual([1, 2, 3]);
});
```

### 2. One Assertion Per Test (Usually)

**❌ Bad (too many assertions):**
```javascript
test('user validation', () => {
  expect(validator.isValidEmail('test@test.com')).toBe(true);
  expect(validator.isValidEmail('invalid')).toBe(false);
  expect(validator.isValidPassword('short')).toBe(false);
  expect(validator.isValidPassword('LongPassword123')).toBe(true);
});
```

**✅ Good (focused tests):**
```javascript
test('accepts valid email format', () => {
  expect(validator.isValidEmail('test@test.com')).toBe(true);
});

test('rejects email without @', () => {
  expect(validator.isValidEmail('invalid')).toBe(false);
});

test('rejects password shorter than 8 characters', () => {
  expect(validator.isValidPassword('short')).toBe(false);
});

test('accepts password with 8+ characters', () => {
  expect(validator.isValidPassword('LongPassword123')).toBe(true);
});
```

### 3. Keep Tests DRY (But Not Too DRY)

**Use setup/teardown:**
```javascript
describe('UserService', () => {
  let userService;
  let mockDatabase;

  beforeEach(() => {
    mockDatabase = createMockDatabase();
    userService = new UserService(mockDatabase);
  });

  test('creates user', async () => {
    const user = await userService.create({ name: 'John' });
    expect(user.name).toBe('John');
  });

  test('finds user by id', async () => {
    const created = await userService.create({ name: 'John' });
    const found = await userService.findById(created.id);
    expect(found.name).toBe('John');
  });
});
```

**But don't abstract too much:**
```javascript
// ❌ Too abstract - hard to understand
test('scenario A', () => {
  runTestScenario('A', [1, 2, 3], expect.true);
});

// ✅ Clear and explicit
test('returns true when all items are positive', () => {
  const result = hasAllPositive([1, 2, 3]);
  expect(result).toBe(true);
});
```

### 4. Test Edge Cases

```javascript
describe('calculateDiscount', () => {
  // Happy path
  test('calculates 10% discount for regular users', () => {
    expect(calculateDiscount(100, 'regular')).toBe(10);
  });

  // Edge cases
  test('handles zero amount', () => {
    expect(calculateDiscount(0, 'regular')).toBe(0);
  });

  test('handles negative amount', () => {
    expect(() => calculateDiscount(-100, 'regular'))
      .toThrow('Amount must be positive');
  });

  test('handles unknown user type', () => {
    expect(calculateDiscount(100, 'unknown')).toBe(0);
  });

  test('handles very large amounts', () => {
    expect(calculateDiscount(1000000, 'premium')).toBe(200000);
  });

  test('handles decimal amounts', () => {
    expect(calculateDiscount(99.99, 'regular')).toBe(9.999);
  });
});
```

### 5. Avoid Test Interdependence

**❌ Bad (tests depend on each other):**
```javascript
let userId;

test('creates user', async () => {
  const user = await createUser({ name: 'John' });
  userId = user.id; // Shared state!
  expect(user.name).toBe('John');
});

test('updates user', async () => {
  await updateUser(userId, { name: 'Jane' }); // Depends on previous test
  const user = await getUser(userId);
  expect(user.name).toBe('Jane');
});
```

**✅ Good (independent tests):**
```javascript
test('creates user', async () => {
  const user = await createUser({ name: 'John' });
  expect(user.name).toBe('John');
});

test('updates user', async () => {
  const user = await createUser({ name: 'John' });
  await updateUser(user.id, { name: 'Jane' });
  const updated = await getUser(user.id);
  expect(updated.name).toBe('Jane');
});
```

## TDD (Test-Driven Development)

**Red → Green → Refactor**

### Step 1: Red (Write failing test)
```javascript
test('calculates total price with tax', () => {
  const cart = new ShoppingCart();
  cart.addItem({ price: 100 });
  expect(cart.getTotalWithTax()).toBe(108); // 8% tax
});

// Test fails - function doesn't exist yet
```

### Step 2: Green (Make it pass)
```javascript
class ShoppingCart {
  constructor() {
    this.items = [];
  }

  addItem(item) {
    this.items.push(item);
  }

  getTotalWithTax() {
    const subtotal = this.items.reduce((sum, item) => sum + item.price, 0);
    return subtotal * 1.08;
  }
}

// Test passes
```

### Step 3: Refactor (Improve code)
```javascript
class ShoppingCart {
  constructor() {
    this.items = [];
    this.TAX_RATE = 0.08;
  }

  addItem(item) {
    this.items.push(item);
  }

  getSubtotal() {
    return this.items.reduce((sum, item) => sum + item.price, 0);
  }

  getTax() {
    return this.getSubtotal() * this.TAX_RATE;
  }

  getTotalWithTax() {
    return this.getSubtotal() + this.getTax();
  }
}

// Tests still pass, code is cleaner
```

## Test Coverage

**Coverage is a metric, not a goal.**

### What Coverage Means

- **Line coverage:** % of lines executed
- **Branch coverage:** % of if/else branches taken
- **Function coverage:** % of functions called

### Coverage Guidelines

- 🎯 Aim for 80%+ coverage
- ✅ 100% coverage of critical paths
- ⚠️ Don't obsess over 100% total coverage
- 🚫 Coverage ≠ quality

**Example:**
```javascript
// 100% coverage but useless test
test('adds numbers', () => {
  add(2, 2); // No assertion!
});

// Lower coverage but meaningful test
test('adds numbers correctly', () => {
  expect(add(2, 2)).toBe(4);
  expect(add(-1, 1)).toBe(0);
});
```

## Testing Anti-Patterns

### 1. Testing Too Much
```javascript
// ❌ Testing framework/library
test('Array.push adds element', () => {
  const arr = [];
  arr.push(1);
  expect(arr.length).toBe(1);
});
```

### 2. Fragile Tests
```javascript
// ❌ Breaks when implementation changes
test('sorts using quicksort', () => {
  const spy = jest.spyOn(sorter, 'quickSort');
  sorter.sort([3, 1, 2]);
  expect(spy).toHaveBeenCalled();
});

// ✅ Tests behavior, not implementation
test('sorts array in ascending order', () => {
  expect(sorter.sort([3, 1, 2])).toEqual([1, 2, 3]);
});
```

### 3. Slow Tests
```javascript
// ❌ Slow (real HTTP calls)
test('fetches user data', async () => {
  const data = await fetch('https://api.example.com/user/1');
  expect(data.name).toBe('John');
});

// ✅ Fast (mocked)
test('fetches user data', async () => {
  fetch.mockResolvedValue({ name: 'John' });
  const data = await fetchUser(1);
  expect(data.name).toBe('John');
});
```

## Code Review: Testing Checklist

When reviewing tests:

- [ ] Tests are clear and readable
- [ ] Edge cases are covered
- [ ] No test interdependence
- [ ] Fast execution (<100ms per test)
- [ ] Tests behavior, not implementation
- [ ] Good test names
- [ ] No excessive mocking
- [ ] Assertions are meaningful

## This Week's Practice

### Day 1-2: Write Tests
- [ ] Add tests to an untested feature
- [ ] Aim for 80%+ coverage
- [ ] Test happy path + edge cases

### Day 3-4: TDD
- [ ] Write one feature using TDD
- [ ] Red → Green → Refactor
- [ ] Notice the difference

### Day 5-7: Review
- [ ] Review test quality in PRs
- [ ] Suggest missing test cases
- [ ] Point out brittle tests

## Remember

> "Tests are a safety net, not a goal. Write tests that give you confidence to change code."

**Key principles:**
- Test behavior, not implementation
- Keep tests fast and focused
- Coverage is a guide, not a target
- TDD when it helps
- Tests should make coding easier, not harder

---

**Previous:** [[06 - Design Patterns]]
**Next:** [[08 - Security in Code Reviews]]
**Related:** [[Code Quality Checklist]]
