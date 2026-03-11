# fork、upstream 与贡献工作流
> 目标
> - 30-40 分钟内掌握 fork 型协作里最关键的 5 件事：
>   - 理解 `origin`、`upstream`、fork 仓库分别是什么
>   - 会给本地仓库配置第二个远端
>   - 会同步上游主线到自己的本地和 fork
>   - 会从 fork 的功能分支提交贡献
>   - 能分清“同仓库协作”和“跨仓库贡献”是两种不同工作流

---

## 0. 为什么这一篇要单独拿出来讲

前一篇讲的是最常见的团队协作主流程：

- 大家都围绕同一个仓库工作
- 都能直接看到同一条主线
- 功能分支最终合回同一个仓库

但现实里还有另一种非常常见的协作模型：

- 你不能直接往主仓库推送
- 你先复制一份自己的仓库副本
- 在自己的副本里开发和推送
- 再把改动提交给主仓库审核

这就是 `fork` / `upstream` 工作流。

它常见于：

- 开源项目贡献
- 跨团队协作
- 你只有只读权限，但仍要提交修改建议

所以这一篇真正要解决的问题是：

**当你不是主仓库的直接维护者时，应该怎样围绕自己的 fork 和上游仓库稳定协作。**

---

## 1. 先把三个角色讲清楚

在 fork 型工作流里，最容易混的是这三个东西：

- 原始主仓库
- 你自己的 fork 仓库
- 你本地克隆下来的工作副本

先建立一个最小心智模型：

```text
原始主仓库 --------------------> upstream
你的 fork 仓库 -----------------> origin
你本地的工作目录 ---------------> 同时连接 origin 和 upstream
```

通常可以这样理解：

- `upstream`：原始主仓库，也就是你真正想提交贡献的地方
- `origin`：你自己的 fork 仓库，也就是你通常有推送权限的地方
- 本地仓库：你实际写代码、建分支、提交和同步的地方

这里最关键的一点是：

**`origin` 不一定总是“主仓库”，它只是你当前克隆来源的默认远端名。**

如果你克隆的是自己的 fork，那么：

- `origin` 指向你的 fork
- `upstream` 才指向原始主仓库

这就是为什么在 fork 工作流里，理解远端角色比单纯记命令更重要。

---

## 2. 一个典型的 fork 工作流长什么样

把全过程压缩一下，通常是这样：

1. 先在代码托管平台上 fork 一份自己的仓库副本
2. 把自己的 fork 克隆到本地
3. 在本地给原始主仓库添加一个 `upstream` 远端
4. 从 `upstream/main` 同步最新主线
5. 基于最新主线切功能分支
6. 把分支推到自己的 `origin`
7. 再从自己的 fork 发起到 `upstream` 的 Pull Request

所以这条工作流本质上不是“多了几个命令”，而是多了一层仓库关系：

- 你的改动先进入自己的 fork
- 再通过评审或 PR 进入上游仓库

---

## 3. 本地应该怎么配置 `origin` 和 `upstream`

假设你已经在平台上 fork 了某个仓库，现在先克隆你自己的 fork：

```bash
git clone <your-fork-url>
cd <repo-name>
git remote -v
```

这时你看到的 `origin` 通常会指向你的 fork。

接下来，把原始主仓库加成第二个远端：

```bash
git remote add upstream <original-repo-url>
git remote -v
```

配置完成后，本地一般会同时连接两个远端：

- `origin`：你的 fork
- `upstream`：原始主仓库

之后你就能明确区分两类操作：

- 向 `origin` 推送：把自己的工作同步到自己的 fork
- 从 `upstream` 同步：跟上原始主仓库的最新进度

这是整个 fork 工作流最关键的准备动作。

---

## 4. 为什么要先同步 `upstream`，再开始开发

很多人 fork 之后最容易犯的错，是直接在旧的 fork 主线上开始干活。

这样会带来两个问题：

- 你的开发基线可能已经落后于原始主仓库
- 后面发 PR 时，改动范围会变脏，甚至带上不属于你的旧差异

所以开始新任务前，更稳妥的做法通常是：

```bash
git fetch upstream
git switch main
git merge --ff-only upstream/main
git push origin main
```

这几步分别在做什么：

- `fetch upstream`：先更新你对上游仓库状态的认知
- `switch main`：回到本地主线
- `merge --ff-only upstream/main`：把本地主线快进到上游最新状态
- `push origin main`：把你自己的 fork 主线也一起同步过去

这样做的好处是：

- 本地 `main` 跟上游主线一致
- 你的 fork `main` 也保持干净同步
- 你后面切出的功能分支会有一个更可信的基线

所以在 fork 工作流里，一个非常重要的习惯就是：

**先同步 `upstream`，再开始新的开发。**

---

## 5. 正确的开发流程：从同步后的主线切分支

和普通协作一样，fork 工作流也不建议你长期直接在 `main` 上开发。

更稳妥的流程通常是：

```bash
git fetch upstream
git switch main
git merge --ff-only upstream/main
git push origin main
git switch -c feature/fix-typo
```

然后在这条功能分支上正常开发：

```bash
git add .
git commit -m "docs: fix typo in guide"
git push -u origin feature/fix-typo
```

这里要特别看清楚：

- 你推送的目标通常是自己的 `origin`
- 不是直接推到 `upstream`

因为在 fork 模型里，你往往没有 `upstream` 的直接写权限，或者根本就不应该直接往那里推送。

所以这条线的本质是：

- 本地分支 -> 推到自己的 fork
- 再由 fork -> 向上游仓库发起贡献

---

## 6. PR 在这个工作流里扮演什么角色

在同仓库协作里，分支合并可能直接发生在同一个远端仓库里。

而在 fork 工作流里，你通常还会多一个步骤：

**从自己的 fork 分支，向上游仓库发起 Pull Request。**

这意味着：

- 你的代码先出现在自己的 fork 上
- 上游维护者再决定是否接纳这份修改

所以 PR 在这里不只是“提个合并请求”，而是：

- 明确告诉上游仓库你改了什么
- 让对方基于可审查的分支来讨论和审核
- 把“个人修改”变成“候选贡献”

这一篇不展开平台页面操作，但你至少要建立这个概念：

- `git push origin feature/...` 不是终点
- 它通常只是为后续发 PR 做准备

---

## 7. 一个完整、可复用的 fork 协作模板

如果你想把流程固定下来，可以先记住下面这套模板。

### 第一次配置仓库

```bash
git clone <your-fork-url>
cd <repo-name>
git remote add upstream <original-repo-url>
git remote -v
```

### 开始新任务前

```bash
git fetch upstream
git switch main
git merge --ff-only upstream/main
git push origin main
git switch -c feature/<task-name>
```

### 开发过程中

```bash
git add .
git commit -m "feat: ..."
git push -u origin feature/<task-name>
```

### 上游主线又前进时

```bash
git fetch upstream
git rebase upstream/main
```

或者，如果这条分支是多人共享的，就改成：

```bash
git fetch upstream
git merge upstream/main
```

### 准备贡献时

- 确认分支历史清楚、语义明确
- 确认分支已经基于较新的上游主线
- 再从自己的 fork 分支向 `upstream` 发起 PR

---

## 8. fork 工作流里最常见的误区

- 把 `origin` 误以为永远是“主仓库”
- fork 之后从不配置 `upstream`
- 很久不同步上游主线，直接在过期基线上开发
- 长期直接在 fork 的 `main` 上改
- 误把功能分支直接推向 `upstream`

这些问题的共同根源是：

没有把“我的 fork”和“原始主仓库”这两个远端角色分开理解。

所以你一定要记住：

- `origin` 解决“我把修改推到哪里”
- `upstream` 解决“我应该跟谁保持同步”

---

## 9. 这一篇的过关标准

如果下面这些问题你都能答清楚，就说明这一篇已经过关了：

- 为什么 fork 工作流里需要两个远端
- 为什么 `origin` 不一定是原始主仓库
- 为什么开始新任务前要先同步 `upstream/main`
- 为什么功能分支通常先推到自己的 fork
- PR 在 fork 工作流里承担什么角色

下一篇就可以进入另一个更贴近现代开发现实的问题了：

**当你已经能处理远端协作和 fork 协作之后，怎样把多个并行开发上下文真正隔离开。**
