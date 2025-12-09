## Review Philosophy

- Only comment when you have HIGH CONFIDENCE (>80%) that an issue exists
- Be concise: one sentence per comment when possible
- Focus on actionable feedback, not observations
- When reviewing text, only comment on clarity issues if the text is genuinely confusing or could lead to errors.

## Priority Areas (Review These)
 
### Security & Safety
- Unsafe code blocks without justification
- Command injection risks (shell commands, user input)
- Path traversal vulnerabilities
- Credential exposure or hardcoded secrets
- Missing input validation on external data
- Improper error handling that could leak sensitive info
 
### Correctness Issues
- Logic errors that could cause panics or incorrect behavior
- Race conditions in async code
- Resource leaks (files, connections, memory)
- Off-by-one errors or boundary conditions
- Incorrect error propagation (using `unwrap()` inappropriately)
- Optional types that don’t need to be optional
- Booleans that should default to false but are set as optional
- Error context that doesn’t add useful information (e.g., `.context("Failed to do X")` when error already says it failed)
- Overly defensive code that adds unnecessary checks
- Unnecessary comments that just restate what the code already shows (remove them)
 
### Architecture & Patterns
- Code that violates existing patterns in the codebase
- Missing error handling (should use `anyhow::Result`)
- Async/await misuse or blocking operations in async contexts
- Improper trait implementations

## Skip These (Low Value)
 
Do not comment on:
- **Minor naming suggestions** - unless truly confusing
- **Suggestions to add comments** - for self-documenting code
- **Refactoring suggestions** - unless there’s a clear bug or maintainability issue
- **Multiple issues in one comment** - choose the single most critical issue
- **Logging suggestions** - unless for errors or security events (the codebase needs less logging, not more)
- **Pedantic accuracy in text** - unless it would cause actual confusion or errors. No one likes a reply guy

## Response Format

- Add issues to a "REVIEW-LOG.MD" file
- When you identify an issue:
1. **State the problem** (1 sentence)
2. **Why it matters** (1 sentence, only if not obvious)
3. **Suggested fix** (code snippet or specific action)
 
Example:
This could panic if the vector is empty. Consider using `.get(0)` or add a length check.

## When to Stay Silent
 
If you’re uncertain whether something is an issue, don’t comment. False positives create noise and reduce trust in the review process.
