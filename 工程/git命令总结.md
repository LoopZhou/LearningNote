## Git命令总结

### 普通工作流程

```sh
# 拉取代码
git clone <远程仓库的网址>
# 当前项目配置调整
git config --local user.name  "xxxx"
git config --local user.email "xxxx@xxx"

# 从远程仓库获取最新版本。
$ git pull
# 开发分支
git checkout -b develop
# 把指定的文件添加到暂存区中
git add .
# 查看本地仓库的状态
$ git status
# 把暂存区中的文件提交到本地仓库中并添加描述信息
git commit -m "<提交的描述信息>"
# 修改上次提交的描述信息
git commit --amend
# 把本地仓库的分支推送到远程仓库
git push --set-upstream origin develop

# 把指定的分支合并到当前所在的分支下，并自动进行新的提交
$ git merge develop
```

### 配置git短命令

```sh
# 方式1
git config --global alias.ps push

# 方式2
# 打开全局配置文件
vim ~/.gitconfig

# 写入内容
[alias] 
        co = checkout
        ps = push
        pl = pull
        mer = merge --no-ff
        cp = cherry-pick
```

### 多仓库push

```sh
# 查看
git remote -v

origin  git@github.com:LoopZhou/LearningNote.git (fetch)
origin  git@github.com:LoopZhou/LearningNote.git (push)

# 添加新仓库origin_repo
git remote add origin_repo 'git@github.com:xxxx.git'

# push到origin_repo仓库
git push origin_repo develop

# 从origin_repo仓库拉取分支
git fetch origin_repo fix/packages
```

### stash

```sh
# 保存当前未commit的代码
git stash

# 保存当前未commit的代码并添加备注
git stash save "备注的内容"

# 列出stash的所有记录
git stash list

# 删除stash的所有记录
git stash clear

# 应用最近一次的stash
git stash apply

# 应用第二条记录
git stash apply stash@{1}

# 应用最近一次的stash，随后删除该记录
git stash pop

# 第二条记录
git stash pop stash@{1}

# 删除最近的一次stash
git stash drop

# 第二条记录
git stash drop stash@{1}
```

### reset

回退你已提交的 commit, 并将 commit 的修改内容放回到暂存区

```sh
# 重置暂存区，但文件不受影响
# 相当于将用 "git add" 命令更新到暂存区的内容撤出暂存区，可以指定文件
# 没有指定 commit ID 则默认为当前 HEAD
$ git reset [<文件路径>]
$ git reset --mixed [<文件路径>]

# 将 HEAD 的指向改变，撤销到指定的提交记录，文件未修改
$ git reset <commit ID>
$ git reset --mixed <commit ID>

# 将 HEAD 的指向改变，撤销到指定的提交记录，文件未修改
# 相当于调用 "git reset --mixed" 命令后又做了一次 "git add"
$ git reset --soft <commit ID>

# 将 HEAD 的指向改变，撤销到指定的提交记录，文件也修改了
$ git reset --hard <commit ID>
```

### cherry-pick 

```sh
# 转移提交
git cherry-pick commit1 

# 一次转移多个提交
git cherry-pick commit1 commit2

# 多个连续的commit，也可区间复制
git cherry-pick commit1^..commit2

# 解决代码冲突，让 cherry-pick 继续进行下去
cherry-pick --continue

# 放弃 cherry-pick, 回到操作前的样子，就像什么都没发生过
git cherry-pick --abort

#退出 cherry-pick, 不回到操作前的样子。即保留已经 cherry-pick 成功的 commit
git cherry-pick --quit
```
