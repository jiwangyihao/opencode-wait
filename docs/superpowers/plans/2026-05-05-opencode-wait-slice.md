# opencode-wait 最小切片实现计划

> **面向 AI 代理的工作者：** 必需子技能：使用 superpowers:subagent-driven-development（推荐）或 superpowers:executing-plans 逐任务实现此计划。步骤使用复选框（`- [ ]`）语法来跟踪进度。

**目标：** 将当前 `wait` 工具拆成可独立构建、测试、启用的 `opencode-wait` npm 包，同时让当前 `opencode-copilot-account-switcher` 在迁移期继续注册同一个 `wait` 工具。

**架构：** 在仓库内新增 `packages/opencode-wait` 独立包，包内拥有 `wait` 工具实现、真实 OpenCode 插件入口、测试和 README。当前包通过 npm 本地依赖消费 `opencode-wait`，`src/wait-tool.ts` 仅作为兼容 re-export，不再拥有独立实现。

**技术栈：** Node.js >= 24、TypeScript、ESM、`@opencode-ai/plugin`、`node:test`、npm workspace / local file dependency。

---

## 执行前准备

本实现必须在项目内 worktree 中执行，遵循 `superpowers:using-git-worktrees`。

```powershell
$env:GIT_MASTER = "1"; git check-ignore -q .worktrees
$env:GIT_MASTER = "1"; git worktree add ".worktrees/opencode-wait-slice" -b "feature/opencode-wait-slice"
```

进入 worktree 后安装依赖并确认基线：

```powershell
npm install
npm run build
node --test test/wait-tool.test.js test/index-exports.test.js
```

预期：`npm install` 成功；`npm run build` 退出码为 0；两个 targeted tests 全部 PASS。若基线失败，记录失败命令和错误，不开始实现。

## 文件结构

### 新增文件

- `packages/opencode-wait/package.json`：独立包元数据、导出入口、构建和测试脚本。
- `packages/opencode-wait/tsconfig.json`：独立 TypeScript 编译配置。
- `packages/opencode-wait/tsconfig.build.json`：独立构建配置。
- `packages/opencode-wait/src/wait-tool.ts`：从当前 `src/wait-tool.ts` 迁移来的 `wait` 工具实现，导出 `createWaitTool` 和测试 seam 类型。
- `packages/opencode-wait/src/index.ts`：真实 OpenCode 插件入口，导出 `WaitPlugin`、`createWaitPlugin` 和 `createWaitTool`。
- `packages/opencode-wait/test/wait-tool.test.js`：独立包内 `wait` 行为测试。
- `packages/opencode-wait/test/plugin-entry.test.js`：独立插件入口和 smoke 测试。
- `packages/opencode-wait/README.md`：独立包说明、安装、启用和参数说明。
- `packages/opencode-wait/LICENSE`：复制根目录 `LICENSE`。

### 修改文件

- `package.json`：声明 workspace / local dependency，调整 build 和 wait package test 脚本。
- `package-lock.json`：由 `npm install` 更新，锁定 `opencode-wait` 本地依赖。
- `src/wait-tool.ts`：改为从 `opencode-wait` re-export。
- `README.md`：给当前包的 `wait` 文案增加迁移期说明，不扩展成完整迁移指南。

---

### 任务 1：建立 `opencode-wait` 包骨架和失败的入口测试

**文件：**
- 修改：`package.json`
- 创建：`packages/opencode-wait/package.json`
- 创建：`packages/opencode-wait/tsconfig.json`
- 创建：`packages/opencode-wait/tsconfig.build.json`
- 创建：`packages/opencode-wait/test/plugin-entry.test.js`
- 创建：`packages/opencode-wait/LICENSE`
- 生成：`package-lock.json`

- [ ] **步骤 1：创建包目录**

运行：

```powershell
New-Item -ItemType Directory -Force "packages/opencode-wait/src", "packages/opencode-wait/test" | Out-Null
Copy-Item "LICENSE" "packages/opencode-wait/LICENSE"
```

预期：目录存在，`packages/opencode-wait/LICENSE` 与根目录 `LICENSE` 内容一致。

- [ ] **步骤 2：写入独立包元数据**

创建 `packages/opencode-wait/package.json`：

```json
{
  "name": "opencode-wait",
  "version": "0.1.0",
  "description": "Generic wait tool plugin for OpenCode",
  "main": "./dist/index.js",
  "types": "./dist/index.d.ts",
  "exports": {
    ".": {
      "types": "./dist/index.d.ts",
      "default": "./dist/index.js"
    },
    "./wait-tool": {
      "types": "./dist/wait-tool.d.ts",
      "default": "./dist/wait-tool.js"
    }
  },
  "type": "module",
  "license": "MPL-2.0",
  "author": "jiwangyihao",
  "keywords": [
    "opencode",
    "wait",
    "tool",
    "automation"
  ],
  "engines": {
    "node": ">=24.0.0"
  },
  "files": [
    "dist/",
    "README.md",
    "LICENSE"
  ],
  "scripts": {
    "build": "tsc -p tsconfig.build.json",
    "test": "npm run build && npm run test:built",
    "test:built": "node --test test/plugin-entry.test.js",
    "prepublishOnly": "npm run build"
  },
  "devDependencies": {
    "@types/node": "^24.10.1",
    "typescript": "^5.0.0"
  },
  "dependencies": {
    "@opencode-ai/plugin": "^1.2.26"
  }
}
```

- [ ] **步骤 3：写入独立 TypeScript 配置**

创建 `packages/opencode-wait/tsconfig.json`：

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ES2022",
    "moduleResolution": "Bundler",
    "lib": ["ES2022"],
    "types": ["node"],
    "strict": true,
    "declaration": true,
    "declarationMap": true,
    "emitDeclarationOnly": false,
    "outDir": "dist",
    "rootDir": "src",
    "skipLibCheck": true
  },
  "include": ["src"]
}
```

创建 `packages/opencode-wait/tsconfig.build.json`：

```json
{
  "extends": "./tsconfig.json",
  "compilerOptions": {
    "declaration": true,
    "declarationMap": false,
    "sourceMap": false
  },
  "include": ["src"]
}
```

- [ ] **步骤 4：在根包声明本地包依赖与脚本**

修改根 `package.json`，在顶层 `type` 后增加：

```json
"workspaces": [
  "packages/*"
],
```

修改根 `scripts.build`：

```json
"build": "npm run build -w opencode-wait && tsc -p tsconfig.build.json"
```

在根 `scripts` 中新增：

```json
"test:wait-package": "npm run test:built -w opencode-wait"
```

修改根 `scripts.test`，在 `npm run build &&` 后增加 `npm run test:wait-package &&`，保持后续现有 serial / parallel 测试顺序不变。

在根 `dependencies` 中新增本地依赖：

```json
"opencode-wait": "file:packages/opencode-wait"
```

注意：保留根包已有 `@opencode-ai/plugin`、`@opencode-ai/sdk`、微信和其他依赖，不删除任何现有依赖。

- [ ] **步骤 5：编写失败的插件入口测试**

创建 `packages/opencode-wait/test/plugin-entry.test.js`：

```javascript
import test from "node:test"
import assert from "node:assert/strict"

import WaitPlugin, { WaitPlugin as NamedWaitPlugin, createWaitPlugin, createWaitTool } from "../dist/index.js"

function createContext() {
  return {
    sessionID: "s1",
    messageID: "m1",
    agent: "task",
    directory: "/tmp/project",
    worktree: "/tmp/project",
    abort: new AbortController().signal,
    metadata() {},
    async ask() {},
  }
}

test("wait plugin entry registers only the wait tool", async () => {
  const plugin = createWaitPlugin({
    sleep: async () => {},
  })

  const hooks = await plugin({
    client: {},
    directory: "/tmp/project",
    serverUrl: "http://localhost",
  })

  assert.equal(typeof createWaitTool, "function")
  assert.equal(typeof WaitPlugin, "function")
  assert.equal(NamedWaitPlugin, WaitPlugin)
  assert.deepEqual(Object.keys(hooks.tool ?? {}), ["wait"])
  assert.equal(typeof hooks.tool.wait.execute, "function")
})

test("wait plugin entry exposes the timed wait path through wait", async () => {
  const plugin = createWaitPlugin({
    now: (() => {
      let tick = 0
      return () => 1_700_000_000_000 + tick++ * 30_000
    })(),
    sleep: async () => {},
  })
  const hooks = await plugin({
    client: {},
    directory: "/tmp/project",
    serverUrl: "http://localhost",
  })

  const result = await hooks.tool.wait.execute({ seconds: 30 }, createContext())

  assert.match(result, /^started: /)
  assert.match(result, /waited: 30s/)
  assert.match(result, /; now: /)
})

test("wait plugin entry exposes unavailable event result without session messages", async () => {
  const plugin = createWaitPlugin({
    now: () => 1_700_000_000_000,
  })
  const hooks = await plugin({
    client: {},
    directory: "/tmp/project",
    serverUrl: "http://localhost",
  })

  const result = await hooks.tool.wait.execute({ until: "new_user_message" }, createContext())

  assert.match(result, /ended: unavailable/)
  assert.match(result, /event: new_user_message/)
  assert.match(result, /reason: session messages unavailable/)
})

test("wait plugin entry forwards the OpenCode client into wait", async () => {
  const messageCalls = []
  const plugin = createWaitPlugin({
    now: (() => {
      let tick = 0
      return () => 1_700_000_000_000 + tick++ * 1_000
    })(),
    pollIntervalMs: 1,
  })

  const hooks = await plugin({
    client: {
      session: {
        messages: async () => {
          messageCalls.push(true)
          return {
            data: messageCalls.length === 1
              ? [
                  {
                    info: { id: "m1", role: "user", time: { created: 1_700_000_000_000 } },
                    parts: [],
                  },
                ]
              : [
                  {
                    info: { id: "m2", role: "user", time: { created: 1_700_000_001_000 } },
                    parts: [{ type: "text", text: "from host client" }],
                  },
                  {
                    info: { id: "m1", role: "user", time: { created: 1_700_000_000_000 } },
                    parts: [],
                  },
                ],
          }
        },
      },
    },
    directory: "/tmp/project",
    serverUrl: "http://localhost",
  })

  const result = await hooks.tool.wait.execute({ until: "new_user_message" }, createContext())

  assert.equal(messageCalls.length >= 2, true)
  assert.match(result, /ended: event/i)
  assert.match(result, /reason: new user message/i)
  assert.match(result, /message: m2/i)
})
```

- [ ] **步骤 6：运行红灯验证**

运行：

```powershell
npm install
Push-Location "packages/opencode-wait"
try {
  node --test test/plugin-entry.test.js
} finally {
  Pop-Location
}
```

预期：测试失败，原因是构建产物尚不存在，错误包含 `Cannot find module '../dist/index.js'` 或等价入口缺失错误。不要在此步骤运行 `npm test -w opencode-wait`；若测试直接通过，说明已有实现残留，先停止并检查 worktree 状态。

- [ ] **步骤 7：Commit**

运行：

```powershell
$env:GIT_MASTER = "1"; git add package.json package-lock.json packages/opencode-wait/package.json packages/opencode-wait/tsconfig.json packages/opencode-wait/tsconfig.build.json packages/opencode-wait/test/plugin-entry.test.js packages/opencode-wait/LICENSE
$env:GIT_MASTER = "1"; git commit -m "chore(wait): 建立独立包测试骨架" -m "为 opencode-wait 准备本地包结构和红灯入口测试，后续任务以此实现真实插件入口。"
```

---

### 任务 2：迁移 `wait` 实现并提供真实插件入口

**文件：**
- 创建：`packages/opencode-wait/src/wait-tool.ts`
- 创建：`packages/opencode-wait/src/index.ts`
- 测试：`packages/opencode-wait/test/plugin-entry.test.js`

- [ ] **步骤 1：复制当前 `wait` 工具实现**

运行：

```powershell
Copy-Item "src/wait-tool.ts" "packages/opencode-wait/src/wait-tool.ts"
```

然后修改 `packages/opencode-wait/src/wait-tool.ts` 的类型导出，让测试 seam 成为公开类型：

```typescript
export type WaitToolInput = {
  now?: () => number
  sleep?: (ms: number, signal?: AbortSignal) => Promise<void>
  pollIntervalMs?: number
  client?: {
    session?: {
      messages?: (input: {
        path: { id: string }
        query?: {
          directory?: string
          limit?: number
        }
      }) => Promise<{ data?: SessionMessage[] } | SessionMessage[] | undefined>
    }
  }
}
```

保留其余逻辑不变，包括 `DEFAULT_USER_MESSAGE_POLL_INTERVAL_MS`、`USER_MESSAGE_LOOKBACK_LIMIT`、`createWaitTool()`、synthetic 判定和 unavailable 返回文案。后续版本已取消等待下限，正数 `seconds` 按请求秒数等待，缺失、无效或非正数立即完成。

- [ ] **步骤 2：创建真实 OpenCode 插件入口**

创建 `packages/opencode-wait/src/index.ts`：

```typescript
import type { Plugin } from "@opencode-ai/plugin"
import { createWaitTool, type WaitToolInput } from "./wait-tool.js"

const WAIT_TOOL_DESCRIPTION = "Use for unattended/background waits that do not require user confirmation, including long-running tasks, external jobs, cooldowns, or expected notifications; pass until: \"new_user_message\" to wait for the current session to receive a new user message, including plugin-synthesized notifications, or resume after a timed wait exits early for that reason."

export { createWaitTool }
export type { WaitToolInput }

export function createWaitPlugin(waitInput: WaitToolInput = {}): Plugin {
  return async (input) => ({
    tool: {
      wait: createWaitTool({
        ...waitInput,
        client: waitInput.client ?? input.client,
      }),
    },
    "tool.definition": async (hookInput, output) => {
      if (hookInput.toolID === "wait") {
        output.description = WAIT_TOOL_DESCRIPTION
      }
    },
  })
}

export const WaitPlugin: Plugin = createWaitPlugin()

export default WaitPlugin
```

- [ ] **步骤 3：运行绿灯验证**

运行：

```powershell
npm test -w opencode-wait
```

预期：`plugin-entry.test.js` 全部 PASS。若测试失败，修改 `src/index.ts` 或 `src/wait-tool.ts`，不要修改测试期望。

- [ ] **步骤 4：Commit**

运行：

```powershell
$env:GIT_MASTER = "1"; git add packages/opencode-wait/src/wait-tool.ts packages/opencode-wait/src/index.ts
$env:GIT_MASTER = "1"; git commit -m "feat(wait): 提供独立插件入口" -m "opencode-wait 现在拥有真实 OpenCode 插件入口，并保留 wait 工具的可测试 seam。"
```

---

### 任务 3：迁移 `wait` 行为测试到独立包

**文件：**
- 创建：`packages/opencode-wait/test/wait-tool.test.js`
- 修改：`packages/opencode-wait/package.json`

- [ ] **步骤 1：复制现有行为测试**

运行：

```powershell
Copy-Item "test/wait-tool.test.js" "packages/opencode-wait/test/wait-tool.test.js"
```

修改 `packages/opencode-wait/test/wait-tool.test.js` 的导入路径，保持从独立包构建产物导入：

```javascript
import { createWaitTool } from "../dist/wait-tool.js"
```

同时在 `wait tool returns started waited now timeline` 用例末尾补充超时路径断言：

```javascript
assert.doesNotMatch(result, /ended:/)
```

- [ ] **步骤 2：补充 unavailable 行为测试**

在 `packages/opencode-wait/test/wait-tool.test.js` 末尾追加：

```javascript
test("wait tool returns unavailable for event wait without session messages", async () => {
  const wait = createWaitTool({
    now: () => 1_700_000_000_000,
  })

  const result = await wait.execute({ until: "new_user_message" }, createContext())

  assert.match(result, /ended: unavailable/i)
  assert.match(result, /event: new_user_message/i)
  assert.match(result, /reason: session messages unavailable/i)
  assert.doesNotMatch(result, /requested: 30s/)
})
```

- [ ] **步骤 3：确认包内测试脚本覆盖两个测试文件**

确认 `packages/opencode-wait/package.json` 中脚本为：

```json
"test:built": "node --test test/wait-tool.test.js test/plugin-entry.test.js"
```

- [ ] **步骤 4：运行红绿验证**

先临时修改 `packages/opencode-wait/src/wait-tool.ts` 中 `formatUnavailableEventResult()` 的返回文案，制造不会挂起的红灯：只把 `ended: unavailable` 临时改成 `ended: event`，保留 `eventOnly && !input.client?.session?.messages` 早退判断。

运行：

```powershell
npm test -w opencode-wait
```

预期：新增 unavailable 测试立即 FAIL，错误显示未返回 `ended: unavailable`，且不会挂起。随后恢复 `packages/opencode-wait/src/wait-tool.ts` 中的 `ended: unavailable`，再运行：

```powershell
npm test -w opencode-wait
```

预期：`wait-tool.test.js` 与 `plugin-entry.test.js` 全部 PASS。

- [ ] **步骤 5：Commit**

运行：

```powershell
$env:GIT_MASTER = "1"; git add packages/opencode-wait/test/wait-tool.test.js packages/opencode-wait/package.json
$env:GIT_MASTER = "1"; git commit -m "test(wait): 迁移独立包行为断言" -m "将现有 wait 行为测试迁入 opencode-wait，并补充事件不可用路径断言。"
```

---

### 任务 4：让当前包消费 `opencode-wait`

**文件：**
- 修改：`src/wait-tool.ts`
- 修改：`package.json`
- 生成：`package-lock.json`
- 测试：`test/wait-tool.test.js`
- 测试：`test/index-exports.test.js`

- [ ] **步骤 1：改写当前包兼容 re-export**

将根包 `src/wait-tool.ts` 全文替换为：

```typescript
export { createWaitTool } from "opencode-wait"
export type { WaitToolInput } from "opencode-wait"
```

这保持 `../dist/wait-tool.js` 兼容路径，同时让当前包不再拥有独立 `wait` 实现。

- [ ] **步骤 2：安装并刷新锁文件**

运行：

```powershell
npm install
```

预期：`package-lock.json` 出现 `packages/opencode-wait` 和 `node_modules/opencode-wait` 条目，根包依赖解析成功。

- [ ] **步骤 3：运行兼容红绿验证**

先临时把 `src/wait-tool.ts` 的 re-export 改为错误包名：

```typescript
export { createWaitTool } from "opencode-wait-missing"
export type { WaitToolInput } from "opencode-wait-missing"
```

运行：

```powershell
npm run build
```

预期：FAIL，TypeScript 报找不到 `opencode-wait-missing`。恢复正确 re-export 后运行：

```powershell
npm run build
node --test test/wait-tool.test.js test/index-exports.test.js
```

预期：构建成功，两个测试文件全部 PASS。`test/wait-tool.test.js` 仍从 `../dist/wait-tool.js` 导入，证明当前包兼容路径有效。

- [ ] **步骤 4：Commit**

运行：

```powershell
$env:GIT_MASTER = "1"; git add package.json package-lock.json src/wait-tool.ts
$env:GIT_MASTER = "1"; git commit -m "refactor(wait): 由当前包适配独立 wait 包" -m "当前 Copilot 包通过 opencode-wait 暴露 wait 工具，避免迁移期保留分叉实现。"
```

---

### 任务 5：补充独立包 README 和当前包迁移说明

**文件：**
- 创建：`packages/opencode-wait/README.md`
- 修改：`README.md`

- [ ] **步骤 1：创建 `opencode-wait` README**

创建 `packages/opencode-wait/README.md`：

````markdown
# opencode-wait

`opencode-wait` 是一个通用 OpenCode 插件，只提供 `wait` 工具。它适合无人值守等待、后台任务、冷却时间和预期事件，不依赖 Copilot、Codex、Loop Safety、`notify` 或微信。

## 安装

发布后可以使用明确版本安装：

```bash
opencode plugin opencode-wait@0.1.0 -g
```

本地开发时先运行构建：

```bash
npm run build -w opencode-wait
```

## 在 OpenCode 中启用

默认入口导出的是 OpenCode 插件函数：

```typescript
import WaitPlugin from "opencode-wait"

export default WaitPlugin
```

如果需要在测试中注入时间和等待实现，可以使用 `createWaitPlugin()`：

```typescript
import { createWaitPlugin } from "opencode-wait"

export default createWaitPlugin({
  sleep: async () => {},
})
```

## 用法

计时等待按请求的正数秒数执行；缺失、无效或非正数 `seconds` 会立即完成：

```json
{
  "seconds": 5
}
```

等待当前会话出现新的用户消息：

```json
{
  "until": "new_user_message"
}
```

当等待期间出现真实用户消息，结果会包含 `reason: new user message`。当等待期间出现插件合成用户消息，结果会包含 `reason: new user message (synthetic)`。

## 不包含的能力

- 不提供 `notify`。
- 不改写 `question`。
- 不包含 Copilot、Codex、Loop Safety 或微信功能。
- 不依赖 `opencode-j-super-suite`。
````

- [ ] **步骤 2：在当前包 README 增加迁移期说明**

在根 `README.md` 的中文 `Wait Tool` 简述附近补充一句：

```markdown
> 迁移说明：`wait` 正在拆为通用独立包 `opencode-wait`；当前 Copilot 包在迁移期继续默认组合它，现有用户无需改变安装方式。
```

在英文对应的 `Wait Tool` 简述附近补充一句：

```markdown
> Migration note: `wait` is being split into the generic standalone package `opencode-wait`; this Copilot package keeps composing it during migration, so existing users do not need to change their installation path yet.
```

- [ ] **步骤 3：运行文档与包入口验证**

运行：

```powershell
npm run build
npm run test:wait-package
```

预期：构建成功，`opencode-wait` 包内测试全部 PASS。

- [ ] **步骤 4：Commit**

运行：

```powershell
$env:GIT_MASTER = "1"; git add packages/opencode-wait/README.md README.md
$env:GIT_MASTER = "1"; git commit -m "docs(wait): 说明独立 wait 包用法" -m "补充 opencode-wait 的安装和启用说明，并标注当前包的迁移期组合关系。"
```

---

### 任务 6：最终验证和手工 QA

**文件：**
- 验证：`packages/opencode-wait/dist/index.js`
- 验证：`dist/wait-tool.js`
- 验证：全部修改文件

- [ ] **步骤 1：运行 package 级验证**

运行：

```powershell
npm test -w opencode-wait
```

预期：`packages/opencode-wait/test/wait-tool.test.js` 和 `packages/opencode-wait/test/plugin-entry.test.js` 全部 PASS。

- [ ] **步骤 2：运行当前包 targeted 验证**

运行：

```powershell
npm run build
node --test test/wait-tool.test.js test/index-exports.test.js
```

预期：构建成功，当前包兼容入口和导出测试全部 PASS。

- [ ] **步骤 3：运行完整项目测试**

运行：

```powershell
npm test
```

预期：完整测试链路退出码为 0。若出现与本切片无关的既有失败，记录失败测试名、命令和错误摘要，不修改无关模块。

- [ ] **步骤 4：执行手工 QA 驱动脚本**

运行：

```powershell
node --input-type=module -e "import { createWaitPlugin } from './packages/opencode-wait/dist/index.js'; const context = { sessionID: 's1', messageID: 'm1', agent: 'task', directory: process.cwd(), worktree: process.cwd(), abort: new AbortController().signal, metadata() {}, async ask() {} }; const plugin = createWaitPlugin({ now: (() => { let tick = 0; return () => 1700000000000 + tick++ * 30000 })(), sleep: async () => {} }); const hooks = await plugin({ client: {}, directory: process.cwd(), serverUrl: 'http://localhost' }); if (!hooks.tool?.wait) throw new Error('wait tool not registered'); console.log(await hooks.tool.wait.execute({ seconds: 30 }, context)); console.log(await hooks.tool.wait.execute({ until: 'new_user_message' }, context));"
```

预期输出包含两行结果：第一行包含 `started:`、`waited: 30s`、`now:`；第二行包含 `ended: unavailable`、`event: new_user_message`、`reason: session messages unavailable`。

- [ ] **步骤 5：检查边界没有回退**

运行：

```powershell
$matches = rg -n "plugin-hooks|plugin(?:\.ts|\.js)|notify-tool|loop-safety-plugin|wechat|opencode-j-super-suite|opencode-copilot-account-switcher" "packages/opencode-wait/src" "packages/opencode-wait/package.json"
$code = $LASTEXITCODE
if ($code -eq 1) { Write-Host "0 matches"; exit 0 }
if ($code -ne 0) { exit $code }
$matches
exit 1
```

预期：无匹配时输出 `0 matches` 且退出码为 0；若出现匹配则打印匹配行并以退出码 1 失败。若出现匹配，确认不是 README 文档；`packages/opencode-wait/src` 和 `packages/opencode-wait/package.json` 不得引用这些运行时模块。

- [ ] **步骤 6：确认无需额外提交**

运行：

```powershell
$env:GIT_MASTER = "1"; git status --short
```

预期：只显示本计划之外尚未提交的计划或规格文档；实现 worktree 内没有未提交代码。如果任务 6 暴露了需要改代码的问题，回到对应任务修复并用该任务的 commit 步骤提交，不在最终验证阶段创建混合提交。

---

## 自检清单

- 规格覆盖：`opencode-wait` 独立包、真实插件入口、行为契约、当前包兼容、无共享库、无 suite 运行时依赖、独立启用 smoke 均有对应任务。
- TDD 路径：任务 1、任务 3、任务 4 均要求先制造或观察失败，再实现或恢复正确代码。
- 边界控制：`packages/opencode-wait/src` 不允许引用当前混合插件入口、notify、Loop Safety、WeChat 或 suite 运行时代码。
- 验证路径：包内测试、当前包 targeted 测试、完整 `npm test`、手工 QA 驱动脚本、边界 grep 均列入任务 6。
