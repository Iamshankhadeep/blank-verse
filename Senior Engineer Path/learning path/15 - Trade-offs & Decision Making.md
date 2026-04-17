# 15 - Trade-offs & Decision Making

The hallmark of a senior engineer isn't knowing all the answers—it's understanding the trade-offs and making informed decisions.

## The Senior Mindset

**Junior:** "What's the right answer?"
**Mid:** "What's the best solution?"
**Senior:** "What are the trade-offs, and which one fits our context?"

> Every architectural decision is a trade-off. There are no perfect solutions, only appropriate ones.

## Common Technical Trade-offs

### 1. Performance vs Simplicity

**Scenario:** Optimize this algorithm?

**Option A: Simple but slower**
```javascript
// O(n²) - Nested loops
function findDuplicates(items) {
  const duplicates = [];
  for (let i = 0; i < items.length; i++) {
    for (let j = i + 1; j < items.length; j++) {
      if (items[i] === items[j]) {
        duplicates.push(items[i]);
      }
    }
  }
  return duplicates;
}
```

**Option B: Complex but faster**
```javascript
// O(n) - Hash map
function findDuplicates(items) {
  const seen = new Map();
  const duplicates = new Set();

  for (const item of items) {
    if (seen.has(item)) {
      duplicates.add(item);
    }
    seen.set(item, true);
  }

  return Array.from(duplicates);
}
```

**Trade-off analysis:**
- **Option A:** Simpler to understand, but O(n²)
- **Option B:** Faster, but more complex

**Decision factors:**
- ✅ Use A if: <100 items, rarely called, team is junior
- ✅ Use B if: >1000 items, called frequently, performance matters

**The Rule:** "Make it work, make it right, make it fast" - optimize when measured need exists.

### 2. Consistency vs Availability

**Scenario:** Should our distributed system prioritize consistency or availability?

**Strong Consistency:**
```javascript
// All nodes must agree before responding
// ✅ Always accurate
// ❌ Slower, might be unavailable during network issues
async function transfer(from, to, amount) {
  await lockAccounts([from, to]);
  const result = await coordinatedTransaction(from, to, amount);
  await unlockAccounts([from, to]);
  return result;
}
```

**Eventual Consistency:**
```javascript
// Respond immediately, sync later
// ✅ Always available
// ❌ Might show stale data temporarily
async function transfer(from, to, amount) {
  await queueTransfer(from, to, amount);
  return { status: 'pending' };
  // Background worker processes queue
}
```

**Decision factors:**
- **Strong:** Banking, inventory, anything money-related
- **Eventual:** Social media feeds, analytics, user profiles

### 3. Monolith vs Microservices

**Monolith:**
- ✅ Simple deployment
- ✅ Easy to debug
- ✅ Fast development initially
- ❌ Scale entire app, not parts
- ❌ All-or-nothing deployment

**Microservices:**
- ✅ Scale independently
- ✅ Technology flexibility
- ✅ Team autonomy
- ❌ Complex operations
- ❌ Network overhead
- ❌ Distributed system problems

**When to choose:**
- **Monolith:** Small team (<10), new product, unclear boundaries
- **Microservices:** Large team (>20), clear boundaries, different scaling needs

**The Rule:** Start monolith, extract services when pain is real.

### 4. Build vs Buy

**Scenario:** Need authentication system

**Build:**
- ✅ Full control
- ✅ Customizable
- ❌ Time-consuming
- ❌ Security burden
- ❌ Ongoing maintenance

**Buy (Auth0, Cognito, etc.):**
- ✅ Fast to implement
- ✅ Security handled
- ✅ Maintained by experts
- ❌ Less control
- ❌ Ongoing cost
- ❌ Vendor lock-in

**Decision factors:**
- **Build:** Core business differentiator, unique requirements
- **Buy:** Commodity feature, not your expertise, fast time-to-market

**The Rule:** Don't build what you can buy, unless it's your competitive advantage.

### 5. Normalization vs Denormalization

**Normalized (Relational):**
```sql
-- No duplication, but requires joins
SELECT u.name, o.total, p.name as product
FROM orders o
JOIN users u ON o.user_id = u.id
JOIN order_items oi ON oi.order_id = o.id
JOIN products p ON oi.product_id = p.id
```

✅ Single source of truth
✅ Easier updates
❌ Slower reads (joins)

**Denormalized (NoSQL):**
```javascript
// Duplicate data, but fast reads
{
  orderId: 123,
  userName: "John",  // Duplicated
  userEmail: "john@example.com",  // Duplicated
  items: [
    { productName: "Widget", price: 10 },  // Duplicated
  ]
}
```

✅ Fast reads (no joins)
✅ Scales horizontally
❌ Data duplication
❌ Harder updates

**Decision:** Normalize for write-heavy, denormalize for read-heavy.

### 6. Synchronous vs Asynchronous

**Synchronous:**
```javascript
// Wait for result
const result = await processPayment(order);
res.json({ result });
```

✅ Simple to reason about
✅ Immediate feedback
❌ Slower for user
❌ Can block

**Asynchronous:**
```javascript
// Queue and respond immediately
await queue.publish('process-payment', order);
res.json({ status: 'processing' });
```

✅ Fast response
✅ Better resilience
❌ More complex
❌ Eventual feedback

**Decision:**
- **Sync:** User needs immediate result, fast operation (<100ms)
- **Async:** Slow operation (>1s), can fail/retry, not urgent

## Framework for Decision Making

### Step 1: Understand the Context

Ask:
- What's the business goal?
- Who are the users?
- What's the timeline?
- What are the constraints?
- What's the team's skill level?

### Step 2: Identify Options

List 3-5 realistic options:
- Current state (do nothing)
- Minimal viable solution
- Ideal solution
- Compromise solutions

### Step 3: Analyze Trade-offs

For each option, consider:

**Technical:**
- Performance
- Scalability
- Maintainability
- Reliability
- Security

**Business:**
- Cost (development + ongoing)
- Time to market
- Risk
- Flexibility

**Team:**
- Skill fit
- Learning curve
- Developer experience
- Velocity impact

### Step 4: Make Decision Matrix

| Option | Pros | Cons | Cost | Risk | Time | Recommendation |
|--------|------|------|------|------|------|----------------|
| A | ... | ... | $ | Low | 1w | ⭐ |
| B | ... | ... | $$$ | High | 4w | |
| C | ... | ... | $$ | Med | 2w | |

### Step 5: Decide & Document

1. **Make the call**
   - Consider trade-offs
   - Align with business goals
   - Get input from team
   - Make decision

2. **Document reasoning**
   ```markdown
   ## Decision: Use PostgreSQL for primary database

   ### Context
   Need to store user data and transactions with ACID guarantees.

   ### Options Considered
   1. PostgreSQL
   2. MongoDB
   3. DynamoDB

   ### Decision
   Chose PostgreSQL.

   ### Rationale
   - Need strong consistency for transactions
   - Complex queries required for reporting
   - Team has PostgreSQL expertise
   - Open source (no vendor lock-in)

   ### Trade-offs Accepted
   - Harder to scale horizontally (acceptable for now)
   - More complex setup than managed solution (team can handle)

   ### Future Revisit
   If we exceed 100k transactions/day, consider sharding or
   hybrid approach with read replicas.
   ```

3. **Commit to the decision**
   - Support it publicly
   - Give it a fair chance
   - Measure results

## Common Decision Patterns

### When to Optimize

❌ **Don't optimize:**
- Before you measure
- Something called once per day
- When it's "fast enough"
- During initial development

✅ **Do optimize:**
- Measured bottleneck
- User-facing slowness
- Resource cost issues
- After profiling

**The Rule:** "Premature optimization is the root of all evil" - Donald Knuth

### When to Refactor

❌ **Don't refactor:**
- Code that rarely changes
- When under tight deadline
- Without tests
- To show off skills

✅ **Do refactor:**
- Actively changing code
- Code smells causing bugs
- Before adding feature to area
- With good test coverage

**The Rule:** Make it work, then make it right (if it matters).

### When to Add Abstraction

❌ **Don't abstract:**
- Single use case
- Requirements unclear
- Before second instance
- To be "fancy"

✅ **Do abstract:**
- After 3rd duplication (Rule of Three)
- Clear pattern emerged
- Multiple teams need it
- Proven stable requirement

**The Rule:** "Duplication is better than the wrong abstraction" - Sandi Metz

### When to Use New Technology

❌ **Don't adopt:**
- Because it's trendy
- Without evaluation
- For everything
- Without team buy-in

✅ **Do adopt:**
- Solves specific problem
- After proof of concept
- Team is excited and capable
- Bounded experiment

**The Rule:** Choose boring technology (for most things).

## Handling Disagreements

### Disagree and Commit

Sometimes your option isn't chosen. Handle it professionally:

1. **Share your concerns clearly**
   - "My concern is X because Y"
   - "Have we considered Z?"

2. **Listen to counterarguments**
   - "That's a good point about..."
   - "I didn't consider..."

3. **If decision is made, support it**
   - "I shared my concerns, but let's give this approach our full effort"
   - Don't sabotage or say "I told you so"

4. **Set success criteria**
   - "Let's measure X and revisit in 2 weeks"

### When to Push Back

**Do push back hard on:**
- Security vulnerabilities
- Data loss risks
- Major architectural mistakes
- Unsustainable technical debt

**Don't die on hills of:**
- Naming preferences
- Code style (use linter)
- Which library when both work
- Personal preferences

## Questions for Better Decisions

### Before Deciding

- What problem are we really solving?
- What's the simplest solution?
- What are we NOT solving? (Non-goals)
- What's the cost of being wrong?
- Can we try small first?
- Is this reversible?

### During Implementation

- Is this still the right approach?
- What have we learned?
- Should we adjust?

### After Shipping

- Did this solve the problem?
- What worked? What didn't?
- What would we do differently?
- Should we revisit the decision?

## Red Flags in Decision Making

🚩 **Warning signs:**
- "This is how we've always done it"
- "Let's use X because I know it"
- "We'll fix it later"
- "This will handle everything forever"
- "It's too late to change now"
- No one can explain the "why"

## This Week's Practice

### Day 1-2: Analysis
- [ ] Identify 3 trade-offs in your current project
- [ ] Document the reasoning behind them
- [ ] Consider if you'd decide differently today

### Day 3-4: Review
- [ ] In PRs, look for trade-off discussions
- [ ] Ask "What alternatives were considered?"
- [ ] Discuss pros/cons

### Day 5-7: Decision
- [ ] Make one technical decision using the framework
- [ ] Document it thoroughly
- [ ] Share with team

## Remember

> "Good judgment comes from experience. Experience comes from bad judgment."

**Key principles:**
- No perfect solutions, only trade-offs
- Context matters immensely
- Reversible decisions can be made fast
- Irreversible decisions need more thought
- Document the "why"
- Measure and iterate

**The senior approach:**
"Here are the options, here are the trade-offs, here's my recommendation based on our context, and here's how we'll know if we're right."

---

**Previous:** [[14 - Technical Communication]]
**Next:** [[16 - Raising the Bar]]
**Related:** [[09 - System Design Basics]]
