---
description: Drive the running app in a real browser (Playwright MCP) to test a feature end-to-end, then clean up. App-agnostic.
argument-hint: [what to test — omit to test whatever was just worked on]
allowed-tools: Bash, Read, Edit, mcp__playwright__browser_navigate, mcp__playwright__browser_evaluate, mcp__playwright__browser_snapshot, mcp__playwright__browser_take_screenshot, mcp__playwright__browser_console_messages
---

Live-test this feature by driving the running app in a real browser, then report what you actually observed.

**Target:** $ARGUMENTS

If the target above is empty, test **whatever was just worked on this session** — infer it from the recent edits / `git diff --stat` and the conversation, and state what you decided to test.

Playbook (adapt to whatever app and feature is in scope — web app, dashboard, API, etc.):

1. **Find the running app.** Look up its dev/start command and port from `package.json` scripts (or `README`, `Makefile`, compose file). Confirm it's actually listening (`lsof -nP -iTCP -sTCP:LISTEN` or `curl -sI` the URL). If it isn't running, say so and ask the user to start it rather than guessing — don't launch it yourself unless told to. Then `browser_navigate` to it. Note: the Playwright browser is a SEPARATE session from the user's own tab — you can't read theirs, only reproduce.

2. **Reach the feature through the real UI.** Navigate to the relevant screen and interact with the real controls (find them via `browser_evaluate` + `document.querySelector`, then click/type). Driving the real controls exercises the same code path the user does. For pure backend/API work, hit the endpoints directly with `fetch()` / `curl` and assert the responses.

3. **Inspect live state.** Read the app's real state, not just pixels:
   - Vite/ESM dev servers serve source modules — `await import('/src/…')` (or the app's module path) inside `browser_evaluate` gives the same singleton the UI uses (after a fresh page load). Read framework state from there: Svelte stores (`let v; const u = store.subscribe(x=>v=x); u();`), exported getters, a Redux store on `window`, React devtools hooks, etc.
   - Or assert via the DOM and network requests when no state module is reachable.
   - If state looks out of sync with the UI, reload the page — HMR can leave a stale module instance split-brained from the mounted one.

4. **Verify with state + DOM + a screenshot + console.** Assert concrete values (counts, flags, positions, response bodies) over eyeballing. `browser_take_screenshot` for the visual, and check `browser_console_messages` (error level) for runtime errors.

5. **SAFETY — never trigger irreversible or outward-facing actions against live systems.** No real submissions, sent messages, payments, deletes, or writes to production. If a test needs a destructive/external step, stub it, use a test fixture, or stop short and report — unless the user has explicitly authorized that action this session. Don't burn paid API/credits beyond what the test needs.

6. **Clean up after.** Undo any test state you created, cancel anything you started, and `rm -rf .playwright-mcp` plus any screenshots you saved. If your testing left the system in a churned state (e.g. queued jobs, test rows), say so and how to reset.

7. **Report honestly.** State what you verified with evidence, and explicitly what you did NOT exercise (e.g. "the queued/priority path was verified; an end-to-end submission was not driven").
