# 01 - Code Review Fundamentals

## What Makes a Great Code Review?

A senior engineer's review goes beyond syntax - it considers **impact, maintainability, and team growth**.

## The Review Mindset

### 1. Understand the Context First
- Read the PR description thoroughly
- Understand the problem being solved
- Check linked issues/tickets
- Ask "why" before "how"

**Practice:** Before commenting on your next PR, write down:
- What problem is this solving?
- Who are the users affected?
- What's the business impact?

### 2. Review Layers (In Order)

#### Layer 1: Architecture & Design (5 min)
- Does this fit the system's design?
- Is this the right place for this code?
- Are we creating the right abstractions?

#### Layer 2: Logic & Correctness (10 min)
- Does the code do what it claims?
- Are edge cases handled?
- What happens when things fail?

#### Layer 3: Security & Performance (5 min)
- See [[08 - Security in Code Reviews]]
- Any obvious performance issues?
- Resource leaks?

#### Layer 4: Readability & Style (5 min)
- Can I understand this in 6 months?
- Are names clear?
- Is complexity justified?

#### Layer 5: Tests (5 min)
- Are critical paths tested?
- Do tests actually test the right thing?
- Are they maintainable?

## The Art of Feedback

### ✅ DO:
```
❓ "Could we extract this into a separate function?
   It would make testing easier and improve readability."

💡 "Nice solution! Have you considered using X?
   It might handle the Y edge case better."

📝 "Can you add a comment explaining why we're doing this?
   The 'why' isn't obvious from the code."
```

### ❌ DON'T:
```
"This is wrong." (No explanation)
"We don't do it this way." (No reasoning)
"Nit: [subjective preference]" (Blocking for style)
```

## Review Categories

### 1. 🚫 MUST BLOCK (Request Changes)
- Security vulnerabilities
- Data loss risks
- Breaking changes without migration
- Critical bugs
- Secrets/credentials in code

### 2. 🟡 SHOULD ADDRESS (Comment)
- Missing error handling
- Performance concerns
- Missing tests for critical paths
- Confusing code structure
- Technical debt being added

### 3. 🟢 MINOR (Approve with comments)
- Nits about naming
- Style preferences
- Suggestions for future improvements
- Learning opportunities

## Common Review Mistakes

1. **Nitpicking style** - Use linters/formatters for this
2. **Rewriting in your style** - Different doesn't mean wrong
3. **Review fatigue** - Large PRs are hard to review well
4. **Not testing locally** - Sometimes you need to run it
5. **Ignoring the PR description** - Context matters

## Your Action Items

This week, review 3 PRs and:
- [ ] Start each review by writing down the problem being solved
- [ ] Use the 5-layer approach
- [ ] Categorize each comment (BLOCK/SHOULD/MINOR)
- [ ] Ask at least one question instead of making a demand
- [ ] Find one thing to praise

## Questions to Ask Yourself

- Would I be proud to maintain this code?
- Is this code easier to understand than before?
- Does this make the system better or just different?
- Am I being pedantic or is this genuinely important?

---

**Previous:** [[🎯 START HERE - Roadmap]]
**Next:** [[02 - Code Smells & Anti-Patterns]]
**Related:** [[PR Review Checklist]]
