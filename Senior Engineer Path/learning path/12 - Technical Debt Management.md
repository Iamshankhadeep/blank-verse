# 12 - Technical Debt Management

Technical debt isn't inherently bad—it's a tool. Senior engineers know when to incur debt, when to pay it down, and how to communicate the costs.

## What is Technical Debt?

> "Shipping first-time code is like going into debt. A little debt speeds development so long as it is paid back promptly with refactoring." - Ward Cunningham

**Technical debt is:**
- ✅ Shortcuts taken knowingly to ship faster
- ✅ Trade-off between speed and quality
- ✅ Investment in future maintainability

**Technical debt is NOT:**
- ❌ Sloppy code
- ❌ Lack of skill
- ❌ Excuse for poor quality

## Types of Technical Debt

### 1. Deliberate & Prudent (Good debt)

**Example:** "We need to ship by Friday. Let's hardcode these values for now and refactor next sprint."

```javascript
// TODO: Move to configuration system (Ticket #123)
const TAX_RATES = {
  'US': 0.08,
  'CA': 0.13,
  'UK': 0.20
};
```

**Characteristics:**
- ✅ Conscious decision
- ✅ Documented
- ✅ Scheduled for payback
- ✅ Acceptable trade-off

### 2. Deliberate & Reckless (Dangerous debt)

**Example:** "We don't have time for tests, just ship it."

```javascript
// No tests, no error handling, hardcoded values
function processPayment(amount) {
  fetch('http://payment-api.com/charge', {
    method: 'POST',
    body: { amount: amount, key: 'sk_live_abc123' }  // Key in code!
  });
}
```

**Characteristics:**
- ⚠️ Shortcuts that create serious issues
- ⚠️ Security risks
- ⚠️ Data loss potential
- ❌ Should be avoided

### 3. Inadvertent & Prudent (Learning debt)

**Example:** "Now we know how we should have done it."

After shipping:
- Learned a better pattern
- Discovered a better library
- Understood the domain better

**Characteristics:**
- ✅ Natural part of learning
- ✅ Refactor when touching the code
- ✅ Document the better approach

### 4. Inadvertent & Reckless (Worst kind)

**Example:** "What's layered architecture?"

```javascript
// Controller directly manipulating database
class UserController {
  async createUser(req, res) {
    // Validation in controller
    if (!req.body.email.includes('@')) {
      return res.status(400).send('Invalid email');
    }

    // Direct database access
    const id = Math.random(); // "Unique" ID
    await db.query(
      `INSERT INTO users VALUES ('${id}', '${req.body.email}', '${req.body.password}')`
      // SQL injection, plaintext password, everything wrong
    );

    // Business logic in controller
    if (req.body.isPremium) {
      await db.query(`INSERT INTO premium_users VALUES ('${id}')`);
    }

    res.send('OK');
  }
}
```

**Characteristics:**
- ❌ Lack of knowledge
- ❌ No design
- ❌ Creates cascading problems
- 🚨 Requires immediate attention

## The Technical Debt Quadrant

```
         Prudent
           │
   Deliberate  │  Inadvertent
   "Ship now,  │  "Now we
   fix later"  │   know better"
───────────────┼───────────────
   "No time    │  "What's
   for design" │   design?"
           │
        Reckless
```

## Measuring Technical Debt

### 1. Interest Rate (Pain)

How much does this debt cost us?

**Low interest:**
- Rarely touched code
- Isolated component
- Doesn't block new features

**High interest:**
- Changes frequently
- Multiple dependencies
- Slows down every feature

### 2. Principal (Effort to fix)

How much work to pay it back?

**Small:**
- Hours to days
- Single file/component
- Clear solution

**Large:**
- Weeks to months
- System-wide changes
- Unclear path forward

### 3. Priority Matrix

```
High Interest │
     ↑        │  PAY NOW    │  PLAN TO PAY
     │        │             │
     │        ├─────────────┼──────────────
     │        │             │
     │        │  MONITOR    │  LEAVE FOR NOW
Low Interest │             │
             └─────────────┴──────────────→
            Small Principal  Large Principal
```

**Pay Now:** High pain, easy fix
**Plan to Pay:** High pain, needs planning
**Monitor:** Low pain, watch for changes
**Leave:** Low pain, expensive fix

## Managing Technical Debt

### 1. Make Debt Visible

**Create debt tickets:**
```markdown
# [TECH-DEBT] Refactor UserService

## Context
UserService has grown to 1500 lines and handles authentication,
authorization, validation, email sending, and database operations.

## Problem
- Hard to test
- Changes affect multiple concerns
- Violates SRP
- New features take longer

## Proposal
Extract into:
- UserService (core domain logic)
- UserRepository (database)
- UserValidator (validation)
- EmailService (notifications)
- AuthService (authentication)

## Effort: 3-5 days
## Impact: Reduces feature time by ~30%
## Priority: Medium (touch frequently but manageable)
```

### 2. Track Debt Over Time

```javascript
// Add TODO comments with context
// TODO(debt): Extract validation logic (TECH-123)
// Created: 2024-03-15
// Impact: Medium - slows down form changes
// Effort: 1 day

// HACK: Temporary workaround for API limitation (TECH-124)
// Created: 2024-03-10
// Impact: High - breaks when API updates
// Effort: 2 days, waiting for API team
```

### 3. Include Debt in Planning

**Each sprint:**
- 70-80% features
- 20-30% debt paydown + bugs

**Quarterly:**
- Review debt backlog
- Prioritize highest interest debt
- Schedule debt-focused sprint if needed

### 4. Boy Scout Rule

> "Leave the code better than you found it."

When touching code with debt:
```javascript
// Before working on feature
function processOrder(order) {
  // 200 lines of messy code
}

// After your change
function processOrder(order) {
  validateOrder(order);  // Extracted
  const total = calculateTotal(order);  // Extracted
  processPayment(order, total);  // Extracted
  sendConfirmation(order);  // Extracted
}

// Small improvements every time
```

## Preventing Technical Debt

### 1. Code Review Standards

**Block for:**
- Security vulnerabilities
- No tests for critical paths
- Copy-pasted code (DRY violation)
- Magic numbers/strings

**Accept with debt ticket for:**
- Suboptimal but working solution
- Missing edge case handling
- Performance that's "good enough"

### 2. Definition of Done

✅ **Not done until:**
- Tests written and passing
- Code reviewed and approved
- Documentation updated
- No critical TODOs

### 3. Architecture Decision Records

Document trade-offs:
```markdown
# ADR-005: Skip caching for v1

## Decision
We will not implement caching for the initial release.

## Rationale
- Current load is <100 requests/day
- Time to market is critical
- Simple infrastructure preferred for MVP

## Debt Incurred
- No caching layer
- May be slow at scale

## Payback Trigger
When we reach 1000 requests/day, implement Redis caching.

## Estimated Payback Cost: 3-5 days
```

## Communicating Debt

### To Management

❌ **Don't say:**
"The code is a mess, we need to rewrite everything."

✅ **Do say:**
"Our current architecture adds 2-3 days to each new feature. Investing 2 weeks in refactoring would reduce future feature time by 30%."

**Use business terms:**
- Time to market
- Cost per feature
- Risk of bugs
- Maintenance cost

### To Team

**In PR reviews:**
```
💡 SUGGESTION: This works but creates technical debt.
Consider extracting this logic into a separate service
to avoid tight coupling. If time is tight, let's create
a ticket to refactor next sprint.

Ticket created: TECH-456
```

**In planning:**
```
"This feature will take 5 days with our current setup.
If we spend 2 days refactoring the payment system first,
this feature drops to 2 days. Plus future payment features
become faster. Should we tackle the refactor first?"
```

## Debt Paydown Strategies

### 1. Big Refactor (Risky)

❌ **The Problem:**
- Stop all feature work
- Spend weeks refactoring
- Big bang migration
- High risk

✅ **When appropriate:**
- Complete rewrite is needed
- System is unusable
- Clear deadline and scope

### 2. Incremental Improvement (Safer)

✅ **The Strategy:**
- Fix debt when touching the code
- Small improvements continuously
- Always leave code better
- Low risk

**Example:**
```javascript
// Week 1: Extract validation while adding feature
function createUser(data) {
  validateUser(data);  // New!
  // ... rest of messy code
}

// Week 2: Extract repository while fixing bug
function createUser(data) {
  validateUser(data);
  userRepository.save(data);  // New!
  // ... rest of messy code
}

// Week 3: Continue improving
function createUser(data) {
  const user = new User(data);  // New!
  user.validate();
  return userRepository.save(user);
  // Much cleaner!
}
```

### 3. Strangler Pattern

**Gradually replace old system:**
1. Build new system alongside old
2. Migrate one feature at a time
3. Route to new system when ready
4. Remove old system when complete

```javascript
// Router decides which system to use
function handleRequest(req, res) {
  if (isFeatureMigrated(req.path)) {
    newSystem.handle(req, res);
  } else {
    legacySystem.handle(req, res);
  }
}
```

## Red Flags

### 🚩 Dangerous Debt Patterns

**1. "We'll fix it later" (but no ticket)**
```javascript
// TODO: Fix this
// (No ticket, no plan, will never be fixed)
```

**2. "It's just temporary" (6 months later...)**
```javascript
// Temporary workaround 2023-03-15
// Still here in 2024
```

**3. "Nobody understands this code"**
```javascript
// Don't touch this, it works somehow
// Last modified: 2015
// Original author: Left company
```

**4. "We can't change this without breaking everything"**
```
File A depends on File B depends on File C...
Circular dependencies everywhere
```

### ⚠️ Warning Signs

- Features take longer each sprint
- Bugs in production increase
- Developers avoid certain areas
- "It's complicated" is common phrase
- New team members struggle
- Testing is difficult/skipped

## This Week's Practice

### Day 1-2: Identify
- [ ] List 5 pieces of technical debt in your codebase
- [ ] Categorize each (deliberate/inadvertent, prudent/reckless)
- [ ] Estimate interest rate and principal

### Day 3-4: Prioritize
- [ ] Create debt tickets for top 3
- [ ] Include context, impact, effort
- [ ] Discuss with team

### Day 5-7: Pay Down
- [ ] Fix one small piece of debt
- [ ] Apply Boy Scout Rule to code you touch
- [ ] Document what you improved

## Remember

> "It's not the debt that kills you, it's the interest."

**Key principles:**
- Some debt is okay (strategic)
- Most debt is inadvertent (learning)
- Make debt visible
- Pay interest regularly
- Communicate in business terms
- Prevent reckless debt
- Fix small debts immediately

**The senior approach:**
"We're making a trade-off. We'll ship faster now but will need to refactor X in Y weeks. The cost is Z, but the business value is worth it."

---

**Previous:** [[11 - Architecture Patterns]]
**Next:** [[13 - Mentoring Through Reviews]]
**Related:** [[15 - Trade-offs & Decision Making]]
