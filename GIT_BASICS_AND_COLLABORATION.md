# 中科融道 Git 基础与团队协作指南

## 目录

- [1. 代码管理基础理论](#1-代码管理基础理论)
  - [1.1 为什么需要代码管理](#11-为什么需要代码管理)
  - [1.2 代码管理的基础原则](#12-代码管理的基础原则)
  - [1.3 Git 代码管理知识的两个层次](#13-git-代码管理知识的两个层次)
- [2. Git 本地仓库基础](#2-git-本地仓库基础)
  - [2.1 本地仓库管理的目标](#21-本地仓库管理的目标)
  - [2.2 本地 Git 的核心模型](#22-本地-git-的核心模型)
  - [2.3 本地 Git 主要功能、命令和输出](#23-本地-git-主要功能命令和输出)
  - [2.4 本地仓库开发完整流程](#24-本地仓库开发完整流程)
- [3. Git 远程仓库与协作](#3-git-远程仓库与协作)
  - [3.1 为什么需要远程仓库](#31-为什么需要远程仓库)
  - [3.2 本地仓库和远程仓库的关系](#32-本地仓库和远程仓库的关系)
  - [3.3 远程相关的主要 Git 命令](#33-远程相关的主要-git-命令)
  - [3.4 GitLab / GitHub 的核心功能](#34-gitlab--github-的核心功能)
  - [3.5 基于远程仓库的协同开发标准流程](#35-基于远程仓库的协同开发标准流程)
  - [3.6 多人开发同一个需求](#36-多人开发同一个需求)
  - [3.7 远程协作中的冲突处理](#37-远程协作中的冲突处理)
- [4. 本地仓库与远程仓库命令对照](#4-本地仓库与远程仓库命令对照)
  - [4.1 只在本地使用](#41-只在本地使用)
  - [4.2 涉及远程仓库](#42-涉及远程仓库)
  - [4.3 最核心的区别](#43-最核心的区别)
- [5. 文件纳管与安全边界](#5-文件纳管与安全边界)
  - [5.1 哪些文件应该由 Git 管理](#51-哪些文件应该由-git-管理)
  - [5.2 Git 操作安全边界](#52-git-操作安全边界)

## 1. 代码管理基础理论

### 1.1 为什么需要代码管理

软件代码会持续变化，并且通常由多人共同维护。如果只依靠复制文件夹、压缩包、聊天附件或在服务器上直接修改文件，会产生以下问题：

- 不知道哪一份代码是最新版。
- 不知道谁修改了什么、为什么修改。
- 修改出错后无法准确恢复到原来的状态。
- 多个人同时修改时容易覆盖彼此代码。
- 无法可靠地进行代码评审和自动测试。
- 无法确认测试环境和生产环境运行的是哪个版本。
- 人员变化后，代码修改过程和设计原因难以追溯。

代码管理需要解决四个核心问题：

```text
变化可记录
协作可合并
版本可追踪
问题可恢复
```

Git 通过 Commit、分支、Tag 和历史记录解决代码版本问题；GitLab、GitHub 在 Git 基础上进一步解决团队共享、代码评审、权限、自动测试和发布管理问题。

### 1.2 代码管理的基础原则

#### 1.2.1 版本记录与可信来源

##### 1.2.1.1 代码仓库是唯一可信来源

只涉及本地仓库时，以本地 Git 历史为准；涉及代码共享和协作时，以 GitLab 或 GitHub 正式仓库为准。

普通目录副本、服务器临时文件、压缩包和聊天附件都不能代替正式仓库。

##### 1.2.1.2 所有有意义的变化都应形成记录

每次有明确目的的修改都应该形成 Commit。Commit 应能够说明：

- 谁修改了代码。
- 修改了哪些内容。
- 为什么修改。
- 修改发生在什么时间。

Commit 应小而完整，不把多个无关功能、临时文件和大范围格式化混在一起。

##### 1.2.1.3 正式版本必须固定和可追踪

正式发布使用不可移动的 Tag，例如 `v1.0.0`。Tag、构建制品和生产环境版本应当一一对应。

不能只用“最新代码”“main 最新版”或 `latest` 表示正式版本，因为这些名称会继续变化。

#### 1.2.2 分支协作与历史管理

##### 1.2.2.1 开发代码与稳定代码隔离

新功能和缺陷修复使用独立分支，不直接在稳定分支上开发：

```text
稳定分支
└── 任务分支
```

任务分支出现问题时，不会立即影响稳定代码。

##### 1.2.2.2 先检查和测试，再合并

在本地完成合并前应查看 Diff 并运行测试；涉及远程协作时，还应通过 MR/PR、CI 和其他成员的代码评审。

```text
开发
→ 查看修改
→ 自动测试
→ 代码评审
→ 合并稳定分支
```

##### 1.2.2.3 已共享的历史不能随意改写

已经 Push 到远程并被其他人使用的 Commit 属于共享历史。不得随意强推、删除或重写，否则可能造成他人的代码丢失或分支历史混乱。

##### 1.2.2.4 合并后及时清理短生命周期分支

功能或修复分支完成合并后应及时删除。删除已合并分支只会删除分支指针，不会删除已经进入目标分支的 Commit。

#### 1.2.3 文件安全与操作边界

##### 1.2.3.1 配置、密钥和运行产物不进入源码仓库

以下内容不能提交到 Git：

- 生产 `.env` 和环境专用配置。
- 密码、Token 和 SSH 私钥。
- 数据库备份。
- 运行日志和缓存。
- 临时压缩包和构建产物。
- 包含敏感信息的测试数据。

##### 1.2.3.2 只管理应该进入版本历史的文件

Git 不是项目目录的全量备份工具。源码、测试、构建配置和必要文档通常应该进入 Git；密钥、本地环境、缓存、日志和可重新生成的产物通常不应进入 Git。

判断重点是：这个文件是否需要由团队共同维护，以及是否是可靠复现、构建、测试或部署项目所必需的输入。

##### 1.2.3.3 不理解命令影响时先停止操作

执行命令前先确认当前分支和工作区状态。遇到不理解的错误时，先使用 `git status`、`git diff` 和 `git log` 获取信息，不要尝试强推、硬重置或强制删除。

### 1.3 Git 代码管理知识的两个层次

#### 1.3.1 第一层：Git 本地仓库基础

第一层只关注本地仓库中的文件和版本历史：

```text
编辑文件
→ 选择要记录的修改
→ 创建本地 Commit
→ 创建本地分支
→ 查看或恢复历史版本
```

这一层只需要 Git，不依赖 GitLab、GitHub 或网络。

#### 1.3.2 第二层：Git 远程仓库与协作

在本地 Git 基础上增加 GitLab 或 GitHub 远程仓库，用于代码共享和多人协同开发：

```text
本地 Git 仓库
↕ fetch / pull / push
GitLab 或 GitHub 远程仓库
↕
其他成员的本地 Git 仓库
```

远程平台进一步提供 MR/PR、代码评审、CI、权限和版本发布等协作功能。

#### 1.3.3 两个知识层次的关系

远程仓库与协作建立在本地 Git 之上。应先理解工作区、暂存区、Commit 和本地分支，再学习 Push、Pull、远程分支和 MR/PR。

```mermaid
flowchart LR
    W["工作区"] -->|git add| S["暂存区"]
    S -->|git commit| L["本地 Git 仓库"]
    L -->|git push| R["GitLab / GitHub"]
    R -->|git fetch / pull| L
    R --> T["其他成员的本地仓库"]
```

## 2. Git 本地仓库基础

### 2.1 本地仓库管理的目标

本地 Git 主要解决以下问题：

- 记录每一次有意义的代码变化。
- 查看某个时间点修改了什么。
- 给不同功能建立互不干扰的分支。
- 新代码出现问题时找到旧版本。
- 暂时保存未完成的修改。
- 给重要版本增加本地 Tag。

本地 Git 仓库通常位于项目目录的 `.git` 中。不要手工修改或删除 `.git` 目录。

### 2.2 本地 Git 的核心模型

#### 2.2.1 工作区、暂存区和本地仓库

```mermaid
flowchart LR
    W["工作区<br/>正在编辑的文件"] -->|git add| S["暂存区<br/>本次准备提交的内容"]
    S -->|git commit| L["本地仓库<br/>Commit 历史"]
    L -->|git restore / switch| W
```

| 区域 | 含义 |
|---|---|
| 工作区 | 磁盘上正在查看和修改的文件 |
| 暂存区 | 已经选中、准备进入下一个 Commit 的修改 |
| 本地仓库 | `.git` 中保存的 Commit、分支和 Tag 历史 |

必须区分：

- 保存文件不等于 `git add`。
- `git add` 不等于 `git commit`。
- `git commit` 后代码已经进入本地历史，但还没有任何远程备份。

#### 2.2.2 文件状态

| 状态 | 含义 |
|---|---|
| Untracked | 新文件，Git 尚未跟踪 |
| Modified | 已跟踪文件发生修改 |
| Staged | 修改已经加入暂存区 |
| Committed | 修改已经生成本地 Commit |

状态变化：

```text
创建或修改文件
→ git add
→ 暂存区
→ git commit
→ 本地 Commit
```

#### 2.2.3 Commit

Commit 是一次有明确目的的代码变更记录，包含修改内容、作者、时间、说明和唯一 ID。

推荐格式：

```text
<类型>(<模块>): <修改目的>
```

示例：

```text
feat(history): 增加历史票查询
fix(history): 修复空关键字异常
test(history): 增加查询测试
docs(git): 补充 Git 学习材料
```

#### 2.2.4 分支

分支是指向某个 Commit 的可移动指针。不同开发任务应使用分支进行隔离：

```text
main:              A──B──C
feature/example:         └──D──E
```

开发功能时在 `feature/example` 上产生 Commit，完成后再合入本地 `main`。

#### 2.2.5 Tag

Tag 是指向固定 Commit 的版本标记：

```text
main: A──B──C──D
              ↑
            v1.0.0
```

分支会继续移动，Tag 通常保持不变。

### 2.3 本地 Git 主要功能、命令和输出

#### 2.3.1 初始化仓库：`git init`

作用：把当前普通目录初始化为一个本地 Git 仓库。

```bash
mkdir my-project
cd my-project
git init
```

典型输出：

```text
Initialized empty Git repository in /path/to/my-project/.git/
```

| 输出部分 | 含义 |
|---|---|
| `Initialized empty Git repository` | 已创建一个空 Git 仓库 |
| `/path/to/my-project/.git/` | Git 历史和配置保存位置 |

同一个项目只需要初始化一次。已有 `.git` 的项目不要重复初始化。

#### 2.3.2 配置身份：`git config`

```bash
git config --global user.name "你的姓名"
git config --global user.email "你的公司邮箱"
git config --global --list
```

| 命令 | 作用 |
|---|---|
| `git config --global user.name` | 设置 Commit 作者姓名 |
| `git config --global user.email` | 设置 Commit 作者邮箱 |
| `git config --global --list` | 查看全局 Git 配置 |

设置成功时通常没有输出。查看配置可能输出：

```text
user.name=张三
user.email=zhangsan@example.com
init.defaultbranch=main
```

等号左边是配置项，右边是配置值。

#### 2.3.3 查看状态：`git status`

```bash
git status
git status --short
```

作用：查看当前分支、文件状态以及哪些内容会进入下一个 Commit。

典型输出：

```text
On branch feature/example

Changes to be committed:
  new file:   tests/test_example.py

Changes not staged for commit:
  modified:   src/example.py

Untracked files:
  notes.txt
```

| 输出部分 | 含义 |
|---|---|
| `On branch ...` | 当前所在分支 |
| `Changes to be committed` | 已暂存，会进入下一个 Commit |
| `Changes not staged for commit` | 已修改但尚未加入暂存区 |
| `Untracked files` | 新文件，Git 尚未跟踪 |
| `working tree clean` | 没有未提交修改 |

短格式：

```text
M  tests/test_example.py
 M src/example.py
?? notes.txt
```

前两列分别表示暂存区和工作区：

| 标记 | 含义 |
|---|---|
| `M ` | 修改已暂存 |
| ` M` | 修改未暂存 |
| `A ` | 新文件已暂存 |
| `D ` | 删除已暂存 |
| `??` | 未跟踪文件 |

#### 2.3.4 查看差异：`git diff`

```bash
git diff
git diff -- path/to/file
git diff --staged
```

| 命令 | 作用 |
|---|---|
| `git diff` | 查看尚未暂存的修改 |
| `git diff -- <file>` | 只查看指定文件 |
| `git diff --staged` | 查看已经暂存、将进入 Commit 的修改 |

典型输出：

```diff
diff --git a/src/example.py b/src/example.py
--- a/src/example.py
+++ b/src/example.py
@@ -10,2 +10,3 @@ def query():
     validate()
+    return result
```

| 输出部分 | 含义 |
|---|---|
| `--- a/...` | 修改前文件 |
| `+++ b/...` | 修改后文件 |
| `@@ -10,2 +10,3 @@` | 修改前后对应的行号范围 |
| `-` 开头 | 删除内容 |
| `+` 开头 | 新增内容 |
| 无符号行 | 未变化的上下文 |

没有输出表示对应范围没有差异。

#### 2.3.5 选择提交内容：`git add`

```bash
git add src/example.py
git add src/example.py tests/test_example.py
```

作用：把指定修改加入暂存区。

成功时通常没有输出，应使用以下命令确认：

```bash
git status
git diff --staged
```

初学阶段慎用 `git add .`，它可能把所有临时文件一起加入。

#### 2.3.6 取消暂存：`git restore --staged`

```bash
git restore --staged notes.txt
```

作用：把文件移出暂存区，但保留工作区修改。

成功时通常没有输出。再次执行 `git status --short`，状态可能从 `M ` 变为 ` M`。

#### 2.3.7 创建本地 Commit：`git commit`

```bash
git commit -m "feat(example): 增加示例功能"
```

作用：把暂存区内容生成一个本地 Commit。

典型输出：

```text
[feature/example 3ab92ef] feat(example): 增加示例功能
 2 files changed, 35 insertions(+), 4 deletions(-)
 create mode 100644 tests/test_example.py
```

| 输出部分 | 含义 |
|---|---|
| `feature/example` | Commit 创建所在分支 |
| `3ab92ef` | 新 Commit 的短 ID |
| 后方文字 | Commit Message |
| `2 files changed` | 修改了 2 个文件 |
| `35 insertions` | 新增 35 行 |
| `4 deletions` | 删除 4 行 |
| `create mode 100644` | 创建普通非可执行文件 |

`nothing to commit` 表示暂存区没有内容。

#### 2.3.8 查看历史：`git log`

```bash
git log --oneline --decorate -10
```

作用：显示最近 10 个 Commit 的简洁历史。

```text
3ab92ef (HEAD -> feature/example) feat(example): 增加示例功能
b0f5f1d (main) chore: 初始化项目
```

| 输出部分 | 含义 |
|---|---|
| `3ab92ef` | Commit 短 ID |
| `HEAD -> feature/example` | 当前检出的是该分支和 Commit |
| `main` | 本地 `main` 指向的位置 |
| 最后文字 | Commit Message |

#### 2.3.9 查看单个 Commit：`git show`

```bash
git show 3ab92ef
```

作用：查看指定 Commit 的作者、时间、说明和代码差异。

```text
commit 3ab92ef...
Author: 张三 <zhangsan@example.com>
Date:   Thu Jul 16 10:00:00 2026 +0800

    feat(example): 增加示例功能

diff --git ...
```

`commit` 后是完整 ID，`Author` 是作者，`Date` 是提交时间，`diff` 后是具体修改。

#### 2.3.10 查看和创建分支：`git branch`

```bash
git branch
git branch -vv
git branch --show-current
```

| 命令 | 作用 |
|---|---|
| `git branch` | 查看本地分支 |
| `git branch -vv` | 查看分支最新 Commit 和跟踪信息 |
| `git branch --show-current` | 只显示当前分支 |

本地场景下的输出：

```text
* feature/example 3ab92ef feat(example): 增加示例功能
  main            b0f5f1d chore: 初始化项目
```

`*` 表示当前分支，其后依次是分支名、Commit ID 和最新 Commit Message。

#### 2.3.11 切换和创建分支：`git switch`

```bash
git switch main
git switch -c feature/example
```

| 命令 | 作用 |
|---|---|
| `git switch main` | 切换到已有 `main` |
| `git switch -c feature/example` | 从当前位置创建并切换到新分支 |

典型输出：

```text
Switched to a new branch 'feature/example'
```

如果本地修改会被覆盖，Git 会拒绝切换。不要强制操作，应先检查 `git status`。

#### 2.3.12 合并本地分支：`git merge`

先切换到接收修改的目标分支，再执行 merge：

```bash
git switch main
git merge feature/example
```

作用：把 `feature/example` 的 Commit 合入当前 `main`。

可能输出：

```text
Updating b0f5f1d..3ab92ef
Fast-forward
```

`Fast-forward` 表示只移动 `main` 指针，没有创建额外 Merge Commit。

如果出现：

```text
CONFLICT (content): Merge conflict in src/example.py
```

表示 Git 无法自动决定结果，需要人工处理冲突。

#### 2.3.13 删除已完成分支：`git branch -d`

```bash
git branch -d feature/example
```

作用：安全删除已经合并的本地分支。

```text
Deleted branch feature/example (was 3ab92ef).
```

括号内是分支删除前指向的 Commit。分支尚未合并时，Git 通常会拒绝删除。

#### 2.3.14 临时保存：`git stash`

```bash
git stash push -u -m "临时保存说明"
git stash list
git stash pop
```

| 命令 | 作用 |
|---|---|
| `git stash push -u` | 临时保存修改和未跟踪文件 |
| `git stash list` | 查看临时保存列表 |
| `git stash pop` | 恢复最近一次临时保存 |

列表输出：

```text
stash@{0}: On main: 临时保存说明
```

`stash@{0}` 是编号，`On main` 是保存时所在分支，最后是说明。

#### 2.3.15 丢弃未提交修改：`git restore`

```bash
git restore src/example.py
```

作用：把文件恢复为 Git 最近记录的状态，丢弃尚未提交的修改。

成功时通常没有输出，但修改可能已经永久丢失，执行前必须检查 `git diff`。

#### 2.3.16 创建本地版本标记：`git tag`

```bash
git tag --list
git tag -a v1.0.0 -m "Release v1.0.0"
```

| 命令 | 作用 |
|---|---|
| `git tag --list` | 查看本地 Tag |
| `git tag -a` | 在当前 Commit 创建带说明的 Tag |

创建成功时通常没有输出，`git tag --list` 会逐行显示 Tag 名称。

### 2.4 本地仓库开发完整流程

下面以开发“历史票查询”功能为例说明完整的本地流程。

#### 2.4.1 初始化项目

```bash
mkdir workticket-demo
cd workticket-demo
git init
```

#### 2.4.2 创建第一个版本

编写文件后：

```bash
git status
git diff
git add README.md src/main.py
git diff --staged
git commit -m "feat: 初始化工作票项目"
```

#### 2.4.3 创建功能分支

```bash
git switch -c feature/history-search
```

开发后：

```bash
git status --short
git diff
git add src/history.py tests/test_history.py
git diff --staged
git commit -m "feat(history): 增加历史票查询"
```

#### 2.4.4 合并回主分支

```bash
git switch main
git merge feature/history-search
git branch -d feature/history-search
```

#### 2.4.5 标记本地版本

```bash
git tag -a v1.0.0 -m "Release v1.0.0"
git log --oneline --decorate -10
```

这就是不依赖远程平台的完整本地版本管理过程。

## 3. Git 远程仓库与协作

### 3.1 为什么需要远程仓库

本地 Git 能管理本地代码，但涉及共享和多人协作时还需要解决：

- 团队共享同一份正式代码。
- 获取其他成员的 Commit。
- 把自己的 Commit 提供给其他成员。
- 审查代码后再进入稳定分支。
- 自动运行测试和构建。
- 管理权限、版本和发布记录。

GitLab 和 GitHub 都以 Git 为基础。两者核心原理相同，主要页面名称略有差异：

| 协作概念 | GitLab | GitHub |
|---|---|---|
| 合并请求 | Merge Request，MR | Pull Request，PR |
| 自动流水线 | GitLab CI/CD | GitHub Actions |
| 需求管理 | Issue | Issue |
| 远程仓库 | Repository | Repository |
| 版本发布 | Release | Release |

本文后续使用“MR/PR”表示同一类合并请求。

### 3.2 本地仓库和远程仓库的关系

#### 3.2.1 `origin`

`origin` 是本地为远程仓库设置的默认名称：

```bash
git remote -v
```

```text
origin  git@gitlab.example.com:team/project.git (fetch)
origin  git@gitlab.example.com:team/project.git (push)
```

| 输出部分 | 含义 |
|---|---|
| `origin` | 远程仓库在本地的名称 |
| 仓库地址 | GitLab/GitHub 项目地址 |
| `(fetch)` | 获取代码使用的地址 |
| `(push)` | 推送代码使用的地址 |

#### 3.2.2 本地分支、远程分支和远程跟踪分支

```text
dev                 本地分支
GitLab 上的 dev      真正的远程分支
origin/dev          本地记录的远程 dev 状态
```

三者不是同一个对象：

- 本地 `dev` 只会在本地 Commit、Merge、Pull 时移动。
- GitLab 上的 `dev` 在有人 Push 或合并 MR/PR 时移动。
- `origin/dev` 在本地执行 Fetch/Pull 时更新。

```mermaid
flowchart LR
    LD["本地 dev"] -->|git push| RD["GitLab/GitHub dev"]
    RD -->|git fetch| OD["本地 origin/dev"]
    OD -->|merge / pull| LD
```

#### 3.2.3 ahead、behind 和 diverged

```text
ahead 2     本地比远程多 2 个 Commit，需要 Push
behind 3    本地比远程少 3 个 Commit，需要同步
diverged    本地和远程各自有新 Commit，需要合并处理
```

查看关系：

```bash
git status
git branch -vv
```

### 3.3 远程相关的主要 Git 命令

#### 3.3.1 复制远程仓库：`git clone`

```bash
git clone git@gitlab.example.com:team/project.git
```

作用：下载远程仓库的文件、Commit、分支和 Tag，并自动配置 `origin`。

典型输出：

```text
Cloning into 'project'...
remote: Enumerating objects: 1250, done.
Receiving objects: 100% (1250/1250), done.
Resolving deltas: 100% (620/620), done.
```

| 输出部分 | 含义 |
|---|---|
| `Cloning into` | 正在创建本地项目目录 |
| `Enumerating objects` | 远程正在统计 Git 对象 |
| `Receiving objects` | 本地正在下载对象 |
| `Resolving deltas` | 本地正在还原对象差异 |

#### 3.3.2 获取远程变化：`git fetch`

```bash
git fetch origin
git fetch --prune origin
```

作用：更新 `origin/dev`、`origin/main` 等远程跟踪记录，不直接修改当前工作区和本地分支。

```text
From gitlab.example.com:team/project
   357ea8b..b0f5f1d  dev        -> origin/dev
 * [new branch]      feature/a  -> origin/feature/a
 - [deleted]         (none)     -> origin/feature/old
```

| 输出部分 | 含义 |
|---|---|
| `From ...` | 变化来自哪个远程仓库 |
| `old..new` | 远程分支从旧 Commit 移到新 Commit |
| `dev -> origin/dev` | 本地远程跟踪记录已更新 |
| `[new branch]` | 远程新增分支 |
| `[deleted]` | `--prune` 清理已删除分支记录 |

#### 3.3.3 更新本地分支：`git pull --ff-only`

```bash
git switch dev
git pull --ff-only origin dev
```

作用：先获取远程 `dev`，再安全快进本地 `dev`。

已经最新：

```text
Already up to date.
```

成功更新：

```text
Updating 357ea8b..b0f5f1d
Fast-forward
 src/example.py | 10 ++++++++--
```

`Fast-forward` 表示只移动本地分支指针，没有产生额外 Merge Commit。

`Not possible to fast-forward` 表示本地存在独立 Commit，命令为保护历史而停止。

#### 3.3.4 推送分支：`git push`

首次推送：

```bash
git push -u origin feature/example
```

后续推送：

```bash
git push
```

作用：把本地新增 Commit 上传到远程对应分支。

```text
To gitlab.example.com:team/project.git
 * [new branch]      feature/example -> feature/example
branch 'feature/example' set up to track 'origin/feature/example'.
```

| 输出部分 | 含义 |
|---|---|
| `To ...` | Push 的目标远程仓库 |
| `[new branch]` | 远程创建了新分支 |
| `local -> remote` | 本地分支推送到哪个远程分支 |
| `set up to track` | 已建立本地和远程跟踪关系 |

没有新 Commit：

```text
Everything up-to-date
```

被拒绝：

```text
! [rejected] feature/example -> feature/example (non-fast-forward)
```

表示远程存在本地没有的 Commit，应先 Fetch 和合并，不能强推覆盖。

#### 3.3.5 删除远程分支

```bash
git push origin --delete feature/example
```

作用：删除远程任务分支。只能在确认 MR/PR 已合并后执行。

```text
To gitlab.example.com:team/project.git
 - [deleted]         feature/example
```

`[deleted]` 表示远程分支指针已删除，已经合入目标分支的 Commit 不会丢失。

#### 3.3.6 推送 Tag

```bash
git push origin v1.0.0
```

作用：把本地 Tag 上传到远程平台。

```text
To gitlab.example.com:team/project.git
 * [new tag]         v1.0.0 -> v1.0.0
```

`[new tag]` 表示远程新增正式版本标记。

### 3.4 GitLab / GitHub 的核心功能

#### 3.4.1 远程 Repository

远程仓库集中保存：

- Commit 历史。
- 远程分支。
- Tag。
- 项目文件。
- 文件和版本差异。

#### 3.4.2 MR/PR

MR/PR 用于请求把任务分支合入目标分支：

```text
feature/example → dev
```

核心作用：

- 展示代码差异。
- 进行代码评审。
- 触发 CI。
- 记录讨论和审批。
- 检查冲突。
- 执行合并。

#### 3.4.3 Code Review

评审人员检查：

- 功能是否正确。
- 是否影响已有功能。
- 边界和异常是否处理。
- 是否存在安全风险。
- 是否有必要测试。
- 是否包含不应提交的配置和数据。

#### 3.4.4 CI/CD

GitLab 使用 `.gitlab-ci.yml`，GitHub 使用 GitHub Actions Workflow。二者都可以自动执行：

```text
代码检查
→ 单元测试
→ 前端构建
→ Docker 构建
→ 部署
```

关键 Job 失败时不应合并 MR/PR。

#### 3.4.5 Issue

Issue 用于记录需求、缺陷、技术债和任务：

```text
Issue
→ feature/fix 分支
→ Commit
→ MR/PR
→ 合并后关闭 Issue
```

#### 3.4.6 权限与受保护分支

`main`、`dev` 应设置为受保护分支：

- 禁止直接 Push。
- 禁止 Force Push。
- 只能通过 MR/PR 合并。
- CI 成功后才能合并。
- 要求指定人员审批。

#### 3.4.7 Tag、Release 和制品

| 功能 | 作用 |
|---|---|
| Tag | 固定正式版本对应 Commit |
| Release | 保存版本说明和发布日期 |
| Artifacts | 保存 CI 产生的测试报告或构建文件 |
| Container Registry | 保存带版本号的 Docker 镜像 |

### 3.5 基于远程仓库的协同开发标准流程

```mermaid
flowchart TB
    R["GitLab/GitHub 上的 dev"] -->|clone 或 pull| LD["本地 dev"]
    LD -->|switch -c| F["本地 feature 分支"]
    F --> C["开发、add、commit"]
    C -->|push| RF["远程 feature 分支"]
    RF --> MR["创建 MR/PR"]
    MR --> Q["CI + 代码评审"]
    Q -->|失败或需修改| C
    Q -->|通过| M["合并远程 dev"]
    M --> D["删除远程 feature 分支"]
    D -->|pull| LD
```

#### 3.5.1 第一次获取项目

```bash
git clone git@gitlab.example.com:team/project.git
cd project
git remote -v
git branch -vv
```

#### 3.5.2 从最新 `dev` 创建任务分支

```bash
git fetch origin
git switch dev
git pull --ff-only origin dev
git switch -c feature/history-search
git push -u origin feature/history-search
```

#### 3.5.3 开发并创建本地 Commit

```bash
git status
git diff
git add src/history.py tests/test_history.py
git diff --staged
git commit -m "feat(history): 增加历史票查询"
```

#### 3.5.4 Push 任务分支

```bash
git push
```

Push 只更新对应的远程任务分支，不会自动进入 `dev`。

#### 3.5.5 合并前同步远程最新代码

```bash
git fetch origin
git switch feature/history-search
git merge origin/dev
```

解决冲突并测试后：

```bash
git push
```

#### 3.5.6 创建 MR/PR

```text
Source：feature/history-search
Target：dev
```

描述应包含修改背景、修改内容、测试结果、数据库变化、配置变化和风险。

#### 3.5.7 根据评审意见修改

继续在原任务分支修改：

```bash
git add <修改文件>
git commit -m "fix(history): 根据评审补充参数校验"
git push
```

原 MR/PR 自动更新，不需要重新创建。

#### 3.5.8 合并后清理

GitLab/GitHub 合并时删除远程源分支。本地执行：

```bash
git switch dev
git pull --ff-only origin dev
git branch -d feature/history-search
git fetch --prune origin
```

下一项任务重新从最新 `dev` 创建分支。

### 3.6 多人开发同一个需求

大需求使用公共需求分支和任务子分支：

```mermaid
flowchart TB
    D["dev"] --> P["feature/history-ticket<br/>公共需求分支"]
    P --> A["feature/history-ticket-api<br/>开发者 A"]
    P --> B["feature/history-ticket-ui<br/>开发者 B"]
    P --> C["feature/history-ticket-model<br/>开发者 C"]
    A -->|MR/PR| P
    B -->|MR/PR| P
    C -->|MR/PR| P
    P -->|总 MR/PR| D
```

协作规则：

1. 需求负责人维护公共需求分支。
2. 每个开发任务从公共需求分支创建独立子分支。
3. 任务子分支通过 MR/PR 合入公共需求分支。
4. 公共需求分支完成集成测试后合入 `dev`。
5. 公共分支有新合并后，其他成员及时 Fetch 和 Merge。
6. 不建议多人长期直接 Push 同一个分支。

创建任务子分支：

```bash
git fetch origin
git switch -c feature/history-ticket-api origin/feature/history-ticket
git push -u origin feature/history-ticket-api
```

同步公共需求分支：

```bash
git fetch origin
git merge origin/feature/history-ticket
git push
```

### 3.7 远程协作中的冲突处理

如果两个人修改同一文件的同一区域，Git 可能输出：

```text
CONFLICT (content): Merge conflict in src/example.py
Automatic merge failed; fix conflicts and then commit the result.
```

冲突文件：

```text
<<<<<<< HEAD
当前任务分支内容
=======
远程目标分支内容
>>>>>>> origin/dev
```

处理步骤：

1. 使用 `git status` 查看冲突文件。
2. 与相关开发人员确认双方业务意图。
3. 手工整理正确结果。
4. 删除冲突标记。
5. 运行测试。
6. `git add` 冲突文件。
7. `git commit` 完成合并。
8. `git push` 更新远程任务分支。

如果合并方向错误且尚未提交：

```bash
git merge --abort
```

## 4. 本地仓库与远程仓库命令对照

### 4.1 只在本地使用

```bash
git init
git config
git status
git diff
git add
git restore --staged
git commit
git log
git show
git branch
git switch
git merge
git stash
git restore
git tag
```

### 4.2 涉及远程仓库

```bash
git clone
git remote -v
git fetch
git pull
git push
git push origin --delete <branch>
git push origin <tag>
```

### 4.3 最核心的区别

```text
本地 Git：管理自己的文件、Commit、分支和历史
远程 Git：在本地 Git 基础上增加共享、同步和多人协作
GitLab/GitHub：在远程 Git 基础上增加 MR/PR、评审、CI、权限和发布
```

## 5. 文件纳管与安全边界

### 5.1 哪些文件应该由 Git 管理

#### 5.1.1 判断文件是否应该提交的原则

判断一个文件是否应该进入 Git，可以依次询问：

1. 它是项目的源文件，还是运行后产生的结果？
2. 其他成员是否需要它才能开发、测试、构建或部署？
3. 它是否可以通过源码和命令稳定地重新生成？
4. 它是否包含密码、Token、个人信息或生产数据？
5. 它是否体积过大、变化频繁或不适合文本审查？

推荐判断流程：

```mermaid
flowchart TB
    A["发现一个项目文件"] --> B{"包含密钥、密码或敏感数据？"}
    B -->|是| X["禁止提交<br/>使用安全变量或受控存储"]
    B -->|否| C{"是开发、测试、构建或部署的必要输入？"}
    C -->|是| D{"体积和变化频率适合 Git？"}
    D -->|是| Y["加入 Git 管理"]
    D -->|否| Z["使用 LFS、制品库、对象存储或 Registry"]
    C -->|否| E{"可以通过命令重新生成？"}
    E -->|是| N["不提交<br/>加入 .gitignore"]
    E -->|否| F["由项目负责人判断<br/>并记录原因"]
```

#### 5.1.2 通常应该加入 Git 管理的文件

| 类型 | 常见示例 | 为什么需要提交 |
|---|---|---|
| 业务源码 | `src/**/*.py`、`frontend/src/**/*.tsx` | 项目的核心实现 |
| 自动测试 | `tests/**/*.py`、`*.test.tsx` | 保证功能可验证和持续回归 |
| 依赖声明 | `pyproject.toml`、`package.json` | 定义项目依赖和工具配置 |
| 依赖锁文件 | `package-lock.json`、`poetry.lock`、`uv.lock` | 固定实际依赖版本，提高构建可复现性 |
| 构建配置 | `Dockerfile`、`docker-compose.yml`、`vite.config.ts` | 定义如何构建和运行项目 |
| CI/CD 配置 | `.gitlab-ci.yml`、`.github/workflows/*.yml` | 定义自动检查、构建和部署流程 |
| 数据库迁移 | `alembic/versions/*.py` | 记录数据库结构演进过程 |
| 配置模板 | `.env.example`、`config.example.yml` | 告诉使用者需要哪些配置项 |
| 团队脚本 | `scripts/deploy.sh`、数据迁移脚本 | 保证团队执行统一、可审查的操作 |
| 项目文档 | `README.md`、`docs/*.md` | 保存架构、开发、部署和使用说明 |
| Git 规则 | `.gitignore`、`.gitattributes` | 统一忽略规则、行尾和文件处理方式 |
| 项目元数据 | `LICENSE`、`CHANGELOG.md` | 记录许可证和版本变化 |
| 必要的小型静态资源 | 图标、模板、小型测试样例 | 项目运行或测试所需输入 |

测试代码通常应该提交。不要因为测试文件较多就直接忽略整个 `tests/` 目录，否则其他成员和 CI 无法执行同一套测试。

数据库 migration 也必须提交。数据库结构是代码版本的一部分，不能只在某台数据库上手工修改而不留下历史。

#### 5.1.3 通常不应该加入 Git 管理的文件

| 类型 | 常见示例 | 不提交的原因 |
|---|---|---|
| 密钥和生产配置 | `.env`、API Key、Token、私钥 | 泄露后会造成安全风险 |
| 本地虚拟环境 | `.venv/`、`venv/`、`node_modules/` | 体积大，可通过依赖声明重建 |
| 编译和构建结果 | `dist/`、`build/`、前端打包目录 | 可通过构建命令重新生成 |
| 语言缓存 | `__pycache__/`、`*.pyc` | 运行时自动产生 |
| 工具缓存 | `.pytest_cache/`、`.mypy_cache/`、`.ruff_cache/` | 本地工具自动产生 |
| 覆盖率结果 | `.coverage`、`htmlcov/`、`coverage.xml` | 测试执行结果，不是源输入 |
| 日志 | `*.log`、`logs/` | 变化频繁，可能包含敏感信息 |
| 本地数据库 | `*.db`、`*.sqlite3` | 可能包含运行数据且不适合合并 |
| 数据库备份 | `*.sql`、`*.dump`、备份压缩包 | 体积大，通常包含敏感数据 |
| 操作系统文件 | `.DS_Store`、`Thumbs.db` | 与项目无关 |
| IDE 私有配置 | `.idea/`、`.vscode/` 中的个人设置 | 容易包含个人路径并产生无关变化 |
| 临时文件 | `tmp/`、`*.swp`、`*~` | 编辑器或程序临时产生 |
| 导出结果 | `exports/`、批处理输出、报表结果 | 通常可重新生成且变化频繁 |
| 发布制品 | Docker 镜像包、部署 `tar.gz` | 应放 Registry、Release 或制品库 |
| 下载的第三方依赖 | 手工下载的包和缓存 | 应通过依赖管理工具获取 |

不提交并不等于不保存。生产配置、数据库备份和发布制品应该进入权限受控、具备备份和生命周期管理的专用存储。

#### 5.1.4 需要根据项目情况判断的文件

| 文件类型 | 建议判断方式 |
|---|---|
| IDE 配置 | 团队统一的格式化、调试配置可提交；个人窗口布局和绝对路径不提交 |
| 生成代码 | 如果构建时能稳定生成，优先不提交；如果下游工具必须直接使用，应明确生成和更新规则 |
| 测试快照 | 作为测试预期结果且经过审查时可提交；临时运行输出不提交 |
| 示例数据 | 小型、脱敏、稳定并用于测试时可提交；真实业务数据不提交 |
| SQLite 文件 | 作为经过审查的只读知识库资源时可提交；本地运行数据库不提交 |
| 图片、Excel、PDF | 项目必要且体积可控时可提交；大文件或频繁变化文件使用 Git LFS/对象存储 |
| 本地 Compose 覆盖 | 团队统一文件可提交；个人端口、挂载路径写入 `docker-compose.override.yml` 并忽略 |
| 一次性脚本 | 对迁移、审计或问题复现有长期价值时提交；纯临时调试脚本不提交 |

“二进制文件能否提交”不能只看扩展名，而要同时考虑必要性、体积、变化频率、敏感性和是否可审查。

#### 5.1.5 `.gitignore` 的作用

`.gitignore` 用来声明哪些未跟踪文件不应进入 Git：

```gitignore
# 环境和密钥
.env
.env.*
!.env.example

# Python
__pycache__/
*.py[cod]
.venv/
.pytest_cache/

# Node.js
node_modules/
frontend/dist/

# 日志和临时文件
*.log
tmp/

# 操作系统和 IDE
.DS_Store
*.swp
```

常见规则含义：

| 规则 | 含义 |
|---|---|
| `.env` | 忽略任意目录匹配到的 `.env` 文件 |
| `*.log` | 忽略所有 `.log` 文件 |
| `tmp/` | 忽略名为 `tmp` 的目录及内容 |
| `/dist/` | 只忽略仓库根目录的 `dist` |
| `**/dist/` | 忽略任意层级的 `dist` 目录 |
| `!.env.example` | 即使前面忽略 `.env.*`，仍允许提交 `.env.example` |
| `# 注释` | 解释规则原因，不参与匹配 |

`.gitignore` 本身应该提交，使所有成员和 CI 使用相同规则。

#### 5.1.6 检查文件为什么被忽略

```bash
git check-ignore -v path/to/file
```

作用：显示是哪一个忽略文件、哪一条规则匹配了目标文件。

典型输出：

```text
.gitignore:18:*.log  logs/app.log
```

| 输出部分 | 含义 |
|---|---|
| `.gitignore` | 生效的忽略规则文件 |
| `18` | 规则所在行号 |
| `*.log` | 匹配到的规则 |
| `logs/app.log` | 被检查的文件 |

命令没有输出通常表示文件没有被忽略。

#### 5.1.7 `.gitignore` 不能影响已经跟踪的文件

如果文件已经提交，后来再加入 `.gitignore`，Git 仍会继续跟踪它。

需要保留本地文件但停止跟踪时，经确认后执行：

```bash
git rm --cached path/to/file
git commit -m "chore(git): 停止跟踪本地生成文件"
```

`git rm --cached` 只从 Git 暂存索引移除文件，通常保留工作区文件；仍应先备份并执行 `git status` 确认影响。

如果文件包含已经泄露的密钥，仅停止跟踪并不能清除历史，必须立即轮换密钥，并由有经验人员评估历史清理方案。

#### 5.1.8 配置文件最佳实践

真实配置与配置模板分离：

```text
.env             本地或生产真实配置，不提交
.env.example     配置键和安全示例值，提交
```

`.env.example` 示例：

```dotenv
DATABASE_URL=mysql+asyncmy://user:password@localhost:3306/workticket
LLM_BASE_URL=https://example.invalid/v1
LLM_API_KEY=replace-me
LLM_MODEL=model-name
```

模板中只能使用占位值，不能复制真实密码或 Token。新增配置项时，代码、配置模板和文档应在同一个 MR/PR 中更新。

#### 5.1.9 依赖文件最佳实践

- 应用项目通常提交依赖锁文件，例如 `package-lock.json`、`uv.lock`。
- 不提交 `node_modules/`、`.venv/` 或下载缓存。
- 依赖声明和锁文件发生变化时，应在同一个 Commit 或 MR/PR 中说明原因。
- 不要手工修改自动生成的锁文件；使用对应包管理命令更新。

#### 5.1.10 大文件和发布制品最佳实践

Git 不适合保存体积大、频繁变化的二进制文件。建议：

| 内容 | 推荐位置 |
|---|---|
| Docker 镜像 | Container Registry |
| 正式发布压缩包 | GitLab/GitHub Release 或制品库 |
| CI 构建输出 | Pipeline Artifacts |
| 大型模型和数据集 | 模型仓库、对象存储或数据平台 |
| 必须版本化的大型二进制文件 | 经审批后使用 Git LFS |
| 数据库备份 | 加密的备份系统 |

不要仅因为文件小于 GitLab 上传限制，就判断它适合进入 Git 历史。Git 中的历史文件通常会长期保留，即使后来删除也会继续占用仓库体积。

#### 5.1.11 不要使用过于宽泛的忽略规则

以下规则风险较高：

```gitignore
tests/
docs/
scripts/
*.json
*.xlsx
```

它们可能误伤正式测试、文档、运维脚本和必要数据资源。更好的方式是忽略明确的临时目录或文件：

```gitignore
scripts/output/
docs/generated/
tests/tmp/
*.local.json
```

新增宽泛规则前，应检查它会匹配哪些现有和未来文件，并在规则旁写明原因。

#### 5.1.12 提交前检查清单

执行：

```bash
git status --short
git diff
git diff --staged
```

确认：

- [ ] 只包含当前任务相关文件。
- [ ] 没有 `.env`、Token、密码和私钥。
- [ ] 没有日志、缓存、临时文件和本地数据库。
- [ ] 没有无意生成的大型二进制文件。
- [ ] 必要测试、migration、配置模板和文档已经包含。
- [ ] 新增忽略规则不会误伤正式源码、测试和文档。
- [ ] Commit Message 能说明修改目的。

### 5.2 Git 操作安全边界

禁止直接在共享 `main`、`dev` 上开发。

初学人员不得自行执行：

```bash
git push --force
git reset --hard
git clean -fd
git rebase dev
git branch -D <branch>
```

| 命令 | 风险 |
|---|---|
| `git push --force` | 可能覆盖远程历史和他人 Commit |
| `git reset --hard` | 可能永久删除未提交修改 |
| `git clean -fd` | 删除未跟踪文件和目录 |
| `git rebase dev` | 重写当前分支 Commit 历史 |
| `git branch -D` | 不检查是否合并就强制删除分支 |

必须遵守：

1. 开始操作先执行 `git status`。
2. 提交前查看 `git diff` 和 `git diff --staged`。
3. 一项任务使用一个独立任务分支。
4. 只 `git add` 本任务相关文件。
5. 密码、Token、私钥和生产配置不进入 Git。
6. 共享代码通过 MR/PR、CI 和评审合入。
7. 合并前同步目标分支。
8. 冲突处理必须确认双方业务意图。
9. MR/PR 合并后及时删除任务分支。
10. 不理解命令影响时停止操作并寻求帮助。
