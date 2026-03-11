# 单人练习
> 目标
> - 30 分钟内掌握 Git 的 6 个核心能力：
>  	- 建仓
>  	- 提交
> 	- 看差异
>  	- 分支
>  	- 合并 / 冲突
>  	- 回退 / 找回
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

---

## 1. 第一次提交：理解“工作区 → 暂存区 → 提交”

创建一个文件，在`powshell`中输入以下命令
```powershell
"hello git" | Set-Content note.txt
```
然后切换至 `git bash`
```bash
git status
```
把它放进暂存区：
```bash
git add note.txt
git status
```
提交：
```bash
git commit -m "feat: add note"
git status
git log --oneline --graph --decorate --all
```
你现在应该看到：

- `working tree clean`
- 历史里出现 1 个提交

你要记住：

- `git add` = 放进“下一次提交”
- `git commit` = 把暂存区做成正式快照

---

## 2. 理解差异：工作区和暂存区不是一回事

继续改文件：
在`powshell`中输入以下命令
```powershell
"line 2" | Add-Content note.txt
```
切换至`git bash`
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
提交：
```bash
git commit -m "docs: update note"
git log --oneline --graph --decorate --all
```
你现在应该理解：

- `git diff` 看的是“还没暂存的改动”
- `git diff --cached` 看的是“已经暂存、准备提交的改动”

---

## 3. 分支：理解“分支只是指针”

创建功能分支：
在`git bash`中输入以下命令
```bash
git switch -c feature/theme
```
切换至`powshell`
```powershell
"blue" | Set-Content theme.txt
```
再次切换回`git bash`
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
你现在应该理解：

- 分支不是复制仓库
- 分支只是“指向某个提交的名字”

---

## 4. 冲突：理解 Git 为什么需要你做决定
> 从本章开始，不再区分powershell和bash，请自行判断

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

从 reflog 里找到刚才那个提交哈希，再恢复：

```bash
git reset --hard <刚才那条提交哈希>
git log --oneline -n 3
```

你现在应该理解：

- restore 处理文件
- reset 处理历史/指针
- reflog 是你的后悔药

---

## 7. 30 分钟结束时，你必须会检查这 5 件事

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

## 8. 你现在的熟练度指标

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
