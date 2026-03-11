# Git worktree 与并行开发
> 目标
> - 30 分钟内理解并掌握 `worktree` 的 5 个关键点：
>   - 为什么并行开发不该靠复制文件夹
>   - `worktree` 到底解决什么问题
>   - 会创建、查看、删除 worktree
>   - 理解 worktree 和分支的关系
>   - 知道如何把它用于“一个人协调多个 AI 同时开工”

---

## 0. 为什么这一篇值得单独学

如果你只做单线开发，那么“切分支再切回来”勉强也能工作。

但现实开发里很快就会遇到这些情况：

- 主线要保持稳定
- 一个功能正在开发
- 一个紧急修复必须插进来
- 一个 AI 给了方案 A
- 另一个 AI 给了方案 B

这时如果你还在一个工作目录里来回切来切去，问题会马上出现：

- 当前目录到底是哪条线
- 未提交的改动挡住你切分支
- 为了临时比较方案，只能复制整份文件夹
- 复制出来的目录和仓库历史越来越脱节

`worktree` 的价值就在这里。

它解决的不是“多一个技巧”，而是：

**同一个仓库，如何同时保持多个互不打扰的开发上下文。**

---

## 1. 先把核心概念讲清楚

你可以把 `git worktree` 理解成：

- 同一个 Git 仓库
- 挂出多个工作目录
- 每个工作目录各自站在不同分支上工作

最重要的不是命令形式，而是这个结果：

- 各个目录互不干扰
- 历史对象仍然在同一个仓库里
- 你可以真正并行地处理多条开发线

所以 `worktree` 和“复制一份项目目录”最大的区别是：

- 复制目录会复制一堆混乱状态
- `worktree` 会保持这些目录仍然共享同一份 Git 历史

---

## 2. 用一个最小例子体验 worktree

先新建一个练习仓库。

在 `PowerShell` 中输入：

```powershell
cd $HOME # 或其他你认为合适的路径
mkdir git-worktree-lab -Force
cd git-worktree-lab
mkdir demo -Force
cd demo
"base" | Set-Content app.txt
git init -b main
git add app.txt
git commit -m "feat: init demo"
```

先看当前只有一个工作目录：

```bash
git branch
git worktree list
```

现在新增两个工作上下文：

```bash
git worktree add ..\demo-api -b feature/api main
git worktree add ..\demo-hotfix -b hotfix/login main
git worktree list
```

这时你会看到：

- `demo` 目录还在 `main`
- `demo-api` 目录站在 `feature/api`
- `demo-hotfix` 目录站在 `hotfix/login`

你已经开始真正并行开发了。

---

## 3. worktree 和分支之间到底是什么关系

这里要记住几个关键事实：

- 每个 worktree 都有自己的工作区
- 每个 worktree 都有自己的 `HEAD`
- 这些 worktree 共享同一个仓库历史

这意味着：

- 你在 `demo-api` 提交的内容，`demo-hotfix` 也能在同一仓库历史里看到
- 但它们的当前文件状态彼此独立

还有一个非常重要的约束：

**同一条分支不能同时被多个 worktree 检出。**

原因很简单：如果两处目录同时都声称“我就是这条分支当前工作区”，Git 就没法保证状态一致了。

所以一个很自然的工作方式就是：

- 稳定主线一个 worktree
- 每个功能或实验各自一个分支、各自一个 worktree

---

## 4. 为什么它特别适合并行开发和多 AI 协作

这篇真正的重点在这里。

在 AI 参与开发之后，单个开发者越来越像是在协调一个小型并行开发现场。

你可能会同时想做这些事：

- 保持 `main` 作为稳定主线
- 让 AI-A 尝试一种实现方案
- 让 AI-B 尝试另一种实现方案
- 自己单独开一条修复线处理紧急问题

如果这些事情都挤在同一个工作目录里，你很容易遇到：

- 切分支前必须先清空当前改动
- 不同 AI 的结果混在一起
- 某个实验做到一半，不知道是否值得保留
- 明明已经比较过两个方案，最后却只记得结论

而 worktree 很适合这样编排：

```bash
git worktree add ..\demo-ai-a -b experiment/ai-a main
git worktree add ..\demo-ai-b -b experiment/ai-b main
git worktree add ..\demo-fix -b hotfix/urgent main
git worktree list
```

对应的思路是：

- `main` worktree 维持稳定主线
- `experiment/ai-a` 给 AI 的方案 A
- `experiment/ai-b` 给 AI 的方案 B
- `hotfix/urgent` 专门处理紧急修复

这样做的价值不是“看起来专业”，而是你真正获得了：

- 可隔离
- 可追踪
- 可比较
- 可回退

这也是为什么在并行开发场景下，`worktree` 经常比“来回切分支”更实用。

---

## 5. 并行开发之后，怎么比较和收敛

有了多个 worktree，并不意味着你要长期保留所有分支。

真正重要的是：

**你要能比较不同尝试，然后把有价值的结果收敛回来。**

常见做法包括：

```bash
git log --oneline --graph --decorate --all
git diff main..experiment/ai-a
git diff experiment/ai-a..experiment/ai-b
```

如果你决定采纳某条线，可以再根据情况：

- `merge` 回主线
- `cherry-pick` 某几个关键提交
- 或者直接放弃某条实验分支

所以 `worktree` 不是替代分支、替代提交、替代合并。

它是在帮你把这些操作放进更清晰的上下文里。

---

## 6. 用完之后怎么清理

如果某个上下文已经结束，就把它清掉。

例如：

```bash
git worktree remove ..\demo-ai-a
git worktree remove ..\demo-hotfix
git worktree prune
git worktree list
```

这里的理解也要到位：

- `remove` 是删掉这个工作目录
- `prune` 是清理失效记录

如果对应分支也不需要了，再单独删除分支。

不要把 worktree 当成永久堆积区。

---

## 7. 最常见的几个误区

- 觉得 `worktree` 只是“多开几个文件夹”
- 在同一个 worktree 里同时放多个实验改动
- 忘了哪个目录对应哪条分支
- 实验已经结束，却长期不清理 worktree
- 把 `worktree` 当成替代提交历史的手段

正确理解应该是：

- `branch` 负责表达开发线
- `commit` 负责表达阶段状态
- `worktree` 负责给这些开发线提供真正隔离的工作上下文

---

## 8. 这一篇真正的过关标准

如果下面这些问题你都能答清楚，就说明这一篇已经过关：

- 为什么并行开发不应该靠复制工程目录
- `worktree` 和普通切分支最大的差异是什么
- 为什么同一条分支不能同时在两个 worktree 里检出
- 为什么它很适合“一个人协调多个 AI 同时开工”的场景
- worktree 用完之后为什么要及时清理

下一篇再补上一组真正高频的进阶能力，你的 Git 工作流就会完整很多。
