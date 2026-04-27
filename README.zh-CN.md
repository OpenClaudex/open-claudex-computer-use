# Open Claudex Computer Use

**[English](README.md) | 简体中文**

面向 Claude Code、Codex 和任意 MCP agent 的开源 macOS 后台 computer-use MCP server。

[![Release](https://img.shields.io/github/v/release/OpenClaudex/open-claudex-computer-use?include_prereleases&label=release)](https://github.com/OpenClaudex/open-claudex-computer-use/releases)
[![License: MIT](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)
[![macOS](https://img.shields.io/badge/macOS-13%2B-black)](docs/install.md)
[![Swift](https://img.shields.io/badge/Swift-5.9%2B-orange)](Package.swift)
[![MCP](https://img.shields.io/badge/MCP-stdio-blue)](docs/install.md)

`claudex-computer-use` 是一个原生 Swift MCP server。它把 macOS Accessibility、ScreenCaptureKit、CGEvent fallback 和虚拟光标封装成 MCP tools，让 AI agent 可以读取和操作真实 Mac App。

它填的是 browser automation 和完整虚拟机之间的空位：

- 操作真实桌面 App，不只操作网页。
- 本地运行，不需要云端虚拟桌面。
- 可接入 Claude Code、Codex App 插件、Codex CLI 风格 MCP 配置、Cursor 和通用 MCP client。
- 提供实时虚拟光标，适合 demo、录屏和人工观察。
- 返回 Codex 风格的 post-action state，agent 不需要每一步都重新手动 snapshot。

**状态：** `0.1.0-alpha`

> 本项目不是 Anthropic、OpenAI、Apple 或官方 Codex Computer Use 插件的官方项目。

## Demos

| 原生 App 操作 | 后台跨 App 工作流 | 飞书 / Lark Best-Effort |
|---|---|---|
| ![原生 macOS Calculator demo](docs/assets/demo-calculator.gif) | ![Safari 和 Notes 后台工作流 demo](docs/assets/demo-background-safari-notes.gif) | ![飞书和 Lark 安全演示 demo](docs/assets/demo-feishu-lark.gif) |
| 通过 Accessibility 读取和点击原生 macOS App，并显示虚拟光标。 | agent 在 Safari 和 Notes 中工作，同时人可以继续使用电脑。 | 对 WebView-heavy 企业 App 使用 AX + 坐标 fallback。素材只包含假数据。 |

## 项目起源

这个项目来自两个方向的实验：

- Codex 风格的 background computer use：agent 可以在后台看屏幕、点按钮、输入内容，同时不抢你的真实鼠标。
- Claude Code 风格的 MCP 扩展：把外部能力封装成工具，让 agent 在同一套 tool-call 协议下调用。

我们缺的是一个可以复用的开源执行层：一个本地 macOS MCP server，任何 agent harness 都可以接进来。Open Claudex Computer Use 做的就是这件事。

这里的 “Claudex” 指的是 Claude Code + Codex 两类工作流的组合，不是一个官方产品名。

## 为什么做

大多数 computer-use 项目分三类：浏览器优先、虚拟机优先、或者 agent harness 优先。Open Claudex Computer Use 是 app-first：

| 路线 | 适合场景 | 代价 |
|---|---|---|
| 浏览器自动化 | 网站和 Web App | 覆盖不了原生 Mac App |
| 远程 VM / 虚拟桌面 | 隔离和可复现 | 配置重，不是你的真实桌面 |
| 本项目 | 真实本地 macOS App | 受 macOS Accessibility 质量限制 |

目标不是假装平台限制不存在，而是把限制清晰暴露成一个实用的 MCP contract。

## 快速开始

```bash
git clone https://github.com/OpenClaudex/open-claudex-computer-use.git
cd open-claudex-computer-use
swift build
```

运行 smoke tests：

```bash
python3 scripts/smoke-mcp.py
python3 scripts/smoke-mcp-ndjson.py
python3 scripts/test-stale-guard.py
```

要求：

- macOS 13.0+
- Swift 5.9+ / Xcode 15+
- Accessibility 权限
- Screen Recording 权限

完整安装说明见：[docs/install.md](docs/install.md)

## Claude Code 安装

Claude Code 可以运行本地 stdio MCP server。

```bash
git clone https://github.com/OpenClaudex/open-claudex-computer-use.git
cd open-claudex-computer-use
swift build

claude mcp add claudex-computer-use -- "$(pwd)/.build/debug/claudex-computer-use"
claude mcp list
```

然后重启 Claude Code。

权限要授予运行 Claude Code 的宿主进程。通常是 Terminal.app、iTerm2、Warp 等终端 App，需要在系统设置里打开 Accessibility 和 Screen Recording。

## Codex App 安装

仓库里带了一个本地 Codex plugin scaffold：

```text
plugins/claudex-computer-use/.codex-plugin/plugin.json
plugins/claudex-computer-use/.mcp.json
```

先构建：

```bash
swift build
```

然后把这个本地 plugin 目录加到 Codex：

```text
/path/to/open-claudex-computer-use/plugins/claudex-computer-use
```

插件会启动：

```text
.build/debug/claudex-computer-use
```

如果 Codex 是宿主进程，Accessibility 和 Screen Recording 权限需要授予 Codex。

## Codex CLI / 通用 MCP 安装

也可以把它当成普通 stdio MCP server 使用：

```json
{
  "mcpServers": {
    "claudex-computer-use": {
      "command": "/absolute/path/to/open-claudex-computer-use/.build/debug/claudex-computer-use"
    }
  }
}
```

server 会自动识别两种 MCP stdio 帧格式：

- NDJSON：新版 Claude Code / MCP SDK client 常用
- `Content-Length`：旧版 MCP client 和 Codex 风格 transport 常用

## 能做什么

server 暴露 23 个 MCP tools：

| 类别 | Tools |
|---|---|
| App 状态 | `get_app_state`, `list_apps`, `list_windows`, `capture_window` |
| 操作动作 | `click`, `scroll`, `drag`, `press_key`, `type_text`, `set_value` |
| AX 辅助 | `find_ui_element`, `press_element`, `perform_action`, `perform_secondary_action` |
| 桌面控制 | `acquire_desktop`, `desktop_status`, `release_desktop`, `stop` |
| 虚拟光标 | `get_virtual_cursor`, `set_virtual_cursor` |
| 安全和诊断 | `doctor`, `get_allowlist`, `set_allowlist` |

mutation tools 会尽量返回动作后的 app state、截图元数据和 app guidance。这和 Codex 风格 computer-use system 的交互模式一致。

## Demo Prompt

### Safari + Notes

```text
只使用 Claudex Computer Use tools，在后台完成这个任务：

1. 用 Safari 搜索 Tokyo population。
2. 计算 Tokyo 人口占 8.1 billion 的比例。
3. 把结果写入 Notes。
4. 汇报你写入了什么。

不要使用 web search tools。除非必要，不要把 App 切到前台。
```

### Calculator

```text
使用 Claudex Computer Use，在 Calculator 里计算 42 * 17。
尽量使用 app state 和 element click，最后读取结果。
```

### 飞书 / Lark Best-Effort 测试

```text
只使用 claudex-computer-use MCP tools。

1. 调用 set_virtual_cursor(preset="codexDemo", clear=true) 打开虚拟光标。
2. 获取 Feishu 或 Lark 的 app state。
3. 尝试打开搜索。
4. 搜索一个可见联系人或会话。
5. 除非明确要求，不要发送消息。

最后汇报哪些 tools 成功了，哪些地方 AX 信息不完整。
```

更多 demo：[docs/demos.md](docs/demos.md)

## 虚拟光标

Open Claudex Computer Use 内置同进程虚拟光标 overlay，主要用于录屏、观察和建立信任感。

```text
set_virtual_cursor(preset="codexDemo", clear=true)
set_virtual_cursor(mode="hybrid", style="ghostArrow", show_trail=false)
```

当前行为：

- 在 pointer action 前平滑移动。
- 尽量跟随被操作 App 的可见区域。
- 不会无条件浮在其他 App 上方。
- 对弱 AX 坐标使用低置信度视觉反馈。
- desktop session 结束或 stop 后隐藏。

## App 兼容性

| 等级 | App | 预期表现 |
|---|---|---|
| Stable | Safari, Notes, TextEdit, Calculator, Finder, System Settings | AX tree 完整，截图稳定，语义点击和 `set_value` 支持好 |
| Limited | Chrome, Edge, VS Code, Slack, Cursor | AX 部分可用，需要坐标 fallback 和 pasteboard typing |
| Best-effort | WeChat, Feishu/Lark, 自绘或 WebView-heavy 界面 | AX 稀疏，frame 不稳定，需要更多 fallback |

详细矩阵：[docs/compatibility.md](docs/compatibility.md)

## 架构

```text
MCP client
  -> claudex-computer-use stdio server
    -> ClaudexComputerUseCore
      -> Accessibility / ScreenCaptureKit / CGEvent / Pasteboard
      -> Same-process virtual cursor overlay
      -> App-specific guidance and Codex-compatible responses
```

主要模块：

- `ClaudexComputerUseMCP`：stdio MCP server
- `ClaudexComputerUseCore`：原生 macOS 执行层
- `ClaudexComputerUseCLI`：本地 debug CLI
- `plugins/claudex-computer-use`：Codex plugin scaffold

这个仓库是执行引擎，不是完整 agent harness。

## 安全模型

- Local-first：动作都在你的机器上执行。
- macOS 权限显式可控：Accessibility 和 Screen Recording。
- 可用 allowlist tools 限制允许操作的 bundle IDs。
- `doctor` 会报告权限和运行状态。
- desktop session tools 让 agent 何时接管桌面变得可见。

## Roadmap

近期方向：

- 改善 Feishu/Lark、WeChat、Electron 等弱 AX App 的 guidance。
- 打磨虚拟光标 preset 和录屏示例。
- 提供打包后的 release artifacts。
- 补充更多 MCP client 安装文档。

见：[ROADMAP.md](ROADMAP.md)

## 文档

- [安装和集成](docs/install.md)
- [Demo Pack](docs/demos.md)
- [App 兼容性矩阵](docs/compatibility.md)
- [Testing](docs/testing.md)
- [Codex Native Trace Kit](docs/codex-native-trace-kit.md)
- [Roadmap](ROADMAP.md)

## License

[MIT](LICENSE)
