### 常用命令清单

#### 参考文章
> https://github.com/guanguans/notes/blob/master/Git/%E5%B8%B8%E7%94%A8Git%E5%91%BD%E4%BB%A4%E6%B8%85%E5%8D%95.md
>https://www.ruanyifeng.com/blog/2015/12/git-cheat-sheet.html

#### git mv
```
# 重命名
$ git mv test.js test2.js


# 移动
$ git mv test.js ./new/path/test.js


# 改名文件，并且将这个改名放入暂存区
$ git mv [file-original] [file-renamed]


# 强制重命名或移动
# 这个文件已经存在，将要覆盖掉
$ git mv -f myFile existingFile
```
* * *

#### git fetch/git remote/git push/git pull
```
# 下载远程仓库的所有变动
$ git fetch [remote]


# 显示所有远程仓库
$ git remote -v


# 显示某个远程仓库的信息
$ git remote show [remote]


# 增加一个新的远程仓库，并命名
$ git remote add [shortname] [url]


# 查看远程服务器地址和仓库名称
$ git remote -v


# 添加远程仓库地址
$ git remote add origin git@ github:xxx/xxx.git


# 设置远程仓库地址(用于修改远程仓库地址)
$ git remote set-url origin git@ github.com:xxx/xxx.git


# 删除远程仓库
$ git remote rm <repository>


# 上传本地指定分支到远程仓库
# 把本地的分支更新到远端origin的master分支上
# git push <远端> <分支>
# git push 相当于 git push origin master
$ git push [remote] [branch]


# 强行推送当前分支到远程仓库，即使有冲突
$ git push [remote] --force


# 推送所有分支到远程仓库
$ git push [remote] —all


# 从远端origin的master分支更新版本库
# git pull <远端> <分支>
$ git pull origin master


# 抓取远程仓库所有分支更新并合并到本地，不要快进合并
$ git pull --no-ff
```

* * *



### git merge
```
# 将其他分支合并到当前分支
$ git merge branchName


# 在合并时创建一个新的合并后的提交
# 不要 Fast-Foward 合并，这样可以生成 merge 提交
$ git merge --no-ff branchName
```

* * *

### git log
```
# 显示所有提交
$ git log


# 显示某几条提交信息
$ git log -n 10


# 仅显示合并提交
$ git log --merges


# 查看该文件每次提交记录
$ git log <file>


# 查看每次详细修改内容的diff
$ git log -p <file>


# 查看最近两次详细修改内容的diff
$ git log -p -2


#查看提交统计信息
$ git log --stat
```

* * *

### git reset/git revert
```
# 恢复暂存区的指定文件到工作区
$ git checkout [file]


# 恢复某个commit的指定文件到暂存区和工作区
$ git checkout [commit] [file]


# 恢复暂存区的所有文件到工作区
$ git checkout .


# 重置暂存区的指定文件，与上一次commit保持一致，但工作区不变
$ git reset [file]


# 重置暂存区与工作区，与上一次commit保持一致
$ git reset --hard


# 重置当前分支的指针为指定commit，同时重置暂存区，但工作区不变
$ git reset [commit]


# 重置当前分支的HEAD为指定commit，同时重置暂存区和工作区，与指定commit一致
$ git reset --hard [commit]


# 重置当前HEAD为指定commit，但保持暂存区和工作区不变
$ git reset --keep [commit]


# 新建一个commit，用来撤销指定commit
# 后者的所有变化都将被前者抵消，并且应用到当前分支
$ git revert [commit]


# 恢复最后一次提交的状态
$ git revert HEAD


# 暂时将未提交的变化移除，稍后再移入
$ git stash
$ git stash pop


# 列所有stash
$ git stash list


# 恢复暂存的内容
$ git stash apply


# 删除暂存区
$ git stash drop
```

* * *

### git checkout
```
# 恢复暂存区的指定文件到工作区
$ git checkout [file]


# 恢复某个commit的指定文件到暂存区和工作区
$ git checkout [commit] [file]


# 恢复暂存区的所有文件到工作区
$ git checkout .

```
* * *

### git branch
```
# 查看所有的分支和远程分支
$ git branch -a


# 创建一个新的分支
$ git branch [branch-name]


# 重命名分支
# git branch -m <旧名称> <新名称>
$ git branch -m [branch-name] [new-branch-name]


# 编辑分支的介绍
$ git branch [branch-name] --edit-description


# 列出所有本地分支
$ git branch


# 列出所有远程分支
$ git branch -r


# 新建一个分支，但依然停留在当前分支
$ git branch [branch-name]


# 新建一个分支，并切换到该分支
$ git checkout -b [branch]


# 新建一个分支，指向指定commit
$ git branch [branch] [commit]


# 新建一个分支，与指定的远程分支建立追踪关系
$ git branch --track [branch] [remote-branch]


# 建立追踪关系，在现有分支与指定的远程分支之间
$ git branch --set-upstream [branch] [remote-branch]

# 删除分支
$ git branch -d [branch-name]


# 强制删除某个分支 (未被合并的分支被删除的时候需要强制)
$ git branch -D <branch>

```

* * *

### git init
```
# 在当前目录新建一个Git代码库
$ git init

# 新建一个目录，将其初始化为Git代码库
$ git init [project-name]

# 下载一个项目和它的整个代码历史
$ git clone [url]
```

* * *
### git commit
```
# 提交暂存区到仓库区
$ git commit -m [message]

# 提交暂存区的指定文件到仓库区
$ git commit [file1] [file2] ... -m [message]

# 提交工作区自上次commit之后的变化，直接到仓库区
$ git commit -a

# 提交时显示所有diff信息
$ git commit -v

# 使用一次新的commit，替代上一次提交
# 如果代码没有任何新变化，则用来改写上一次commit的提交信息
$ git commit --amend -m [message]

# 重做上一次commit，并包括指定文件的新变化
$ git commit --amend [file1] [file2] ...
```
* * *

### git diff
```
# 显示暂存区和工作区的差异
$ git diff

# 显示暂存区和上一个commit的差异
$ git diff --cached [file]

# 显示工作区与当前分支最新commit之间的差异
$ git diff HEAD

# 显示两次提交之间的差异
$ git diff [first-branch]...[second-branch]

# 显示今天你写了多少行代码
$ git diff --shortstat "@{0 day ago}"
```

### git add/git rm
```
# 添加指定文件到暂存区
$ git add [file1] [file2] ...

# 添加指定目录到暂存区，包括子目录
$ git add [dir]

# 添加当前目录的所有文件到暂存区
$ git add .

# 添加每个变化前，都会要求确认
# 对于同一个文件的多处变化，可以实现分次提交
$ git add -p

# 删除工作区文件，并且将这次删除放入暂存区
$ git rm [file1] [file2] ...

# 停止追踪指定文件，但该文件会保留在工作区
$ git rm --cached [file]

# 改名文件，并且将这个改名放入暂存区
$ git mv [file-original] [file-renamed]
```

### git config
```
1.显示当前的配置
git config —list

2.配置名称与邮箱
git config --global user.email "MyEmail@gmail.com”
git config --global user.name "My Name”

3.定义git命令的快捷方式：
git config --global alias.<alias-name> <git-command>

例如：git config --global alias.s status
 
那么执行git s，就等于执行了git status


```

