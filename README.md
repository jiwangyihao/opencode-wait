# opencode-wait

[![npm version](https://img.shields.io/npm/v/opencode-wait.svg)](https://www.npmjs.com/package/opencode-wait)
[![npm downloads](https://img.shields.io/npm/dw/opencode-wait.svg)](https://www.npmjs.com/package/opencode-wait)
[![License: MPL-2.0](https://img.shields.io/badge/License-MPL--2.0-brightgreen.svg)](LICENSE)

> **Initial in v0.1.0 | v0.1.0 初始版本**
>
> - Adds a standalone OpenCode `wait` tool plugin | 新增独立 OpenCode `wait` 工具插件
> - Supports timed waits with a 30-second minimum | 支持最短 30 秒的计时等待
> - Supports waiting for new user messages in the current session | 支持等待当前会话的新用户消息

[中文](#中文) | [English](#english)

---

<a name="中文"></a>

## 中文

`opencode-wait` 是一个专注于等待场景的 **OpenCode** 插件。它提供 `wait` 工具，让智能体在无人值守等待、后台任务、冷却时间和预期事件之间保持可恢复的执行节奏。

它适合把「等一段时间」和「等用户下一条消息」从阻塞式提问中拆出来：需要明确用户决策时继续使用 `question`，只需要时间流逝或预期事件时使用 `wait`。

## 功能一览

- **计时等待** — `wait({ seconds })` 会等待指定秒数，最短 30 秒。
- **参数归一化** — 缺失、无效、小于 30 的 `seconds` 会按 30 秒处理。
- **事件等待** — `wait({ until: "new_user_message" })` 会等待当前会话出现新的用户消息。
- **早退原因** — 计时等待期间如果检测到新用户消息，会提前结束并返回原因。
- **测试友好** — `createWaitPlugin()` 支持注入时间、睡眠和客户端实现，便于稳定测试。

---

## 安装

<details open>
<summary><b>面向人类用户</b></summary>

**选项 A：让 LLM 帮你安装**

把下面这段话丢给任意 LLM（OpenCode / Claude Code / Cursor 等）：

```text
请先查看 opencode-wait 最新 GitHub Release 正文里的“升级方式”，然后严格执行其中带明确版本号的安装/升级命令。默认应使用带 `-g/--global` 的全局命令来更新你平时实际在用的 OpenCode 实例；只有你明确想改当前项目目录下的 `.opencode` 时，才省略 `-g`。不要手动编辑 OpenCode 配置，也不要使用裸包名或 latest。参考说明：https://github.com/jiwangyihao/opencode-wait/releases/latest
```

**选项 B：手动安装**

1. **先查看最新 GitHub Release** 的 `## 升级方式` 小节。

2. **执行其中带明确版本号的全局命令**。例如当前版本是：

   ```bash
   opencode plugin opencode-wait@0.1.0 --force -g
   ```

3. **如果你是刻意只想改当前项目目录下的 `.opencode`**，才使用不带 `-g` 的局部命令；否则默认不要省略 `-g`。

4. **如果升级后看起来仍然没有变化**，先清理 OpenCode 插件缓存，再重新执行同一条带版本号命令。

   ```bash
   rm -rf ~/.cache/opencode/packages/opencode-wait@*
   ```

5. **如果 OpenCode 已在运行，重启后再验证**。

</details>

<details>
<summary><b>面向 LLM 智能体</b></summary>

### 步骤指引

1. 读取本仓库最新 GitHub Release 正文，提取 `## 升级方式` 中带明确版本号的命令。

2. 直接执行该命令。默认使用带 `-g/--global` 的全局命令；只有用户明确要改当前项目目录下的 `.opencode` 时，才省略 `-g`。例如当前版本：

   ```bash
   opencode plugin opencode-wait@0.1.0 --force -g
   ```

3. 不要手动编辑 OpenCode 配置，也不要使用裸包名或 `latest`。

4. 如果执行后版本看起来仍然没更新，先清理 OpenCode 插件缓存，再重新执行同一条带版本号命令：

   ```bash
   rm -rf ~/.cache/opencode/packages/opencode-wait@*
   ```

5. 如果 OpenCode 已在运行，重启后再验证。

### 验证

在任务中调用 `wait({ seconds: 30 })` 后，返回结果应包含类似信息：

```text
started: 2026-05-05T00:00:00.000Z; waited: 30s; now: 2026-05-05T00:00:30.000Z
```

</details>

---

## 使用方式

默认入口导出 OpenCode 插件函数：

```typescript
import WaitPlugin from "opencode-wait"

export default WaitPlugin
```

如果需要在测试中注入时间、睡眠或客户端实现，可以使用 `createWaitPlugin()`：

```typescript
import { createWaitPlugin } from "opencode-wait"

export default createWaitPlugin({
  sleep: async () => {},
})
```

## 工具参数

计时等待：

```json
{
  "seconds": 30
}
```

等待当前会话出现新的用户消息：

```json
{
  "until": "new_user_message"
}
```

当等待期间出现真实用户消息，结果会包含 `reason: new user message`。当等待期间出现插件合成用户消息，结果会包含 `reason: new user message (synthetic)`。

## 适合谁使用

- 想让 OpenCode 智能体在后台任务、冷却期或外部作业期间继续保持可恢复状态的用户。
- 需要把无人值守等待和必须用户决策的交互区分开的用户。
- 正在构建组合式 OpenCode 插件，希望复用一个小而稳定的等待工具的开发者。

## 本地开发

```bash
npm install
npm test
npm run build
```

更多设计与实现记录见 `docs/superpowers/`。

---

<a name="english"></a>

## English

`opencode-wait` is a focused **OpenCode** plugin for wait-oriented workflows. It provides a `wait` tool so agents can handle unattended waits, background jobs, cooldowns, and expected events without turning those waits into blocking user prompts.

Use `question` when a real user decision is required. Use `wait` when the only thing needed is time passing or a known event in the current session.

## What You Get

- **Timed waits** — `wait({ seconds })` waits for the requested duration, with a 30-second minimum.
- **Input normalization** — missing, invalid, or too-small `seconds` values become 30 seconds.
- **Event waits** — `wait({ until: "new_user_message" })` waits for the current session to receive a new user message.
- **Early-return reasons** — timed waits can end early when a new user message arrives, and the result explains why.
- **Test-friendly seams** — `createWaitPlugin()` accepts injected time, sleep, and client implementations for stable tests.

---

## Installation

<details open>
<summary><b>For Humans</b></summary>

**Option A: Let an LLM do it**

Paste this into any LLM agent (OpenCode, Claude Code, Cursor, etc.):

```text
First read the latest GitHub Release for opencode-wait and execute the exact versioned install/upgrade command from its "Upgrade / 升级方式" section. By default, use the global command with `-g/--global` so you update the OpenCode instances you actually run; omit `-g` only if you intentionally want to change the current project's `.opencode`. Do not manually edit the OpenCode config, and do not use a bare package name or latest. Reference: https://github.com/jiwangyihao/opencode-wait/releases/latest
```

**Option B: Manual setup**

1. **Read the latest GitHub Release** and look for the `## Upgrade` section.

2. **Run the exact versioned global command** from that section. For the current version:

   ```bash
   opencode plugin opencode-wait@0.1.0 --force -g
   ```

3. **Only omit `-g` if you intentionally want to update the current project's `.opencode` instead of the global OpenCode config.**

4. **If an upgrade still looks stale**, clear the OpenCode plugin cache and rerun the same versioned command:

   ```bash
   rm -rf ~/.cache/opencode/packages/opencode-wait@*
   ```

5. **If OpenCode is already running, restart it before verification**.

</details>

<details>
<summary><b>For LLM Agents</b></summary>

### Step-by-Step Instructions

1. Read the latest GitHub Release body for this repository and extract the exact versioned command from `## Upgrade / 升级方式`.

2. Execute that command directly. By default, use the global command with `-g/--global`; only omit `-g` if you intentionally want to update the current project's `.opencode`. For the current version:

   ```bash
   opencode plugin opencode-wait@0.1.0 --force -g
   ```

3. Do not install or upgrade this plugin by hand-editing the OpenCode config, and do not use a bare package name or `latest`.

4. If the installed version still does not change, clear the OpenCode plugin cache and rerun the same versioned command:

   ```bash
   rm -rf ~/.cache/opencode/packages/opencode-wait@*
   ```

5. If OpenCode is already running, restart it before verification.

### Verification

Calling `wait({ seconds: 30 })` should return a result similar to:

```text
started: 2026-05-05T00:00:00.000Z; waited: 30s; now: 2026-05-05T00:00:30.000Z
```

</details>

---

## Usage

The default export is the OpenCode plugin function:

```typescript
import WaitPlugin from "opencode-wait"

export default WaitPlugin
```

For tests, inject time, sleep, or client behavior through `createWaitPlugin()`:

```typescript
import { createWaitPlugin } from "opencode-wait"

export default createWaitPlugin({
  sleep: async () => {},
})
```

## Tool Input

Timed wait:

```json
{
  "seconds": 30
}
```

Wait for a new user message in the current session:

```json
{
  "until": "new_user_message"
}
```

When a real user message appears during the wait, the result includes `reason: new user message`. When a plugin-synthesized user message appears, it includes `reason: new user message (synthetic)`.

## Who Should Use This

- Users who want OpenCode agents to resume cleanly after background work, cooldowns, or external jobs.
- Users who want unattended waits to stay separate from interactions that need real user decisions.
- Plugin authors who want a small, stable wait tool for composed OpenCode workflows.

## Local Development

```bash
npm install
npm test
npm run build
```

Design and implementation notes live under `docs/superpowers/`.

---

## License

MPL-2.0 License. See [LICENSE](LICENSE) for details.
