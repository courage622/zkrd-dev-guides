# 团队开发规范

本项目用于统一管理团队开发规范相关的文件，为团队成员提供集中、清晰且可持续维护的开发指引。

## 规范文档

| 文档 | 内容说明 |
|---|---|
| [Git 基础与团队协作指南](./GIT_BASICS_AND_COLLABORATION.md) | 介绍 Git 本地仓库基础、远程仓库操作及 GitLab 多人协作知识，适合团队成员学习和查阅。 |
| [GitLab 开发与发布流程规范](./GITLAB_DEVELOPMENT_WORKFLOW.md) | 规定功能开发、分支管理、代码合并、版本发布、生产部署和紧急修复流程。 |
| [GitLab 项目配置与开发管理规范](./GITLAB_PROJECT_MANAGEMENT.md) | 规定项目创建、成员权限、仓库初始化、开发过程管控和版本发布等项目管理要求。 |
| [GitLab Merge Request 代码审查规范](./GITLAB_CODE_REVIEW.md) | 规定 Merge Request 的创建、代码审查、意见处理、复审和合并流程。 |
| [软件版本管理快速指南](./VERSIONING_QUICK_GUIDE.md) | 提供版本文件、版本号更新、CHANGELOG、发布 Tag 和 CI 检查规则的日常速查指引。 |

## 推荐阅读顺序

1. 新成员先阅读 [Git 基础与团队协作指南](./GIT_BASICS_AND_COLLABORATION.md)，了解 Git 和 GitLab 基础。
2. 开始日常开发前阅读 [GitLab 开发与发布流程规范](./GITLAB_DEVELOPMENT_WORKFLOW.md) 和 [GitLab Merge Request 代码审查规范](./GITLAB_CODE_REVIEW.md)。
3. 项目负责人阅读 [GitLab 项目配置与开发管理规范](./GITLAB_PROJECT_MANAGEMENT.md)，完成项目及权限配置。
4. 准备版本发布或日常查阅版本规则时，阅读 [软件版本管理快速指南](./VERSIONING_QUICK_GUIDE.md)。

## 使用方式

团队成员在开展开发工作前，应先查阅并遵循仓库中的相关规范。规范发生变化时，请及时更新对应文件，并通过代码评审确保变更内容准确、清晰且获得团队共识。

## 维护原则

- 规范文件应按主题分类，保持目录结构清晰。
- 内容应简洁明确，并在必要时提供示例。
- 重要变更应记录原因及影响范围。
- 已失效或被替代的规范应及时更新或移除。
