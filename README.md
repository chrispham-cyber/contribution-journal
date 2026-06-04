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
