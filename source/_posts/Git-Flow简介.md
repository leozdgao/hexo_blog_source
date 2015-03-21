title: "Git Flow简介"
date: 2015-02-28 11:13:19
tags: git
categories:
---

摘抄自：[原文](http://nvie.com/posts/a-successful-git-branching-model/)，[译文](http://www.juvenxu.com/2010/11/28/a-successful-git-branching-model/)

![git flow](http://7sbm5t.com1.z0.glb.clouddn.com/gitflow_0.png)

<!--more-->

####主要分支

- develop：反映了开发过程中最新的提交变更。
- master：当develop分支的源码达到一个稳定的状态时，develop上的变更都应该以某种方式合并回master分支，每一个commit对应一个版本，并使用发布版本号打上标签。

####特性分支

可能的分支来源：develop
必须合并回：develop
分支命名约定：任何除master，develop，release-\*，或hotfix-\*以外的名称

特性分支用来开发下个版本的新特性，最终会被合并回develop分支，往往只存在于开发者的仓库中，而不会出现在origin。

```
> git checkout develop
Switch to branch ‘develop’
> git merge -no-ff myfeature
Updating ea1b82a..05e9557
(Summary of changes)
> git branch -d myfeature
Deleted branch myfeature (was 05e9557).
> git push origin develop
```

####发布分支

可能的分支来源：develop
必须合并回：develop和master
分支命名约定：release-*

发布分支为准备新的产品版本发布做支持，它允许你在最后时刻检查所有的细节。在分布分支创建的时候，对应的版本发布才获得一个版本号。

**创建一个发布分支**

发布分支从develop分支创建，develop分支的状态已经是准备好“下一版本”发布了。

```
> git checkout -b releases-1.2 develop
Switched to a new branch “release-1.2”
> ./bump-version.sh 1.2
Files modified successfully. version bumped to 1.2.
> git commit -a -m “Bumped version number to 1.2”
[release-1.2 74d9424] Bumped version number to 1.2
1 files changed. 1 insertions(+). 1 deletions(-)
```

**结束一个发布分支**

当发布分支达到一个可以正式发布的状态时，将发布分支合并到master，接着master分支上的commit必须被打上标签（tag）。

```
> git checkout master
Switched to branch ‘master’
> git merge –no-ff release-1.2
Merge made by recursive.
(Summary of changes)
> git tag -a 1.2
```

为了能保留发布分支上的变更，我们还需要将分支合并回develop。

```
> git checkout develop
Switched to branch ‘develop’
> git merge –no-ff release-1.2
Merge made by recursive.
(Summary of changes)
```

现在工作才算真正完成了，最后一步是删除发布分支。

```
> git branch -d release-1.2
Deleted branch release-1.2 (was ff452fe).
```

####热补丁分支

可能的分支来源：master
必须合并回：develop和master
分支命名约定：hotfix-*

当产品版本发现未预期的问题时，需要着手处理，这时就需要热补丁分支。

**创建一个热补丁分支**

热补丁分支从master分支创建。不要忘记在创建热补丁分支后设定一个新的版本号。

```
> git checkout -b hotfix-1.2.1 master
Switched to a new branch “hotfix-1.2.1″
> ./bump-version.sh 1.2.1
Files modified successfully, version bumped to 1.2.1.
> git commit -a -m “Bumped version number to 1.2.1″
[hotfix-1.2.1 41e61bb] Bumped version number to 1.2.1
1 files changed, 1 insertions(+), 1 deletions(-)
```

修复bug并提交。

```
> git commit -m “Fixed severe production problem”
[hotfix-1.2.1 abbe5d6] Fixed severe production problem
5 files changed, 32 insertions(+), 17 deletions(-)
```

**结束一个热补丁分支**

修复完成后，热补丁分支需要合并回master并打上标签，同时也需要被合并回develop。

```
> git checkout develop
Switched to branch ‘develop’
> git merge –no-ff hotfix-1.2.1
Merge made by recursive.
(Summary of changes)
```

*注：如果这个时候有发布分支存在，热补丁分支的变更则应该合并至发布分支，而不是develop。*

最后删除临时的热补丁分支：

```
> git branch -d hotfix-1.2.1
Deleted branch hotfix-1.2.1 (was abbe5d6).
```
