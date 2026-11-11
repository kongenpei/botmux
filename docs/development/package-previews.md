# PR 预览包

`Package Preview` workflow 在 PR、`master` push 和手动触发时，通过
[pkg.pr.new](https://github.com/stackblitz-labs/pkg.pr.new) 发布当前提交的预览包。

仓库管理员需要先将 [pkg.pr.new GitHub App](https://github.com/apps/pkg-pr-new)
安装到目标仓库。fork 需要单独安装；只配置 workflow 并不能代替 App 安装。
不需要 npm token。

## 使用

在 PR 的 pkg.pr.new 评论中找到 **botmux** 的完整 URL，例如：

```sh
npx https://pkg.pr.new/OWNER/botmux/botmux@COMMIT --version
```

临时试用可以使用 `npx`。执行 `start`、`restart` 等命令会操作当前用户的
botmux 数据；需要独立测试时设置单独的 `HOME`。

预览包版本为 `0.0.0-preview-<提交 SHA 前七位>`。URL 带上 owner、仓库名和提交，
因此 fork 的预览不会被解析到上游 npm 包。各个平台的二进制和主包一起发布，
主包的 optional dependencies 由 pkg.pr.new 重写为同一提交的 URL。

## 验证与范围

- Linux x64 / arm64：复用正式发布的 glibc 2.28 构建与 smoke。
- Linux x64 / arm64 musl：在对应架构的 Alpine 中编译 native addon 并执行 smoke。
- macOS x64 / arm64：检查架构、ad-hoc 签名和 canvas native；执行宿主架构 smoke。
- 发布后，在隔离的 HOME 和 npm prefix 中安装实际 URL，执行 `--version` 和
  `capabilities --json`。Actions summary 和 `preview-evidence` artifact 留存结果。

预览 macOS 二进制只有 ad-hoc 签名，不经过正式发布的 Developer ID 签名流程。
`botmux-workflow-core` 和桌面安装包不在这个 workflow 的发布范围内。
预览使用单独的暂存目录，不修改提交中的版本字段，也不发布 npm dist-tag 或
GitHub Release。

发布工具固定在 `.github/pkg-pr-new/package-lock.json`，通过 `npm ci` 安装；
仓库本身的构建仍使用 `bun@1.4.3` 和 `bun.lock`。
