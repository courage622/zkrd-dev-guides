# 中科融道软件版本与发布管理规范

## 1. 目的与适用范围

本规范用于统一团队对版本号、版本文件、发布提交、Git Tag、发布制品和发布记录的管理方式。

本文档是软件版本与发布管理的完整规范。日常操作可查阅配套的[软件版本管理快速指南](./VERSIONING_QUICK_GUIDE.md)；如两份文档存在不一致，以本文档为准。

所有由团队维护并正式发布的软件仓库都必须遵守本规范。一个仓库包含多个共同构建、共同部署的组件时，这些组件使用同一个产品版本；确需独立发布的组件视为独立发布单元，分别执行本规范。

每个正式版本必须形成以下唯一对应关系：

```text
产品版本
→ Git Tag
→ Git Commit
→ 不可变发布制品
→ 制品 checksum/digest
→ Release 和部署记录
```

## 2. 统一版本来源

### 2.1 强制规则

每个发布单元必须在根目录维护一个名为 `VERSION` 的纯文本文件，并将其作为产品版本的唯一来源（Single Source of Truth）。

`VERSION` 只允许包含一个不带 `v` 的版本号和结尾换行：

```text
1.4.0
```

其他位置出现的版本号都属于映射值，必须由版本工具同步修改或由 CI 检查与 `VERSION` 一致，包括：

- 语言或构建工具的项目清单。
- 依赖锁定文件中的项目版本。
- 应用版本接口或页面展示。
- 安装包、压缩包和容器标签。
- 部署清单和发布记录。

禁止把任意组件清单、环境变量或 `latest` 标签作为独立的第二版本来源。

### 2.2 多组件仓库

同一仓库内共同发布的组件必须共用根目录 `VERSION`，不得各自决定产品版本。

本仓库的前端和后端共同部署，因此统一使用：

```text
VERSION                         0.2.0
pyproject.toml                  0.2.0
frontend/package.json           0.2.0
frontend/package-lock.json      0.2.0
Git Tag                         v0.2.0
后端镜像                        workticket-backend:0.2.0
前端镜像                        workticket-frontend:0.2.0
```

当前仓库后端版本为 `0.2.0`、前端版本为 `0.1.0`，且尚无根目录 `VERSION`。首次正式发布前必须增加 `VERSION` 并统一所有映射值。

如果一个仓库中的组件确实独立发布，则每个组件目录必须有自己的 `VERSION`，Tag 统一使用：

```text
<component>-v<MAJOR.MINOR.PATCH>
```

例如 `backend-v1.2.0`。同一组件不得同时使用仓库级版本和组件级版本。

## 3. 版本号格式与升级规则

### 3.1 正式版本

正式版本强制采用语义化版本：

```text
MAJOR.MINOR.PATCH
主版本.次版本.修订版本
```

| 变化类型 | 强制更新规则 | 示例 |
|---|---|---|
| 向后兼容的缺陷、安全或性能修复 | `PATCH + 1` | `1.4.2 → 1.4.3` |
| 向后兼容的新功能 | `MINOR + 1`，`PATCH` 归零 | `1.4.2 → 1.5.0` |
| 不兼容的接口、配置或数据变化 | `MAJOR + 1`，其余归零 | `1.4.2 → 2.0.0` |

同一版本同时包含多种变化时，按影响最大的变化升级。例如同时包含新功能和普通修复，升级 `MINOR`。

### 3.2 `0.x` 阶段

未发布 `1.0.0` 的项目统一执行：

- 普通修复：增加 `PATCH`。
- 新功能或不兼容变化：增加 `MINOR`，`PATCH` 归零。
- 开始向使用方承诺稳定兼容性时发布 `1.0.0`。

本仓库示例：

```text
0.2.0 → 0.2.1    普通生产修复
0.2.1 → 0.3.0    新功能或明显不兼容变化
0.x   → 1.0.0    首个稳定兼容版本
```

### 3.3 预发布版本

预发布版本只使用以下格式：

```text
X.Y.Z-alpha.N
X.Y.Z-beta.N
X.Y.Z-rc.N
```

- `alpha`：功能尚未完整，仅供内部早期验证。
- `beta`：功能已经完整，用于扩大测试。
- `rc`：发布候选，只允许修复阻断发布的问题。
- `N` 从 `1` 开始连续递增，不得覆盖旧候选版本。

例如：

```text
0.3.0-rc.1
0.3.0-rc.2
0.3.0
```

## 4. 版本号更新时间

### 4.1 日常开发不升版

`feature/*` 和普通 `fix/*` 分支不得修改 `VERSION` 及其映射值。开发构建统一使用 Commit SHA 或 Pipeline ID 标识，例如：

```text
dev-a1b2c3d
pipeline-1024
```

日常变化必须写入 `CHANGELOG.md` 的 `Unreleased` 部分，但不提前确定正式版本。

例如，本仓库新增历史工作票功能时，功能分支仍保持当前版本；只有该功能进入确定发布范围后才统一升版。

### 4.2 发布准备时升版

发布范围确定后，从最新开发主线创建 `release/vX.Y.Z`，并立即完成版本更新：

```text
确定发布范围
→ 判断 MAJOR / MINOR / PATCH
→ 创建 release/vX.Y.Z
→ 更新 VERSION 和所有映射值
→ 整理 CHANGELOG
→ 提交版本准备变更
```

版本准备提交的格式固定为：

```text
chore(release): prepare vX.Y.Z
```

本仓库准备发布 `0.3.0` 时，应在 `release/v0.3.0` 中统一修改：

```text
VERSION
pyproject.toml
frontend/package.json
frontend/package-lock.json
CHANGELOG.md
```

前端清单和锁定文件使用包管理工具同步修改：

```bash
cd frontend
npm version 0.3.0 --no-git-tag-version
```

发布准备提交不得混入普通功能实现。

### 4.3 热修复时升版

生产紧急修复必须从当前生产对应的稳定分支或 Tag 创建 `hotfix/vX.Y.Z`，并升级 `PATCH`：

```text
v0.3.0
→ hotfix/v0.3.1
→ VERSION 更新为 0.3.1
→ 修复、测试和发布
```

禁止修改代码后继续使用原版本 `v0.3.0`。

### 4.4 发布后不预升下一版本

正式发布完成后，开发主线保持最近一次正式版本号。团队不使用 `next`、`snapshot`、`dev` 等预升版本。

下一版本只在下一次发布范围确定并创建发布分支时更新。开发中的精确代码由 Commit SHA 标识。

## 5. 版本修改内容与提交要求

### 5.1 必须修改的内容

每次升版必须一次性检查和更新：

1. 根目录 `VERSION`。
2. 各组件项目清单中的映射版本。
3. 锁定文件中的项目版本。
4. 应用运行时显示的版本来源。
5. `CHANGELOG.md`。
6. 部署清单中的制品版本引用。

本仓库版本准备提交示例：

```bash
git add VERSION pyproject.toml frontend/package.json \
  frontend/package-lock.json CHANGELOG.md
git commit -m "chore(release): prepare v0.3.0"
```

### 5.2 提交规则

- 版本更新必须形成独立 Commit。
- 版本 Commit 只包含版本、变更日志和发布元数据。
- 候选版本发现问题时增加新的修复 Commit，不得修改已经共享的 Commit。
- 版本更新必须通过 Merge Request、CI 和非作者评审。
- 禁止直接向稳定主分支提交版本修改。

## 6. CHANGELOG 规范

仓库根目录必须维护 `CHANGELOG.md`。

日常开发写入：

```markdown
## [Unreleased]

### Added

- 增加历史工作票检索功能。

### Fixed

- 修复异常日志敏感信息泄漏问题。
```

发布 `0.3.0` 时整理为：

```markdown
## [Unreleased]

## [0.3.0] - 2026-07-16

### Added

- 增加历史工作票检索功能。

### Fixed

- 修复异常日志敏感信息泄漏问题。
```

版本条目固定使用以下分类，没有内容的分类可以省略：

- `Added`：新增功能。
- `Changed`：兼容性变化。
- `Deprecated`：计划废弃但仍可使用。
- `Removed`：已删除功能。
- `Fixed`：问题修复。
- `Security`：安全修复。

数据库、配置、部署和回滚要求必须写在对应版本条目中。变更日志面向使用者描述结果，不得仅复制 Commit 标题。

## 7. 候选制品、Tag 与正式发布

### 7.1 候选制品

发布分支通过基础 CI 后，必须生成至少一个候选制品。候选制品使用目标版本、`rc` 序号和 Commit SHA 标识：

```text
artifact:0.3.0-rc.1
commit:a1b2c3d
```

每个候选制品必须对应唯一 Commit 和唯一制品 digest。发现问题后创建新 Commit 和新候选制品，`rc` 序号递增，不得覆盖旧候选制品。

### 7.2 正式 Tag

发布变更合入稳定主分支，且该提交构建的候选制品通过预发布验收后，在同一提交上创建附注 Tag：

```bash
git tag -a v0.3.0 -m "Release v0.3.0"
git push origin v0.3.0
```

正式 Tag 必须满足：

- Tag 名为 `v` 加 `VERSION` 内容。
- Tag 指向稳定主分支上的发布提交。
- 对应 Commit 已通过 CI、回归测试和发布审批。
- Tag 设置为 Protected Tag。
- Tag 创建后永久保留，不得移动、覆盖或重建。

### 7.3 制品晋级

正式发布遵守“构建一次、逐级晋级”：

```text
稳定主分支 Commit
→ 构建带 Commit SHA 的候选制品
→ 部署预发布并验收
→ 创建正式 Git Tag
→ 给同一制品 digest 增加正式版本标签
→ 部署同一 digest 到生产
```

本仓库发布 `0.3.0` 时：

```text
workticket-backend:<commit-sha>  → workticket-backend:0.3.0
workticket-frontend:<commit-sha> → workticket-frontend:0.3.0
```

正式镜像标签必须指向已经验收的同一 digest，禁止以同一个 `0.3.0` 标签重新构建不同内容。

生产部署必须固定正式版本或 digest。`latest` 只能作为非生产辅助别名，不能用于生产部署、追溯或回滚。

## 8. 运行时版本信息

正式应用必须能够查询实际运行版本，至少返回：

```json
{
  "version": "0.3.0",
  "commit": "a1b2c3d",
  "build_time": "2026-07-16T10:30:00+08:00"
}
```

版本、Commit 和构建时间必须在 CI 构建时注入，不得由生产人员手工修改。前端应在“关于”页面或诊断信息中展示相同版本和 Commit。

## 9. CI 强制校验

所有仓库必须设置版本校验 Job。当前仓库至少校验：

```text
VERSION == pyproject.toml version
VERSION == frontend/package.json version
VERSION == frontend/package-lock.json project version
```

Tag Pipeline 额外校验：

```text
Git Tag 去掉 v 后 == VERSION
目标 Tag 尚不存在于正式发布记录
目标正式制品标签尚未被占用
Tag 来源为允许的稳定分支提交
CHANGELOG 包含目标版本
```

任意检查失败时，Pipeline 必须失败并阻止合并、Tag 发布或制品发布。

## 10. 分支与版本关系

| 分支类型 | 是否允许修改版本 | 统一要求 |
|---|---|---|
| `feature/*` | 否 | 使用 Commit SHA 标识开发构建 |
| 普通 `fix/*` | 否 | 随下一正式版本发布 |
| `release/vX.Y.Z` | 是 | 更新为 `X.Y.Z` 并整理 CHANGELOG |
| `hotfix/vX.Y.Z` | 是 | 从生产版本增加 `PATCH` |
| 稳定主分支 | 禁止直接修改 | 只接收通过 MR 的发布变更 |

发布分支中的版本提交和修复必须同步到后续开发主线；确认同步完成后删除发布分支。Git Tag 和正式制品永久保留。

## 11. 角色与责任

每次发布必须指定一名发布负责人，统一执行升版和 Tag 操作。其他开发人员不得各自修改正式版本号或创建正式 Tag。

| 角色 | 责任 |
|---|---|
| 开发人员 | 维护 `Unreleased`、完成测试，不在普通分支升版 |
| 发布负责人 | 确定版本、统一修改、创建发布 MR 和 Tag |
| 评审人员 | 检查升版规则、兼容性、CHANGELOG 和风险 |
| CI | 检查版本一致性、Tag、制品唯一性和测试结果 |
| 发布审批人 | 确认预发布验收、部署和回滚条件 |

## 12. 禁止事项

- 禁止存在多个互不校验的版本来源。
- 禁止在普通功能分支中修改正式版本号。
- 禁止只修改部分版本位置。
- 禁止由多人分别创建正式版本和 Tag。
- 禁止覆盖、移动、删除后重建已发布 Tag。
- 禁止用同一版本号发布内容不同的制品。
- 禁止仅使用 `latest`、分支名或 Pipeline ID 标识正式版本。
- 禁止把数据库 migration、API 版本或数据版本当作产品版本。
- 禁止在 CI、评审和验收未通过时创建正式 Tag。

## 13. 发布检查清单

### 发布准备

- [ ] 已指定发布负责人。
- [ ] 已根据变更范围确定版本增量。
- [ ] 已创建 `release/vX.Y.Z`。
- [ ] `VERSION` 和所有映射位置完全一致。
- [ ] `CHANGELOG.md` 已包含版本、日期和重要变化。
- [ ] 版本修改已形成独立 `chore(release)` 提交。
- [ ] CI、回归测试、迁移测试和预发布验收已通过。
- [ ] 部署和回滚方案已确认。

### 正式发布

- [ ] 发布变更已通过 MR 合入稳定主分支。
- [ ] 正式 Tag 创建在正确的发布提交上。
- [ ] Tag 与 `VERSION` 完全一致。
- [ ] 正式制品来自已验收的同一 Commit 和 digest。
- [ ] 已记录 Commit SHA、制品 digest 和发布时间。
- [ ] Release 已包含变更、迁移、部署和回滚说明。

### 发布完成

- [ ] 已确认生产环境实际运行的版本和 Commit。
- [ ] 已完成发布后监控和核心功能验证。
- [ ] 发布修改和修复已同步到后续开发主线。
- [ ] 临时发布分支已删除。
- [ ] 正式 Tag、制品和 Release 已永久保留。
