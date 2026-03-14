# Git worktree 与并行开发
> 目标
> - 30 分钟内理解并掌握 `worktree` 的 6 个关键点：
>   - 为什么并行开发不该靠复制文件夹
>   - `worktree` 到底解决什么问题
>   - 会创建、查看、删除 worktree
>   - 理解 worktree 和分支、`HEAD` 的关系
>   - 知道如何把它用于“一个人协调多个 AI 同时开工”
>   - 能把一轮并行开发从创建上下文、比较到收敛跑完整

---

## 0. 这一篇要解决的问题

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

这一篇会按这个顺序展开：

1. 先理解 `worktree` 这项能力到底是什么
2. 再理解最核心的命令 `git worktree add` 是怎么工作的
3. 再把 `worktree`、分支、`HEAD` 和 `git worktree list` 的关系讲清楚
4. 然后用一轮完整实战把并行开发跑一遍
5. 最后再引到 AI 时代下的并行开发价值和清理动作

---

## 1. 先把 `worktree` 当成一种能力理解

先不要急着背命令。

你可以先把 `git worktree` 理解成这样一种能力：

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

这里最容易漏掉的一步是：

**当你 `git init` 或 `git clone` 完进入仓库之后，当前这个仓库目录本身就已经是一个工作上下文了。**

也就是说，在你第一次写 `git worktree add` 之前，并不是“仓库里还没有 `worktree`”，而是：

- 当前目录已经承担着第一个工作目录
- 你只是还没有额外挂出第二个、第三个工作目录

先从最小状态看，会更容易建立这个认知：

```text
git init / git clone 之后：

+--------------------------------------------------+
|         同一个 Git 仓库 / 同一份提交历史           |
|                  main -> A                       |
+--------------------------------------------------+
                        |
                        | 当前就在这个目录里工作
                        v
            +----------------------+
            | 当前目录: demo/       |
            | HEAD -> main         |
            | 这是第一个工作目录     |
            +----------------------+
```

所以后面 `git worktree add` 做的事情，不是“第一次创建 `worktree`”，而是：

- 保留当前目录这个已有工作目录
- 再额外挂出新的工作目录

先用一张静态关系图看，会更直观：

```text
同一个 Git 仓库 / 同一份提交历史

分支:
- main                -> A
- feature/report-a    -> A
- feature/report-b    -> A
- hotfix/login-copy   -> A

挂出的工作目录:
- demo/               HEAD -> main
- report-a/           HEAD -> feature/report-a
- report-b/           HEAD -> feature/report-b
- login-fix/          HEAD -> hotfix/login-copy
```

这张图表达的是：

- 多个 `worktree `共享同一份仓库历史
- 每个 `worktree `都有自己的工作区和自己的 `HEAD`
- 你挂出来的是“多个工作上下文”，不是“多份互不相干的仓库副本”

如果把第 6 章里的“分支”理解成开发线，那么这里的 `worktree` 就是在给这些开发线提供真正隔离的工作空间。

---

## 2. 最核心的命令：`git worktree add`

```bash
git worktree add
```

它真正负责“新增一个独立工作上下文”。

到这一步要先带着一个清晰前提往下看：

- 当前仓库目录已经是一个已有工作目录
- `git worktree add` 做的是在它之外再新增一个工作目录

### 这条命令解决什么问题

当你已经有一个仓库目录，但又想：

- 保留当前目录不动
- 同时拉起另一条功能线
- 让另一条线在另一个目录里独立工作

这时你需要的能力就是：

**从同一个仓库历史上，再挂出一个新的工作目录。**

这正是 `git worktree add` 在做的事。

### 命令骨架

本章最常用的一种写法是：

```bash
git worktree add -b <new-branch> <path> <start-point>
```

这一行里每一段分别在表达什么：

- `add`
  含义：新增一个 `worktree`
- `-b <new-branch>`
  含义：顺手创建一条新分支，并让新 worktree 站在这条分支上
- `<path>`
  含义：这个新工作目录放在哪里
- `<start-point>`
  含义：这条新分支从哪条线或哪个提交开始

在这套教程里，最常见的起手式是：

- 用 `-b` 新建一条分支
- 用 `main` 作为起点
- 用一个新的目录名挂出这个上下文

例如，如果后面要做“方案 A”的实验线，可以写成：

```bash
git worktree add -b feature/report-a ..\demo-report-a main
```

可以直接读成：

- 基于 `main`
- 新建一条 `feature/report-a`
- 在 `..\demo-report-a` 这个目录里把它检出来

这里的 `..\demo-report-a` 是相对路径，意思是：

- 当前仓库目录叫 `demo`
- 新 worktree 会建在它的同级目录 `demo-report-a`

把它拆成“执行前 / 执行后”来看，会更容易理解：

```text
执行前：当前仓库只有一个工作目录

仓库历史:   A
分支:       main -> A

worktree:
- demo/         HEAD -> main

执行后：当前仓库多了一个新增的 worktree

仓库历史:   A
分支:       main -> A
           feature/report-a -> A

worktree:
- demo/         HEAD -> main
- demo-report-a/  HEAD -> feature/report-a
```

这里真正新增的是：

- 一个新的工作目录
- 一条新的分支
- 一个新的“目录 <-> 分支”检出关系

这里没有新增的是：

- 第二份独立的 Git 历史
- 第二个互不相干的仓库

所以如果你在“执行前”图里看到 `demo/` 已经存在，不应该把它理解成“图画错了”，而应该理解成：

- 当前目录本来就是第一个工作目录
- `git worktree add` 正在基于这个已存在的上下文，再挂出一个新的工作目录

---

## 3. 在实战前，先把 `worktree`、分支、`HEAD` 和 `git worktree list` 讲清楚

前面已经理解了 `git worktree add` 的骨架。

在真正进入实战之前，还需要把四个关系先固定下来：

- 一个 `worktree` 对应一个工作目录
- 每个工作目录都有自己的 `HEAD`
- 多个 `worktree` 共享同一个仓库历史
- `git worktree list` 看的是“当前有哪些工作目录，以及每个目录现在站在哪条分支上”

可以先把它理解成这样：

```text
同一个 Git 仓库 / 同一份提交历史

分支:
- main                -> A
- feature/report-a    -> A
- hotfix/login-copy   -> A

worktree:
- demo/               HEAD -> main
- demo-report-a/      HEAD -> feature/report-a
- demo-login-fix/     HEAD -> hotfix/login-copy
```

这张图里最关键的不是目录名，而是两层关系：

- 分支负责表达开发线
- `worktree` 负责给这些开发线提供独立工作目录

`HEAD` 则表示“当前这个目录现在站在哪条线”。所以当你切换到不同目录时，看到的 `HEAD` 其实也是不同的。

这里还有一条必须先记住的约束：

**同一条分支不能同时被两个 worktree 检出。**

原因不是语法限制，而是状态一致性。如果两个目录都同时声称“我现在就是 `main` 的工作区”，Git 就没法判断这条分支到底该以哪一个目录的状态为准。

用“正确 / 错误”看会更直接：

```text
正确：

- demo/               -> main
- demo-report-a/      -> feature/report-a
- demo-login-fix/     -> hotfix/login-copy

错误：

- demo/               -> main
- demo-copy/          -> main
```

`git worktree list` 正是在观察这组关系。

如果当前只有一个工作目录，它的输出可以先被读成：

```text
- demo/               -> main
```

如果后来挂出了多个工作目录，它看的也仍然是同一件事：

```text
- demo/               -> main
- demo-report-a/      -> feature/report-a
- demo-report-b/      -> feature/report-b
- demo-login-fix/     -> hotfix/login-copy
```

所以 `git worktree list` 不是在比较提交，也不是在列出所有分支。它只是把“目录 <-> 当前分支”的对应关系摆到你面前。

---

## 4. 实战模拟：把并行开发从头到尾跑一遍

前面几节已经把概念和关系讲清楚了。

这一节把它们合起来，做一轮更像真实工作的练习：

- `main` 保持稳定
- `feature/report-a` 代表方案 A
- `feature/report-b` 代表方案 B
- `hotfix/login-copy` 代表一条独立紧急修复线

重点不是“多开几个目录”，而是完整体验这条闭环：

- 先把多条线挂出来
- 再分别产生差异
- 再统一比较
- 最后只收敛真正要留下的那一条

### 第一步：先准备统一的练习仓库

先把整轮练习放进一个干净环境里，后面所有动作都围绕同一个仓库继续推进。

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

做到这里时，仓库里还只有一个工作目录，也就是当前的 `demo/`。

### 第二步：从稳定主线挂出三个额外上下文

当前目标是把一个已有工作目录，扩展成四条可以并行推进的线。

这样做的原因很简单：

- `main` 继续作为稳定主线
- 两个功能方案互不干扰
- 紧急修复也不需要堵住功能开发

在主目录 `demo/` 中执行：

```bash
git switch main
git worktree add -b feature/report-a ..\demo-report-a main
git worktree add -b feature/report-b ..\demo-report-b main
git worktree add -b hotfix/login-copy ..\demo-login-fix main
git worktree list
```

这一步之后，当前仓库里的目录和分支关系可以先看成：

```text
仓库历史:
- main                -> A
- feature/report-a    -> A
- feature/report-b    -> A
- hotfix/login-copy   -> A

worktree:
- demo/               HEAD -> main
- demo-report-a/      HEAD -> feature/report-a
- demo-report-b/      HEAD -> feature/report-b
- demo-login-fix/     HEAD -> hotfix/login-copy
```

这一步想让你看懂的是：

- 当前目录 `demo/` 仍然保留
- 新增的是三个额外工作目录
- 每个目录各自站在不同分支上工作

### 第三步：让每条线真的产生差异

这一阶段的目标是：

- 让方案 A 和方案 B 真正做出不同实现
- 让 hotfix 成为一条独立于功能尝试的平行线

先到 `demo-report-a/`：

```powershell
cd ..\demo-report-a
"report view: card" | Set-Content report.txt
git add report.txt
git commit -m "feat: add report view A"
```

再到 `demo-report-b/`：

```powershell
cd ..\demo-report-b
"report view: table" | Set-Content report.txt
"report sort: updated-at" | Add-Content report.txt
git add report.txt
git commit -m "feat: add report view B"
```

最后到 `demo-login-fix/`：

```powershell
cd ..\demo-login-fix
"login copy: please retry" | Set-Content login.txt
git add login.txt
git commit -m "fix: add login retry copy"
```

这时三条线的意义是：

- `feature/report-a` 是一个候选方案
- `feature/report-b` 是另一个候选方案
- `hotfix/login-copy` 是应该独立判断发布时间的修复线

### 第四步：回到主目录，统一观察现在的分支状态

并行开发最怕的不是线太多，而是看不清它们现在分别走到了哪里。

所以这一步不要急着合并，而是先回到稳定主线所在的目录统一观察：

```powershell
cd ..\demo
git worktree list
git log --oneline --graph --decorate --all
```

如果把当前状态抽象成一张提交图，大致会接近这样：

```text
* E  hotfix/login-copy
| * D  feature/report-b
|/
| * C  feature/report-a
|/
* B  main
```

这里真正要看懂的是：

- `B` 是当前这轮练习的共同基线，对应实际执行里的 `feat: init demo`
- `main` 仍然停在基线 `B`
- `feature/report-a`、`feature/report-b`、`hotfix/login-copy` 都是基于 `B` 分出去的
- 现在要比较的是这些分支结果，而不是这些目录名本身

这里的 `B/C/D/E` 只是为了读图方便的抽象记号。你本地看到的会是实际提交哈希，例如 `f3cee98`、`7db5086`、`ce4f5f0`、`7094537`。

### 第五步：先比较，再决定哪条线值得收敛

`worktree` 的价值不只是“同时开工”，更重要的是你终于能把不同尝试摆在同一张桌面上比较。

在主目录 `demo/` 中执行：

```bash
git diff main..feature/report-a
git diff main..feature/report-b
git diff feature/report-a..feature/report-b
```

这里三条命令分别在回答三个不同问题：

- `git diff main..feature/report-a`
  看方案 A 相比主线改了什么
- `git diff main..feature/report-b`
  看方案 B 相比主线改了什么
- `git diff feature/report-a..feature/report-b`
  直接比较两个候选方案之间的差异

但对新手来说，真正关键的不只是“执行哪条命令”，而是**看到输出之后，能不能把它读成一个明确判断**。

这里可以顺手把第 4 章学过的 `git diff` 知识点接过来：

- 第 4 章里，`git diff` 比较的是工作区、暂存区和最新提交
- 这一章里，`git diff A..B` 比较的是两条分支结果
- 输出长相还是同一套 diff 结构，只是比较对象换成了“分支 vs 分支”

所以这一节不再重新完整解释 `diff --git`、`--- / +++`、`@@` 的通用语法，而是重点看：

- 这次比较的两边分别是谁
- 输出里到底说明了什么差异
- 这些差异为什么足以支撑后面的收敛判断

### 先看 `git diff main..feature/report-a`

它的代表性输出会接近这样：

```text
diff --git a/report.txt b/report.txt
--- /dev/null
+++ b/report.txt
@@ -0,0 +1 @@
+report view: card
```

沿用第 4 章的读法，这里最重要的信号是：

- 出现了 `--- /dev/null`
  说明相对 `main` 来说，这是一个新增文件
- 出现了 `+report view: card`
  说明这条线多出了一行报表内容

把这一段读成人话就是：

**方案 A 相对 `main` 新增了一个 `report.txt`，里面只有一行 `report view: card`。**

### 再看 `git diff main..feature/report-b`

它的代表性输出会接近这样：

```text
diff --git a/report.txt b/report.txt
--- /dev/null
+++ b/report.txt
@@ -0,0 +1,2 @@
+report view: table
+report sort: updated-at
```

这一段的读法和上面一样，但结论已经不同了：

- 这条线同样是新增了 `report.txt`
- 但它不是只加一行
- 它同时引入了“表格视图”和“按更新时间排序”两部分内容

读成人话就是：

**方案 B 相对 `main` 不只是新增了一个报表视图，还额外多了一步排序规则。**

### 最后看 `git diff feature/report-a..feature/report-b`

这一条命令最有价值，因为它直接把两个候选方案摆在一起比较。

代表性输出会接近这样：

```text
diff --git a/report.txt b/report.txt
--- a/report.txt
+++ b/report.txt
@@ -1 +1,2 @@
-report view: card
+report view: table
+report sort: updated-at
```

这里不需要再把每一行语法重新定义一遍，只要抓住这次比较里最有决策价值的部分：

- `-report view: card`
  表示这是方案 A 有、方案 B 没保留的内容
- `+report view: table`
  表示方案 B 把视图改成了 table
- `+report sort: updated-at`
  表示方案 B 还额外加了一行排序规则

把这一段读成人话就是：

**方案 B 相比方案 A，既换了展示方式，又多加了一步排序。**

所以这三条 `diff` 真正帮助你完成的是三类判断：

- `main..feature/report-a`
  看方案 A 到底给主线增加了什么
- `main..feature/report-b`
  看方案 B 到底给主线增加了什么
- `feature/report-a..feature/report-b`
  直接看两个方案的核心差异到底在哪里

做到这一步，你就已经从“目录并行”进入了真正重要的阶段：

- 比较不同开发线的结果
- 选择一条值得进入主线的线
- 明确哪些线要保留，哪些线可以放弃

### 第六步：只收敛真正要留下的那一条

假设比较之后，你决定：

- 采纳 `feature/report-b`
- 放弃 `feature/report-a`
- 暂时保留 `hotfix/login-copy`，等待独立的发布窗口

这就是最真实的并行开发结果：

- 不是所有线都要合回主线
- 有的线进入主线
- 有的线保留待定
- 有的线直接放弃

这时在主目录 `demo/` 中执行：

```bash
git switch main
git merge --ff-only feature/report-b
git log --oneline --graph --decorate --all
```

如果把收敛后的状态画出来，大致会是：

```text
* E  hotfix/login-copy
| * D  main, feature/report-b
|/
| * C  feature/report-a
|/
* B
```

这里保留下来的事实是：

- `main` 通过快进从 `B` 前进到了 `D`
- `feature/report-b` 的结果已经进入 `main`
- `feature/report-a` 还存在，但已经被明确判定为不采纳
- `hotfix/login-copy` 仍然是一条独立线，不会因为这次功能收敛就被强行合并

到这里，这轮并行开发最关键的部分已经完成了：

- 方案 A 和方案 B 真的并行存在过
- 你不是靠复制目录比较它们，而是靠分支结果比较它们
- 你只把真正要留下的那一条收敛回了主线

至于已经结束的工作目录怎么清掉，后面第 6 节会沿用这套场景继续往下讲。

---

## 5. 为什么这在 AI 时代尤其重要

到这里再回头看 AI 时代下的并行开发，理解会自然很多。

因为你刚刚已经亲手跑过了一次通用场景：

- 稳定主线一直保留
- 两条候选线并行推进
- 修复线独立存在
- 最后只选一条进入主线

把这套结构直接映射到 AI 协作，几乎是一一对应的：

```text
稳定主线:
- demo/               -> main

AI 并行尝试:
- demo-ai-a/          -> experiment/ai-a
- demo-ai-b/          -> experiment/ai-b

独立修复线:
- demo-fix/           -> hotfix/urgent
```

你真正获得的不是“又多一个 Git 命令”，而是协调多个上下文的能力：

- AI-A 可以独立尝试方案 A
- AI-B 可以独立尝试方案 B
- 你自己或另一个 AI 可以单独处理紧急修复
- 最后回到主目录统一比较、统一收敛

所以 AI 并没有削弱 Git 的必要性，反而把 `worktree` 这样的能力放大了。

AI 越能并行产出结果，你越需要：

- 把不同尝试隔离开
- 把不同结果追踪清楚
- 把比较建立在分支结果上
- 把收敛过程保持可回退

这也是为什么在 AI 时代下，`worktree` 更像一种组织并行开发的基础设施，而不只是一个小技巧。

---

## 6. 如何清理 `worktree`：`remove` 和 `prune` 不是一回事

如果某个上下文已经结束，就应该把它清掉。

但这里也不要只记命令，要先分清两种能力：

- `git worktree remove`
  能力：删掉某个工作目录
- `git worktree prune`
  能力：清理失效的 worktree 记录

它们解决的不是同一个问题。

### 先看最常见的两条命令

```bash
git worktree remove <path>
git worktree prune
```

这里的参数含义也很简单：

- `remove`
  删除指定路径上的 worktree
- `<path>`
  指向要删除的那个工作目录
- `prune`
  清理仓库里已经失效的 worktree 记录

沿用上一节的实战场景，可以先把已经结束的两个工作目录清掉：

```bash
git worktree remove ..\demo-report-a
git worktree remove ..\demo-report-b
git worktree prune
git worktree list
```

把这几步拆开看，大致会是这样：

```text
执行前：

worktree:
- demo/            -> main
- demo-report-a/   -> feature/report-a
- demo-report-b/   -> feature/report-b
- demo-login-fix/  -> hotfix/login-copy

执行 `git worktree remove ..\demo-report-a`
执行 `git worktree remove ..\demo-report-b` 后：

worktree:
- demo/            -> main
- demo-login-fix/  -> hotfix/login-copy

分支:
- feature/report-a     可能还在
- feature/report-b     可能还在
- hotfix/login-copy    可能还在

执行 `git worktree prune` 后：

- 清掉失效的 worktree 记录
- 不会删除仍然存在的提交历史
```

这里最容易搞混的是：

- `remove` 清的是工作目录
- `prune` 清的是失效记录
- 分支要不要删，是另一件单独决定的事

所以不要把它们记成一组“清理命令”就完了，而要分清各自到底在清什么。

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

- `worktree` 本质上新增的能力是什么
- `git worktree add -b feature/report-a ..\demo-report-a main` 这一行里每一段分别在表达什么
- 为什么 `git worktree list` 只是看状态，而不改状态
- 为什么同一条分支不能同时在两个 worktree 里检出
- 能不能说清 `worktree`、分支和 `HEAD` 之间的关系
- 为什么它特别适合“一个人协调多个 AI 同时开工”的场景
- 能否把一轮并行开发从挂出上下文、比较方案、选择收敛到清理目录完整走一遍
- 为什么 `remove` 和 `prune` 不是一回事

下一篇再补上一组真正高频的进阶能力，你的 Git 工作流就会完整很多。
