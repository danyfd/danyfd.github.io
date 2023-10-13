---
title: Git笔记
date: 2023-10-13 18:35:26
tags: "Git"
categories: "工具"
---

### Git命令用法

#### git add和git stash

##### git stash

将未提交的修改（工作区和暂存区）保存至堆栈中用于后续恢复当前工作目录，故`stash`的内容可以恢复任意分支

###### git stash两种应用场景

- 改动同一分支

  本地修改后发现远程被改动造成冲突，此时无法`push`和`pull`，但是可以使用`git stash`

  ```
  git stash save "本地缓存内容标识"	//将本地改动暂存
  git pull		//拉取远端分支
  git stash pop 	//将栈顶改动内容重新加回本地分支，此时可以继续修改
  ```

- 不小心改动其它分支

  当正在`dev`上开发，此时项目中出现一个bug需要紧急修复，但是正在开发的内容只完成一半还不想提交，此时可以用`git stash`将修改的内容保存至堆栈区，然后切换到`fixbug`分支进行修复；

  或者本该在`dev`分支开发的内容却在`master`上进行了开发，需要重新切回`master`,用`git stash`将内容存至堆栈，切回`dev`后再次恢复内容即可，如

  ```
  git stash save "本地缓存内容标识"	//将本地内容暂存，此时master分支恢复到上次拉取时状态
  git checkout dev		//切换到需要改动的分支
  git stash pop			//将改动pop到当前的分支
  ```

###### git stash用法

- `git stash save “save message”`：执行存储时，添加备注，方便查找
- `git stash list`：查看`stash`了哪些存储
- `git stash show`：显示做了哪些改动，默认show第一个存储，若要显示其它存储，后面加`satsh@{$num}`，比如第二个 `git stash show stash@{1}`
- `git stash apply`：取出栈顶内容，不会把存储从存储列表中删除，默认第一个存储，即`stash@{0}`，若要使用第二个：`git stash apply stash@{1}`
- `git stash pop`：取出栈顶内容并删除，默认第一个 `stash`即 `stash@{0}`，若要应用并删除第二个：`git stash pop stash@{1}`。
- `git stash drop stash@{$num}`：丢弃`stash@{$num}`存储，列表中删除
- `git stash clear`：删除所有缓存的`satsh`

##### git add和git stash的关系

`git add`只是将文件加到版本控制中，`git stash`能正确存储的前提是文件必须在版本控制中才行；常规`git stash`的一个限制是会暂存所有文件，有时，只想备份某些文件的方法如下：

1. add`不想备份的文件
2. 调用`git stash --keep-index`只会备份没有被add的文件
3. 调用`git reset`取消已经`add`的文件的备份，继续自己的工作

#### git revert 和 git reset 版本回退

##### git revert

创建一个新的版本，该版本内容与要回退的目标版本一致，HEAD指针指向新生成的版本，而不是目标版本

- `git revert HEAD`撤销前一次commit
- `git revert HEAD^`撤销前前一次commit
- `git revert CommitVersionNum`撤销指定的版本

##### git reset

用于回退版本，可以指定退回某一次提交的版本

```
git reset [--soft | --mixed | --hard] [HEAD]
```

**--mixed** 为默认，可以不用带该参数，用于重置暂存区的文件与上一次的提交(commit)保持一致，工作区文件内容保持不变

```
$ git reset HEAD^            # 回退所有内容到上一个版本  
$ git reset HEAD^ hello.php  # 回退 hello.php 文件的版本到上一个版本 
$ git  reset  052e           # 回退到指定版本
```

**--soft** 参数用于回退到某个版本：

```
$ git reset --soft HEAD~3   # 回退上上上一个版本 
```

**--hard** 参数撤销工作区中所有未提交的修改内容，将暂存区与工作区都回到上一次版本，并删除之前的所有信息提交：

```
$ git reset --hard HEAD~3  # 回退上上上一个版本  
$ git reset –hard bae128  # 回退到某个版本回退点之前的所有信息。 
$ git reset --hard origin/master    # 将本地的状态回退到和远程的一样 
```

谨慎使用 **–-hard** 参数，它会删除回退点之前的所有信息

##### git revert和git reset的区别

- `git revert`是用一次新的`commit`回滚之前的`commit`，`git reset`是直接删除指定的`commit`，操作后HEAD指针的指向不同
- 在回滚操作上看，效果差不多，但在日后继续`merge`以前的老版本时有区别，`git revert`是用一次逆向的`commit`“中和”，因此合并老`branch`时导致这部分改变不会再次出现，但`git reset`是把某些`commit`在某个`branch`上删除，和老`branch`再次`merge`时，这些被回滚的`commit`应该还会被引入

#### git remote

```
git remote -v			#显示所有远程仓库
git remote rm name		#删除远程仓库
git remote rename old_name new_name		#修改仓库名
```

#### git branch

```
git branch			#列举所有分支，与git branch --list同义
git branch <branch>	#创建一个名为branch的分支，不自动检出新创建的分支
git branch -d <branch>	#删除指定分支，当分支中有未合并的变更时，git会阻止这一次的删除操作
git branch -D <branch>	#强制删除指定分支，即便其中含有未合并的变更，该命令常见于当开发者希望永久删除某一开发过程中的所有commit
git branch -m <branch>	#将当前分支重命名为<branch>
git branch -a			#列举所有远程分支
```

#### git checkout

```
git checkout branchname （切换到本地分支）	
git checkout -b 本地分支名 origin/远程分支名	//切换远程分支（需要将远程分支与本地分支关联）
//该命令可以将远程仓库指定的分支拉到本地，并在本地创建一个分支与指定远程仓库分支关联起来，并切换到新建的本地分支中

//放弃所有工作区的修改
git checkout .
git checkout – filename		//放弃对指定文件的修改
git checkout -f				//放弃工作区和暂存区的所有修改
```

#### git merge

- 开发分支（dev）上的代码达到上线标准后，合并到master分支

  ```
  git checkout dev
  git pull 
  git checkout master
  git pull
  
  # merge --no-ff参数，表示Fast forward；可以保存之前的分支历史。能够更好地查看merge历史以及branch分支状态；保证版本提交、分支结构清晰
  
  git merge --no-ff dev
  git push -u origin master
  ```

  当`master` 分支为保护分支时，执行`git push -u origin master`会提示远程服务器拒绝，此时需要在浏览器进行远程仓库`merge`操作

- 当master代码改动，需要更新开发分支（dev）上的代码

  ```
  git checkout master 
  git pull 
  git checkout dev
  
  # merge --no-ff参数，表示禁用Fast forward;可以保存之前的分支历史能够更好的查看merge历史，以及branch状态;保证版本提交、分支结构清晰
  
  git merge --no-ff  master
  git push -u origin dev
  ```

#### Git命令图解

![](https://img-blog.csdnimg.cn/2020010917233675.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9zaHE1Nzg1LmJsb2cuY3Nkbi5uZXQ=,size_16,color_FFFFFF,t_70)

##### 删除远程仓库文件及文件夹

在`github`上只能删除仓库，无法删除文件夹或文件，只能通过命令解决

```
$ git pull origin master			# 将远程仓库的项目拉下来
$ dir								# 查看有哪些文件夹
$ git rm -r --cached .idea			# 删除.idea文件夹
$ git commit -m '删除.idea'			# 提交，添加操作说明
$ git push -u origin master			# 将本次更改更新到github上
```

### 常见操作

#### 分支操作

##### 查看分支

```
git branch -a 	# 查看所有分支：本地分支白色，当前分支绿色，远程分支红色
```

##### 合并分支

###### 进入合并分支

`.git`同文件夹下进入`git bash`，进入要合并的分支（如develop分支合并到release，进入release目录）

```
git checkout release 		# 切换分支
git pull					# 拉取最新的代码
```

###### 合并分支

```
git merge develop	# develop在上一步是白色的，不建议直接合并远程分支
```

###### 查看状态

```
git status			# 查看是否有冲突
```

###### 解决冲突

在编辑器中解决冲突后`git add`提交至暂存区，之后`git commit`提交

```
git commit -m "说点什么"（-m后面的是本次提交的信息，如果不填，直接使用git commit,系统会自动生成）
```

###### 推送

```
git push			# 已提交的变动推送至远程
```

##### 命令集

###### 刷新分支

```
git remote update origin --prune
```

###### 查看所有分支

```
git branch -a
```

###### 查看远程分支

```
git branch -r
```

###### 查看本地分支所关联的远程分支

```
git branch -vv
```

###### 修改本地分支名称

```
git branch -m old_branch new_branch
```

###### 删除远程旧分支

```
git push origin :old_branch
```

###### 将新分支推送到远程仓库

```
git push --set-upstream origin new_branch
或者	git push -u origin new_branch 
```

###### 分支切换、合并

```
git checkout --track origin/dev		切换到远程dev分支
git branch -D master develop		删除本地库develop
git checkout -b dev					建立一个新的本地分支dev
git merge origin/dev				将dev与当前分支合并
git checkout dev					切换到本地dev分支
```

##### 命令图

![](https://img-blog.csdnimg.cn/f4d4bbb3de864cb882e9cf9faff30bca.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBATm8gU2lsdmVyIEJ1bGxldA==,size_20,color_FFFFFF,t_70,g_se,x_16)

#### 提交指定文件

- 应用场景：本地代码未完成需要紧急修复bug，某些开发阶段代码不想提交

  ```
  git status -s		# 查看仓库状态
  git add 文件名			# 添加需要提交的文件名
  git stash -u -k		# 提交时不提交未被add的文件
  git commit -m		# 提交备注信息
  git pull			# 拉取合并
  git push			# 推送到远程仓库
  git stash pop		# 恢复之前忽略的文件
  ```

#### 移动文件

当需要重命名文件时，可以使用`git mv [oldFileName] [new FileName]`，`Git`对于重命名操作分为三步进行，第一步首先重命名文件，然后再从仓库中删除原有文件，最后将新文件添加进暂存区等待提交

```
$ mv README.md LOOKME.md
$ git rm README.md
$ git add LOOKME.md
```

若通过软件进行批量修改文件时，也要按照该流程先删除原文件再添加新文件

#### 查看操作历史

使用`git log`打印所有参与者的提交记录

- `-p --patch`显示每次提交所引入的差异，也可以限制显示条目数量，如：

  ```
  git log -p --patch -2
  ```

  ![](https://img-blog.csdnimg.cn/2021072010075819.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3N1bmh1YXFpYW5nMQ==,size_16,color_FFFFFF,t_70)

- `--stat`：在每次提交的下面列出所有被修改的文件、有多少文件被修改了以及被修改的文件哪些行被移除或是添加了。在每次提交的最后还有一个总结

  ![](https://img-blog.csdnimg.cn/20210720100637195.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3N1bmh1YXFpYW5nMQ==,size_16,color_FFFFFF,t_70)

##### 将日志信息显示在一行上

```
$ git log --pretty=oneline
```

##### 以short格式输出仓库修改信息

```
$ git log --pretty=short
```

##### 以 full 格式输出仓库修改信息

```
$ git log --pretty=full
```

##### 以 fuller 格式输出仓库修改信息

```
$ git log --pretty=fuller
```

修改文件与提交文件可以不是同一个人，故在查询日志时会区分修改人与提交人

#### 修改分支名称

##### 修改本地分支名称

```
git branch -m oldBranchName newBranchName
```

##### 将本地分支的远程分支删除

```
git push origin :oldBranchName
```

##### 将改名后的本地分支推送到远程，并将本地分支与之关联

```
git push --set-upstream origin newBranchName
```

### 其它

#### 文件颜色

- 红褐色：创建之后没有`add`，没提交，不在版本控制范围内，需要先`add`文件
- 绿色：`add`之后文件为绿色，没有提交
- 蓝色：原本有一个文件，改动之后没有提交为蓝色

#### git分支管理策略

##### 企业项目一般分支策略

###### 主分支master

代码库应有且仅有一个主分支。所有给用户使用的正式版本都在该主分支上发布

###### 开发分支develop

主分支只用于发布重大版本。日常开发应在开发分支上生成代码最新版本

###### 功能分支feature

为了开发某种特定功能，从`develop`上分出来的，开发完成后再次并入，功能分支的名字，可以采用`feature-*`的形式命名

###### 预发布分支release

在合并到`master`分支前，可能需要一个预发布版本进行测试，预发布分支从`develop上分出来`，预发布结束后必须合并进`develop`和`master`分支，它的命名可以采用`release-*`的形式

###### bug分支fixbug

该分支用于bug修补，从`master`分支上分出来，修补结束以后，再合并进`Master`和`Develop`分支，它的命名可以采用`fixbug-*`的形式

###### 其它分支other

#### 仓库文件状态

##### 工作区

编写代码文件、管理资源文件的区域，细分为受版本控制和不受版本控制的文件

##### 暂存区

该区域和`index`文件取得了联系，实际执行`git add`的文件都生成了对应的object对象，放在`.git/objects`目录下，状态变为`staged`，当提交到版本库时，分支会引用这些对象

##### 版本库

最终的修改提交到版本库，此时提交的文件状态变成`committed`，其实也是一种`unmodified`状态，版本库会记录每一次提交，可以追溯每次修改的内容

文件状态通常可以分为：

- 不受版本控制的`untracked`状态
- 受版本控制且已修改的`modified`状态
- 受版本控制已修改并提交到暂存区的 `staged` 状态
- 从暂存区已经提交到本地仓库的 `committed` 状态
- 提交到本地仓库未修改或者从远程仓库克隆下来的 `unmodified` 状态

```
·Changes to be committed为暂存区已存在，需要进行提交进仓库的文件；
·Changes not staged for commit为文件被操作尚未提交至暂存区的文件，此类文件需要使用add将其添加至缓存区再提交进仓库；
·Untracked files为未入暂存区文件；
```

```
git status 只能查看未传送提交的次数，不能查看具体文件信息
git cherry -v 只能查看未传送提交的描述/说明
git log master ^origin/master 则可以查看未传送提交的详细信息
```