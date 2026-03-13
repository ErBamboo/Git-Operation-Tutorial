# 远端与同步基础
> 目标
> - 30-40 分钟内掌握 Git 远端协作的 5 个核心能力：
>   - 理解 `origin`、远端分支、跟踪分支
>   - 会用 `clone`
>   - 区分 `fetch`、`pull`、`push`
>   - 看懂“本地领先 / 落后 / 分叉”的状态
>   - 能把自己的分支安全推到远端

---

## 0. 远端操作

学完本地提交、分支、回退之后，下一步最自然的问题就是：

**我怎么和“远端上的版本真相”打交道？**

因为一旦开始跟远端同步，Git 管理的就不只是你本机上的文件状态了，还包括：

- 远端仓库现在长什么样
- 你的本地分支和远端分支差了多少
- 你这次拉取的是谁的修改
- 你这次推送会不会覆盖别人的工作

所以这一篇的重点不是“学会连上 GitHub”，而是先彻底理解：

**本地仓库和远端仓库之间，到底是怎么同步状态的。**

---

## 1. 用本地 bare 仓库模拟一个远端

先不要急着上 GitHub。

为了把原理看清楚，我们先在本地造一个“远端服务器”。

在 `PowerShell` 中输入：

```powershell
cd $HOME # 或其他你认为合适的路径
mkdir git-remote-lab -Force
cd git-remote-lab
git init --bare --initial-branch=main server.git
git clone .\server.git alice
cd alice
git switch main
"hello remote" | Set-Content readme.txt
git add readme.txt
git commit -m "docs: init remote demo"
git push -u origin main
cd ..
git clone --branch main .\server.git bob
```

这里最关键的是：

- `server.git` 是一个 bare 仓库，可以把它理解成“服务器上的仓库”
- `alice` 和 `bob` 是两个普通工作副本
- `origin` 是 Git 给克隆来源起的默认远端名

这里要特意说明一下：我们显式把 bare 仓库的初始分支固定成了 `main`。

原因是不同 Git 环境里的默认分支名可能不一致。如果这里不固定，远端 `HEAD` 有可能仍然指向 `master`，而后面的练习又是围绕 `main` 展开的，这样 `bob` 克隆后就可能默认跟踪 `origin/master`，后面在 `git pull --ff-only` 那一步报错。

如果你的 Git 版本较老，不支持 `--initial-branch`，可以改用下面这组命令达到同样效果：

```bash
git init --bare server.git
git -C server.git symbolic-ref HEAD refs/heads/main
```

你现在已经有了一个最小可练习环境：

- 一个远端：`server.git`
- 两个本地开发者副本：`alice`、`bob`

---

## 2. `origin`、远端分支、跟踪分支分别是什么

先进入 `alice`：

```powershell
cd $HOME\git-remote-lab\alice # 此处 $HOME 应更改为你自定义的路径，后续不做赘述
```

```bash
git remote -v
git branch -vv
git log --oneline --graph --decorate --all
```

你应该先建立三个概念：

- `origin`：一个远端仓库的名字
- `origin/main`：你上次知道的“远端 main 指到了哪里”
- `main`：你本地正在工作的分支

把它们放在一起看，可以先建立这样一张关系图：

```text
远端仓库 server.git:      A---B
                            ^
                           main

alice 对远端的本地记录:  A---B
                            ^
                       origin/main

alice 当前本地分支:      A---B
                            ^
                       main, HEAD
```

把常用观察命令放进这张图里看，会更容易理解它们到底在看什么：

```text
`git branch -vv` 主要在看：
main, HEAD  ----比较---->  origin/main

`git status` 主要在看：
当前分支 + 工作区/暂存区 是否干净
```

如果 `main` 设置了上游分支，那么它就会“跟踪” `origin/main`。

这也是为什么 `git branch -vv` 很重要。它会直接告诉你：

- 当前分支在跟踪谁
- 你领先远端多少
- 你落后远端多少

初学阶段，一个非常重要的习惯就是：

**每次搞不清状态时，优先看 `git status` 和 `git branch -vv`。**

---

## 3. `push`：把本地正式状态送到远端

刚才初始化仓库时你已经做过一次：

```bash
git push -u origin main
```

这里要明确理解两个点：

- `push` 是把本地提交对象和分支指针推到远端
- `-u` 是把当前本地分支和远端分支建立跟踪关系

所以它的意义不只是“上传代码”，而是：

- 把你的本地历史同步给远端
- 告诉 Git：以后 `main` 默认对应 `origin/main`

如果把 `push` 看成“移动远端分支指针”，它的效果大致是这样：

```text
执行前：

alice 本地分支:          A---B---C
                                ^
                           main, HEAD

alice 已知远端状态:      A---B
                            ^
                       origin/main

远端仓库 server.git:     A---B
                            ^
                           main

执行 `git push origin main` 后：

alice 本地分支:          A---B---C
                                ^
                           main, HEAD

alice 已知远端状态:      A---B---C
                                ^
                           origin/main

远端仓库 server.git:     A---B---C
                                ^
                               main
```

这里真正变化的是：

- 远端仓库里的 `main`
- 你本地记录的 `origin/main`

这里没变化的是：

- 你当前本地分支 `main` 仍然停在 `C`
- `HEAD` 仍然站在 `main` 上

以后再推同一条分支时，通常就可以直接写：

```bash
git push
```

但在刚开始学习时，我更建议你先写全：

```bash
git push origin main
```

原因很简单：这样不容易把“我正在推哪条线”这件事想糊涂。

---

## 4. `fetch` 和 `pull` 不是一回事

这一步最值得反复练。

先让 `alice` 产生一个新的远端更新：

```powershell
cd $HOME\git-remote-lab\alice
"line from alice" | Add-Content readme.txt
```

```bash
git add readme.txt
git commit -m "docs: update from alice"
git push
```

然后切到 `bob`：

```powershell
cd $HOME\git-remote-lab\bob
```

先看状态：

```bash
git status
git branch -vv
git log --oneline --graph --decorate --all
```

这时 `bob` 还不知道远端已经变了。

执行：

```bash
git fetch origin
git status
git branch -vv
git log --oneline --graph --decorate --all
```

你现在要看懂：

- `origin/main` 已经前进了
- 但你本地 `main` 还没动

这时对象变化可以画成：

```text
执行前 `git fetch origin`：

远端仓库 server.git:     A---B---C
                                ^
                               main

bob 已知远端状态:        A---B
                            ^
                       origin/main

bob 当前本地分支:        A---B
                            ^
                       main, HEAD

执行后：

远端仓库 server.git:     A---B---C
                                ^
                               main

bob 已知远端状态:        A---B---C
                                ^
                           origin/main

bob 当前本地分支:        A---B
                            ^
                       main, HEAD
```

这就是 `fetch` 的本质：

**只更新你对远端状态的认知，不直接改你当前工作的本地分支。**

换句话说，`git fetch origin` 真正移动的是 `origin/main`，而不是你当前正在工作的 `main`。

再执行：

```bash
git pull --ff-only
git branch -vv
git log --oneline --graph --decorate --all
```

这里的 `--ff-only` 表示：

- 只允许“快进式更新” (fast-forward)
- 如果需要额外合并，直接报错，不偷偷帮你做复杂整合

执行后通常可观察到：

- 本地 `main` 追上了 `origin/main`
- 没有额外产生新的 merge commit
- 你的工作区只是被更新到了远端已经存在的那个提交

### 什么是 fast-forward

“快进”可以理解成：**当前分支只是把指针往前挪到了已有提交上**。

比如刚才 `bob` 已经执行完 `git fetch origin`，但还没 `pull` 时，关系大致是这样：

```text
执行前 `git pull --ff-only`：

远端仓库 server.git:     A---B---C
                                ^
                               main

bob 已知远端状态:        A---B---C
                                ^
                           origin/main

bob 当前本地分支:        A---B
                            ^
                       main, HEAD
```

执行 `git pull --ff-only` 之后，会变成：

```text
远端仓库 server.git:     A---B---C
                                ^
                               main

bob 已知远端状态:        A---B---C
                                ^
                           origin/main

bob 当前本地分支:        A---B---C
                                ^
                           main, HEAD
```

这里发生的事情只有一件：

- 本地 `main` 从 `B` 移动到了 `C`

所以 fast-forward 的特点是：

- 不会新建 merge commit
- 不会改写旧提交
- 只是把当前分支移动到更靠前的已有历史上

所以在初学阶段，一个很稳妥的理解方式是：

- `fetch` = 先看远端变成什么样
- `pull` = 在此基础上，把本地分支更新上去

而 `pull` 本质上通常可以理解成：

```bash
git fetch
git merge
```

或者某些配置下：

```bash
git fetch
git rebase
```

所以，别把 `pull` 当成一个“单纯下载代码”的命令。

顺手补一个小知识点：

- `git pull` 和 `git merge` 都有 `--ff` / `--no-ff` / `--ff-only`
- 它们都和“是否允许快进、是否保留 merge commit”有关
- 这一篇你先只需要掌握 `git pull --ff-only`

`git push` 不带这一组选项，但远端通常会拒绝会改写远端历史的 non-fast-forward push。这个问题后面讲协作和发布流时再展开。

### 常见问题：`refs/heads/master` 报错

如果你在这里看到类似报错：

```text
Your configuration specifies to merge with the ref 'refs/heads/master'
from the remote, but no such ref was fetched.
```

通常说明：

- 你当前分支还在跟踪 `origin/master`
- 但这个练习实际使用的是 `origin/main`

这时你先看清当前状态：

```bash
git branch -vv
git branch -a
```

如果你看到 `bob` 当前站在 `master`，或者它在跟踪 `origin/master`，就执行：

```bash
git fetch origin
git switch -c main --track origin/main
```

这里的 `--track` 表示：

- 在创建本地分支 `main` 的同时，让它跟踪 `origin/main`
- 建立跟踪关系后，`git branch -vv` 和 `git status` 才知道该和哪条远端分支比较
- 默认的 `git pull` / `git push` 也会更清楚地围绕这条关系工作

在这一篇里，可以先把“跟踪”理解成：**给本地分支指定一个默认对应的远端分支。**

```text
执行 `git switch -c main --track origin/main` 后：

本地分支:   main -> C
远端记录:   origin/main -> C
跟踪关系:   main ----> origin/main
HEAD:       HEAD -> main
```

如果你的 Git 版本较老，也可以用：

```bash
git fetch origin
git checkout -b main origin/main
```

这条旧写法在这里也会建立同样的跟踪关系。

然后再确认一次：

```bash
git status
git branch -vv
```

这里还要多记住一点：

- `git pull` 更新的是当前分支对应的工作区状态
- 它不会“额外生成一个新文件夹”

---

## 5. 看懂“领先 / 落后 / 分叉”，并把每种状态走完整

这一节只做一件事：把远端同步里最常见的三种关系状态拆开看清楚。

它们分别是：

- 领先：本地有、远端无
- 落后：远端有、本地无
- 分叉：本地和远端各自都有独占提交

### 1) 领先：本地有、远端无

在 `bob` 里再做一个本地提交：

```powershell
"line from bob" | Add-Content readme.txt
```

```bash
git add readme.txt
git commit -m "docs: update from bob"
git branch -vv
git status
```

此时通常会看到：

- 本地 `main` 领先 `origin/main`

```text
A---B---C
    ^   ^
    |   main, HEAD
    |
 origin/main
```

这通常是刚执行完本地 `git commit`、但还没 `git push` 的结果。

这表示：

- 新提交只存在于本地
- 远端还没有这段历史

这种状态最典型的处理方式是把本地历史推到远端：

把它推上去：

```bash
git push
git branch -vv
```

推送完成后，`main` 和 `origin/main` 会重新回到同一条历史上。

### 2) 落后：远端有、本地无

切换回 `alice`：

```powershell
cd $HOME\git-remote-lab\alice
```

```bash
git fetch origin
git branch -vv
```

此时通常会看到：

- 本地 `main` 落后于 `origin/main`

```text
A---B---C
    ^   ^
    |   origin/main
    |
  main, HEAD
```

这通常是先看到远端有新提交，再执行 `git fetch origin` 后暴露出来的状态。

这表示：

- 远端已经有了新的提交
- 本地还没有把这段历史接过来

这种状态的典型处理方式是快进更新：

```bash
git pull --ff-only
git branch -vv
```

执行完成后，`alice` 的 `main` 会再次和 `origin/main` 对齐。

这就是“落后”场景最常见的处理路径：

- 先 `fetch` 看清远端
- 确认只是单纯落后
- 再用 `git pull --ff-only` 做快进更新

### 3) 分叉：本地和远端各自都有独占提交

“分叉”不是单纯的领先，也不是单纯的落后，而是两边都已经各自前进了一步，这在多人协作开发的场景十分常见。

为了单独观察这种状态，先在 `alice` 里创建一条专门用于演示的本地分支，并同样让它跟踪 `origin/main`，便于继续观察 ahead / behind 状态：

```bash
git switch -c demo/diverged --track origin/main
```

在 `PowerShell` 中输入：

```powershell
"line only in alice demo" | Add-Content readme.txt
```

```bash
git add readme.txt
git commit -m "docs: local only change on demo branch"
git branch -vv
```

此时通常会看到：

- `demo/diverged` 领先 `origin/main`

这说明 `demo/diverged` 已经比远端多出一个本地提交。

接着切换到 `bob`：

```powershell
cd $HOME\git-remote-lab\bob
```

在 `PowerShell` 中输入：

```powershell
"line from bob again" | Add-Content readme.txt
```

```bash
git add readme.txt
git commit -m "docs: update from bob again"
git push
```

再切换回 `alice`：

```powershell
cd $HOME\git-remote-lab\alice
```

```bash
git switch demo/diverged
git fetch origin
git branch -vv
git log --oneline --graph --decorate --all
```

此时通常可观察到一种新的状态：

- 本地分支有自己的提交
- `origin/main` 也有本地分支没有的提交
- `demo/diverged` 相对 `origin/main` 同时表现为 ahead 和 behind

```text
      C  demo/diverged, HEAD
     /
A---B
     \
      D  origin/main
```

这通常是“本地先做了一次提交，然后远端也前进了一次，最后本地再 `fetch`”之后得到的结果。

这就进入了“分叉”状态：

- 本地分支有自己的提交
- 远端也有本地分支没有的提交

在这种情况下，问题已经不能再被理解成“谁比较新”，而必须开始处理“怎么整合两边的历史”。（你可以理解为，因为分叉了，有两条路可以走，所以 git 不知道下一步该怎么走了）

可以直接验证一下：

```bash
git pull --ff-only
```

这一步会失败。原因是：

- fast-forward 只适用于“本地只是单纯落后”的情况
- 一旦本地和远端各自都有独占提交，分支指针就不能只靠往前移动解决问题

这时就要进入真正的历史整合：

- 要么 `merge`
- 要么 `rebase`

这两种方式下一篇和后续协作章节会专门展开。这里需要记住的是：

**`--ff-only` 很适合帮助识别“这次同步到底是不是简单更新”，一旦它失败，通常就说明已经进入了更复杂的分叉场景。**

为了让后续练习继续从干净主线开始，最后切回 `main`：

```bash
git switch main
git pull --ff-only
```

---

## 6. 把新分支推到远端

很多实际开发不是直接在 `main` 上改，而是先开分支。

在 `alice` 中试一下：

```bash
git switch -c feature/remote-note
```

在 `PowerShell` 中输入：

```powershell
"feature line" | Add-Content readme.txt
```

```bash
git add readme.txt
git commit -m "feat: add remote note"
git push -u origin feature/remote-note
git branch -vv
```

你现在应该理解：

- 新分支默认只存在于本地
- `git push -u origin <branch>` 会把它同步到远端
- 建立跟踪关系后，这条线后续就能更自然地推拉

这个能力是后面学习协作开发的前提。

---

## 7. 这一篇学完后，你至少应该建立的习惯

- 每次同步前先看 `git status`
- 每次看分支关系时先看 `git branch -vv`
- 不把 `pull` 当成“无脑更新”
- 能先 `fetch` 看清远端状态，再决定怎么整合
- 能分清“领先 / 落后 / 分叉”分别意味着什么
- 第一次推新分支时明确写出 `git push -u origin <branch>`

---

## 8. 你现在的熟练度指标

如果下面这些问题你都能回答，就说明这一篇已经过关：

- `origin` 和 `origin/main` 分别是什么
- `fetch` 为什么不会直接修改当前分支
- `--track` 建立的是什么关系
- 什么叫 fast-forward
- `pull --ff-only` 比直接 `pull` 稳在哪里
- 本地“领先”和“落后”各自意味着什么
- 为什么“分叉”时 `git pull --ff-only` 会失败
- 为什么 `git branch -vv` 能显示领先 / 落后信息
- 为什么新建分支后要先 `push -u`

下一篇，就该进入真正的协作主流程了：

**“多个人怎么围绕同一个仓库稳定协作”。**
