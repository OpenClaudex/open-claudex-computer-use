# Open Claudex Computer Use

**English | [简体中文](README.zh-CN.md)**

Open-source background computer use for Claude Code, Codex, and MCP agents on macOS.

[![Release](https://img.shields.io/github/v/release/OpenClaudex/open-claudex-computer-use?include_prereleases&label=release)](https://github.com/OpenClaudex/open-claudex-computer-use/releases)
[![License: MIT](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)
[![macOS](https://img.shields.io/badge/macOS-13%2B-black)](docs/install.md)
[![Swift](https://img.shields.io/badge/Swift-5.9%2B-orange)](Package.swift)
[![MCP](https://img.shields.io/badge/MCP-stdio-blue)](docs/install.md)

`claudex-computer-use` is a native Swift MCP server that lets AI agents inspect and operate real Mac apps through Accessibility, ScreenCaptureKit, and CGEvent fallbacks.

It is built for the gap between browser automation and full virtual machines:

- Works with real desktop apps, not just webpages.
- Runs locally on your Mac, with no cloud desktop required.
- Supports Claude Code, Codex App plugin workflows, Codex CLI-style MCP setups, Cursor, and generic MCP clients.
- Keeps a live virtual cursor overlay for demos, observation, and trust.
- Returns Codex-style post-action state so agents can keep acting without excessive re-snapshotting.

**Status:** `0.1.0-alpha`

> Not affiliated with Anthropic, OpenAI, Apple, or the official Codex Computer Use plugin.

## Demos

| Native App Control | Background Cross-App Work | Feishu / Lark Best-Effort |
|---|---|---|
| ![Native macOS Calculator demo](docs/assets/demo-calculator.gif) | ![Background Safari and Notes demo](docs/assets/demo-background-safari-notes.gif) | ![Sanitized Feishu and Lark demo](docs/assets/demo-feishu-lark.gif) |
| Click and read native macOS apps through Accessibility, with a visible virtual cursor. | Let the agent work in Safari and Notes while you keep using the Mac. | Operate WebView-heavy enterprise apps with mixed AX and coordinate fallbacks. Sanitized demo data only. |

## Origin

This project started as an experiment around two converging workflows:

- Codex-style background computer use, where an agent can inspect and operate Mac apps without stealing your real mouse.
- Claude Code-style extensibility, where external capabilities are exposed as MCP tools.

The missing piece was a reusable open-source execution layer: a local macOS MCP server that any agent harness can plug into. Open Claudex Computer Use is that layer.

## Why

Most computer-use projects are browser-first, VM-first, or harness-first. Open Claudex Computer Use is app-first:

| Approach | Best For | Tradeoff |
|---|---|---|
| Browser automation | Websites and web apps | Does not cover native Mac apps |
| Remote VM / virtual desktop | Isolation and reproducibility | Heavy setup, not your real desktop |
| This project | Real local macOS apps | Bound by macOS Accessibility quality |

The goal is not to hide the platform boundary. The goal is to expose it cleanly through a practical MCP contract.

## Quick Start

```bash
git clone https://github.com/OpenClaudex/open-claudex-computer-use.git
cd open-claudex-computer-use
swift build
```

Run smoke tests:

```bash
python3 scripts/smoke-mcp.py
python3 scripts/smoke-mcp-ndjson.py
python3 scripts/test-stale-guard.py
```

Requires:

- macOS 13.0+
- Swift 5.9+ / Xcode 15+
- Accessibility permission
- Screen Recording permission

Full setup: [docs/install.md](docs/install.md)

## Install For Claude Code

Claude Code can run local stdio MCP servers.

```bash
git clone https://github.com/OpenClaudex/open-claudex-computer-use.git
cd open-claudex-computer-use
swift build

claude mcp add claudex-computer-use -- "$(pwd)/.build/debug/claudex-computer-use"
claude mcp list
```

Then restart Claude Code.

Grant Accessibility and Screen Recording permissions to the terminal app that runs Claude Code, such as Terminal.app, iTerm2, or Warp.

## Install For Codex App

This repo includes a local Codex plugin scaffold:

```text
plugins/claudex-computer-use/.codex-plugin/plugin.json
plugins/claudex-computer-use/.mcp.json
```

Build first:

```bash
swift build
```

Then add the local plugin directory to Codex:

```text
/path/to/open-claudex-computer-use/plugins/claudex-computer-use
```

The plugin starts:

```text
.build/debug/claudex-computer-use
```

Grant Accessibility and Screen Recording permissions to Codex if Codex is the host process.

## Install For Codex CLI / Generic MCP

Use the server as a normal stdio MCP command:

```json
{
  "mcpServers": {
    "claudex-computer-use": {
      "command": "/absolute/path/to/open-claudex-computer-use/.build/debug/claudex-computer-use"
    }
  }
}
```

The server auto-detects both MCP stdio formats:

- NDJSON, used by newer Claude Code / MCP SDK clients
- `Content-Length`, used by older clients and Codex-style transports

## What It Can Do

The server exposes 23 MCP tools:

| Area | Tools |
|---|---|
| App state | `get_app_state`, `list_apps`, `list_windows`, `capture_window` |
| Actions | `click`, `scroll`, `drag`, `press_key`, `type_text`, `set_value` |
| AX helpers | `find_ui_element`, `press_element`, `perform_action`, `perform_secondary_action` |
| Desktop control | `acquire_desktop`, `desktop_status`, `release_desktop`, `stop` |
| Virtual cursor | `get_virtual_cursor`, `set_virtual_cursor` |
| Safety / diagnostics | `doctor`, `get_allowlist`, `set_allowlist` |

Mutation tools return post-action state, screenshot metadata, and app guidance where possible. This matches the interaction pattern agents expect from Codex-style computer-use systems.

## Demo Prompts

### Safari + Notes

```text
Using only Claudex Computer Use tools, complete this task entirely in the background:

1. Use Safari to search for the population of Tokyo.
2. Calculate Tokyo's population as a percentage of 8.1 billion.
3. Write the result into Notes.
4. Report what you wrote.

Do not use web search tools. Do not bring apps to the foreground unless required.
```

### Calculator

```text
Using Claudex Computer Use, calculate 42 * 17 in the Calculator app.
Use app state and element clicks where possible, then read the result.
```

### Feishu / Lark Best-Effort Test

```text
Use only the claudex-computer-use MCP tools.

1. Enable the virtual cursor with set_virtual_cursor(preset="codexDemo", clear=true).
2. Get the app state for Feishu or Lark.
3. Try to open search.
4. Search for a visible contact or chat.
5. Stop before sending any message unless explicitly instructed.

Report which tools succeeded and where AX information was incomplete.
```

More demos: [docs/demos.md](docs/demos.md)

## Virtual Cursor

Open Claudex Computer Use includes a same-process virtual cursor overlay designed for recordings and human supervision.

```text
set_virtual_cursor(preset="codexDemo", clear=true)
set_virtual_cursor(mode="hybrid", style="ghostArrow", show_trail=false)
```

Current behavior:

- Smoothly moves before pointer actions.
- Stays visually tied to the operated app when possible.
- Obeys app-level visibility instead of floating above unrelated windows.
- Uses lower-confidence visuals for weak AX coordinates.
- Hides when the desktop session is released or stopped.

## App Compatibility

| Tier | Apps | Expected Behavior |
|---|---|---|
| Stable | Safari, Notes, TextEdit, Calculator, Finder, System Settings | Strong AX tree, screenshots, semantic clicks, `set_value` |
| Limited | Chrome, Edge, VS Code, Slack, Cursor | Partial AX, coordinate fallback, pasteboard-heavy typing |
| Best-effort | WeChat, Feishu/Lark, self-drawn or WebView-heavy surfaces | Sparse AX, unreliable frames, more fallback logic |

Details: [docs/compatibility.md](docs/compatibility.md)

## Architecture

```text
MCP client
  -> claudex-computer-use stdio server
    -> ClaudexComputerUseCore
      -> Accessibility / ScreenCaptureKit / CGEvent / Pasteboard
      -> Same-process virtual cursor overlay
      -> App-specific guidance and Codex-compatible responses
```

Main modules:

- `ClaudexComputerUseMCP`: stdio MCP server
- `ClaudexComputerUseCore`: native macOS execution layer
- `ClaudexComputerUseCLI`: local debug CLI
- `plugins/claudex-computer-use`: Codex plugin scaffold

This repo is the execution engine, not a full agent harness.

## Safety Model

- Local-first: actions run on your machine.
- macOS permissions are explicit: Accessibility and Screen Recording.
- Optional allowlist tools restrict which bundle IDs can be operated.
- `doctor` reports permission and runtime status.
- Desktop session tools make it visible when an agent has control.

## Roadmap

Short-term:

- Better weak-AX app guidance for Feishu/Lark, WeChat, and Electron apps
- More polished virtual cursor presets and recording examples
- Packaged release artifacts
- Broader MCP client setup docs

See [ROADMAP.md](ROADMAP.md).

## Docs

- [Installation & Integration](docs/install.md)
- [Demo Pack](docs/demos.md)
- [App Compatibility Matrix](docs/compatibility.md)
- [Testing](docs/testing.md)
- [Codex Native Trace Kit](docs/codex-native-trace-kit.md)
- [Roadmap](ROADMAP.md)

## License

[MIT](LICENSE)
