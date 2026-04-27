# Open Claudex Computer Use

Open-source background computer use for Claude and Codex on macOS.

Open Claudex Computer Use is a local-first MCP server and native Swift runtime that lets AI agents drive Mac apps without stealing your visible cursor, forcing focus changes, or requiring a separate virtual desktop.

**Status:** `0.1.0-alpha`

- 23 MCP tools for app state, clicks, typing, scrolling, dragging, desktop sessions, and virtual cursor control
- Mixed-mode execution: Accessibility first, CGEvent fallback when AX is weak
- Same-process live virtual cursor for demos, observation, and recording
- NDJSON + Content-Length stdio support for Claude Code, Codex, Cursor, and other MCP clients

## Why This Exists

Most open computer-use projects are browser-first or VM-first. This project is macOS-native and app-first: it works directly against real desktop apps, keeps execution local, and is designed for background operation.

The honest boundary is the platform boundary:

- Native AppKit and SwiftUI apps are the strongest path
- Electron and WebView-heavy apps are workable with fallbacks
- Self-drawn apps like WeChat and some Feishu surfaces remain best-effort

## Quick Start

```bash
# Build
swift build

# Add to Claude Code
claude mcp add claudex-computer-use -- /path/to/open-claudex-computer-use/.build/debug/claudex-computer-use

# Verify
python3 scripts/smoke-mcp.py
python3 scripts/smoke-mcp-ndjson.py
```

Requires macOS 13+, Accessibility permission, and Screen Recording permission. Full setup is in [docs/install.md](docs/install.md).

## Core Capabilities

- `get_app_state`: numbered AX tree, screenshot, and app-specific guidance
- `click`, `scroll`, `drag`, `press_key`, `type_text`, `set_value`: mixed semantic and coordinate interaction paths
- `acquire_desktop`, `desktop_status`, `release_desktop`: session control for background automation
- `get_virtual_cursor`, `set_virtual_cursor`: demo and observability controls
- `doctor`, `list_apps`, `capture_window`, `find_ui_element`, `press_element`: diagnostics and fallback tooling

Mutation tools return post-action state, so the client can often keep going without forcing a fresh `get_app_state` after every step.

## App Support

| Tier | Apps | What Works |
|------|------|-----------|
| **Stable** | Safari, Notes, TextEdit, Calculator, Finder, System Settings | Full AX tree, semantic clicks, screenshots, strong `set_value` support |
| **Limited** | Chrome, Edge, VS Code, Slack, Cursor | Partial AX, coordinate fallback, pasteboard-heavy typing |
| **Best-effort** | WeChat, Feishu/Lark, self-drawn apps | Sparse AX, fragile focus, less reliable CGEvent delivery |

See [docs/compatibility.md](docs/compatibility.md) for the full matrix and strategy guidance.

## Virtual Cursor

The default desktop overlay is a same-process visual cursor intended for demos, recording, and human trust. It is independent from the real system cursor and can be combined with screenshot overlays when you need traces for debugging.

Common presets:

```text
set_virtual_cursor(preset="codexDemo", clear=true)
set_virtual_cursor(preset="debugTrace", clear=true)
set_virtual_cursor(mode="hybrid", style="secondCursor", show_trail=false)
```

## Architecture

- `ClaudexComputerUseCore`: native execution layer for AX, CGEvent, ScreenCaptureKit, sessions, and cursor overlays
- `claudex-computer-use`: stdio MCP server
- `plugins/claudex-computer-use`: repo-local Codex plugin wrapper

This repo is not an agent harness. It is the execution engine that other harnesses plug into.

## Docs

- [Installation & Integration](docs/install.md)
- [App Compatibility Matrix](docs/compatibility.md)
- [Testing](docs/testing.md)
- [Demo Pack](docs/demos.md)
- [Codex Native Trace Kit](docs/codex-native-trace-kit.md)
- [Roadmap](ROADMAP.md)

## License

[MIT](LICENSE)
