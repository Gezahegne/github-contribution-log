# Contribution [#]: Provide email address for reaching PSF Board members
 
**Contribution Number:** #14909 
**Student:** Gezahegne Yirefu  
**Issue:** https://github.com/ggml-org/llama.cpp/issues/14909 
**Status:** Phase I Complete

---

## Why I Chose This Issue

I chose the ABS operation for the OpenCL backend because it looks like a good beginner-friendly contribution to the llama.cpp codebase. The operation itself is simple to understand: it takes an input value and returns its absolute value. Since it is a basic unary operation, I feel I can focus more on learning how backend operations are implemented in llama.cpp without getting overwhelmed by a very complex algorithm.

This issue also matches my current learning goals in the CodePath AI301 program. I want to get more comfortable reading a large open-source C/C++ codebase, understanding how different backends are organized, and making a small but meaningful contribution. Implementing ABS for OpenCL will help me learn how ggml operations are mapped to backend-specific kernels, how to test backend support using test-backend-ops, and how to prepare a focused pull request.

I also chose this item because it appears to be available: it is marked as not implemented in the backend support table, and I did not find an existing open request or pull request specifically for ABS in OpenCL. This makes it a good issue to claim because the scope is clear, the expected result is testable, and the contribution would fill a real missing piece in the project.

---

## Understanding the Issue

### Problem Description

[In your own words, what's broken or missing?]

### Expected Behavior

[What should happen?]

### Current Behavior

[What actually happens?]

### Affected Components

[Which parts of the codebase are involved?]

---

## Reproduction Process

### Environment Setup

[Notes on setting up your local development environment - challenges you faced, how you solved them]

### Steps to Reproduce

1. [Step 1]
2. [Step 2]
3. [Observed result]

### Reproduction Evidence

- **Commit showing reproduction:** [Link to commit in your fork]
- **Screenshots/logs:** [If applicable]
- **My findings:** [What you discovered during reproduction]

---

## Solution Approach

### Analysis

[Your analysis of the root cause - what's causing the issue?]

### Proposed Solution

[High-level description of your fix approach]

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** [Restate the problem]

**Match:** [What similar patterns/solutions exist in the codebase?]

**Plan:** [Step-by-step implementation plan]
1. [Modify file X to do Y]
2. [Add function Z]
3. [Update tests]

**Implement:** [Link to your branch/commits as you work]

**Review:** [Self-review checklist - does it follow the project's contribution guidelines?]

**Evaluate:** [How will you verify it works?]

---

## Testing Strategy

### Unit Tests

- [ ] Test case 1: [Description]
- [ ] Test case 2: [Description]
- [ ] Test case 3: [Description]

### Integration Tests

- [ ] Integration scenario 1
- [ ] Integration scenario 2

### Manual Testing

[What you tested manually and results]

---

## Implementation Notes

### Week [X] Progress

[What you built this week, challenges faced, decisions made]

### Week [Y] Progress

[Continue documenting as you work]

### Code Changes

- **Files modified:** [List]
- **Key commits:** [Links to important commits]
- **Approach decisions:** [Why you chose certain approaches]

---

## Pull Request

**PR Link:** [GitHub PR URL when submitted]

**PR Description:** [Draft or final PR description - much of the content above can be adapted]

**Maintainer Feedback:**
- [Date]: [Summary of feedback received]
- [Date]: [How you addressed it]

**Status:** [Awaiting review / Iterating / Approved / Merged]

---

## Learnings & Reflections

### Technical Skills Gained

[What you learned technically]

### Challenges Overcome

[What was hard and how you solved it]

### What I'd Do Differently Next Time

[Reflection on your process]

---

## Resources Used

- [Link to helpful documentation]
- [Tutorial or Stack Overflow post that helped]
- [GitHub issues or discussions that helped]
