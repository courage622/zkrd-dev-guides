# 中科融道软件版本管理快速指南

本文档用于日常快速查阅，完整要求请参阅[软件版本与发布管理规范](./VERSIONING_AND_RELEASE.md)。如两份文档存在不一致，以完整规范为准。

## 目录

- [1. 版本管理基础](#1-版本管理基础)
  - [1.1 必须存在的文件](#11-必须存在的文件)
  - [1.2 其他地方如何使用版本号](#12-其他地方如何使用版本号)
  - [1.3 版本号更新规则](#13-版本号更新规则)
- [2. 版本变更与记录](#2-版本变更与记录)
  - [2.1 在什么时候、什么分支修改](#21-在什么时候什么分支修改)
  - [2.2 CHANGELOG.md 怎么写](#22-changelogmd-怎么写)
  - [2.3 版本更新怎么提交](#23-版本更新怎么提交)
- [3. 正式发布与校验](#3-正式发布与校验)
  - [3.1 正式 Tag 怎么创建](#31-正式-tag-怎么创建)
  - [3.2 CI 必须检查](#32-ci-必须检查)
- [4. 执行流程](#4-执行流程)

## 1. 版本管理基础

### 1.1 必须存在的文件

每个项目根目录必须有：

```text
VERSION
CHANGELOG.md
```

| 文件 | 作用 |
|---|---|
| `VERSION` | 保存项目当前版本，是当前版本号的唯一来源 |
| `CHANGELOG.md` | 记录每个版本增加、修改和修复了什么 |

`VERSION` 只写一个不带 `v` 的版本号：

```text
1.4.0
```

禁止写成 `VERSION=1.4.0`、`v1.4.0` 或包含其他说明。

### 1.2 其他地方如何使用版本号

除 `VERSION` 和 `CHANGELOG.md` 外，不得再手工维护独立版本号。

所有需要当前版本的代码和文件必须：

1. 直接读取根目录 `VERSION`；或
2. 在构建时由脚本把 `VERSION` 自动注入；或
3. 由版本同步脚本自动更新，并由 CI 检查一致性。

不同项目统一按下面方式处理：

| 场景 | 统一要求 |
|---|---|
| Python 项目清单需要版本 | 使用动态版本配置或构建脚本从 `VERSION` 读取 |
| Python 代码查询版本 | 读取打包元数据、构建变量或 `VERSION`，不得写死 `__version__` |
| Node.js 项目清单需要版本 | 发布脚本根据 `VERSION` 自动生成或同步 |
| 前端页面显示版本 | 构建时从 `VERSION` 注入，不得写死在源码中 |
| Java、Go、Rust 等项目 | 构建配置从 `VERSION` 读取或自动同步 |
| 容器镜像标签 | CI 从 `VERSION` 生成，例如 `application:1.4.0` |
| Git Tag | 在 `VERSION` 前加 `v`，例如 `v1.4.0` |
| 版本接口、日志、关于页面 | 读取或展示构建时注入的 `VERSION` |

如果某个构建工具不能直接读取 `VERSION`，项目必须提供自动同步脚本。开发人员不得分别手工修改多个版本位置。

### 1.3 版本号更新规则

版本号使用：

```text
MAJOR.MINOR.PATCH
主版本.次版本.修订版本
```

| 修改类型 | 更新规则 | 示例 |
|---|---|---|
| 向后兼容的问题修复 | 增加 `PATCH` | `1.4.0 → 1.4.1` |
| 向后兼容的新功能 | 增加 `MINOR`，`PATCH` 归零 | `1.4.1 → 1.5.0` |
| 不兼容的重大变化 | 增加 `MAJOR`，其余归零 | `1.5.0 → 2.0.0` |

`0.x` 项目执行：

- 普通修复增加 `PATCH`。
- 新功能或不兼容变化增加 `MINOR`。
- 开始承诺稳定兼容性时发布 `1.0.0`。

同一版本同时包含多种变化时，按影响最大的变化升级。已经发布过的版本号不得再次使用。

## 2. 版本变更与记录

### 2.1 在什么时候、什么分支修改

#### 2.1.1 普通开发分支

`feature/*` 和普通 `fix/*` 分支：

- 不修改 `VERSION`。
- 功能或修复说明写入 `CHANGELOG.md` 的 `Unreleased`。
- 完成后通过 MR 合入开发主线。

#### 2.1.2 正式发布分支

确定发布范围后，从开发主线创建：

```text
release/vX.Y.Z
```

发布分支完成：

1. 把 `VERSION` 修改为 `X.Y.Z`。
2. 把 `CHANGELOG.md` 的 `Unreleased` 整理为 `[X.Y.Z]`。
3. 运行版本同步脚本。
4. 提交版本更新并创建发布 MR。

禁止直接在开发主线或稳定主线上修改版本号。

#### 2.1.3 生产热修复分支

生产紧急修复从稳定主线创建：

```text
hotfix/vX.Y.Z
```

热修复分支完成：

1. 修复生产问题。
2. 增加 `PATCH` 版本。
3. 更新 `VERSION` 和 `CHANGELOG.md`。
4. 合入稳定主线和开发主线。
5. 在稳定主线创建新 Tag。

禁止修改代码后继续使用原来的版本号。

### 2.2 CHANGELOG.md 怎么写

日常开发写入：

```markdown
# Changelog

## [Unreleased]

### Added

- 增加某项功能。

### Fixed

- 修复某项问题。
```

准备发布 `1.4.0` 时整理为：

```markdown
## [Unreleased]

## [1.4.0] - YYYY-MM-DD

### Added

- 增加某项功能。

### Fixed

- 修复某项问题。
```

至少写清楚：新增功能、问题修复、数据库或配置变化、不兼容变化和注意事项。

### 2.3 版本更新怎么提交

先检查：

```bash
cat VERSION
git diff
git status
```

版本提交只包含：

- `VERSION`。
- `CHANGELOG.md`。
- 自动同步且必须提交的构建元数据。

提交信息固定为：

```text
chore(release): prepare vX.Y.Z
```

例如：

```bash
git add VERSION CHANGELOG.md
git commit -m "chore(release): prepare v1.4.0"
git push
```

版本更新必须单独提交，不得混入功能代码或无关修改。

## 3. 正式发布与校验

### 3.1 正式 Tag 怎么创建

发布 MR 合入稳定主线后，在对应发布提交上执行：

```bash
git tag -a v1.4.0 -m "Release v1.4.0"
git push origin v1.4.0
```

创建前必须满足：

```text
VERSION：1.4.0
Git Tag：v1.4.0
CHANGELOG：包含 [1.4.0]
```

Tag 创建后不得移动、覆盖或删除后重建。后续问题必须发布新版本，例如 `v1.4.1`。

### 3.2 CI 必须检查

CI 至少检查：

1. `VERSION` 存在且格式为 `X.Y.Z`。
2. `CHANGELOG.md` 存在。
3. 自动生成或同步的版本与 `VERSION` 一致。
4. 正式 Tag 去掉 `v` 后与 `VERSION` 一致。
5. `CHANGELOG.md` 包含正式 Tag 对应的版本。

任意检查失败时，禁止正式发布。

## 4. 执行流程

```text
普通开发：
feature/* 或 fix/*
→ 不修改 VERSION
→ 更新 CHANGELOG 的 Unreleased
→ MR 合入开发主线

正式发布：
release/vX.Y.Z
→ 修改 VERSION
→ 整理 CHANGELOG
→ 自动同步其他版本位置
→ 提交 chore(release): prepare vX.Y.Z
→ MR 合入稳定主线
→ 创建 vX.Y.Z Tag

生产热修复：
hotfix/vX.Y.Z
→ 修复问题并增加 PATCH
→ 修改 VERSION 和 CHANGELOG
→ 合入稳定主线和开发主线
→ 创建 vX.Y.Z Tag
```
