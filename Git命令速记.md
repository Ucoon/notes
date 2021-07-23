---
title: Git命令速记
---

- ```git reset --soft HEAD^```：撤销commit

- ```git reset  --hard HEAD^```：

- ```git fetch origin master```：从远程origin的master主干分支上获取到最新版本到本地origin/master分支上，不会merge

- ```git log -p master..origin/master```：比较本地的master分支和origin/master分支的区别

- ```git merge origin/master```：合并

- ```git pull origin master```：相当于进行了```git fetch```和```git merge```两步操作

- ```git branch -a```：查看远程分支

- ```git branch```：查看本地分支

- ```git checkout branchName```：将当前工作分支切换到branchName

- ```git checkout -b newBranch```：新建分支的同时切换分支：

  该命令相当于下面两条命令的执行结果：

  ```java
  git branch newBranch
  git checkout newBranch
  ```

- ```git branch -D branchName```：删除本地分支

- ```git push origin branchName```：分支上传

- ```git checkout -b newBranch origin/oldBranch```：在已有分支基础上新建分支

- ```git checkout -b newBranch oldTag```：在已有tag基础上新建分支

- 上游项目（以uprtream为例）新增分支（以production分支为例）的同步

  ```java
  git fetch upstream production:production：获取远程分支内容
  git checkout production：切换分支
  git push origin production：推送本地分支到origin
  git branch --set-upstream-to=origin/production production：关联本地production与远程production
  ```

- ```git remote set-url origin url```：切换远程仓库地址

- ```git remote add upstream url```：关联上游项目

- 回退单个文件

  ```java
  git log xxxx.file : 获取某个文件的提交记录
  git reset commitId xxxx.file：回退至某个commit状态
  ```

- tag 操作：

  ```java
  git tag 查看本地tag
  git tag -d v1.0.3 删除本地名为“v1.0.3”的Tag
  git push origin –-delete v1.0.3 删除远程名为“v1.0.3”的Tag
  git push origin :refs/tags/v1.0.3 也能删除远程名为“v1.0.3”的Tag
  ```

- 远程分支重命名

  ```java
  git branch -m oldName newName 重命名远程分支对应的本地分支
  git push --delete origin oldName 删除远程分支
  git push origin newName 上传新命名的本地分支
  git branch --set-upstream-to origin/newName 把修改后的本地分支与远程分支关联
  ```

- Git - .gitignore怎么忽略已经被版本控制的文件

  使用命令`git rm --cached filename`，然后将该文件写入`.gitignore`中即可。
  
- `git cherry-pick commitId` 合并特定commit到当前分支

**常见错误**

1. error: You have not concluded your merge (MERGE_HEAD exists)

```groovy
解决办法一：保留本地的更改，中止合并->重新合并->重新拉取
git merge --abort
git reset --merge
git pull

解决办法二：舍弃本地代码，远端版本覆盖本地版本
git fetch --all
git reset --hard origin/master
```

2. ```git pulll```更新代码，遇到如下问题：
   ```
   error: Your local changes to the following files would be overwritten by merge:
           xxxxx.json
   Please commit your changes or stash them before you merge.
   Aborting
   
   ```

   解决办法：

   ```
   1. git stash 
   2. git pull
   3. git stash pop
   ```

   ```git stash```：备份当前的工作区的内容，从最近的一次提交中读取相关内容，让工作区保证和上次提交的内容一致。同时，将当前的工作区内容保存到Git栈中

   ```git stash pop```： 从Git栈中读取最近一次保存的内容，恢复工作区的相关内容。由于可能存在多个Stash的内容，所以用栈来管理，pop会从最近的一个stash中读取内容并恢复。

   ```git stash list```： 显示Git栈内的所有备份，可以利用这个列表来决定从那个地方恢复。

   ```giy stash clear```： 清空Git栈

3. ```java
   Please enter a commit message to explain why this merge is necessary, # especially if it merges an updated upstream into a topic branch.
   ```

   解决办法：

   ```java
   1. press "i"
   2. write your merge message
   3. press "esc"
   4. write ":wq"
   5. then press enter
   ```

   



   