# browser-testing

A Claude Code skill for testing and debugging web UIs using Chrome DevTools MCP — accessibility-tree snapshots, form automation, network inspection, and navigation testing.

![Claude Code skill](https://img.shields.io/badge/Claude_Code-skill-blue)

## What it does

Drives a headless Chrome browser through accessibility-tree interaction. Finds elements by label, fills forms, clicks buttons, inspects network requests, and verifies page state — all without a visual display.

## Install

```bash
npx skills add browser-testing
```

Or clone manually:

```bash
git clone https://github.com/ccwriter369-beep/claude-skill-browser-testing \
  ~/.claude/skills/browser-testing
```

**Prerequisite:** Chrome DevTools MCP must be installed and running.

## Usage

```
test the UI
check if the form works
debug the API call
verify the page loaded correctly
browser testing
```

## Core workflow

```
snapshot → identify UIDs → interact → verify (network or snapshot)
```

**Critical:** UIDs change between snapshots. Always re-snapshot before interacting with elements.

## Common patterns

**Form submission test:**
```
1. take_snapshot                              # Find form elements
2. fill uid=X value="..."                     # Fill each field
3. click uid=Y includeSnapshot=true           # Submit + get new state
4. list_network_requests resourceTypes=["fetch","xhr"]
5. get_network_request reqid=N                # Check request/response body
```

**Debug failed request:**
```
1. list_network_requests resourceTypes=["fetch","xhr"]
2. Find failed request (look for 4xx/5xx status)
3. get_network_request reqid=N
4. Inspect request headers, body, response
```

## When NOT to use

- **WebGL / 3D components** (Three.js, Canvas) — will fail, no GPU context in headless Chrome. Use the `desktop-automation` skill instead.
- **Visual / CSS debugging** — snapshots show structure, not styling. Use screenshots.
- **Performance profiling** — use `performance_start_trace` / `performance_stop_trace` directly.

## License

MIT
