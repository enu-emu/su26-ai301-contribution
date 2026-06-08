# su26-ai301-contribution

# Contribution 1: Add documentation on how to run the Burr UI server

**Contribution Number:** 1  
**Student:** Ena Salazar
**Issue:** [apache/burr #272 — Add documentation on how to run the Burr UI server](https://github.com/apache/burr/issues/272)  
**Status:** Phase I — Complete

---

## Why I Chose This Issue

I chose this issue because it felt like a well-scoped first contribution. Documentation work lets me focus on understanding the project without needing to immediately dive into complex code changes. I'm also drawn to tools that work with LLMs, and Burr's UI server is a core part of what makes the framework usable in practice. Writing this documentation will force me to actually run and understand the tool end to end, which is exactly the kind of hands-on learning I'm looking for.

---

## Understanding the Issue

### Problem Description

Burr has a built-in UI server that lets you visualize and debug your AI application's execution in real time. However, there is no dedicated section in the documentation explaining how to install it, run it, or use it. New users have no clear reference point for getting the UI up and running.

### Expected Behavior

The documentation should have a dedicated section covering:

1. How to install the UI server (`pip install "apache-burr[start]"`)
2. How to run it (`burr` command)
3. Optional commands and flags (e.g. creating demo data)
4. What you can do with the UI: browse runs by project, inspect individual steps, replay state for debugging, etc.

### Current Behavior

The only mention of the UI server lives in `docs/concepts/tracking.rst`, where it gets a single line: "run `burr` in the terminal." There is no installation guidance, no explanation of optional commands, and no description of what the UI actually shows or how to use it.

### Affected Components

- `docs/concepts/tracking.rst` — contains the only existing mention of the UI server
- Likely a new file: `docs/concepts/ui.rst` or an expanded section within `tracking.rst`

---

## Reproduction Process

### Environment Setup

- Forked `apache/burr` to `github.com/enu-emu/burr`
- Cloned fork locally into `~/CodePath_su26_ai301/burr`
- Added upstream remote: `git remote add upstream https://github.com/apache/burr.git`
- Created virtual environment: `python -m venv burr-fork && source burr-fork/bin/activate`
- Installed the package: `pip install -e .`
- Created working branch: `git checkout -b docs/burr-ui-server`

### Steps to Reproduce

1. Visit the [Burr documentation](https://burr.dagworks.io/)
2. Search for any dedicated page on the UI server — none exists
3. Check `docs/concepts/tracking.rst` — only a one-line mention of running `burr`
4. A new user trying to get the UI running has no complete reference

### Reproduction Evidence

- **Commit showing reproduction:** [To be added]
- **Screenshots/logs:** [To be added]
- **My findings:** The gap is confirmed. `tracking.rst` has a minimal mention but no dedicated UI documentation exists anywhere in the `docs/` folder.

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
