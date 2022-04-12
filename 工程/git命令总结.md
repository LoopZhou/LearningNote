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