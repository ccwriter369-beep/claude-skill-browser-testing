---
name: browser-testing
description: Test and debug web UIs using Chrome DevTools MCP. Use when verifying form submissions, debugging API calls from the browser, testing navigation flows, or inspecting page state. Triggers on "test the UI", "check if the form works", "debug the API call", "verify the page", "browser testing", or when Chrome DevTools MCP tools are available and UI verification is needed.
---

# Browser Testing with Chrome DevTools MCP

Test web UIs through accessibility-tree snapshots, form automation, and network inspection.

## Core Workflow

```
snapshot → identify UIDs → interact → verify (network or snapshot)
```

**Critical**: UIDs change between snapshots. Always re-snapshot before interacting with elements.

## Tool Selection

| Goal | Tool | Notes |
|------|------|-------|
| See page structure/state | `take_snapshot` | Accessibility tree, not visual |
| See visual appearance | `take_screenshot` | Use for CSS/layout issues |
| Fill input/textarea | `fill` | Pass `uid` and `value` |
| Click button/link | `click` | Use `includeSnapshot: true` for efficiency |
| Check API calls | `list_network_requests` → `get_network_request` | Filter by `resourceTypes: ["fetch", "xhr"]` |
| Wait for content | `wait_for` | Wait for specific text to appear |
| Keyboard shortcuts | `press_key` | e.g., `Control+k`, `Meta+n`, `Escape` |
| Handle alert/confirm/prompt | `handle_dialog` | `action: "accept"` or `"dismiss"` |

## Pattern: Form Submission Test

```
1. take_snapshot                              # Find form elements
2. fill uid=X value="..."                     # Fill each field
3. click uid=Y includeSnapshot=true           # Submit + get new state
4. list_network_requests resourceTypes=["fetch","xhr"]  # Verify API called
5. get_network_request reqid=N                # Check request/response body
```

## Pattern: Navigation + Verify

```
1. take_snapshot                    # Find nav element
2. click uid=X includeSnapshot=true # Navigate + get new page
3. Verify expected content in snapshot
```

## Pattern: Debug Failed Request

```
1. list_network_requests resourceTypes=["fetch","xhr"]
2. Find failed request (look for 4xx/5xx status)
3. get_network_request reqid=N
4. Inspect: request headers, request body, response body
```

## Pattern: Discover Backend Wiring

Different routes in a UI may hit different backends (or none). To map what a page actually uses:

```
1. Navigate to page
2. list_network_requests resourceTypes=["fetch","xhr"]
3. Note which endpoints were called (or if none → mock/static data)
4. Repeat for each major route
```

Builds a map of UI route → API endpoint. Reveals if pages use real backend vs demo data.

## Pattern: Command Palette / Keyboard Shortcuts

```
1. press_key key="Control+k"           # Open command palette
2. Snapshot shows dialog with options
3. fill uid=X value="search term"      # Filter options
4. click uid=Y                         # Select option
5. If alert appears → handle_dialog action="accept"
```

Note: Use `Control` for Ctrl, `Meta` for Cmd (macOS). Escape closes most dialogs.

## Snapshot Interpretation

Snapshots show accessibility tree, not DOM:
- `button "Submit"` - clickable button with label
- `textbox "Email"` - input field
- `link "Home" url="..."` - navigation link
- `StaticText "..."` - visible text content
- `combobox expandable` - dropdown/select

**Find elements by**: label text, surrounding context, hierarchy position.

## Quirks

1. **UIDs are ephemeral** - Never cache UIDs across snapshots
2. **No CSS visibility** - Snapshots show structure, not styling. Use `take_screenshot` for visual bugs
3. **Form state** - After submission, form may retain values (not always reset)
4. **Network timing** - Request may not appear immediately; re-list if needed
5. **Toasts/notifications** - Often in a `region "Notifications"` or `list` at page bottom
6. **`wait_for` is exact match** - Text must match literally. "Loading complete" ≠ "Loading completed". When unsure, use `take_snapshot` instead and check manually
7. **WebGL/Canvas fails** - Three.js, Pixi.js, WebGL visualizations won't render in automated Chrome (no GPU context). Error: "Error creating WebGL context". Skip these components or use screenshots on real browser
8. **Dialogs block interaction** - Browser alerts/confirms/prompts must be handled with `handle_dialog` before any other action works

## When NOT to Use

- **Visual/CSS debugging** → Use screenshots + user description
- **Performance profiling** → Use `performance_start_trace` / `performance_stop_trace`
- **Console errors** → Use `list_console_messages` / `get_console_message`
- **3D/WebGL components** → Will fail; need real browser with GPU
