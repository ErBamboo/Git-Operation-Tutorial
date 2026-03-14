# 单人练习
> 目标
> - 30 分钟内掌握 Git 的 6 个核心能力：
>   - 建仓
>   - 提交
>   - 看差异
>   - 分支
>   - 合并 / 冲突
>   - 回退 / 找回

这一篇只做一件事：

- 把第 3 篇里那套模型，先放到“单仓库内部”做纵向练习

所以这里先不引入远端，不引入协作，也不引入 fork。
先只看最基础的一条线：

- 工作区
- 暂存区
- 提交
- 分支和 `HEAD` 在本地怎么变化

---

## 0. 安全准备：单独建一个练习目录

```powershell
cd $HOME # 请自行寻找一个合适的路径
mkdir git-lab -Force
cd git-lab
mkdir demo -Force
cd demo
```
```bash
git --version
git config --global user.name
git config --global user.email
```
如果 user.name / user.email 是空的，先设置：
```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```
初始化仓库：
```bash
git init -b main
git status
```
你现在应该理解：

- 目录刚变成一个 Git 仓库
- 但还没有任何提交

说明：

- 下文只在需要创建或修改文件时单独标注 `PowerShell`
- 其余 Git 命令默认直接在你常用的 Git 终端执行

---

## 1. 第一次提交：理解“工作区 → 暂存区 → 提交”

创建一个文件，在 `PowerShell` 中输入以下命令：
```powershell
"hello git" | Set-Content note.txt
```
```bash
git status
```
把它放进暂存区：
```bash
git add note.txt
git status
```
这里的 `note.txt` 表示“只暂存这个文件”，避免你在刚开始练习时一次性把别的改动也带进去。

如果把 `git add note.txt` 看成“把当前文件快照送进暂存区”，它的效果大致是：

```text
执行前：

工作区:     note.txt -> "hello git"
暂存区:     (空)
提交历史:   (空)
分支/HEAD:  main, HEAD -> 尚无提交

执行 `git add note.txt` 后：

工作区:     note.txt -> "hello git"
暂存区:     note.txt -> "hello git"
提交历史:   (空)
分支/HEAD:  main, HEAD -> 尚无提交
```

这里变化的是：`note.txt` 被放进了暂存区；还没变化的是：提交历史、分支指针和 `HEAD`。

提交：
```bash
git commit -m "feat: add note"
git status
git log --oneline --graph --decorate --all
```
其中：

- `-m` 表示直接在命令后面写提交说明
- `--oneline` 表示每个提交压缩成一行
- `--graph` 表示把分支和合并关系画出来
- `--decorate` 表示显示分支名、`HEAD` 等引用
- `--all` 表示把所有分支一起显示出来

你现在应该看到：

- `working tree clean`
- 历史里出现 1 个提交

如果把 `git commit -m "feat: add note"` 看成“把暂存区正式做成一个提交对象”，它的效果大致是：

```text
执行前：

工作区:     note.txt -> "hello git"
暂存区:     note.txt -> "hello git"
提交历史:   (空)
分支/HEAD:  main, HEAD -> 尚无提交

执行 `git commit -m "feat: add note"` 后：

工作区:     note.txt -> "hello git"
暂存区:     与最新提交一致
提交历史:   A
分支/HEAD:  main, HEAD -> A
```

这里真正新产生的是提交对象 `A`；同时前移的是当前分支和 `HEAD`。

你要记住：

- `git add` = 放进“下一次提交”
- `git commit` = 把暂存区做成正式快照

---

## 2. 理解差异：工作区和暂存区不是一回事

继续改文件：
在 `PowerShell` 中输入以下命令：

```powershell
"line 2" | Add-Content note.txt
```
```bash
git diff
git status
```
把修改加入暂存区：
```bash
git add note.txt
git diff --cached
git status
```
这时得分清的是：`git diff` 和 `git diff --cached` 看的不是同一个范围。

```text
在执行 `git add note.txt` 之前：

工作区 note.txt:  "hello git\nline 2"
暂存区 note.txt:  "hello git"
最新提交 A:       "hello git"

`git diff`           看 工作区  vs 暂存区
`git diff --cached`  看 暂存区  vs 最新提交
```

执行 `git add note.txt` 后：

```text
工作区 note.txt:  "hello git\nline 2"
暂存区 note.txt:  "hello git\nline 2"
最新提交 A:       "hello git"

这时：
- `git diff` 通常已经没有这部分差异了
- `git diff --cached` 会显示“准备提交的变化”
```

这里要刻意区分：

- `git diff` 默认看“工作区”和“暂存区”的差异
- `git diff --cached` 中的 `--cached` 表示看“暂存区”和“最新提交”的差异

提交：
```bash
git commit -m "docs: update note"
git log --oneline --graph --decorate --all
```
你现在应该理解：

- `git diff` 看的是“还没暂存的改动”
- `git diff --cached` 看的是“已经暂存、准备提交的改动”

### 补充：怎么看 `git diff` 的输出

上面已经知道了 `git diff` 在“看谁和谁比”。

再往前走一步，还应该知道它的输出长什么样。以后不管你是在看工作区、暂存区，还是在比较两条分支，`diff` 的基本结构都还是这一套。

如果现在执行 `git diff --cached`，代表性输出会接近这样：

```text
diff --git a/note.txt b/note.txt
--- a/note.txt
+++ b/note.txt
@@ -1 +1,2 @@
 hello git
+line 2
```

这里最值得先看懂的是这几行：

- `diff --git a/note.txt b/note.txt`
  表示下面这一段变化发生在 `note.txt`
- `--- a/note.txt`
  表示旧版本这一侧是谁
- `+++ b/note.txt`
  表示新版本这一侧是谁
- `@@ -1 +1,2 @@`
  表示下面这一个变化片段发生在文件的哪一段位置

再看变化正文：

- 前面带空格的行
  表示这行内容是上下文，本身没有变化
- 前面带 `+` 的行
  表示这行是新增内容
- 前面带 `-` 的行
  表示这行是旧版本里有、现在被删掉或替换掉的内容

所以这段输出读成人话就是：

- `note.txt` 被改了
- 原来的 `hello git` 还在
- 新增了一行 `line 2`

还有一种你后面很快会遇到的形状，是“新增文件”：

```text
diff --git a/report.txt b/report.txt
--- /dev/null
+++ b/report.txt
@@ -0,0 +1 @@
+report view: card
```

这里的关键点是：

- `--- /dev/null`
  表示旧版本这一侧根本没有这个文件
- `+++ b/report.txt`
  表示新版本这一侧新增了 `report.txt`

也就是说，当你看到 `/dev/null` 时，通常可以先把它理解成：

**这次比较里，有一边原本没有这个文件。**

后面到了远端、分支和 `worktree` 的场景，`git diff` 仍然复用这套读法。变化的不是输出结构，而是“谁和谁在比较”。

---

## 3. 分支：理解“分支只是指针”

创建功能分支：
```bash
git switch -c feature/theme
```
这里的 `-c` 表示“创建并切换到新分支”；如果没有 `-c`，`git switch` 只负责切换，不负责创建。

在这套教程里，“切出一条分支”可以先理解成：

- 以当前所在提交为基线，新建一个分支名
- 然后立刻把 `HEAD` 切换到那条分支上继续工作

所以 `git switch -c feature/theme` 可以直接读成：

- 从当前 `main` 所在提交切出 `feature/theme`
- 并立即切换到 `feature/theme`

它的变化大致是：

```text
执行前：

提交历史: A---B
分支指针: main -> B
HEAD:     HEAD -> main

执行 `git switch -c feature/theme` 后：

提交历史: A---B
分支指针: main -> B
         feature/theme -> B
HEAD:     HEAD -> feature/theme
```

在 `PowerShell` 中写入一个新文件：
```powershell
"blue" | Set-Content theme.txt
```
```bash
git add theme.txt
git commit -m "feat: add theme"
git log --oneline --graph --decorate --all
```
回到 main，做另一条历史：
同理，此处不做工具切换说明。

```bash
git switch main
```

```powershell
"main branch file" | Set-Content readme.txt
```
```bash
git add readme.txt
git commit -m "docs: add readme"
git log --oneline --graph --decorate --all
```
现在合并：
```bash
git merge feature/theme
git log --oneline --graph --decorate --all
```
这一步最容易误解成“把功能分支整个复制回来”，但 `merge` 真正做的是“保留两边原历史，再额外生成一个整合结果”。

```text
执行前 `git merge feature/theme`：

A---B---D  main, HEAD
     \
      C    feature/theme

执行后：

A---B---D---M  main, HEAD
     \     /
      C---/   feature/theme
```

这里要看懂三件事：

- `C` 和 `D` 都是 merge 之前就已经存在的提交
- `M` 才是 merge 新生成的整合提交
- merge 不会把旧提交改掉，它是在旧历史之上再补一个汇合点

你现在应该理解：

- 分支不是复制仓库
- 分支只是“指向某个提交的名字”

---

## 4. 冲突：理解 Git 为什么需要你做决定
> 从本章开始，只在需要写文件时标注 `PowerShell`；其余 Git 命令默认直接执行。

先在 main 建一个文件：
```powershell
"base line" | Set-Content app.txt
```
```bash
git add app.txt
git commit -m "feat: add app file"
```

切一条冲突分支并修改：

```bash
git switch -c feature/conflict
```
```powershell
"feature version" | Set-Content app.txt
```
```bash
git add app.txt
git commit -m "feat: edit app on feature"
```

回到 main，改同一个文件：

```bash
git switch main
```
```powershell
"main version" | Set-Content app.txt
```
```bash
git add app.txt
git commit -m "feat: edit app on main"
```

触发冲突：

```bash
git merge feature/conflict
git status
```
```powershell
Get-Content app.txt
```

你会看到冲突标记。手工改成你要的结果，例如：

```text
执行 `git merge feature/conflict` 后（冲突尚未解决）：

A---B---C---E  main, HEAD
         \
          D    feature/conflict

工作区: app.txt = 同时包含两边内容和冲突标记
暂存区: Git 还没有得到最终整合结果
下一步: 手工编辑 -> git add app.txt -> git commit
```

这一步的关键是：Git 已经知道“两条线要整合”，但它还不知道最终文件应该长什么样，所以不会直接替你生成 merge commit。

```powershell
@"
main version
feature version
"@ | Set-Content app.txt
```

告诉 Git 冲突已解决：

```bash
git add app.txt
git commit -m "merge: resolve app conflict"
git log --oneline --graph --decorate --all
```

你现在应该理解：

- 冲突不是坏事
- Git 只是不会替你决定“哪一版才是对的”

---

## 5. Rebase：理解“重放提交”

切一条新分支：

```bash
git switch -c feature/rebase
```
```powershell
"rebase demo" | Set-Content rebase.txt
```
```bash
git add rebase.txt
git commit -m "feat: add rebase demo"
```

回到 main，再加一个提交：

```bash
git switch main
```
```powershell
"another main change" | Add-Content readme.txt
```
```bash
git add readme.txt
git commit -m "docs: update readme again"
```

回到功能分支，执行 rebase：

```bash
git switch feature/rebase
git rebase main
git log --oneline --graph --decorate --all
```

你现在应该理解：

- merge 是“汇合两条线”
- rebase 是“把我的提交重新接到新的基线后面”

如果把第 3 节里的 `merge` 和这里的 `rebase` 放到同一种分叉场景里看，可以画成下面这三张图：

- `P`：分叉前共同的基线
- `F`：功能分支上的提交
- `M`：`main` 后来新增的提交
- `G`：merge 产生的汇合提交
- `F'`：rebase 后重新生成的提交

### 执行前

```text
...---P---M                  main
      \
       F                     feature/*
```

这表示：

- 你先从 `P` 切出功能分支
- 在功能分支上做了一个提交 `F`
- 与此同时，`main` 又往前走了一个提交 `M`

此时两条线已经分叉。

### merge 后

```text
...---P---M------G           main
      \         /
       F-------/             feature/*
```

如果你这时做的是 `merge`，Git 的做法是：

- 保留原来的 `F`
- 保留原来的 `M`
- 额外创建一个新的提交 `G`，表示“把两条线汇合后的结果”

所以 `merge` 的特点是：

- 旧提交都还在
- 历史关系没有被改写
- 只是新增了一个“汇合点”

### rebase 后

```text
...---P---M---F'             feature/*
```

如果你这时做的是 `rebase`，Git 的做法是：

- 先找出功能分支独有的提交 `F`
- 暂时把 `F` 的修改效果记下来
- 让功能分支先站到 `M` 后面
- 再把 `F` 的修改重新应用一次

这时你看到的已经不是原来的 `F`，而是新的 `F'`。

原因是 Git 的每个提交都包含“父提交是谁”这个信息：

- 原来的 `F` 的父提交是 `P`
- rebase 后的 `F'` 的父提交变成了 `M`

父提交一变，这个提交对象本身就不是原来的那个了，所以 Git 只能重新生成一个新的提交。

这也是为什么：

- `merge` 通常不会改写已有历史，因为它不动旧提交，只新增一个汇合提交
- `rebase` 会改写这条分支上的历史，因为它把旧提交换成了基于新基线重新生成的提交

所以你可以把这两者先记成一句话：

- `merge` = 保留两边原历史，再做一次汇合
- `rebase` = 把我的提交拆下来，接到新的基线后面重新播放

---

## 6. 回退与找回：新手最该掌握的保命能力

### 撤销工作区修改

```powershell
"temp line" | Add-Content note.txt
```
```bash
git status
git restore note.txt
git status
```
`git restore note.txt` 的含义是：放弃工作区里这个文件尚未暂存的修改，回到最近一次暂存或提交时的状态。

```text
执行前：

工作区 note.txt:  已提交内容 + temp line
暂存区 note.txt:  已提交内容
最新提交:         已提交内容

执行 `git restore note.txt` 后：

工作区 note.txt:  已提交内容
暂存区 note.txt:  已提交内容
最新提交:         已提交内容
```

这里变化的是工作区；暂存区和提交历史都没有变化。

### 取消暂存

```powershell
"staged line" | Add-Content note.txt
```
```bash
git add note.txt
git status
git restore --staged note.txt
git status
```
其中 `--staged` 表示“只从暂存区撤下来”，不会删除你工作区里已经改好的内容。

```text
执行前：

工作区 note.txt:  已提交内容 + staged line
暂存区 note.txt:  已提交内容 + staged line
最新提交:         已提交内容

执行 `git restore --staged note.txt` 后：

工作区 note.txt:  已提交内容 + staged line
暂存区 note.txt:  已提交内容
最新提交:         已提交内容
```

这里变化的是暂存区；工作区里你刚写好的内容仍然保留。

### 小补充：HEAD 位移

从这里开始，后面会第一次用到像 `HEAD~1` 这样的写法。

这是一类通用的**提交引用表达式**，用来表示“当前想站到哪一个提交上”。

这类位移还可以连续写。一个足够实用的读法是：

- 从左到右读
- 每多写一段，就在前一个结果上继续移动一次

先看最常见的线性历史：

```text
A---B---C---D
            ^
          HEAD

HEAD~1 -> C
HEAD~2 -> B
HEAD^  -> C
HEAD^^ -> B
HEAD~1^ -> B
```

这里可以先记住：

- `HEAD`：当前所在提交
- `HEAD~1`：沿第一父链往上退 1 个提交
- `HEAD~n`：沿第一父链连续往上退 `n` 个提交
- `HEAD^`：当前提交的第一个父提交

在线性历史里，`HEAD~1` 和 `HEAD^` 经常会落到同一个提交上，所以看起来很像。

进一步看组合写法，你会发现在线性历史里：

- `HEAD~2`
- `HEAD^^`
- `HEAD~1^`

这些写法经常会落到同一个位置。也正因为如此，新手很容易误以为 `~` 和 `^` 没区别。

它们真正拉开区别，是在 merge commit 上：

```text
      E---F
     /     \
A---B---C---M
            ^
          HEAD

HEAD^   -> C
HEAD^2  -> F
HEAD~1  -> C
HEAD^^  -> B
HEAD^2~1 -> E
```

这时可以这样理解：

- `HEAD^`：看第一个父提交，通常就是当前分支原本那条主线
- `HEAD^2`：看第二个父提交，通常就是被 merge 进来的那条线
- `HEAD~1`：仍然只会沿第一父链往上退，不会跑到第二父提交去

这里的组合例子可以直接这样读：

- `HEAD^^`：先到第一个父提交，再到那个提交的第一个父提交
- `HEAD^2~1`：先到第二个父提交，再沿第一父链往上退一格

所以这组写法本质上是在描述：

- “从当前提交往上退几步”
- “或者当前这个提交的第几个父提交是谁”

后面如果看到 `git reset --hard HEAD~1`，你就可以直接把它读成：

- 把当前分支、暂存区和工作区一起退回到 `HEAD` 的上一个提交

### 体验 reflog 救命

先做一个测试提交：

```bash
git add note.txt
git commit -m "docs: reflog demo"
git log --oneline -n 3
```

硬回退一格：

```bash
git reset --hard HEAD~1
git log --oneline -n 3
git reflog --oneline -n 5
```
如果把 `git reset --hard HEAD~1` 看成“把分支、暂存区和工作区一起退回上一个提交”，它的效果大致是：

```text
执行前：

提交历史:   A---B
分支/HEAD:  main, HEAD -> B
工作区:     与 B 一致
暂存区:     与 B 一致

执行 `git reset --hard HEAD~1` 后：

提交历史:   A---B
分支/HEAD:  main, HEAD -> A
工作区:     与 A 一致
暂存区:     与 A 一致
reflog:     仍然记得 B 刚刚还是 HEAD
```

这里要特别小心：

- `HEAD~1` 用的是上面那套提交引用表达式，表示沿第一父链退回当前提交的上一个提交
- `--hard` 会同时重置 `HEAD`、暂存区和工作区
- 如果你有尚未提交的重要改动，`git reset --hard` 会直接把它们丢掉

从 reflog 里找到刚才那个提交哈希，再恢复：

```bash
git reset --hard <刚才那条提交哈希>
git log --oneline -n 3
```

你现在应该理解：

- restore 处理文件
- reset 处理历史/指针
- reflog 是你的后悔药

这里再多记一层：`git reflog` 看的不是“正式提交历史长什么样”，而是“`HEAD` 和分支指针最近走过哪些位置”。

---

## 7. 常用场景与常用参数

下面这一章不是新的主线练习，而是把你刚刚做过的动作，整理成今后最常遇到的本地场景速查。

### 1. 我现在到底处于什么状态

场景：刚打开仓库，不知道自己在哪个分支、有没有改动、最近历史长什么样。

推荐命令：

```bash
git status
git log --oneline --graph --decorate --all
git branch -vv
```

参数说明：

- `--oneline`：压缩显示提交，适合快速扫历史
- `--graph`：画出分支走向
- `--decorate`：显示分支名、标签、`HEAD`
- `-vv`：让 `git branch` 显示每个分支指向的提交；如果配置了远端跟踪分支，也会一并显示

典型用法：

```bash
git status
git branch -vv
```

常见误区：不要只盯着文件列表看，先看 `git status`，再决定下一步操作。

### 2. 我改了什么，哪些还没提交

场景：你已经编辑过文件，但不确定改动目前在工作区、暂存区，还是都已经提交。

推荐命令：

```bash
git diff
git diff --cached
```

参数说明：

- `git diff`：看“工作区相对暂存区”的改动
- `--cached`：看“暂存区相对最新提交”的改动

典型用法：

```bash
git diff
git add note.txt
git diff --cached
```

常见误区：`git diff` 没输出，不代表没改动，可能是你已经把改动暂存了。

### 3. 我只想把这次要提交的内容放进去

场景：你改了多个文件，但这次只想提交其中一部分。

推荐命令：

```bash
git add note.txt
git add .
```

参数说明：

- `git add <file>`：只暂存指定文件
- `.`：表示当前目录，把当前目录下能加入的改动一起加入暂存区

典型用法：

```bash
git add note.txt
git status
```

常见误区：`git add .` 很方便，但也容易把你没打算提交的文件一起放进暂存区。

### 4. 我想正式保存这次修改

场景：你已经确认暂存区内容没问题，准备形成一次正式提交。

推荐命令：

```bash
git commit -m "docs: update note"
git commit --amend --no-edit
```

参数说明：

- `-m`：直接在命令行写提交说明
- `--amend`：修改最近一次提交，常用于补文件或改提交说明
- `--no-edit`：保留最近一次提交说明，不再打开编辑器让你修改文字

典型用法：

```bash
git add note.txt
git commit --amend --no-edit
```

常见误区：`--amend` 会改写最近一次提交，不要在已经推送并被别人依赖的提交上随便使用。

### 5. 我想切分支、看分支、准备合并

场景：你要开始一个新任务，或者想确认当前分支情况后再合并。

推荐命令：

```bash
git switch -c feature/demo
git switch main
git branch -vv
git merge feature/demo
```

参数说明：

- `-c`：创建并切换到新分支
- `-vv`：查看分支更详细的信息

典型用法：

```bash
git branch -vv
git switch main
git merge feature/theme
git status
```

常见误区：合并前先确认你当前站在目标分支上，例如“把功能分支合回 `main`”时，你应该先 `git switch main`。

### 6. 我想删除或重命名文件

场景：文件名写错了，或者某个文件已经不再需要。

推荐命令：

```bash
git rm old.txt
git rm --cached keep-local.txt
git mv old-name.md new-name.md
```

参数说明：

- `git rm <file>`：删除文件，并把“删除”记录进下一次提交
- `--cached`：只取消 Git 跟踪，本地文件保留
- `git mv <旧名> <新名>`：在 Git 视角下完成重命名

典型用法：

```bash
git mv "练习草稿.md" "练习草稿-本地.md"
git status
```

常见误区：如果你只在文件管理器里删文件或改文件名，却没有把“删除/重命名”一起提交，远端就可能保留旧文件。

### 7. 我改错了，想撤销或找回

场景：你刚改坏了文件、暂存错了内容，或者把提交回退掉之后想找回来。

推荐命令：

```bash
git restore note.txt
git restore --staged note.txt
git reset --hard HEAD~1
git reflog --oneline -n 5
```

参数说明：

- `--staged`：只处理暂存区
- `--hard`：同时重置历史、暂存区和工作区
- `HEAD~1`：提交引用表达式，表示沿第一父链退回当前提交的上一个提交
- `-n 5`：只显示最近 5 条记录

典型用法：

```bash
git reset --hard HEAD~1
git reflog --oneline -n 5
git reset --hard <要找回的提交哈希>
```

常见误区：`git restore` 主要用于文件层面，`git reset` 主要用于提交和指针层面，这两类命令不要混着理解。

---

## 8. 30 分钟结束时，你必须会检查这 5 件事

每次操作前后，先看：

```bash
git status
git log --oneline --graph --decorate --all
git diff
git diff --cached
git branch -vv
```

你要能回答：

- 我现在在哪个分支？
- 我改了什么？
- 哪些已经暂存？
- 最近提交是什么？
---

## 9. 你现在的熟练度指标

如果下面这些你都能独立完成，就已经即将新手期了：

- 独立创建仓库并做 2 次提交
- 看懂 git status
- 区分 git diff 和 git diff --cached
- 独立创建 / 切换分支
- 完成一次 merge
- 手工解决一次冲突
- 理解 rebase 的含义
- 用 restore 撤销文件改动
- 用 reflog 找回“消失的提交”
