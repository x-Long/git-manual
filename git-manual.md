# Git 使用记录

- **对实际项目开发中git的使用做一个简单的记录。**  
- **适用人群：对git有简单的使用基础，熟悉linux操作环境。**


## 零、三个重要区域
Git的所有操作实际上是在操作这三个区域的状态（或内容）
- HEAD  : **指向最近一次commit里的所有snapshot**
- Index 缓存区域 : **只有Index区域里的东西才可以被commit**
- Working Directory : **用户操作区域**
> note:我们在使用git status 或者诸如sourcetree等图形化工具时，一般会误解为，Index中的内容是空的，只有git add后才会有东西，实际上不是，**Index里一直是有东西的**。

## 一、git reset 

> **commit level：**
- reset --soft [commit]    
  + 只会修改HEAD的指向，INDEX和Working Directory不变，这个我曾经用git reset --soft HEAD~3 来替代git rebase -i 合并连续的commit
- reset --mixed [commit]  
  + --mixed是默认参数，工作区保持不变，HEAD和INDEX改变
- reset --hard [commit]
  + HEAD,INDEX,Working Directory 都会改变，用于回退版本
```
总结：reset(commit level)实际上有3个步骤，根据不同的参数可以决定执行到哪个步骤(--soft, --mixed, --hard)。
- 改变HEAD所指向的commit (--soft)
- 执行第1步，将Index区域更新为HEAD所指向的commit里包含的内容 (--mixed)
- 执行第1、2步，将Working Directory区域更新为HEAD所指向的commit里包含的内容 (--hard)
```

> **file level：**
- git reset --mixed [commit] file.txt  
  + 用于将某个[commit]中的文件恢复到暂存区，这个操作不会修改当前Working Directory  
  + --mixed 可以忽略, 并且不存在--soft 和 --hard

## 二、git checkout
> **commit level：**
- checkout [commit]
  + 生成一个detect branch，用来查看某次提交的运行情况
  
> **file level：**
- checkout [commit] file.txt 
  + 同时改变index区和Working Directory 
  + note：git reset [commit] file.txt 只改变index区，不影响Working Directory
  
## 三、git 分支
```
查看分支：git branch
查看所有分支(包括远程分支)：git branch -a 

创建分支：git branch <name>
切换分支：git checkout <name>或者
创建+切换分支：git checkout -b <name>

合并某分支到当前分支：git merge <name>

删除分支：git branch -d <name>
强制删除分支：git branch -D <name>
```

## 三、分支合并
假设我们现在位于master分支  
- 快速合并，只移动指针：git merge test
  + 会把test比master多出来的提交在master上重新提交一遍（如果可以的话）
- 非快速合并：git merge –no-ff test
  + 会在master分支生成一个单独的commit
- squash合并：git merge –squash test
  + 与-no-ff很像，区别是不会保留对test的引用
- rebase： git rebase test 保持干净的合并记录
  + 过程中如果发生冲突，修改冲突后，执行 git add file & git rebase --continue
  + 这个过程中可以随时执行git rebase --abort 取消rebase
- cherry-pick
  + 你想把那个节点merge过来就把那个节点merge过来，其合入的不是分支而是提交节点

## 四、个性化 git 配制文件

简化 git 命令，修改家目录下[.gitconfig]文件

```ini
[alias]
  ck = checkout
	mm = commit --amend
	cm = commit -m
	ca = commit -am
	ci = commit
	st = status
	br = branch
	df = diff
	rst = reset --hard
	mg = merge
	mt = mergetool
	# 使用命令 git lg 可以查看更简洁的历史记录
	lg = log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit

	# 该配置使得每次推送只需要 git push 即可，默认 push 当前 branch 
	# 如果没有该配置，推送代码一般要 git push origin master
[push]
	default = current
```

## 五、提高 bash 使用效率
在家目录创建如下文件 *.bash_profile* 

```ini
set repo_root=/path/to/repo

# command alias
alias gc="git checkout"

# 切换到主分支
alias gcm="git checkout master"

# 修改上一次提交信息
alias gamd="git commit --amned"

# (慎用)撤销当前所有修改
alias grst="git reset --hard HEAD"

alias gpl="git pull"
alias gps="git push"

# 一键切换到工作目录
alias q="cd $repo_root"
```

## 六、常用功能备忘
```
1. 修改当前提交：git commit --amend 

2. git stash & git satsh pop

3. 合并多个提交：git rebase -i 

4. git rm -r --cached file.txt: 
如果一个文件被commit 过后在修改.gitignore 文件会发现这个文件还是会被追踪，这个时候需要git rm -r --cached file.txt

5. 批量删除 untracked files：git clean -f 

6. 连untracked 的目录也一起删掉：git clean -fd 

7. 全局配置：
git config --global user.name "xlong"
git config --global user.email "liang_xaut@163.com"

8. git rebase -i HEAD~3 合并commit

9. git checkout -b dev lxl/dev

10. git pull 等于 git fetch 加 git merge
```

```
1. git revert [commit] 撤销某次提交的修改 并新建一个提交

2. 列出所有tag: git tag

3. 新建一个tag在当前commit: git tag [tag]

4. 新建一个tag在指定commit: git tag [tag] [commit]

5. 删除本地tag: git tag -d [tag]

6. 删除远程tag: git push origin :refs/tags/[tagName]

7. 查看tag信息: git show [tag]

8. 提交指定tag: git push [remote] [tag]

9. 提交所有tag: git push [remote] --tags

10. 新建一个分支，指向某个tag: git checkout -b [branch] [tag]
```

## 七、git 团队协作-git flow 工作流

> 此部分详细内容待续

**两个长期分支：**  
- 主分支master ：用于存放对外发布的版本，任何时候在这个分支拿到的，都是稳定的分布版；  
- 开发分支develop： 用于日常开发，存放最新的开发版。  

**三种短期分支：**
- 预发分支（release branch）：用于测试
- 功能分支（feature branch）
- 补丁分支（hotfix branch）
**一旦完成开发，它们就会被合并进develop或master，然后被删除。**

**Pull Request：**
- 需要进行code review ，然后在进行合并


