# Git-Operation-Tutorial

一套按学习顺序组织的 Git 入门与进阶教程，目标不是只教你记命令，而是帮助你建立完整的版本管理、协作开发和并行开发工作流。

## 学习顺序

1. [Git 入门指南](docs/1-Git入门指南.md)
   写作背景与整体目标。
2. [为什么需要使用 Git](docs/2-为什么需要使用Git.md)
   先理解 Git 在解决什么问题，而不是先背命令。
3. [Git —— 从第一性原理出发](docs/3-从第一性原理出发.md)
   建立工作区、暂存区、本地仓库、分支和远端的底层心智模型。
4. [单人练习](docs/4-Git初级练习-本地.md)
   用最小练习掌握本地提交、分支、冲突、rebase、回退与找回。
5. [远端与同步基础](docs/5-Git远端与同步基础.md)
   学会 `clone`、`fetch`、`pull`、`push`、跟踪分支和同步状态判断。
6. [协作开发主流程](docs/6-Git协作开发主流程.md)
   建立多人协作中的功能分支、同步主线、冲突处理和 `merge` / `rebase` 判断。
7. [Git fork、upstream 与贡献工作流](docs/7-Git-fork、upstream与贡献工作流.md)
   理解 fork 型协作里的 `origin`、`upstream`、同步上游和发起贡献。
8. [Git worktree 与并行开发](docs/8-Git-worktree与并行开发.md)
   处理主线、实验线、修复线以及“一人协调多个 AI 并行开发”的多上下文场景。
9. [Git 高频进阶能力](docs/9-Git高频进阶能力.md)
   补齐 `stash`、`cherry-pick`、`tag`、`restore` / `reset` / `revert` 等常用能力。

## 推荐路径

- 如果你是第一次系统学习 Git，按 1 -> 9 顺序读即可。
- 如果你已经做完第 4 篇本地练习，下一步优先看第 5、6、7 篇，再进入第 8 篇的 `worktree`。
- 如果你会参与开源项目或跨仓库协作，第 7 篇会先帮你理清 `fork` 和 `upstream`。
- 如果你已经在用 AI 协助开发，第 8 篇会直接帮助你把并行开发上下文管理清楚。
