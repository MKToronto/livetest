# livetest — Claude Code Plugin

A [Claude Code](https://claude.com/claude-code) plugin that makes Claude **test a feature in the actual running app** — driving a real browser via the Playwright MCP server, inspecting live app state, verifying with evidence, and cleaning up after — instead of only reasoning about whether the code is correct.

App-agnostic: works for any web app, dashboard, or API. Not tied to any one project.

## Installation

```
/plugin marketplace add MKToronto/livetest
/plugin install livetest@livetest
```

Then restart Claude Code (`/exit` followed by `claude`) to load the plugin.

For local testing without installing:

```sh
git clone https://github.com/MKToronto/livetest.git
claude --plugin-dir /path/to/livetest
```

### Updating

```
/plugin marketplace update livetest
```

## Requirements

This plugin drives a real browser through the **Playwright MCP server**, so that server must be configured in Claude Code. It provides the `mcp__playwright__browser_*` tools the command uses.

Add it once (user scope, available in every project):

```sh
claude mcp add playwright npx @playwright/mcp@latest
```

No separate browser-install step is needed — the server sets up its browser on first use. (If you ever need to install browsers manually: `npx playwright install`.)

Verify it registered:

```sh
claude mcp list
```

The app under test must also be **running locally** — Claude confirms it's listening before driving it and won't launch it for you unless you tell it to.

## Usage

```
/livetest <what to test>
```

- `/livetest the date-range filter` — test a specific thing.
- `/livetest` (no argument) — test whatever was just worked on this session, inferred from recent edits + the conversation.

## What it does

Claude follows a fixed playbook:

1. **Find the running app** — looks up the dev/start command and port, confirms it's listening, then navigates a real browser to it.
2. **Reach the feature through the real UI** — interacts with the real controls (or hits the API directly for backend work), exercising the same code path a user does.
3. **Inspect live state** — reads the app's real state (Svelte stores / Redux / exported getters / DOM / network), not just pixels.
4. **Verify with evidence** — asserts concrete values (counts, flags, response bodies) plus a screenshot and the console error log.
5. **Clean up** — undoes test state, cancels anything it started, removes screenshots and `.playwright-mcp`.
6. **Report honestly** — states what was verified with evidence, and explicitly what was *not* exercised.

The full playbook lives in [`commands/livetest.md`](./commands/livetest.md).

## Safety

livetest will **not** trigger irreversible or outward-facing actions against live systems — no real submissions, sent messages, payments, deletes, or writes to production — unless you've explicitly authorized that action for the session. Destructive or external steps are stubbed, fixtured, or reported short of execution.

## License

MIT — see [LICENSE](./LICENSE).
