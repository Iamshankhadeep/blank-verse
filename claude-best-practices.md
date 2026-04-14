# Claude 4 Best Practices for Claude Code

## Core Principles

### Be Explicit
Claude 4 models follow instructions precisely. Request behaviors explicitly:
- **Less effective:** "Create an analytics dashboard"
- **More effective:** "Create an analytics dashboard. Include relevant features and interactions. Go beyond the basics."

### Ask to review by becoming expert
- "adopt the persona of an expert in this field"
### Provide Context
Explain *why* behind instructions for better results:
- **Less effective:** "NEVER use ellipses"
- **More effective:** "Your response will be read aloud by TTS, so never use ellipses since TTS won't know how to pronounce them."

### Action vs Suggestion
Claude 4 takes instructions literally. Be clear about intent:
- "Can you suggest changes?" → Claude will only suggest
- "Make these changes" → Claude will implement

### Use Examples
Demonstrate desired format, tone, or style through examples rather than description alone. One well-chosen example often outperforms lengthy explanations. Claude pays close attention to example details.

### Allow Uncertainty
Grant explicit permission to acknowledge limitations. This reduces hallucinations:
```
If you're unsure or don't have enough information, say so rather than guessing.
```

---

## Context Engineering

### Context as Finite Resource
LLMs experience "context rot" - accuracy decreases as context grows. Every token depletes the model's attention budget. Curate carefully.

### High-Signal Tokens
Guiding principle: find the smallest set of high-signal tokens that maximize desired outcomes. Remove redundancy, keep what matters.

### Compaction Strategies
When approaching context limits:
- Summarize conversation history
- Preserve architectural decisions and unresolved bugs
- Discard redundant tool outputs
- Keep structured state (tests.json) over verbose logs

### Structured Notes
Persist critical information outside the context window:
- `progress.txt` for freeform progress notes
- `tests.json` for structured state tracking
- Git commits for checkpoints and history

### Just-in-Time Retrieval
Prefer lightweight identifiers over loading all data upfront:
- **Better:** Keep file paths, load content when needed
- **Worse:** Dump entire files into context "just in case"

---

## Tool Design & Usage

### Parallel Execution
Claude 4 excels at parallel tool calls. Make independent calls simultaneously:
- Read multiple files at once
- Run concurrent searches
- Execute independent bash commands together

### Minimal Overlap
Design tools with clear, unambiguous use cases. If humans can't definitively choose the right tool, neither will Claude.

### Self-Contained Tools
Make tools robust to error and explicit about intended purpose. Descriptive input parameters leverage model strengths.

### Reduce Over-Engineering
Claude 4 may create extra files or abstractions. Keep solutions minimal:
- Only make directly requested changes
- Don't add unnecessary error handling
- Don't create helpers for one-time operations
- Clean up temporary files when done

---

## Sub-Agents

### When to Delegate
Deploy specialized sub-agents for focused tasks with clean context windows. The coordinator maintains the high-level plan while sub-agents perform detailed work.

### Condensed Summaries
Sub-agents should return condensed summaries (1-2k tokens) of extensive exploration. This separates detailed search context from synthesis responsibilities.

---

## Prompting Techniques

### Chain of Thought
Request step-by-step reasoning before answers:
```
Think through this step-by-step before providing your answer.
```

### Prompt Chaining
Break complex tasks into sequential steps with separate prompts. Each stage feeds into the next - trades latency for higher accuracy.

### Prefill Responses
Guide format and structure by starting the response. Effective for enforcing JSON/XML output or skipping preambles.

---

## Long-Running Tasks

### State Management
- Use `tests.json` for structured state tracking
- Use `progress.txt` for freeform progress notes
- Use git for checkpoints and history
- Focus on incremental progress

### Multi-Context Workflows
1. First context: set up framework (tests, scripts)
2. Future contexts: iterate on todo-list
3. Create `init.sh` setup scripts for resumption
4. Review `progress.txt`, `tests.json`, git logs when resuming

---

## Code Quality

### Prevent Hallucinations
Always read files before proposing edits. Don't speculate about unread code.
```
Never speculate about code you have not opened. If the user references a file, read it first.
```

### Avoid Hard-Coding
Write general solutions, not test-specific workarounds. Implement actual logic, not shortcuts that only pass test cases.

### Code Exploration
Be thorough when searching code. Review style and conventions before implementing new features.

---

## Anti-Patterns

Avoid these common mistakes:
- **Hardcoding conditional logic** in prompts (fragile, breaks on edge cases)
- **Assuming shared context** without explicit specification
- **Edge case lists** instead of canonical examples (examples > rules)
- **Ambiguous tools** where selection criteria aren't clear
- **Loading entire objects** upfront instead of just-in-time retrieval
- **Overly aggressive compaction** that loses subtle but critical context

---

## Communication & Formatting

### Communication Style
Claude 4 is more concise and direct. If you want updates:
```
After completing a task involving tool use, provide a quick summary of the work done.
```

### Formatting Control
1. Tell Claude what TO do, not what NOT to do
2. Use XML tags: `<smoothly_flowing_prose_paragraphs>`
3. Match prompt style to desired output style

### Thinking Sensitivity
Claude Opus 4.5 is sensitive to "think" when extended thinking is disabled. Use alternatives: "consider", "believe", "evaluate".


