# 14 - Technical Communication

Senior engineers aren't just great coders—they're great communicators. The ability to explain technical concepts clearly separates senior from staff engineers.

## Why Communication Matters

**Mid-level:** Solves technical problems
**Senior:** Solves technical problems AND ensures everyone understands the solution

Good communication:
- Gets your ideas adopted
- Builds trust with team and stakeholders
- Prevents misunderstandings and rework
- Enables others to build on your work

## Written Communication

### PR Descriptions

**❌ Poor PR description:**
```
Fixed the bug
```

**✅ Good PR description:**
```
Fix race condition in payment processing

## Problem
Users reported duplicate charges when clicking "Pay" multiple times
quickly. Investigation showed no idempotency check in the payment
endpoint.

## Solution
- Added idempotency key using request ID
- Store processing payments in Redis with 10-minute TTL
- Return existing result if payment already processed

## Testing
- Added unit tests for idempotency logic
- Tested manually with rapid button clicks
- Verified in staging with load testing

## Impact
- Prevents duplicate charges
- Low risk: fails open if Redis unavailable
- No database migration needed

## Rollback Plan
Feature flag `enable_payment_idempotency` - can disable if issues arise
```

**Template:**
```markdown
## Problem
[What issue are we solving? Why now?]

## Solution
[How did we solve it? Key technical decisions?]

## Testing
[How did we verify this works?]

## Impact
[Who is affected? What are the risks?]

## Rollback Plan
[How do we undo this if needed?]
```

### Code Comments

**When to comment:**
✅ WHY decisions were made
✅ Non-obvious implications
✅ Workarounds for known issues
✅ Business logic context
✅ Performance considerations

**When NOT to comment:**
❌ What the code does (code should be self-documenting)
❌ Obvious statements
❌ Redundant information
❌ Apologies or TODOs without tickets

**Examples:**

```javascript
// ❌ BAD: States the obvious
// Loop through users
for (const user of users) {
  // Check if user is active
  if (user.isActive) {
    // Process the user
    processUser(user);
  }
}

// ✅ GOOD: Explains WHY
// Process only active users to avoid billing inactive accounts
// (Business requirement from ticket #1234)
for (const user of users) {
  if (user.isActive) {
    processUser(user);
  }
}

// ✅ GOOD: Explains non-obvious decision
// Using polling instead of websockets because corporate
// firewalls block websocket connections for 40% of users
// (See analysis in doc/network-analysis.md)
pollForUpdates();

// ✅ GOOD: Performance note
// Intentionally using linear search instead of hash map
// because array is always <10 items and we need to maintain
// insertion order for display
const result = items.find(item => item.id === targetId);
```

### Documentation

**README should answer:**
- What does this do?
- Why does it exist?
- How do I get started?
- How do I deploy?
- Who do I ask for help?

**Architecture docs should explain:**
- System overview (diagram)
- Key components and their responsibilities
- Data flow
- External dependencies
- Design decisions and trade-offs

**API documentation:**
- Request/response formats
- Authentication
- Error codes
- Rate limits
- Examples

### Design Documents

For large features, write a design doc:

**Template:**
```markdown
# [Feature Name] Design Doc

## Context
What problem are we solving? Why?

## Goals
- Primary goal
- Secondary goals

## Non-Goals
What we're explicitly NOT doing

## Proposed Solution
High-level approach

### Architecture
[Diagram]

### Data Model
[Schema changes]

### API Changes
[New endpoints, changes to existing]

### Security Considerations
[Auth, validation, data privacy]

### Performance Implications
[Expected load, scaling considerations]

## Alternatives Considered
### Option 1: [Alternative approach]
Pros: ...
Cons: ...
Why not chosen: ...

### Option 2: ...

## Trade-offs
What are we sacrificing? Why is it worth it?

## Implementation Plan
1. Phase 1: ...
2. Phase 2: ...

## Testing Strategy
How will we validate this works?

## Metrics & Monitoring
What will we measure? What alerts needed?

## Rollout Plan
How do we deploy safely?

## Rollback Plan
What if we need to undo this?

## Open Questions
- Question 1?
- Question 2?
```

## Verbal Communication

### Code Reviews (Written)

**Use the feedback sandwich (when appropriate):**
1. Start positive
2. Constructive feedback
3. End encouraging

**Example:**
```
✨ Nice use of the builder pattern here - makes this much more readable!

⚠️ CONCERN: I noticed we're not handling the case where the API
returns a 429 (rate limit). Should we add retry logic with
exponential backoff? See similar implementation in user-service.ts:45

The overall structure is solid and this will be a great addition.
Let me know if you need help with the retry logic!
```

**Tone guidelines:**
- Ask questions, don't command: "Could we..." vs "You should..."
- Suggest, don't demand: "Consider X" vs "Do X"
- Explain WHY: "This could cause Y because Z"
- Acknowledge good work

### Technical Discussions

**Structure your arguments:**

1. **State the problem clearly**
   - "We're seeing 503 errors during peak hours"

2. **Provide evidence**
   - "Our monitoring shows CPU at 95% from 12-2pm"
   - "Queries are timing out after 30 seconds"

3. **Propose solution(s)**
   - "Option 1: Scale vertically (add CPU)"
   - "Option 2: Add read replicas"
   - "Option 3: Implement caching layer"

4. **Compare trade-offs**
   - "Option 1 is fastest to implement but has limits"
   - "Option 2 requires code changes but scales better"

5. **Make recommendation**
   - "I recommend Option 2 because..."

**Listen actively:**
- Repeat back to confirm understanding
- Ask clarifying questions
- Don't interrupt
- Consider other viewpoints

### Disagreements

**When you disagree:**

1. **Understand first**
   - "Help me understand your thinking..."
   - "What problem does this solve?"

2. **Find common ground**
   - "I agree that X is important..."
   - "I share your concern about Y..."

3. **Present your view**
   - "My concern is..."
   - "Have we considered...?"

4. **Focus on trade-offs**
   - Not "right vs wrong"
   - But "pros and cons"

5. **Disagree and commit**
   - If decision is made, support it fully
   - "I shared my concerns, but let's move forward with this approach"

**Example:**
```
❌ BAD:
"That approach won't work. We should use X instead."

✅ GOOD:
"I understand the appeal of that approach. My concern is
scalability—we'd need to modify every endpoint. Have we
considered using middleware instead? That would centralize
the logic and reduce the maintenance burden. What do you think?"
```

## Communication with Non-Technical Stakeholders

### Translating Technical to Business

**When explaining to non-engineers:**

❌ Don't: "We need to refactor the monolith into microservices with eventual consistency using event sourcing."

✅ Do: "Our current system is like a large department store—everything's connected. We want to split it into specialized shops. This will let us fix one shop without closing the whole mall."

**Use analogies:**
- Database index → Book index
- Cache → Sticky note on your desk
- Load balancer → Restaurant host seating guests
- API → Restaurant menu
- Rate limiting → "Sorry, limit 2 per customer"

### Estimating Work

**Never say:** "It'll take 2 days"

**Instead:**
- Break down the work
- Identify unknowns
- Give a range
- Explain assumptions

**Example:**
```
"Based on the requirements, here's my estimate:

Known work:
- API endpoint: 4-6 hours
- Database schema: 2-3 hours
- Frontend form: 4-6 hours
- Tests: 3-4 hours
Total: 13-19 hours (~2-3 days)

Unknowns:
- Integration with payment system (never done before): +1-2 days
- Potential performance issues: +0.5-1 day buffer

Realistic estimate: 4-6 days

Assumptions:
- No major design changes
- Payment API works as documented
- Can deploy independently

I'll update you after the spike on payment integration (day 1).
```

### Status Updates

**Structure:**
1. **Progress:** What's done
2. **Current:** What you're working on
3. **Next:** What's coming up
4. **Blockers:** What's stopping you
5. **Risks:** What might go wrong

**Example:**
```
✅ Completed:
- Database schema migration
- API endpoint implementation

🔄 In Progress:
- Frontend integration (70% done)

📅 Next:
- Testing
- Documentation

🚧 Blocked:
- Need design approval for error states (pinged Sarah)

⚠️ Risks:
- Third-party API has been flaky in staging
- Might need fallback plan
```

## Senior Communication Patterns

### Teaching & Mentoring

**When helping junior engineers:**

❌ Don't: Give the answer immediately
✅ Do: Guide them to discover it

**Example:**
```
Junior: "Why isn't this working?"

❌ BAD: "Change line 45 to X"

✅ GOOD:
"Let's debug together. What do you expect to happen?
What's actually happening? Have you checked what value
is being passed to that function? [pause for them to check]
Interesting! Why might it be undefined? What does that
tell us about when this function is called?"
```

### Saying No (Constructively)

**When you disagree with a request:**

1. **Acknowledge the request**
   - "I understand this feature is important to users"

2. **Explain the constraint**
   - "The challenge is we're already committed to X and Y this sprint"

3. **Provide alternatives**
   - "Could we do a lightweight version?"
   - "Could this wait until next sprint?"
   - "Could another team handle this?"

4. **Make it a conversation**
   - "What's the urgency? Can we discuss priorities?"

### Admitting You Don't Know

**Senior engineers admit what they don't know:**

✅ "I'm not sure. Let me research that and get back to you."
✅ "That's outside my expertise. Let's ask Sarah who knows this area better."
✅ "Good question. I don't know off the top of my head."

❌ Bluffing or making up answers
❌ "I think..." when you have no idea

## Communication Checklist

### Before Sending
- [ ] Is this clear?
- [ ] Is this necessary?
- [ ] Is this kind?
- [ ] Is this actionable?
- [ ] Did I explain WHY?
- [ ] Would I want to receive this?

### In Meetings
- [ ] Come prepared
- [ ] Take notes
- [ ] Ask clarifying questions
- [ ] Confirm action items
- [ ] Follow up with summary

## This Week's Practice

### Day 1-2: Writing
- [ ] Write thorough PR descriptions for your next 3 PRs
- [ ] Add meaningful comments to complex code
- [ ] Update one piece of outdated documentation

### Day 3-4: Reviewing
- [ ] Write thoughtful, kind review comments
- [ ] Practice the feedback sandwich
- [ ] Ask questions instead of making demands

### Day 5-7: Speaking
- [ ] Explain one technical concept to a non-engineer
- [ ] Present one design decision in standup
- [ ] Help a junior engineer debug (guide, don't solve)

## Remember

> "The single biggest problem in communication is the illusion that it has taken place." - George Bernard Shaw

Great code that nobody understands is useless.
Good code that everyone understands is valuable.

---

**Previous:** [[13 - Mentoring Through Reviews]]
**Next:** [[15 - Trade-offs & Decision Making]]
**Related:** [[01 - Code Review Fundamentals]]
