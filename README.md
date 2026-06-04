# Contribution 1: Multiple errors trying to fetch lyrics during import

**Contribution Number:** 1
**Student:** Hoang (Chris) Pham
**Issue:** https://github.com/beetbox/beets/issues/5998
**Status:** Phase I — In Progress

---

## Why I Chose This Issue

I chose this issue because it sits squarely in my main area of focus — Python — and it is a concrete, well-scoped bug rather than a vague feature or architectural task. The beets project is a mature, widely-used music library manager with a clean codebase, a real test suite, and clear contribution documentation, which makes it a good environment to learn real-world open-source workflows. The bug is about the lyrics plugin throwing multiple errors when fetching lyrics during a music import, which is understandable in a single sentence and is the kind of error-handling problem I am comfortable debugging.

It also matches my learning goals well: I want to get better at reading other people's Python code, tracing exceptions through a plugin architecture, and writing tests that prove a bug is fixed. The issue is labeled "bug" and "good first issue," has no assignee and no open pull request, and has helpful discussion from other users and maintainers — exactly the criteria for a realistic first contribution that I can finish within a few weeks.

---

## Understanding the Issue

### Problem Description
When a user runs an import with the lyrics plugin enabled (on beets v2.3.0), lyrics fail to fetch and the import produces multiple errors instead of either succeeding or failing quietly. The discussion traces the failures largely to the "tekstowo" lyrics backend, which appears to be blocking requests (user-agent blocking) and/or returning content the plugin can no longer parse. The result is a noisy, broken import experience rather than a clean "lyrics not found" outcome. This matters because lyrics fetching is a popular plugin feature, and a single failing backend should degrade gracefully rather than spam errors during every import.

### Expected Behavior
Lyrics fetching should either succeed, or fail gracefully with a single clean, non-fatal warning, without throwing multiple unhandled errors during the import process.

### Current Behavior
With the lyrics plugin enabled, importing an album surfaces several errors while attempting to fetch lyrics (notably from the tekstowo backend), cluttering the verbose import output and preventing lyrics from being saved.

### Affected Components
Most likely the lyrics plugin in beetsplug/lyrics.py and its per-backend source handlers (especially the tekstowo backend), plus the related error handling that decides how a failed lyrics source is reported during import. To be confirmed during reproduction.

---

## Reproduction Process

### Environment Setup
[Notes on setting up a local beets development environment — Python version, virtualenv, installing in editable mode, enabling the lyrics plugin. To be filled in as I work.]

### Steps to Reproduce
1. Install/check out beets with the lyrics plugin enabled and at least one affected source (e.g. tekstowo) configured.
2. Run a verbose import on a sample album: `beet -vv import "/path/to/album"`.
3. Observe the multiple lyrics-fetch errors in the output and that lyrics are not saved.

### Reproduction Evidence
- **Commit showing reproduction:** [Link to commit in my fork]
- **Screenshots/logs:** [If applicable]
- **My findings:** [What I discovered during reproduction]

---

## Solution Approach

### Analysis
[My analysis of the root cause — what's causing the errors. To be completed.]

### Proposed Solution
[High-level description of my fix approach. To be completed.]

### Implementation Plan
Using UMPIRE framework (adapted):

**Understand:** [Restate the problem]

**Match:** [What similar error-handling patterns exist elsewhere in the beets codebase?]

**Plan:** [Step-by-step implementation plan]
1. [Modify the lyrics plugin to handle the failing source gracefully]
2. [Add/adjust error handling so failures don't raise during import]
3. [Update tests]

**Implement:** [Link to my branch/commits as I work]

**Review:** [Self-review checklist — does it follow beets' contribution guidelines?]

**Evaluate:** [How I'll verify it works]

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
[What I tested manually and the results]

---

## Implementation Notes

### Week 1 Progress
[What I built this week, challenges faced, decisions made]

### Week 2 Progress
[Continue documenting as I work]

### Code Changes
- **Files modified:** [List]
- **Key commits:** [Links to important commits]
- **Approach decisions:** [Why I chose certain approaches]

---

## Pull Request

**PR Link:** [GitHub PR URL when submitted]

**PR Description:** [Draft or final PR description — much of the content above can be adapted]

**Maintainer Feedback:**
- [Date]: [Summary of feedback received]
- [Date]: [How I addressed it]

**Status:** [Awaiting review / Iterating / Approved / Merged]

---

## Learnings & Reflections

### Technical Skills Gained
[What I learned technically]

### Challenges Overcome
[What was hard and how I solved it]

### What I'd Do Differently Next Time
[Reflection on my process]

---

## Resources Used
- [Link to helpful documentation]
- [Tutorial or Stack Overflow post that helped]
- [GitHub issues or discussions that helped]
