# Changelog

本文件记录团队研发协作规范的重要新增、调整和删除内容。

## [Unreleased]

### Added

- 增加 Git 基础、文件纳管和远程协作指南。
- 增加 GitLab 开发、发布、项目管理和 Merge Request 代码审查规范。
- 增加软件版本管理快速指南。

### Changed

- 优化规范文档的名称、章节结构和推荐阅读顺序。
- 将代码管理基础原则按版本记录、分支协作和文件安全重新分层。
- 补充 `git rebase` 的适用场景、冲突处理、交互式整理和安全推送指南。
- 本地仓库初始化示例统一使用 `git init -b main`。
- `main` 和 `dev` 统一由 Maintainers 负责合并。
- 明确 `release/*`、`hotfix/*` 必须同时合入 `main` 和 `dev` 后才能删除。
- 更新 `.gitignore`，忽略 macOS 系统文件和文档转换输出目录。

### Removed

- 删除与当前仓库定位不一致的完整软件版本与发布管理规范，保留软件版本管理快速指南。
