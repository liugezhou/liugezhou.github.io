---
layout: post
title: git 相关
author: liugezhou
date: 2021-08-08
updated: 2021-08-08
category: 
  前端小技
tags: 
  git
---

### 一、tag
#### 1.显示所有的tag
> git tag

#### 2. 查看某个版本系统的tag(过滤查看)
> git tag -l 'v1.0.*'

#### 3.列出仓库远程所有的分支
> git ls-remote --refs

#### 4. 创建标签
> - git tag -a v1.0.0 -m '内容'
> - git tag v1.0.0

#### 5.查看标签的详情
> git show v1.0.0

#### 6.推送标签
> - 推送单个分支tag：git push origin v1.0.0 
> - 推送本地所有tag：git push origin --tags

#### 7.删除标签
> 删除本地标签
> - git tag -d v1.0.0
> 
删除远程标签
> - git push origin :refs/tags/v1.0.0

#### 8.完整的打tag
> - git add *
> - git commit -m 'v1.0.0'
> - git tag v1.0.0
> - git push
> - git push origin v1.0.0

#### 9.给指定的commit号加tag
> 打tag不必要在head上打，也可以在之前的版本上打tag，需要知道某个提交对象的校验和(通过git log获取，取校验和的前几位数字即可)
> - git tag -a v1.0.3 9fedrf -m 'my tag'

### 二、stash

---

git stash的内容与branch无关
#### 2-1 git stash
> 保存当前工作进度，会把暂存区和工作区的改动保存起来，执行完这个命令后，执行git status会发现当前是一个干净的工作区，没有任何改动.
> 可以使用 **git stash save 'message'**添加一些注释
> 说明：stash是本地的，不会通过git push 命令上传到git server上。

#### 2-2 git stash list
> 显示保存进度的列表，也就是说  git stash可以多次执行。

#### 2-3 git stash pop
> - 恢复之前缓存的工作目录，且将缓存堆栈中的第一个stash删除
> - git stash apply会将缓存区的第一个stash应用到工作目录，而不会删除相应的stash存储。

#### 2-4 git stash drop [stash-id]
> 删除一个存储的进度。如果不指定stash_id,则默认删除最新的存储进度。

#### 2-5 git stash clear
> 删除所有存储的进度

#### 2-6 查看制定的stash diff
> 可以使用 git stash show stash@{0}查看stash的文件目录

#### 2-7 暂存未跟踪或忽略的文件
> git stash默认不会缓存在工作目录中的新文件、被呼噜的文件。
> git stash 命令提供了参数用户缓存上面两种类型的文件，使用 -u可以stash  untrakced文件。使用 -a可以stash所有的文件。


### 三、一些其它操作
#### 
#### 3-1 reset
> 有时候，我们在本地 git add && git commit -m '' 之后，我们想要撤回commit，这个时候可以使用
> **git reset --soft HEAD^**
> 这样就成功的撤回了commit ，且本地文件会保留不会删除

#### 3-2 hard
> 如果代码已经push到仓库，但是本地想要去回退到某个版本
> git reset --hard HEAD^ --- 回退到上一个版本
> git reset --hard <commitID> -- 回退到制定commit版本

#### 3-3 提交者和提交次数
> 一个项目想要看到一个项目的提交者和提交次数：git shortlog -sn

#### 3-4 git push -u
> 在使用git push提交代码的时候，本来应该是使用 git push origin main去提交到某分支的，如果不想每次都去写 origin main，那么就可以
> **git push -u origin main**
> 这样，下次提交的时候，直接git push就可以了



