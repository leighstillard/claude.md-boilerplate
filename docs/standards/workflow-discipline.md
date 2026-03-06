# Workflow Discipline

> How to approach work, not just what the output should look like.
>
> Last reviewed: 2026-03-06

These rules address the most common failure modes when AI writes code: wrong assumptions, over-engineering, sprawling changes, and unverified output. They complement the domain-specific standards in the other pillar files.

## Think Before Coding

### DO

- State assumptions explicitly before writing any code. If a requirement has multiple valid interpretations, list them and ask which one applies.
- Present tradeoffs when they exist. If a simpler approach would work, say so before building the complex one.
- Push back on requests that will create maintenance burden, security risk, or unnecessary complexity.
- Stop and name what is unclear rather than filling gaps with guesses. A question now prevents a rewrite later.

### DO NOT

- Pick an interpretation silently and run with it. This is the single most common AI coding failure.
- Assume the user wants the most complete or most flexible solution. Default to the smallest thing that works.
- Start generating code before you understand the problem. Reading existing code and asking questions is not wasted time.

## Simplicity First

### DO

- Write the minimum code that solves the stated problem. Nothing speculative.
- If 200 lines could be 50, rewrite it at 50.
- Use concrete implementations over abstractions until a pattern repeats at least three times.

### DO NOT

- Add features, configurability, or extensibility beyond what was asked.
- Create helpers, utilities, or wrapper abstractions for one-time operations.
- Add error handling for scenarios that cannot occur given the system's constraints.
- Add type annotations, comments, or docstrings to code you did not change.

### The test

Would a senior engineer say "this is overcomplicated"? If yes, simplify.

## Surgical Changes

### DO

- Ensure every changed line traces directly to the user's request.
- Match existing code style, even if you would do it differently.
- Remove imports, variables, or functions that YOUR changes made unused.
- If you notice unrelated issues (dead code, style inconsistencies, potential bugs), mention them in your response — do not fix them silently.

### DO NOT

- "Improve" adjacent code, comments, formatting, or naming that you were not asked to touch.
- Refactor things that are not broken.
- Delete pre-existing dead code unless explicitly asked.
- Add backwards-compatibility shims, `// removed` comments, or renamed `_unused` variables. If something is unused because of your change, delete it cleanly.

## Close the Loop

Design for verifiability. The reason AI is effective at coding is that code can be compiled, tested, and checked. Lean into that.

### DO

- Define verifiable success criteria before starting work. "It works" is not a success criterion. "These 5 tests pass" is.
- Transform vague tasks into testable goals:

| Instead of... | Transform to... |
|---|---|
| "Add validation" | "Write tests for invalid inputs, then make them pass" |
| "Fix the bug" | "Write a test that reproduces it, then make it pass" |
| "Refactor X" | "Ensure tests pass before and after" |

- For multi-step tasks, state a brief plan with verification at each step:

```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

- Run tests after every meaningful change. Do not batch up changes and test at the end.

### DO NOT

- Declare work complete without running verification. "Should work" is not verified.
- Skip verification under time pressure. Unverified code creates more work than it saves.

## Agent-Ergonomic Design

When structuring code, consider that AI agents are now primary consumers of the codebase alongside humans.

### DO

- Use clear, descriptive file and directory names. Agents navigate by name before reading content.
- Keep modules self-contained with explicit interfaces. Agents work better with bounded contexts than with tangled dependencies.
- Maintain reference implementations for recurring patterns. Pointing an agent to "do it like `src/features/auth/`" is more effective than explaining the pattern from scratch.
- Write documentation that explains WHY decisions were made, not just WHAT the code does. Agents can read code; they cannot infer the reasoning behind architectural choices.

### DO NOT

- Rely on implicit conventions that only team members would know. Encode them in the codebase or in these standard files.
- Scatter related logic across many files when co-location would be clearer for both humans and agents.

## Sources

- Behavioural principles adapted from [Karpathy-Inspired Claude Code Guidelines](https://github.com/forrestchang/andrej-karpathy-skills) (Think Before Coding, Simplicity First, Surgical Changes, Goal-Driven Execution).
- Close the Loop and Agent-Ergonomic Design principles derived from Peter Steinberger's development workflow as discussed on the [Pragmatic Engineer Podcast](https://www.youtube.com/watch?v=8lF7HmQ_RgY).
