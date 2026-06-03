# Contribution Journal

A journal documenting my journey contributing to open source, starting with my first selected issue.

## Selected Issue

**Project:** Open WebUI (`open-webui/open-webui`)

**Issue:** [#24577 — "Ask" button renders the box out of the view](https://github.com/open-webui/open-webui/issues/24577)

**Status:** Open · No assignee · No linked pull request

**Target timeline:** 1-5 days to claim and scope; full fix within ~3-4 weeks

---

### What is the issue?

When you select a word near the far-left edge of a chat message and click the "Ask" action, the resulting "Ask a question" input box renders partly off-screen instead of staying fully within the viewport.

### Why does it matter?

It is a visible usability bug in a core interaction. When the popup is clipped, users near the left edge of a message cannot read or use part of the input box, which degrades the "Ask about selection" feature for anyone working on narrow viewports or selecting text at the screen edge.

### Why did I choose it?

It scores well against a beginner-contributor checklist: it is understandable in one sentence, it is a small and self-contained frontend (Svelte + CSS) positioning fix rather than an architectural change, it has no assignee and no linked PR, it has fewer than five comments, and it includes clear reproduction steps. A maintainer (collaborator `silentoplayz`) commented that they could not reproduce it, which makes confirming the exact reproduction conditions a meaningful and well-scoped first contribution. Open WebUI also has clear setup and developer documentation.

### Why am I a good fit to work on it?

My background is in cybersecurity, Python, AI tooling, and bug hunting. This issue sits squarely in the bug-hunting and frontend-debugging space: it requires careful reproduction (browser, viewport width, and selection position), reading the popup-positioning logic, and applying a focused fix. It is a realistic match for skills I already have or can pick up quickly.

### What does a successful fix look like?

Reliably reproduce the clipped popup near the left edge, then update the popup's positioning logic so the "Ask a question" box is always clamped within the visible viewport (no overflow on the left or right edge), verified across browsers. The change is accompanied by a clear description of the reproduction steps and, where appropriate, a test or before/after screenshots.
