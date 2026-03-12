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

这就是 `fetch` 的本质：

**只更新你对远端状态的认知，不直接改你当前工作的本地分支。**

再执行：

```bash
git pull --ff-only
```

这里的 `--ff-only` 表示：

- 只允许“快进式更新”
- 如果需要额外合并，直接报错，不偷偷帮你做复杂整合

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

### 常见问题：`refs/heads/master` 报错

如果你在这里看到类似报错：

```text
Your configuration specifies to merge with the ref 'refs/heads/master'
from the remote, but no such ref was fetched.
```

通常说明的不是 `git pull --ff-only` 本身有问题，而是：

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

如果你的 Git 版本较老，也可以用：

```bash
git fetch origin
git checkout -b main origin/main
```

然后再确认一次：

```bash
git status
git branch -vv
```

这里还要多记住一点：

- `git pull` 更新的是当前分支对应的工作区状态
- 它不会“额外生成一个新文件夹”

所以如果你在 `bob` 里没有看到新的 `readme.txt` 或文件内容变化，通常说明这次 `pull` 根本没有成功更新到正确的分支，而不是“快进更新没有生效”。

---

## 5. 看懂“领先 / 落后 / 分叉”

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

如果你还没推送，这时你通常会看到：

- 本地 `main` 领先 `origin/main`

把它推上去：

```bash
git push
```

再回到 `alice`：

```powershell
cd $HOME\git-remote-lab\alice
```

```bash
git fetch origin
git branch -vv
```

这时 `alice` 会看到：

- 本地 `main` 落后于 `origin/main`

而“分叉”则表示：

- 你本地有自己的提交
- 远端也有你本地没有的提交

这种情况下，你已经不能把问题理解成“谁比较新”了，而必须开始处理“怎么整合两边的历史”。

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
- 第一次推新分支时明确写出 `git push -u origin <branch>`

---

## 8. 你现在的熟练度指标

如果下面这些问题你都能回答，就说明这一篇已经过关：

- `origin` 和 `origin/main` 分别是什么
- `fetch` 为什么不会直接修改当前分支
- `pull --ff-only` 比直接 `pull` 稳在哪里
- 本地“领先”和“落后”各自意味着什么
- 为什么新建分支后要先 `push -u`

下一篇，就该进入真正的协作主流程了：

**不是“我会不会 push”，而是“多个人怎么围绕同一个仓库稳定协作”。**
