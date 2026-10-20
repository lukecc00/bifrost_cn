# bifrost_cn：通过 GitHub Release 分发

本仓库（`lukecc00/bifrost_cn`）是 `bifrost-proxy/bifrost` 的 fork，在上游基础上增加了管理端中文化（默认简体中文）。fork 只发布 CLI 二进制到本仓库的 GitHub Release，不使用上游的 npm（`@bifrost-proxy/bifrost`）、Homebrew 或桌面安装包渠道。fmkit 等工具直接下载 Release 中的二进制。

## 版本与产物

- 版本号：`<上游版本>-cn.<N>`，例如 `0.0.200-cn.1`；同一上游版本重新发版时递增 `N`。
- Tag 为 `v<版本>`。Release 是普通发布，不是 prerelease，所以 `releases/latest` 会指向最新的 `-cn` 版本。
- 版本检查（`crates/bifrost-core/src/version_check.rs`）把 `-cn` 视为稳定渠道，`bifrost upgrade` 只会升级到更新的 `-cn` 版本。
- 每个平台的产物如下，`<target>` 为 Rust target triple：
  - `bifrost-v<版本>-<target>.tar.gz`
  - `bifrost-v<版本>-<target>.tar.xz`（Windows 为 `.zip`）
  - 汇总校验文件：`bifrost-v<版本>-checksums.txt`

| 平台 | target |
| --- | --- |
| macOS Apple Silicon | `aarch64-apple-darwin` |
| macOS Intel | `x86_64-apple-darwin` |
| Linux x64 / arm64 (glibc) | `x86_64-unknown-linux-gnu` / `aarch64-unknown-linux-gnu` |
| Linux x64 / arm64 (musl) | `x86_64-unknown-linux-musl` / `aarch64-unknown-linux-musl` |
| Linux armv7 | `armv7-unknown-linux-gnueabihf` |
| Windows x64 / arm64 | `x86_64-pc-windows-msvc` / `aarch64-pc-windows-msvc` |

下载地址格式：

```
https://github.com/lukecc00/bifrost_cn/releases/download/v<版本>/bifrost-v<版本>-<target>.tar.gz
```

## 指向 fork 的位置

- `install-binary.sh`：默认仓库是 `lukecc00/bifrost_cn`，可用 `BIFROST_INSTALL_REPO` 覆盖。
- `bifrost upgrade`：脚本安装和手动安装两种方式从本仓库的 Release 下载；失败时提示的安装命令也指向本仓库。
- 版本检查：读取本仓库的 Releases。
- `.github/workflows/release.yml`：`build-desktop`、`update-homebrew`、`publish-npm` 只在上游仓库运行；桌面端构建被跳过时，Release 仍会正常创建。

## 发版流程

1. **同步上游**（按需）：执行 `git fetch upstream --tags`，再执行 `git merge upstream/main`，解决冲突后推送到 `main`。上游新增的界面文案需要在 `web/src/i18n/locales/{en,zh-CN}` 中补充词条。
2. **打 tag**：执行 `git tag -a v0.0.200-cn.1 -m "Release v0.0.200-cn.1"`，再执行 `git push origin v0.0.200-cn.1`。`release.yml` 会构建 9 个平台并创建 GitHub Release。
3. **验证**：下载对应平台的产物，用 checksums 文件校验后，确认 `bifrost --version` 输出 `0.0.200-cn.1`。
4. **更新 fmkit**：把 fmkit 中 bifrost 的安装方式改为按平台下载上述 Release 产物，固定版本并校验 checksum。
