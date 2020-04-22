---
title: SVN转Git
categories: 工具
date: 2020-04-22
---
工作中的版本控制系统一直用的是SVN，习惯了SVN的工作流。但保持SVN的使用思路用Git非常的别扭。所以用Git模拟一些使用场景，了解Git的使用方法和思想。

尝试用Git完成以下内容：
1. 创建文件
2. 改动文件
3. 提交改动到暂存区
4. 撤销改动
5. 放弃暂存区的改动
6. 将改动提交到远程分支
7. 创建dev分支
8. 在dev分支上改动文件
9. 合并dev到master分支
10. 改动master分支的文件，使合并dev分支时，与master分支冲突
11. 创建远程dev分支
12. dev分支中的部分功能合并到master分支
13. 发布release版本

<!-- more -->

### 一、基础概念
#### Git与SVN的差异
Git是分布式版本控制系统，SVN是集中式式分布系统。

集中式版本控制系统的版本库是放在中央服务器的，干活的时候用的是自己电脑，需要先从中央服务器获得最新的版本，才能开始干活，当活干完了，再把工作内容推送给中央服务器。有一个很大的弊病，自己无法在本地创建分支，分支的信息必须要存在于中央服务器上。对工作量不大不小的工作，在中央服务器创建分支是不值当的，但是不创建分支又无法进行版本控制，非常糟心。
![l](https://raw.githubusercontent.com/silence394/PicBed/PicGO/l.jpg)

而分布式管理系统是不需要中央服务器的，本地就是一个完整的版本库。那么在不联网的时候，也能进行版本控制。相反，集中式版本控制是无法在不联网/本地环境下进行版本控制的。在分布式管理系统下，只需要把改动互相推送，就可以看到彼此的改动了。但是直接推送给别人还是不方便的。所以在分布式统计系统里也有一台充当中央服务器角色的机器，只不过它的作用仅仅是交换大家的修改，没有它一样能正常工作。
![git](https://raw.githubusercontent.com/silence394/PicBed/PicGO/git.jpg)
#### Git中的关键概念
Git的本地工作环境结构是这样的：
![0](https://raw.githubusercontent.com/silence394/PicBed/PicGO/0.jpg)
- 工作区就是本机的工作目录。
- 版本库中的stage，即暂存区，存放的是我们add进来的内容。
- 版本库中默认的master分支，存放的是我们commit的内容。
- HEAD表示当前版本，上一个版本是HEAD^ ，上上个版本是HEAD^^ ，再网上100个版本是100个^ ，可以写成HEAD~100。

接下来了解下Git分支的概念。
当只有默认创建的master分支的时候，master分支是一条线，master指向最新的提交，HEAD指向master，每次提交master分支都会向前移动一步。
![0](https://raw.githubusercontent.com/silence394/PicBed/PicGO/0.png)
当我们创建并切换新分支dev时，Git新建了一个dev的指针，指向master相同的提交，并把HEAD指针指向了dev，表示当前分支为dev。
![l](https://raw.githubusercontent.com/silence394/PicBed/PicGO/l.png)
在dev分支上提交内容，dev分支后向前移动，而master分支不变。
![l (1)](https://raw.githubusercontent.com/silence394/PicBed/PicGO/l%20(1).png)
这时候切回master分支，可以看到提交在dev分支上，master不变，HEAD指向master。
![0 (3)](https://raw.githubusercontent.com/silence394/PicBed/PicGO/0%20(3).png)
当dev分支的内容开发完毕，把dev合并到master分支上，Git直接把master指向了dev的当前提交，完成了合并。
![0 (1)](https://raw.githubusercontent.com/silence394/PicBed/PicGO/0%20(1).png)
当把dev分支删除后，就只剩下master分支了。
![0 (2)](https://raw.githubusercontent.com/silence394/PicBed/PicGO/0%20(2).png)
### 二、实际操作
#### 创建、修改、回退、提交主分支
初始化Git
```
git init
```
可以看到初始化后，默认是mater分支。

创建文件 c++.txt，内容:
```
11111111111111111
```
添加到暂存区。
```
git add c++.txt
```
提交到仓库
```
$ git commit -m "add c++.txt"
[master (root-commit) 27cb805] add c++.txt
 1 file changed, 1 insertion(+)
 create mode 100644 c++.txt
```
修改c++.txt，查看仓库状态
```
$ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        modified:   c++.txt
```
查看差异
```
$ git diff c++.txt
diff --git a/c++.txt b/c++.txt
index cc583c4..e7fce33 100644
--- a/c++.txt
+++ b/c++.txt
@@ -1 +1,2 @@
 11111111111111111
+22222222222222222
\ No newline at end of file

```
然后提交到仓库
```
$ git add c++.txt
$ git commit -m "add 222222222 to c++.txt"
[master 55c656f] add 222222222 to c++.txt
 1 file changed, 1 insertion(+)
```
在c++.txt中添加333333333，然后不想要这个改动，想把它撤销，需要分两种情况。
一种是没有添加到暂存区的。
```
$ git checkout -- c++.txt
```
可以看到被恢复到了版本库的状态。
```
$ cat c++.txt
11111111111111111
22222222222222222
```
另外一种是已经增加到暂存区里了，那么执行checkout之后仍是暂存区的状态
```
git checkout -- c++.txt

$ cat c++.txt
11111111111111111
22222222222222222
33333333333333333

```

这时候可以把它从暂存区里移出来，然后执行checkout即可。
```
$ git reset HEAD c++.txt
Unstaged changes after reset:
M       c++.txt
```
突然发现上次的提交有问题，需要回到之前的某个版本。先查看提交记录
``` 
$ git log
commit 4dec294ae575211436f41e2dac6d993be1f8af8e (HEAD -> master)
Author: silence394 <silence394@126.com>
Date:   Tue Apr 21 20:15:45 2020 +0800

    add 333333333333

commit 55c656fdad348c495c6a31e5c57d9bace8be41f4
Author: silence394 <silence394@126.com>
Date:   Tue Apr 21 20:00:16 2020 +0800

    add 222222222 to c++.txt

commit 27cb805b167c6edebed4d2a5de0fa9d0a4c1d19a
Author: silence394 <silence394@126.com>
Date:   Tue Apr 21 19:55:28 2020 +0800

    add c++.txt
```
这样不太方便看，执行下面的命令：
```
$ git log --pretty=oneline
4dec294ae575211436f41e2dac6d993be1f8af8e (HEAD -> master) add 333333333333
55c656fdad348c495c6a31e5c57d9bace8be41f4 add 222222222 to c++.txt
27cb805b167c6edebed4d2a5de0fa9d0a4c1d19a add c++.txt
```
可以看到我们现在处于
```
4dec294ae575211436f41e2dac6d993be1f8af8e (HEAD -> master) add 333333333333
```
这个版本中，想回退到上一个版本。
```
$ git reset --hard 55c6
HEAD is now at 55c656f add 222222222 to c++.txt
```
当然执行这个命令也可以达到同样的目的
```
$ git reset --hard HEAD^
HEAD is now at 55c656f add 222222222 to c++.txt
```
上一个版本是HEAD^ ，上上个版本是HEAD^^ ，往前的100个版本是HEAD~100.
最后发现弄错了，想要把它找回来，执行reflog可以看到之前的提交记录。
```
$ git reflog
55c656f (HEAD -> master) HEAD@{0}: reset: moving to 55c6
4dec294 HEAD@{1}: reset: moving to HEAD
4dec294 HEAD@{2}: commit: add 333333333333
55c656f (HEAD -> master) HEAD@{3}: commit: add 222222222 to c++.txt
27cb805 HEAD@{4}: commit (initial): add c++.txt
```
再执行reset命令，回到4dec294 HEAD@{2}: commit: add 333333333333这个版本。
```
$ git reset --hard 4dec
HEAD is now at 4dec294 add 333333333333
```
在dev上开发的功能已经完成了，现在要把本地库推送到github远程仓库中。先与github仓库关联：
```
git remote add origin https://github.com/silence394/LearnGit.git
```
由于在创建github仓库的时候添加了readme.md，所以在push之前，需要pull一下，并用命令“--allow-unrelated-histories”将本地和远程关联起来。
```
git pull origin master --allow-unrelated-histories
```
然后执行push命令，将本地库内容推送到远程分支：
```
$ git push origin master
fatal: HttpRequestException encountered.
   ▒▒▒▒▒▒▒▒ʱ▒▒▒▒
Username for 'https://github.com': silence394@126.com
libpng warning: iCCP: cHRM chunk does not match sRGB
Counting objects: 11, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (6/6), done.
Writing objects: 100% (11/11), 919 bytes | 459.00 KiB/s, done.
Total 11 (delta 0), reused 0 (delta 0)
To https://github.com/silence394/LearnGit.git
   224daaa..feaccb7  master -> master
```

#### 在分支上开发
现在要进行另外一个功能的开发，新建一个分支dev
```
$ git branch dev
```
切换到dev分支，
```
$ git checkout dev
Switched to branch 'dev'
```
创建python.txt，添加改动，并提交到dev本地仓库。
```
$ git add python.txt
$ git commit -m "add python.txt"
[dev 7ca9b59] add python.txt
 1 file changed, 1 insertion(+)
 create mode 100644 python.tx
```
把dev本地仓库推送到远程dev仓库。
```
$ git push origin dev
Enumerating objects: 4, done.
Counting objects: 100% (4/4), done.
Delta compression using up to 4 threads
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 305 bytes | 305.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0)
Everything up-to-date
```
查看所有分支，可以看到已经成功将dev分支推送到远程dev了。
```
$ git branch -a
* dev
  master
  remotes/origin/HEAD -> origin/master
  remotes/origin/dev
  remotes/origin/master
```
第二天到公司，本地没有dev分支，需要从远程库中拉取。不能直接在原有的master分支上pull，否则会把远程分支上的内容合并到本地master分支。应该先查看远程分支：
```
$ git branch -r
  origin/dev
  origin/master
```
创建并切换到origin/dev分支。
```
$ git checkout -t origin/dev
Switched to a new branch 'dev'
Branch 'dev' set up to track remote branch 'dev' from 'origin'.
```
接下来我们就可以在开发，我们把c++.txt末尾，添加一行，并提交到本地分支。

将本地分支切换到master，并在c++.txt末尾做一行不同的改动。把本地dev分支的内容合并到master分支。
```
$ git merge dev
error: Your local changes to the following files would be overwritten by merge:
        c++.txt
Please commit your changes or stash them before you merge.
Aborting
Updating feaccb7..cb7e364
```
报错显示需要先提交或者stash master分支的改动。那将其提交到master分支。然后进行合并。
```
$ git merge dev
Auto-merging c++.txt
CONFLICT (content): Merge conflict in c++.txt
Automatic merge failed; fix conflicts and then commit the result.
```
c++.txt中是这样的，
```
11111111111111111
22222222222222222
33333333333333333
<<<<<<< HEAD
44444444444444444
=======
55555555555555555
>>>>>>> dev
```
将冲突处理之后，查看状态得知需要将冲突处理完毕的文件提交。
```
$ git status
On branch master
You have unmerged paths.
  (fix conflicts and run "git commit")
  (use "git merge --abort" to abort the merge)

Changes to be committed:

        new file:   python.txt

Unmerged paths:
  (use "git add <file>..." to mark resolution)

        both modified:   c++.txt
```
将其提交到本地库，并推送到远程分支。
```
$ git commit -m "merge dev"
[master 88760f5] merge dev

$ git push origin master

Counting objects: 9, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (9/9), done.
Writing objects: 100% (9/9), 861 bytes | 861.00 KiB/s, done.
Total 9 (delta 1), reused 0 (delta 0)
remote: Resolving deltas: 100% (1/1), done.
To https://github.com/silence394/LearnGit.git
   feaccb7..88760f5  master -> master
```

现在切回到dev分支，将开发个新功能lua.txt。提交到本地dev分支和远程dev分支。对lua.txt改动三次，分别提交。
但是我们发现中间那次对lua的改动，是有问题的，需要将其撤销。
```
$ git revert 6615
error: could not revert 6615aa4... add lua 22222222222
hint: after resolving the conflicts, mark the corrected paths
hint: with 'git add <paths>' or 'git rm <paths>'
hint: and commit the result with 'git commit'
```
解决完冲突之后提交。

然后再创建c#.txt，提交到分支和远程分支。

现在dev分支的开发已经完毕，经团队确认后，只需要c#.txt，不需要lua.txt，也就是说我们需要把c#.txt的那次提交，单独merge到master分支。
```
$ git checkout master
Switched to branch 'master'

$ git cherry-pick ed00496c9410f9714462e3ade8186c2148d874cd
[master 3e38826] add c#
 Date: Wed Apr 22 10:09:43 2020 +0800
 1 file changed, 1 insertion(+)
 create mode 100644 C#.txt
```
然后将其推送到远程master分支。

到这个版本功能已经开发并测试完毕，可以发布了。对这个版本打个tag——release1.0，那样，就可以直接通过这个tag而不是一串字符串版本了。把tag推送到远程分支。
```
$ git tag release1.0

$ git push origin release1.0
Total 0 (delta 0), reused 0 (delta 0)
To https://github.com/silence394/LearnGit.git
 * [new tag]         release1.0 -> release1.0
```
### 参考
1. https://www.liaoxuefeng.com/wiki/896043488029600/898732792973664