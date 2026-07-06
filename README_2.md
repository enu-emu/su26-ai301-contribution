# Contribution [2]: [Improve "number of ntapes differs from restart file" error msg #3593]

**Contribution Number:** 2
**Student:** Ena Salazar  
**Issue:** [GitHub issue link](https://github.com/ESCOMP/CTSM/issues/3593)
**Status:** Phase I Complete

---

## Why I Chose This Issue

I chose this issue because it is open, unassigned, labeled `good first issue` and `size: small`, and has no linked pull requests. The requested change is narrow: improve an unclear error message so users know they should use a branch run instead of changing history options during a restart. This matters because better error messages help users understand not only what went wrong, but what to do next. Since I have already completed one pull request, this issue seems like a manageable next step that still lets me practice working in a real source-code file.

---

## Understanding the Issue

### Problem Description

When a user changes the number of history files in the middle of a restart run with `CONTINUE_RUN=TRUE`, CTSM gives an error saying that the number of `ntapes` differs from the restart file. The current message explains that history options cannot be changed on restart, but it does not explain what the user should do instead.

### Expected Behavior

The error message should tell the user that if they need to change history-file options, they should use a branch run instead of trying to change those options during a restart run.

### Current Behavior

The current error message says:

```text
ENDRUN: ERROR in histFileMod.F90 at line 4886
ERROR: number of ntapes differs from restart file. You can NOT change history
options on restart.
```

This tells the user what went wrong, but it does not tell them the recommended next step.

### Affected Components

The issue points to histFileMod.F90, specifically the error message related to history-file settings and restart files.

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
