# 发布流程

## 手动首发

`opencode-wait` 第一次发布到 npm 时，需要先使用本地 npm 账号完成手动发布：

```bash
npm install
npm test
npm publish --access public
```

发布前必须确认：

- `npm test` 通过。
- `npm pack --dry-run` 只包含 `dist/`、`README.md`、`LICENSE` 和 `package.json` 中声明的发布文件。
- `package.json` 的 `repository.url` 指向 `git+https://github.com/jiwangyihao/opencode-wait.git`。

## 后续 GitHub Actions 发布

仓库使用 `.github/workflows/release.yml`。当 GitHub Release 发布后，workflow 会：

1. 检出代码。
2. 使用 Node.js 24。
3. 升级到支持 Trusted Publishing 的新版 npm。
4. 运行 `npm ci`、`npm run build` 和 `npm run test:built`。
5. 如果当前版本还没发布，则通过 npm Trusted Publishing (OIDC) 执行 `npm publish --access public`。

## npm Trusted Publishing 设置

手动首发后，在 npmjs.com 的 `opencode-wait` 包设置中添加 Trusted Publisher：

- Provider: GitHub Actions
- Organization or user: `jiwangyihao`
- Repository: `opencode-wait`
- Workflow filename: `release.yml`
- Environment name: 留空

npm 官方要求 Trusted Publishing 使用 npm CLI 11.5.1 或更高版本、Node.js 22.14.0 或更高版本，并且 GitHub Actions workflow 必须授予 `id-token: write` 权限。
