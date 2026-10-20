# bifrost_cn：通过 bnpm 分发

本仓库（`lukecc00/bifrost_cn`）是 `bifrost-proxy/bifrost` 的 fork，在上游基础上增加了管理端中文化（默认简体中文）。fork 不使用上游的 npmjs、Homebrew 和桌面安装包渠道，CLI 统一发布到字节内部 npm 源 bnpm。fmkit 等内部工具从 bnpm 安装。

## 包与版本

| 项 | 取值 |
| --- | --- |
| 主包 | `@bytedance-dev/bifrost` |
| 平台包 | `@bytedance-dev/bifrost-{darwin-arm64,darwin-x64,linux-x64,linux-arm64,linux-arm,linux-x64-musl,linux-arm64-musl,win32-x64,win32-arm64}` |
| Registry | `https://bnpm.byted.org`（`publishConfig.access: restricted`） |
| 版本号 | `<上游版本>-cn.<N>`，例如 `0.0.200-cn.1`；同一上游版本重新发版时递增 `N` |
| dist-tag | `latest`（`-cn` 版本不进 `next`） |
| GitHub Release | `v<版本>`，标记为 prerelease |

## 运行时行为

- `npm/bifrost`：二进制通过 `optionalDependencies` 安装；可选依赖被跳过时，`install.js` 会从 bnpm 下载对应平台包的 tarball。
- `bifrost upgrade`（npm/pnpm 安装方式，见 `crates/bifrost-cli/src/commands/upgrade/install_method.rs`）执行 `npm install -g --registry=https://bnpm.byted.org @bytedance-dev/bifrost@<版本>`，不会切回上游包。
- 版本检查（`crates/bifrost-core/src/version_check.rs`）读取本仓库的 GitHub Releases。`-cn` 版本属于 `cn` 发布渠道，只会提示升级到更新的 `-cn` 版本。

## 发版流程

1. **同步上游**（按需）：先执行 `git fetch upstream --tags`，再执行 `git merge upstream/main`，解决冲突后推送到 `main`。上游新增的界面文案需要补充 `web/src/i18n/locales/{en,zh-CN}` 词条。
2. **打 tag 触发构建**：执行 `git tag -a v0.0.200-cn.1 -m "Release v0.0.200-cn.1"`，然后执行 `git push origin v0.0.200-cn.1`。
   - `.github/workflows/release.yml` 会构建 9 个平台的 CLI，并创建 GitHub Release。
   - 在 fork 上会跳过三个 job：`build-desktop`、`update-homebrew`、`publish-npm`（npmjs）。
3. **下载产物**：执行 `gh run download <run-id> --repo lukecc00/bifrost_cn --pattern 'cli-*' --dir artifacts`。
4. **发布到 bnpm**：GitHub Actions 访问不到内网，所以需要在本机执行：

   ```bash
   npm login --registry=https://bnpm.byted.org
   BIFROST_VERSION=0.0.200-cn.1 ARTIFACTS_DIR=$PWD/artifacts node scripts/npm-publish.mjs
   ```

   可用环境变量覆盖发布参数：`BIFROST_NPM_SCOPE`（默认 `@bytedance-dev`）、`BIFROST_NPM_REGISTRY`（默认 `https://bnpm.byted.org`）、`BIFROST_NPM_ACCESS`（默认 `restricted`）和 `BIFROST_NPM_TAG`。加 `--dry-run` 可只检查不发布。
5. **验证**：执行 `npx --registry=https://bnpm.byted.org @bytedance-dev/bifrost@0.0.200-cn.1 --version`。
6. **更新 fmkit**：把 `config/tool-catalog.json` 中 bifrost 的安装包改为 `@bytedance-dev/bifrost@<版本>`。fmkit 的 npm 源已经是 bnpm。
