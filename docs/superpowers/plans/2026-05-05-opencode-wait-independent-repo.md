# opencode-wait 独立 GitHub 仓库实现计划

> **面向 AI 代理的工作者：** 必需子技能：使用 superpowers:subagent-driven-development（推荐）或 superpowers:executing-plans 逐任务实现此计划。步骤使用复选框（`- [ ]`）语法来跟踪进度。

**目标：** 将 `packages/opencode-wait` 发布为公开独立 GitHub 仓库 `jiwangyihao/opencode-wait`。

**架构：** 使用 package-only snapshot。独立仓库只包含 `opencode-wait` 包内容和 MPL-2.0 许可证，不保留 monorepo 历史，不包含 Copilot / WeChat / docs coordination 内容。

**技术栈：** Git、GitHub CLI、npm、TypeScript、Node.js test runner。

---

## 文件结构

- 创建临时目录：`C:\Users\34404\AppData\Local\Temp\opencode\opencode-wait-repo`
- 复制来源：`C:\Users\34404\Documents\GitHub\opencode-copilot-analysis\copilot-account-switcher\.worktrees\opencode-wait-slice\packages\opencode-wait`
- 复制许可证：`C:\Users\34404\Documents\GitHub\opencode-copilot-analysis\copilot-account-switcher\.worktrees\opencode-wait-slice\LICENSE`
- 推送远端：`https://github.com/jiwangyihao/opencode-wait.git`

## 任务 1：准备独立快照目录

- [ ] 删除并重建临时目录 `C:\Users\34404\AppData\Local\Temp\opencode\opencode-wait-repo`。
- [ ] 复制 `packages/opencode-wait` 中的源码、测试、配置和 README。
- [ ] 复制根 `LICENSE` 到临时目录。
- [ ] 确认临时目录不包含 `dist/`、`node_modules/`、`.git/` 或 monorepo docs。

## 任务 2：验证独立包

- [ ] 在临时目录运行 `npm install`。
- [ ] 运行 `npm test`，预期全部通过。
- [ ] 运行 `npm run build`，预期退出码为 0。
- [ ] 用最小 Node driver 从 `./dist/index.js` 导入 `createWaitPlugin`，确认返回插件会注册 `wait` 工具。

## 任务 3：创建独立仓库提交

- [ ] 在临时目录运行 `git init -b master`。
- [ ] 添加远端 `origin https://github.com/jiwangyihao/opencode-wait.git`。
- [ ] 检查 `git status --short`，确认只有独立包文件。
- [ ] 创建初始提交，提交信息为 `feat: 初始化 opencode-wait 独立仓库`。

## 任务 4：创建并推送 GitHub 远端

- [ ] 使用 `gh repo view jiwangyihao/opencode-wait` 检查远端是否已存在。
- [ ] 若不存在，运行 `gh repo create jiwangyihao/opencode-wait --public --source . --remote origin --push`。
- [ ] 若已存在且为空，推送 `master`。
- [ ] 若已存在且非空，停止并询问用户，不覆盖远端内容。

## 任务 5：远端验证

- [ ] 运行 `gh repo view jiwangyihao/opencode-wait --json nameWithOwner,url,visibility,defaultBranchRef`，确认仓库为 public 且默认分支为 `master`。
- [ ] 运行 `git ls-remote --heads origin master`，确认远端 `master` 已存在。
- [ ] 运行 `git status --short`，确认独立目录 clean。
