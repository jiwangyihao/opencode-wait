# opencode-wait 独立 GitHub 仓库设计

## 背景

`opencode-wait` 已在 `feature/opencode-wait-slice` 中拆为独立 npm 包，当前位于 `packages/opencode-wait`。用户已确认后续发布目标为公开 GitHub 仓库 `jiwangyihao/opencode-wait`，并选择「独立快照」策略。

## 目标

创建一个只包含 `opencode-wait` 包内容的独立公开仓库，让用户可以在 GitHub 上直接查看、克隆和继续维护这个通用 OpenCode `wait` 插件。

## 非目标

- 不在本次动作中发布 npm 包。
- 不创建 GitHub Release 或 tag。
- 不把 Copilot、WeChat、Loop Safety、`notify` 或当前 monorepo 的其他能力带入独立仓库。
- 不保留 monorepo 完整历史；独立仓库使用新的初始提交。

## 方案

采用 package-only snapshot：

1. 从 `.worktrees/opencode-wait-slice/packages/opencode-wait` 复制独立包源码、测试、配置和 README 到临时发布目录。
2. 将当前仓库根 `LICENSE` 复制到临时发布目录，匹配 `package.json` 中的 `MPL-2.0` 协议声明。
3. 在临时发布目录初始化新的 Git 仓库，默认分支设为 `master`，创建一个初始提交。
4. 使用 GitHub CLI 创建公开远端 `jiwangyihao/opencode-wait`，并推送 `master`。
5. 通过 `gh repo view`、远端 `git ls-remote`、本地 `npm test` 和一个最小导入 driver 验证仓库与包行为。

## 交付边界

独立仓库首版应包含：

- `package.json`
- `package-lock.json`（如果独立目录 install 后生成）
- `tsconfig.json`
- `tsconfig.build.json`
- `src/`
- `test/`
- `README.md`
- `LICENSE`

不包含：

- monorepo 根 `README.md`
- monorepo 根 `package.json`
- `.worktrees/`
- `docs/superpowers/`
- 任何 Copilot / WeChat 相关源码或测试

## 验证标准

- 独立目录中 `npm install` 成功。
- 独立目录中 `npm test` 通过。
- 独立目录中 `npm run build` 通过。
- 最小 Node driver 从构建产物导入 `createWaitPlugin`，并能看到 `wait` 工具注册。
- GitHub 远端存在且可通过 `gh repo view jiwangyihao/opencode-wait` 查询。
- `git ls-remote` 能看到 `refs/heads/master`。

## 风险与处理

- 如果远端仓库已存在，先读取其状态，不覆盖已有内容；必要时停止并询问用户。
- 如果 GitHub CLI 创建仓库失败，不假设后续自动化接管，必须显式报告失败原因。
- 如果验证失败，不推送失败内容；先修复独立快照再提交。
