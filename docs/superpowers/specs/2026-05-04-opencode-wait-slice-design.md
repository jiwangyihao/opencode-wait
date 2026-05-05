# opencode-wait 最小拆分切片设计

## 背景

`opencode-j-super-suite` 总体拆分设计已经明确：`wait`、`notify`、Loop Safety、Copilot、Codex、微信应按独立插件边界逐步拆分。其中 `wait` 是最小、最通用、依赖最少的能力，适合作为第一切片。

当前 `wait` 工具实现位于 `src/wait-tool.ts`，并由 `src/plugin-hooks.ts` 注入到当前 Copilot 混合插件中。测试位于 `test/wait-tool.test.js`，覆盖最小等待时间、无效参数归一化、时间线输出、等待新用户消息、合成用户消息等行为。

本切片只解决一件事：把 `wait` 从当前混合插件中拆成可独立安装、独立启用、独立测试的 OpenCode 插件包 `opencode-wait`。

## 目标

1. 创建独立 npm 包 `opencode-wait`，包名不包含 `plugin`。
2. `opencode-wait` 只提供 `wait` 工具，不包含 Copilot、Codex、Loop Safety、微信或通知能力。
3. `opencode-wait` 的用户可见行为与当前 `src/wait-tool.ts` 中的 `wait` 工具保持一致。
4. 当前 `opencode-copilot-account-switcher` 在迁移期继续可用，并通过依赖或适配使用拆出的 `wait` 能力。
5. 为后续 `opencode-loop-safety` 弱依赖 `opencode-wait` 留出稳定的工具名称和行为契约。

## 非目标

1. 不拆 `notify`。
2. 不拆 Loop Safety，也不重写 prompt 注入策略。
3. 不清理 Copilot、Codex 或微信功能。
4. 不建立共享库。`wait` 暂时没有跨插件共享逻辑需要抽库。
5. 不改变 `wait` 工具的参数语义、最小等待时间、返回文案或事件等待行为。

## 插件边界

### `opencode-wait` 拥有的能力

- `wait` tool 定义。
- `seconds` 参数：可选，最小等待时间为 30 秒，无效值归一化为 30 秒。
- `until` 参数：支持 `new_user_message`。
- 等待新用户消息时读取当前 session messages。
- 识别真实用户消息与插件合成用户消息。
- 输出现有格式的时间线结果。

### `opencode-wait` 不拥有的能力

- 不拥有 `notify` tool。
- 不拥有 `question` tool，也不改写 `question` 描述。
- 不拥有 Loop Safety prompt。
- 不拥有 Copilot/Codex 账号、路由、quota、retry 或 status。
- 不拥有微信通知传输。

实现边界也必须保持干净：`opencode-wait` 只能复用从当前仓库抽出的 `wait` 工具实现和最小插件入口，不得导入 `plugin.ts`、`plugin-hooks.ts`、`notify-tool.ts`、`wechat/`、`loop-safety-plugin.ts` 或 `opencode-j-super-suite` 的运行时代码。

## 行为契约

`opencode-wait` 必须保持以下契约：

1. 工具名为 `wait`。
2. 工具描述必须表达：它用于不需要用户确认的无人值守等待、后台任务、冷却时间或预期事件，是区别于 `question` 的等待路径。
3. `seconds` 小于 30、缺失、非数字、`NaN`、`0` 或负数时，等待时间归一化为 30 秒。
4. 有效非整数秒数向下取整。
5. 返回值包含 `started`、`waited`、`now`，并保留现有 `ended` 标签：超时结束不额外标注，真实或合成用户消息早退时标注 `ended: early`，事件模式命中时标注 `ended: event`，事件不可用时标注 `ended: unavailable`。
6. 当等待期间出现新的真实用户消息时，提前结束并返回 `reason: new user message`。
7. 当等待期间出现插件合成用户消息时，提前结束并返回 `reason: new user message (synthetic)`；合成消息的判定以 `role: "user"` 且任一 part 带有 `synthetic: true` 为准。
8. 当 `until: "new_user_message"` 且未提供 `seconds` 时，不启动超时等待，只等待事件。
9. 如果当前运行环境无法读取 session messages，事件等待必须返回明确的 unavailable 结果，而不是静默挂起。
10. 如果首次 session messages 快照里已经存在 `started` 之后的新用户消息，且该消息不是当前 message，必须立即结束等待，避免漏掉快照返回前已到达的用户消息。

## 与当前包的迁移关系

迁移期内，`opencode-copilot-account-switcher` 仍然可以继续暴露 `wait` 能力，但所有权应转移给 `opencode-wait`。

依赖方向只能是当前 Copilot 包消费或适配 `opencode-wait`。`opencode-wait` 不能反向依赖 `opencode-copilot-account-switcher`，也不能与当前包长期保留两份会分叉的 `wait` 实现。

推荐迁移路径：

1. 先让 `opencode-wait` 独立通过测试。
2. 再让当前包从本地实现切换为依赖或适配 `opencode-wait`。
3. 最后在当前包文档中说明：`wait` 已是通用独立插件，Copilot 包只是兼容期组合它。

这一路径避免一次性删除当前包中的 `wait` 行为，也避免在第一切片同时处理 Loop Safety 或 README 大迁移。

## 最小包结构与本地验证要求

`opencode-wait` 作为独立包时，应具备：

- 独立 `package.json`。
- 独立 `README.md`。
- 独立 TypeScript 构建配置。
- 独立测试入口。
- OpenCode 插件入口导出；可额外导出测试需要的 helper 或 factory，但这些辅助导出不能替代真实插件入口。

本切片不要求 `npm publish`、GitHub Release、tag 或 `opencode-j-super-suite` 集成，只要求本地可构建、可导入、可测试，并能作为独立 OpenCode 插件启用。

README 至少说明：

1. 这个包提供什么。
2. 如何安装。
3. 如何在 OpenCode 中启用。
4. `seconds` 与 `until: "new_user_message"` 的用法。
5. 它不依赖 Copilot，也不提供通知能力。

## 测试策略

本切片需要覆盖 3 层测试。

### 单包行为测试

迁移现有 `test/wait-tool.test.js` 的行为断言，确认独立包内 `wait` 工具行为不变。

测试入口必须保留可控 seam，例如 `now`、`sleep`、`client` 和 `pollIntervalMs`，让 TDD 能稳定覆盖超时、事件轮询和 session messages 场景。

### 当前包兼容测试

当前 `opencode-copilot-account-switcher` 在迁移期仍应能注册 `wait` 工具，避免现有用户升级后丢失工具。

### 打包入口测试

验证 `opencode-wait` 的构建产物能从包入口导入，并导出 OpenCode 可用的插件函数或工具注册能力。

### 独立启用验证

在干净的 OpenCode 环境中，只安装 `opencode-wait` 时，插件应能被启用并暴露 `wait` 工具，不需要 Copilot 包、`notify` 或微信参与。

本地 smoke 验证至少要覆盖：加载独立包入口、确认 `wait` 工具已注册、执行一次 timed wait 路径、执行一次 `until: "new_user_message"` 路径或 session messages 不可用路径。

## 验收标准

1. `opencode-wait` 可以不安装 Copilot 包而独立使用。
2. `opencode-wait` 不引入 Copilot、Codex、微信依赖。
3. 现有 `wait` 行为测试迁移后仍通过。
4. 当前包在迁移期仍能使用 `wait` 工具。
5. 本切片不修改 `notify`、Loop Safety、Copilot、Codex 或微信行为。
6. 没有为了本切片新建共享库。
7. 在干净 OpenCode 环境中，只安装 `opencode-wait` 时可以启用插件并看到 `wait` 工具。
8. `opencode-wait` 运行时不能依赖 `opencode-j-super-suite`，也不能把当前包、Copilot、Codex、Loop Safety、`notify` 或微信作为运行时必需模块。

## 风险与缓解

### 风险 1：切片扩大到 Loop Safety

缓解：本切片只拆 `wait`。Loop Safety 只在后续切片中改为弱依赖 `opencode-wait`。

### 风险 2：当前包兼容行为断裂

缓解：迁移期允许当前包依赖或适配 `opencode-wait`，直到文档和安装路径完成迁移。

### 风险 3：为一个工具过早创建共享库

缓解：本切片禁止新建共享库。只有后续多个插件真实复用稳定逻辑时，再按总体规格评估抽库。

### 风险 4：独立包入口设计过度

缓解：入口只暴露 OpenCode 插件启用所需的最小 API，以及测试需要的可导入工厂函数。

## 后续切片关系

本切片完成后，后续切片可以按顺序处理：

1. `opencode-notify`：拆出通用通知工具。
2. `opencode-loop-safety`：弱依赖 `opencode-wait` 与 `opencode-notify`。
3. `opencode-copilot-account-switcher` 收敛：移除对通用工具和策略的所有权。

本切片不直接启动这些后续工作。
