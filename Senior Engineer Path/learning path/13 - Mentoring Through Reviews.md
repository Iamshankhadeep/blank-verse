# 13 - Mentoring Through Reviews

Senior engineers don't just review code—they grow engineers. Every review is an opportunity to teach, inspire, and raise the team's collective skill.

## The Mentoring Mindset

**Junior reviewer:** "Is this code correct?"
**Senior reviewer:** "Is this code correct AND did I help the author grow?"

### Goals of Mentoring Reviews

1. **Immediate:** Improve this code
2. **Short-term:** Teach a concept/pattern
3. **Long-term:** Develop the engineer's judgment

## Types of Learning Reviews

### 1. Guiding Questions (Best for junior engineers)

**❌ Don't give the answer:**
```
"Change line 45 to use a Set instead of an array."
```

**✅ Ask questions that lead to discovery:**
```
❓ "I notice we're using .includes() in a loop here. What's the
time complexity of this approach? Have you considered using a
different data structure? What would be the trade-offs?"
```

**Follow-up:**
```
"Great thinking! Yes, a Set would be O(1) for lookups. When
would you NOT want to use a Set here?"
```

**Why it works:**
- They discover the answer
- They understand the "why"
- They'll remember it better
- Builds problem-solving skills

### 2. Explaining Context (For mid-level engineers)

**❌ "We don't do it that way."**

**✅ Explain the reasoning:**
```
💡 "This approach would work, but we've standardized on using
the Repository pattern for database access (see user-repository.ts
for an example). The reason is:

1. Easier to test (mock the repository)
2. Centralized data access logic
3. Easy to swap databases if needed

Could you refactor this to follow that pattern? Happy to pair
on it if helpful!"
```

### 3. Sharing Knowledge (For all levels)

**Share patterns and resources:**
```
✨ "Nice solution! This is a great use case for the Strategy
pattern. If you want to learn more, check out:
- refactoring.guru/design-patterns/strategy
- Our team wiki on design patterns

The core idea is: encapsulate algorithms and make them
interchangeable. You've actually implemented a simple version
of it here!"
```

## The Socratic Method in Reviews

### Step 1: Observe

```
"I see you're iterating through all users to find matching ones."
```

### Step 2: Ask about intent

```
"What's the expected number of users? How often will this run?"
```

### Step 3: Explore implications

```
"What happens when we have 100,000 users? How would the
performance change?"
```

### Step 4: Guide toward solution

```
"Have you considered using database filtering instead of
in-memory filtering? What would that look like?"
```

### Step 5: Discuss trade-offs

```
"Good! Database filtering is faster for large datasets.
When might in-memory filtering be better?"
```

## Teaching Through Examples

### Show, Don't Just Tell

**❌ Abstract advice:**
```
"This violates the Single Responsibility Principle."
```

**✅ Concrete example:**
```
⚠️ This function is doing three things:
1. Validating input (lines 5-10)
2. Saving to database (lines 12-15)
3. Sending email (lines 17-20)

Each of these should be its own function:

function createUser(userData) {
  validateUserData(userData);
  const user = saveUser(userData);
  sendWelcomeEmail(user);
  return user;
}

This makes each piece:
- Easier to test independently
- Reusable in other contexts
- Clearer in purpose

Want to try refactoring it this way?
```

## Levels of Mentoring Feedback

### For Junior Engineers

**Focus on:**
- ✅ Fundamentals (naming, structure, testing)
- ✅ Building confidence
- ✅ Safe experimentation

**Example:**
```
✨ Great job getting this working! The logic is sound. Let me
share a few ways to make it even better:

1. **Naming**: `process()` is vague. `processPayment()` is clearer.

2. **Error handling**: What happens if the payment fails? Adding
a try-catch would make this more robust:

   try {
     const result = await processPayment();
   } catch (error) {
     logger.error('Payment failed', error);
     return { success: false, error: error.message };
   }

3. **Testing**: This would be a great function to add a test for!

Want to give these a try? I'm happy to review again or pair if
you have questions!
```

### For Mid-Level Engineers

**Focus on:**
- ✅ Design patterns
- ✅ Architecture awareness
- ✅ Trade-off analysis

**Example:**
```
💭 Solid implementation! I want to discuss the architecture:

You're calling the email service directly from the controller.
This creates tight coupling and makes testing harder.

Consider:
- Controller → Service → Repository pattern
- Dependency injection for the email service
- Event-based approach (emit "user.created" event)

Each has trade-offs:
- DI: More flexible, easier testing, but more setup
- Events: Decoupled, but harder to debug, eventual consistency

For this use case, I'd suggest DI since we want immediate
confirmation that the email was sent. What do you think?
```

### For Senior Engineers

**Focus on:**
- ✅ System-wide impact
- ✅ Long-term maintainability
- ✅ Teaching opportunities

**Example:**
```
🤔 Great design overall. Some thoughts on the broader impact:

1. **Scalability**: This works now, but at 10x load, the
   synchronous email sending will be a bottleneck. Document
   this as tech debt?

2. **Observability**: Consider adding metrics for email send
   rate/failures. Future you will thank present you.

3. **Team knowledge**: This is a excellent example of the
   Command pattern. Consider adding to our architecture docs
   or presenting at Friday tech talk?

Also, this could be a great mentoring opportunity for Jane
who's working on a similar feature. Mind pairing with her?
```

## Common Mentoring Scenarios

### Scenario 1: Over-Engineering

**Junior engineer creates complex solution:**

```javascript
// Over-engineered for simple task
class UserNameFormatterFactory {
  create(type) {
    switch(type) {
      case 'formal': return new FormalUserNameFormatter();
      case 'casual': return new CasualUserNameFormatter();
    }
  }
}

// For just: firstName + lastName
```

**Mentoring response:**
```
💡 I love that you're thinking about design patterns! The
Factory pattern is useful when you have complex object creation
or multiple implementations.

For this case though, we're just formatting a name. A simple
function would be clearer and easier to maintain:

function formatUserName(user, style = 'formal') {
  return style === 'formal'
    ? `${user.lastName}, ${user.firstName}`
    : `${user.firstName} ${user.lastName}`;
}

**Rule of thumb**: Start simple. Add patterns when complexity
demands it. "You Aren't Gonna Need It" (YAGNI) is your friend!

Save the factory pattern for when we have 5+ formatters or
complex instantiation logic. Make sense?
```

### Scenario 2: Missing Edge Cases

**Engineer forgets error handling:**

```javascript
function calculateDiscount(price, discountCode) {
  const discount = DISCOUNTS[discountCode];
  return price - (price * discount);
}
```

**Mentoring response:**
```
❓ Good start! Let's think through some scenarios:

1. What if discountCode doesn't exist in DISCOUNTS?
2. What if price is negative?
3. What if discount is 0.5 but price is $1? (negative result?)

Try running through these cases:
- calculateDiscount(100, 'INVALID')
- calculateDiscount(-50, 'SAVE20')
- calculateDiscount(1, 'HALF_OFF')

What behavior do you want for each? Then add validation and
tests for these edge cases.

💡 Pro tip: Think "what could go wrong?" before writing code.
It's faster to prevent bugs than fix them!
```

### Scenario 3: Good Code, Teaching Moment

**Engineer writes good code but could learn more:**

```javascript
// Good code!
function filterActiveUsers(users) {
  return users.filter(user => user.isActive);
}
```

**Mentoring response:**
```
✅ Approved! This is clean, readable code.

🎓 **Bonus learning**: You might enjoy learning about partial
application and composition. Your code could also be written as:

const isActive = user => user.isActive;
const filterBy = predicate => items => items.filter(predicate);

const filterActiveUsers = filterBy(isActive);

This style enables:
- Better reusability: `filterBy` works for any predicate
- Composition: `filterBy(isActive)(users)`
- Functional programming patterns

Not saying you should change this—your version is great!
Just sharing in case you want to explore functional programming.
Resources:
- JavaScript: The Good Parts (Chapter on Functions)
- Professor Frisby's Mostly Adequate Guide to FP

Keep up the great work! 🎉
```

## Building Psychological Safety

### Create Safe Learning Environment

**❌ Destroys confidence:**
```
"This is wrong. You should know better."
"Did you even read the style guide?"
"Sloppy code. Please fix."
```

**✅ Builds confidence:**
```
"This works! Let me share how we could make it even better..."
"Great first pass! Here's how we typically handle this..."
"I like your approach. Have you considered...?"
```

### Celebrate Growth

```
✨ "Much better! You've applied the feedback from last review
perfectly. I can see you're getting the hang of the Repository
pattern. Keep it up!"

🎉 "This is your first PR without any major issues. Well done!
The code quality is consistently improving."

🚀 "You caught that edge case yourself this time. That's exactly
the kind of thinking that makes a senior engineer!"
```

### Normalize Mistakes

```
💭 "I made this exact same mistake last month! Here's what
I learned..."

📚 "This is a subtle issue that trips up everyone. Even after
10 years, I still have to think carefully about async error
handling."

🤝 "We all write bugs. The difference is: you caught it in
review before it hit production. That's what reviews are for!"
```

## Async Mentoring (Not Just Reviews)

### Documentation

**Create learning resources:**
```markdown
# Team Wiki: Common Review Feedback

## N+1 Queries
[Explanation with examples]
[How to fix]
[How to prevent]

## Missing Edge Cases
[Common scenarios to check]
[Testing strategies]
```

### Pair Programming

```
"I noticed you're working on payment processing. I did something
similar last month. Want to pair for an hour? I can show you
the patterns we use and we can tackle this together."
```

### Tech Talks

```
"Your PR had a really elegant solution to that caching problem.
Would you be interested in presenting it at our Friday tech
talk? It would help the whole team learn this pattern."
```

### Code Examples

**Leave breadcrumbs in the codebase:**
```javascript
/**
 * Example of the Repository pattern
 * See: docs/architecture/repository-pattern.md
 * Related: user-repository.ts, order-repository.ts
 */
class ProductRepository {
  // ... well-documented implementation
}
```

## Mentoring Through Review Comments

### The Comment Template

```markdown
[EMOJI] [CATEGORY]: [OBSERVATION]

[EXPLANATION]

[SUGGESTION/QUESTION]

[RESOURCES] (optional)
```

**Examples:**

```
✨ GREAT: Clean separation of concerns here!

You've properly separated the validation, business logic, and
data access. This makes testing much easier.

This is exactly the pattern we want to see. Nice work!
```

```
🤔 QUESTION: Have you considered the race condition?

If two users try to register with the same email simultaneously,
both might pass the "email exists" check before either is saved.

Could we use a database unique constraint + try/catch to handle
this? Or a distributed lock?

Resources:
- Database constraints: [link]
- Our race condition wiki: [link]
```

```
💡 SUGGESTION: Consider extracting this logic

This validation logic (lines 45-67) is complex and could be
reused in other places. What if we extracted it to a
UserValidator class?

Benefits:
- Reusable in create AND update endpoints
- Easier to test in isolation
- Single source of truth for validation rules

Want to try that? Or we can pair on it!
```

## This Week's Practice

### Day 1-2: Observe Your Style
- [ ] Review 3 PRs
- [ ] Notice how you give feedback
- [ ] Are you asking questions or giving answers?

### Day 3-4: Practice Socratic Method
- [ ] Review using only questions
- [ ] Guide toward solution, don't give it
- [ ] See if author discovers the answer

### Day 5-7: Mentoring Focus
- [ ] Add learning resources to comments
- [ ] Celebrate what's good
- [ ] Explain the "why" behind feedback
- [ ] Follow up on previous feedback

## Remember

> "The best engineers don't just write great code—they make everyone around them better."

**Mentoring principles:**
- Ask, don't tell (when possible)
- Explain the "why"
- Celebrate progress
- Share resources
- Build confidence
- Create psychological safety
- Think long-term growth

**Your goal:** Not just better code, but better engineers.

---

**Previous:** [[12 - Technical Debt Management]]
**Next:** [[14 - Technical Communication]]
**Related:** [[01 - Code Review Fundamentals]]
