# Contribution 1: Multiple errors trying to fetch lyrics during import

**Contribution Number:** 1
**Student:** Hoang (Chris) Pham
**Issue:** https://github.com/beetbox/beets/issues/5998
**Status:** Phase III — Complete

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

> **Confirmed in Phase II:** The root cause is specifically a **User-Agent ban** on the tekstowo backend (HTTP 403), not a parsing failure. The dramatic `AttributeError`/`JSONDecodeError` at the end of the original report turned out to be an *outdated `requests` package* in the reporter's environment, not a beets code defect (details below).

---

## Reproduction Process

### Environment Setup

Local environment: macOS (Darwin), Python 3.12.1. beets installed from my fork as an editable install.

Challenges I hit and how I solved them:

- **beets uses Poetry, but I didn't have Poetry installed.** Instead of installing Poetry globally, I made a virtualenv and did an editable install with the `lyrics` extra, which is enough to run and import the plugin code:
  ```bash
  python3 -m venv .venv
  .venv/bin/python -m pip install -e '.[lyrics]'
  .venv/bin/python -m pip install pytest requests-mock   # for the Phase III test
  ```
- **The tekstowo source is disabled by default.** Current beets disables both `musixmatch` and `tekstowo` by default because they block the beets user-agent, so I had to explicitly enable it (`sources: [tekstowo]`) to trigger the bug.
- **The scariest-looking error is *not* a beets code bug.** The original report ends in `AttributeError: module 'requests' has no attribute 'JSONDecodeError'`. That attribute has existed since `requests` 2.27, and beets pins `requests = ">=2.32.5"` in `pyproject.toml`. So on any supported install the error handler catches the failure and logs a single warning. The reporter had a broken mixed snap + pip install with an outdated `requests` (a maintainer reached the same conclusion in the thread). I reproduced both cases explicitly to prove this.

**Working branch:** https://github.com/chrispham-cyber/beets/tree/fix-issue-5998

### Steps to Reproduce

1. Clone the fork and install in a venv:
   ```bash
   git clone https://github.com/chrispham-cyber/beets.git
   cd beets
   python3 -m venv .venv
   .venv/bin/python -m pip install -e '.[lyrics]'
   ```
2. Confirm the user-agent is what gets banned (pure HTTP, no beets needed):
   ```bash
   URL="https://www.tekstowo.pl/szukaj,School+Supertramp.html"
   curl -s -o /dev/null -w "%{http_code}\n" -A "beets/2.11.0 https://beets.io/" "$URL"  # 403
   curl -s -o /dev/null -w "%{http_code}\n" -A "" "$URL"                                # not blocked
   ```
3. Reproduce through beets' own request code with the script on the branch:
   ```bash
   .venv/bin/python reproduce_issue_5998.py
   ```
4. **Expected:** a failed lyrics source produces one warning and the import continues.
5. **Actual:** with the beets `User-Agent`, tekstowo returns **403 Forbidden**. On a supported `requests` the plugin logs a single warning (correct); on the reporter's outdated `requests` the error handler itself raised `AttributeError`, which aborted the entire import.

### Reproduction Evidence

The `User-Agent` header is the decisive variable (live results, 2026-06-17):

| User-Agent sent | tekstowo.pl response |
| --- | --- |
| `beets/2.11.0 https://beets.io/` | **403 Forbidden** |
| *(empty)* | 200 (not blocked) |
| browser string | 200 (not blocked) |

Output of `reproduce_issue_5998.py`, which drives beets' own `TimeoutAndRetrySession` and `LyricsRequestHandler`:

```
beets uses requests 2.34.2 (JSONDecodeError present: True)

1. tekstowo blocks the beets User-Agent
   User-Agent sent by beets: 'beets/2.11.0 https://beets.io/'
   -> reproduced: HTTP 403 for https://www.tekstowo.pl/szukaj,School+Supertramp.html

2. Empty User-Agent avoids the ban (proposed fix)
   -> success: HTTP 200 (final url: https://www.tekstowo.pl/szukaj/School+Supertramp)

3a. Supported install: failure becomes one warning
   [WARNING] _Handler: Request error: 403 Client Error: Forbidden
   -> import continued; only a warning was logged (no crash)

3b. Outdated requests: the handler itself crashes
   -> reproduced original crash: AttributeError: module 'requests' has no attribute 'JSONDecodeError'
```

Reproduction script: https://github.com/chrispham-cyber/beets/blob/fix-issue-5998/reproduce_issue_5998.py

**My findings:** The errors during import come from tekstowo banning the beets `User-Agent`. Sending an empty `User-Agent` avoids the 403. The `AttributeError` that aborted the original reporter's import is an outdated-`requests` environment issue, not a code bug on supported installs — so the code fix should focus on the tekstowo user-agent.

---

## Solution Approach

### Implementation Plan (UMPIRE)

**Understand.** When the `tekstowo` source is enabled, `tekstowo.pl` blocks the beets `User-Agent` and returns HTTP 403, so lyrics never download from that source and the import is cluttered with errors. The fix is to stop sending the banned agent for tekstowo so its requests succeed (and, when they don't, fail as the single clean warning the handler already produces). The separate `AttributeError`/`JSONDecodeError` symptom is an out-of-date-`requests` environment problem, so it is intentionally **out of scope** for the code change.

**Match.** The codebase already customizes outgoing HTTP headers per backend: the `Genius` backend overrides a `headers` property to add an `Authorization` header (`beetsplug/lyrics.py:623-625`). I'll apply the same per-backend idea to the `User-Agent`, overriding it only for the `Tekstowo` backend — the same approach a maintainer (snejus) sketched in the issue thread.

**Plan.**
1. In `beetsplug/lyrics.py`, add a `get()` override to the `Tekstowo` backend class (`Tekstowo` at line 653; `build_url` at 659, `search` at 664) that sends an empty `User-Agent`:
   ```python
   def get(self, *args, **kwargs):
       kwargs.setdefault("headers", {})
       kwargs["headers"]["User-Agent"] = ""
       return super().get(*args, **kwargs)
   ```
   This covers all tekstowo HTTP calls because they flow through the shared `get_text` → `get()` path (`beetsplug/lyrics.py:207`).
2. Add a regression test in `test/plugins/test_lyrics.py` next to the existing `TestTekstowoLyrics` (line 565), using `requests_mock` (already used in this file, e.g. `test_error_handling`) to assert the outgoing tekstowo request carries an empty `User-Agent`.
3. Add a changelog entry under **Bug fixes** in `docs/changelog.rst`, e.g.
   `` - :doc:`plugins/lyrics`: Avoid the tekstowo HTTP 403 by not sending the beets user-agent. :bug:`5998` ``

**Files I expect to touch.**
- `beetsplug/lyrics.py` — the `Tekstowo.get()` override (the fix).
- `test/plugins/test_lyrics.py` — regression test for the empty user-agent.
- `docs/changelog.rst` — bug-fix changelog entry.

**Implement.** To be done in Phase III on the working branch:
https://github.com/chrispham-cyber/beets/tree/fix-issue-5998

**Review.** beets' `CONTRIBUTING.rst` says a contribution has four parts: code, tests, documentation, and a changelog entry. Before opening the PR I will self-review the diff, run formatting/linting and the test suite (`poe test`), add the changelog entry, and reference the issue (`Fixes #5998`) in the PR description.

**Evaluate.**
- New test passes: the tekstowo request sends an empty `User-Agent`.
- Manual check: re-running `reproduce_issue_5998.py` step 1 should no longer return 403 for the tekstowo request (it returns 200, which step 2 already demonstrates for an empty agent).
- The full `lyrics` test suite still passes (no regressions in other backends).

> **Scope note / honest finding.** Two maintainers disagree on the long-term direction — one prefers the empty-user-agent workaround (this plan), the other prefers removing the bit-rotting `tekstowo` backend entirely because spoofing the agent is "cat and mouse." I'm proceeding with the smaller, lower-risk workaround and will confirm direction with maintainers on the issue before finalizing the PR. The `AttributeError` from the original report reproduces only on outdated `requests` and is therefore an environment issue, not part of this fix.

---

## Implementation Notes

**What I built.** I implemented the Phase II plan as-is — no surprises, because the reproduction had already pinned the root cause.

- **The fix** (`beetsplug/lyrics.py`): added a `get()` override to the `Tekstowo` backend that sets an empty `User-Agent` on its requests, so tekstowo no longer returns HTTP 403. It reuses the existing `RequestHandler.get()` path, so the change is six lines:
  ```python
  def get(self, *args, **kwargs):
      # Tekstowo blocks the default beets user-agent with HTTP 403, so send
      # an empty one instead. See beetbox/beets#5998.
      kwargs.setdefault("headers", {})
      kwargs["headers"]["User-Agent"] = ""
      return super().get(*args, **kwargs)
  ```
- **The test** (`test/plugins/test_lyrics.py`): added `TestTekstowoUserAgent.test_request_omits_beets_user_agent`, which mocks the tekstowo URL with `requests_mock` and asserts the outgoing request's `User-Agent` header is empty.
- **The changelog** (`docs/changelog.rst`): added a bug-fix entry referencing `:bug:`5998``.

**Files modified:**
- `beetsplug/lyrics.py` — the `Tekstowo.get()` override.
- `test/plugins/test_lyrics.py` — the regression test.
- `docs/changelog.rst` — the bug-fix entry.

### Challenges Faced

- **Telling the real bug apart from an environment problem.** The original report's most dramatic symptom (`AttributeError: module 'requests' has no attribute 'JSONDecodeError'`) is *not* a code bug: beets pins `requests >= 2.32.5`, where that attribute exists. The reporter had an outdated `requests` in a broken mixed snap + pip install. I confirmed this by simulating an old `requests` in my reproduction script. Conclusion: keep the fix narrowly on the tekstowo user-agent.
- **No Poetry locally.** beets uses Poetry, which I didn't have. I used a plain venv + editable install (`pip install -e '.[lyrics]'`), which is enough to run the plugin and the tests.
- **Confirming the test actually captures the header.** I verified with `requests_mock.last_request.headers["User-Agent"] == ""`, then ran `ruff format`, which reformatted only my new test (the rest of the diff stayed scoped to my changes).

### Testing Strategy

- **New regression test passes:**
  ```
  test/plugins/test_lyrics.py::TestTekstowoUserAgent::test_request_omits_beets_user_agent PASSED
  ```
- **No regressions:** the full lyrics suite passes — `113 passed, 15 skipped` (the skips are network-gated integration tests that don't run by default).
- **Style gate:** `ruff check` passes and both changed files are `ruff format`-clean (beets' enforced style).
- **Behavioral check:** re-running `reproduce_issue_5998.py` shows the empty user-agent now gets HTTP 200 from tekstowo instead of 403.

### Code Changes

Working branch: https://github.com/chrispham-cyber/beets/tree/fix-issue-5998

Commits on the branch:
- `ac81a94` — `lyrics: avoid tekstowo HTTP 403 by sending an empty user-agent` (fix + changelog)
- `f5eca62` — `lyrics: test that tekstowo requests omit the beets user-agent`
- `c0416b4` — reproduction script (Phase II evidence; will be removed before opening the PR in Phase IV)

> Next (Phase IV): confirm the empty-user-agent direction with maintainers on the issue, drop the reproduction script from the branch, and open the pull request with `Fixes #5998`.
