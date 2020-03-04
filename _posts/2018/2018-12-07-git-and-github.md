---
layout:     post
title:      "2018-12-07 Git and github notes"
subtitle:   "git notes"
date:       2018-12-07
author:     "msg"
header-img: "img/posts/02.jpg"
header-mask: 0.3
catalog:    true

tags:
    - git
    - github
    - 学习
    - 转载
---

> 学习自[廖雪峰Git教程](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)、[git官网](https://git-scm.com/doc)

### 1、在Linux上安装Git(ubuntu为例)

```bash
$ sudo apt-get install git
```

但是这样下载的可能是比较老的版本，如果需要下载更新最新版本的git，需要如下操作：

```bash
$ sudo add-apt-repository ppa:git-core/ppa
$ sudo apt-get update
$ sudo apt-get upgrade
```

### 2、创建版本库

首先，选择一个合适的地方，创建一个空目录：

```bash
$ mkdir learngit
$ cd learngit
$ pwd
/home/msg/learngit
```

第二步，通过`git init`命令把这个目录变成Git可以管理的仓库：

```
$ git init
Initialized empty Git repository in /home/msg/learngit/.git/
```

### 3、把文件添加到版本库

第一步，用命令`git add`告诉Git，把文件添加到仓库：

```bash
$ git add readme.txt
```

第二步，用命令`git commit`告诉Git，把文件提交到仓库：

```bash
$ git commit -m "wrote a readme file"
```

### 4、修改文件

修改`readme.txt`文件之后，用`git diff`命令看看修改的内容：

```bash
$ git diff readme.txt 
```

再次用`git commit`提交：

```bash
$ git commit -m "add distributed"
```

#### 5、版本回退

在Git中，我们用`git log`命令查看：

```bash
$ git log
```

`git log`命令显示从最近到最远的提交日志

如果嫌输出信息太多，看得眼花缭乱的，可以试试加上`--pretty=oneline`参数：

```bash
$ git log --pretty=oneline
```

要把当前版本回退到上一个版本`add distributed`，就可以使用`git reset`命令：

```bash
$ git reset --hard HEAD^
```

最新的版本已经看不到了，想再回去，只要上面的命令行窗口还没有被关掉，就可以找到那个版本的`commit id`是`1094adb...`，于是就可以指定回到未来的某个版本：

```bash
$ git reset --hard 1094a
HEAD is now at 83b0afe append GPL
```

当用`$ git reset --hard HEAD^`回退到上一版本并关闭了`terminal`时，再想恢复，就必须找到`commit id`。Git提供了一个命令`git reflog`用来记录你的每一次命令：

```bash
$ git reflog
e475afc HEAD@{1}: reset: moving to HEAD^
1094adb (HEAD -> master) HEAD@{2}: commit: append GPL
```

可知，`上一个版本`的commit id是`1094adb`。

### 6、工作区和暂存区

#### 工作区（Working Directory）

就是电脑里能看到的目录，比如`learngit`文件夹

#### 版本库（Repository）

工作区有一个隐藏目录`.git`，这个不算工作区，而是Git的版本库。

Git的版本库里存了很多东西，其中最重要的就是称为stage（或者叫index）的暂存区，还有Git为我们自动创建的第一个分支`master`，以及指向`master`的一个指针叫`HEAD`。

创建Git版本库时，Git自动为我们创建了唯一一个`master`分支，所以，现在，`git commit`就是往`master`分支上提交更改。

### 7、撤销修改

`git checkout -- file`可以丢弃工作区的修改：

```
$ git checkout -- readme.txt
```

命令`git checkout -- readme.txt`意思就是，把`readme.txt`文件在工作区的修改全部撤销，这里有两种情况：

* `readme.txt`自修改后还没有被放到暂存区，撤销修改就回到和版本库一模一样的状态；

* `readme.txt`已经添加到暂存区后，又作了修改，撤销修改就回到添加到暂存区后的状态。

总之，就是让这个文件回到最近一次`git commit`或`git add`时的状态。

命令`git reset HEAD <file>`可以把暂存区的修改撤销掉（unstage），重新放回工作区：

```
$ git reset HEAD readme.txt
```

### 8、删除文件

直接在文件管理器中把没用的文件删了，或者用`rm`命令删了：

```
$ rm test.txt
```

Git知道你删除了文件，工作区和版本库就不一致了，`git status`命令会立刻告诉你哪些文件被删除了：

```
$ git status
```

用命令`git rm`删掉，并且`git commit`：

```
$ git rm test.txt
$ git commit -m "remove test.txt"
```

删错了，版本库里还有，可以很轻松地把误删的文件恢复到最新版本：

```
$ git checkout -- test.txt
```

`git checkout`其实是用版本库里的版本替换工作区的版本，无论工作区是修改还是删除，都可以“一键还原”。

### 9、添加远程库

首先，登陆GitHub，然后，在右上角找到“Create a new repo”按钮，创建一个新的仓库

在Repository name填入`learngit`，其他保持默认设置，点击“Create repository”按钮，就成功地创建了一个新的Git仓库

根据GitHub的提示，在本地的`learngit`仓库下运行命令：

```bash
$ git remote add origin git@github.com:XXX/learngit.git
```

下一步，就可以把本地库的所有内容推送到远程库上：

```bash
$ git push -u origin master
```

把本地库的内容推送到远程，用`git push`命令，实际上是把当前分支`master`推送到远程。

由于远程库是空的，第一次推送`master`分支时，加上了`-u`参数，Git不但会把本地的`master`分支内容推送到远程新的`master`分支，还会把本地的`master`分支和远程的`master`分支关联起来，在以后的推送或者拉取时就可以简化命令。

从现在起，只要本地作了提交，就可以通过命令：

```
$ git push origin master
```

#### SSH警告

当你第一次使用Git的`clone`或者`push`命令连接GitHub时，会得到一个警告：

```
The authenticity of host 'github.com (xx.xx.xx.xx)' can't be established.
RSA key fingerprint is xx.xx.xx.xx.xx.
Are you sure you want to continue connecting (yes/no)?
```

这是因为Git使用SSH连接，而SSH连接在第一次验证GitHub服务器的Key时，需要你确认GitHub的Key的指纹信息是否真的来自GitHub的服务器，输入`yes`回车即可。

Git会输出一个警告，告诉你已经把GitHub的Key添加到本机的一个信任列表里了：

```
Warning: Permanently added 'github.com' (RSA) to the list of known hosts.
```

这个警告只会出现一次，后面的操作就不会有任何警告了。

### 10、从远程库克隆

用命令`git clone`克隆一个本地库：

```
$ git clone git@github.com:XXX/gitskills.git
```

`https`除了速度慢以外，还有个最大的麻烦是每次推送都必须输入口令，但是在某些只开放http端口的公司内部就无法使用`ssh`协议而只能用`https`。

### 11、创建与合并分支

Git把每次提交串成一条时间线，这条时间线就是一个分支。一开始只有一条时间线，在Git里，这个分支叫主分支，即`master`分支。`HEAD`严格来说不是指向提交，而是指向`master`，`master`才是指向提交的，所以，`HEAD`指向的就是当前分支。

一开始的时候，`master`分支是一条线，Git用`master`指向最新的提交，再用`HEAD`指向`master`，就能确定当前分支，以及当前分支的提交点。

如果创建新的分支，例如`dev`，Git新建了一个指针叫`dev`，指向`master`相同的提交，再把`HEAD`指向`dev`，就表示当前分支在`dev`上：

从现在开始，对工作区的修改和提交就是针对`dev`分支了，比如新提交一次后，`dev`指针往前移动一步，而`master`指针不变。

假如我们在`dev`上的工作完成了，就可以把`dev`合并到`master`上。Git怎么合并呢？最简单的方法，就是直接把`master`指向`dev`的当前提交，就完成了合并。

合并完分支后，甚至可以删除`dev`分支。删除`dev`分支就是把`dev`指针给删掉，删掉后，我们就剩下了一条`master`分支。

### 实战

#### 创建`dev`分支，然后切换到`dev`分支：

```
$ git checkout -b dev
```

`git checkout`命令加上`-b`参数表示创建并切换，相当于以下两条命令：

```
$ git branch dev
$ git checkout dev
```

#### 用`git branch`命令查看当前分支：

```
$ git branch
```

`git branch`命令会列出所有分支，当前分支前面会标一个`*`号。

然后，可以在`dev`分支上正常提交，比如对readme.txt做个修改，加上一行：

```
Creating a new branch is quick.
```

然后提交：

```
$ git add readme.txt 
$ git commit -m "branch test"
```

#### `dev`分支的工作完成，我们就可以切换回`master`分支：

```
$ git checkout master
```

#### 把`dev`分支的工作成果合并到`master`分支上：

```
$ git merge dev
Updating d46f35e..b17d20e
Fast-forward
 readme.txt | 1 +
 1 file changed, 1 insertion(+)
```

#### `git merge`命令用于合并指定分支到当前分支。

`Fast-forward`就是直接把`master`指向`dev`的当前提交，所以合并速度非常快。

#### 合并完成后，就可以放心地删除`dev`分支了：

```
$ git branch -d dev
```

#### 删除后，查看`branch`，就只剩下`master`分支了：

```
$ git branch
```

#### 删除远程分支

```bash
git push origin --delete dev
```

### 12、解决冲突

准备新的`feature1`分支，继续新分支开发：

```
$ git checkout -b feature1
```

修改`readme.txt`最后一行，改为：

```
Creating a new branch is quick AND simple.
```

在`feature1`分支上提交：

```
$ git add readme.txt
$ git commit -m "AND simple"
```

切换到`master`分支：

```
$ git checkout master
```

Git还会自动提示我们当前`master`分支比远程的`master`分支要超前1个提交。

在`master`分支上把`readme.txt`文件的最后一行改为：

```
Creating a new branch is quick & simple.
```

提交：

```
$ git add readme.txt 
$ git commit -m "& simple"
```

现在，`master`分支和`feature1`分支各自都分别有新的提交。

这种情况下，Git无法执行“快速合并”，只能试图把各自的修改合并起来，但这种合并就可能会有冲突：

```
$ git merge feature1
```

`readme.txt`文件存在冲突，必须手动解决冲突后再提交。

```
$ git status
```

直接查看readme.txt的内容：

```
Git is a distributed version control system.
Git is free software distributed under the GPL.
Git has a mutable index called stage.
Git tracks changes of files.
<<<<<<< HEAD
Creating a new branch is quick & simple.
=======
Creating a new branch is quick AND simple.
>>>>>>> feature1
```

Git用`<<<<<<<`，`=======`，`>>>>>>>`标记出不同分支的内容，我们修改如下后保存：

```
Creating a new branch is quick and simple.
```

再提交：

```
$ git add readme.txt 
$ git commit -m "conflict fixed"
```

用带参数的`git log`也可以看到分支的合并情况：

```
$ git log --graph --pretty=oneline --abbrev-commit
```

最后，删除`feature1`分支：

```
$ git branch -d feature1
```

工作完成。

### 13、分支管理策略

通常，合并分支时，如果可能，Git会用`Fast forward`模式，但这种模式下，删除分支后，会丢掉分支信息。

如果要强制禁用`Fast forward`模式，Git就会在merge时生成一个新的commit，这样，从分支历史上就可以看出分支信息。

下面我们实战一下`--no-ff`方式的`git merge`：

首先，仍然创建并切换`dev`分支：

```
$ git checkout -b dev
Switched to a new branch 'dev'
```

修改readme.txt文件，并提交一个新的commit：

```
$ git add readme.txt 
$ git commit -m "add merge"
```

切换回`master`：

```
$ git checkout master
```

准备合并`dev`分支，请注意`--no-ff`参数，表示禁用`Fast forward`：

```
$ git merge --no-ff -m "merge with no-ff" dev
```

本次合并要创建一个新的commit，所以加上`-m`参数，把commit描述写进去。

合并后，我们用`git log`看看分支历史：

```
$ git log --graph --pretty=oneline --abbrev-commit
```

### 14、Bug分支

Git还提供了一个`stash`功能，可以把当前工作现场“储藏”起来，等以后恢复现场后继续工作：

```
$ git stash
Saved working directory and index state WIP on dev: f52c633 add merge
```

现在，用`git status`查看工作区，就是干净的（除非有没有被Git管理的文件），因此可以放心地创建分支来修复bug。

首先确定要在哪个分支上修复bug，假定需要在`master`分支上修复，就从`master`创建临时分支：

```
$ git checkout master
$ git checkout -b issue-101
```

现在修复bug，然后提交。

修复完成后，切换到`master`分支，并完成合并，最后删除`issue-101`分支：

```
$ git checkout master
$ git merge --no-ff -m "merged bug fix 101" issue-101
```

接着回到`dev`分支:

```
$ git checkout dev
$ git status
```

工作区是干净的，刚才的工作现场存到哪去了？用`git stash list`命令看看：

```
$ git stash list
stash@{0}: WIP on dev: f52c633 add merge
```

工作现场还在，Git把stash内容存在某个地方了，但是需要恢复一下，有两个办法：

一是用`git stash apply`恢复，但是恢复后，stash内容并不删除，你需要用`git stash drop`来删除；

另一种方式是用`git stash pop`，恢复的同时把stash内容也删了：

```
$ git stash pop
```

再用`git stash list`查看，就看不到任何stash内容了：

```
$ git stash list
```

你可以多次stash，恢复的时候，先用`git stash list`查看，然后恢复指定的stash，用命令：

```
$ git stash apply stash@{0}
```

### 15、Feature分支

每添加一个新功能，最好新建一个feature分支，在上面开发，完成后，合并，最后，删除该feature分支。

```bash
git checkout -b feature-vulcan
Switched to a new branch 'feature-vulcan'
```

5分钟后，开发完毕：

```
$ git add vulcan.py
$ git status
$ git commit -m "add feature vulcan"
```

切回`dev`，准备合并：

```
$ git checkout dev
```

就在此时，接到上级命令，因经费不足，新功能必须取消！

虽然白干了，但是这个包含机密资料的分支还是必须就地销毁：

```
$ git branch -d feature-vulcan
```

销毁失败。Git友情提醒，`feature-vulcan`分支还没有被合并，如果删除，将丢失掉修改，如果要强行删除，需要使用大写的`-D`参数。。

现在我们强行删除：

```
$ git branch -D feature-vulcan
Deleted branch feature-vulcan (was 287773e).
```

终于删除成功！

### 16、多人协作

当从远程仓库克隆时，实际上Git自动把本地的`master`分支和远程的`master`分支对应起来了，并且，远程仓库的默认名称是`origin`。

要查看远程库的信息，用`git remote`：

```
$ git remote
origin
```

或者，用`git remote -v`显示更详细的信息：

```
$ git remote -v
origin  git@github.com:michaelliao/learngit.git (fetch)
origin  git@github.com:michaelliao/learngit.git (push)
```

上面显示了可以抓取和推送的`origin`的地址。如果没有推送权限，就看不到push的地址。

#### 推送分支

推送分支，就是把该分支上的所有本地提交推送到远程库。推送时，要指定本地分支，这样，Git就会把该分支推送到远程库对应的远程分支上：

```
$ git push origin master
```

如果要推送其他分支，比如`dev`，就改成：

```
$ git push origin dev
```

但是，并不是一定要把本地分支往远程推送，那么，哪些分支需要推送，哪些不需要呢？

- `master`分支是主分支，因此要时刻与远程同步；
- `dev`分支是开发分支，团队所有成员都需要在上面工作，所以也需要与远程同步；
- `bug`分支只用于在本地修复bug，就没必要推到远程了，除非老板要看看你每周到底修复了几个bug；
- `feature`分支是否推到远程，取决于你是否和你的小伙伴合作在上面开发。

#### 抓取分支

多人协作时，大家都会往`master`和`dev`分支上推送各自的修改。

现在，模拟一个你的小伙伴，可以在另一台电脑（注意要把SSH Key添加到GitHub）或者同一台电脑的另一个目录下克隆：

```
$ git clone git@github.com:XXX/learngit.git
```

当你的小伙伴从远程库clone时，默认情况下，你的小伙伴只能看到本地的`master`分支。不信可以用`git branch`命令看看：

```
$ git branch
* master
```

现在，你的小伙伴要在`dev`分支上开发，就必须创建远程`origin`的`dev`分支到本地，于是他用这个命令创建本地`dev`分支：

```
$ git checkout -b dev origin/dev
```

现在，他就可以在`dev`上继续修改，然后，时不时地把`dev`分支`push`到远程：

```
$ git add env.txt
$ git commit -m "add env"
$ git push origin dev
```

你的小伙伴已经向`origin/dev`分支推送了他的提交，而碰巧你也对同样的文件作了修改，并试图推送：

```
$ cat env.txt
$ git add env.txt
$ git commit -m "add new env"
$ git push origin dev
```

推送失败，因为你的小伙伴的最新提交和你试图推送的提交有冲突，解决办法也很简单，Git已经提示我们，先用`git pull`把最新的提交从`origin/dev`抓下来，然后，在本地合并，解决冲突，再推送：

```
$ git pull
```

`git pull`也失败了，原因是没有指定本地`dev`分支与远程`origin/dev`分支的链接，根据提示，设置`dev`和`origin/dev`的链接：

```
$ git branch --set-upstream-to=origin/dev dev
Branch 'dev' set up to track remote branch 'dev' from 'origin'.
```

再pull：

```
$ git pull
```

这回`git pull`成功，但是合并有冲突，需要手动解决，解决的方法和分支管理中的[解决冲突](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/001375840202368c74be33fbd884e71b570f2cc3c0d1dcf000)完全一样。解决后，提交，再push：

```
$ git commit -m "fix env conflict"
$ git push origin dev
```

因此，多人协作的工作模式通常是这样：

1. 首先，可以试图用`git push origin <branch-name>`推送自己的修改；
2. 如果推送失败，则因为远程分支比你的本地更新，需要先用`git pull`试图合并；
3. 如果合并有冲突，则解决冲突，并在本地提交；
4. 没有冲突或者解决掉冲突后，再用`git push origin <branch-name>`推送就能成功！

如果`git pull`提示`no tracking information`，则说明本地分支和远程分支的链接关系没有创建，用命令`git branch --set-upstream-to <branch-name> origin/<branch-name>`。

这就是多人协作的工作模式，一旦熟悉了，就非常简单。

### 17、Rebase

rebase操作可以把本地未push的分叉提交历史整理成直线；

rebase的目的是使得我们在查看历史提交的变化时更容易，因为分叉的提交需要三方对比。

```bash
$ git rebase
$ git log
```

### 18、创建标签

在Git中打标签非常简单，首先，切换到需要打标签的分支上：

```
$ git branch
* dev
  master
$ git checkout master
Switched to branch 'master'
```

然后，敲命令`git tag <name>`就可以打一个新标签：

```
$ git tag v1.0
```

可以用命令`git tag`查看所有标签：

```
$ git tag
v1.0
```

忘了打标签，找到历史提交的commit id，然后打上就可以了：

```bash
$ git log --pretty=oneline --abbrev-commit
12a631b (HEAD -> master, tag: v1.0, origin/master) merged bug fix 101
4c805e2 fix bug 101
e1e9c68 merge with no-ff
f52c633 add merge
...
```

比方说要对`add merge`这次提交打标签，它对应的commit id是`f52c633`，敲入命令：

```
$ git tag v0.9 f52c633
```

再用命令`git tag`查看标签：

```
$ git tag
v0.9
v1.0
```

注意，标签不是按时间顺序列出，而是按字母排序的。可以用`git show <tagname>`查看标签信息：

```
$ git show v0.9
commit f52c63349bc3c1593499807e5c8e972b82c8f286 (tag: v0.9)
Author: Michael Liao <askxuefeng@gmail.com>
Date:   Fri May 18 21:56:54 2018 +0800

    add merge

diff --git a/readme.txt b/readme.txt
...
```

可以看到，`v0.9`确实打在`add merge`这次提交上。

还可以创建带有说明的标签，用`-a`指定标签名，`-m`指定说明文字：

```
$ git tag -a v0.1 -m "version 0.1 released" 1094adb
```

用命令`git show <tagname>`可以看到说明文字：

```
$ git show v0.1
tag v0.1
Tagger: Michael Liao <askxuefeng@gmail.com>
Date:   Fri May 18 22:48:43 2018 +0800
...
```

### 19、操作标签

如果标签打错了，也可以删除：

```
$ git tag -d v0.1
```

因为创建的标签都只存储在本地，不会自动推送到远程。所以，打错的标签可以在本地安全删除。

如果要推送某个标签到远程，使用命令`git push origin <tagname>`：

```
$ git push origin v1.0
```

或者，一次性推送全部尚未推送到远程的本地标签：

```
$ git push origin --tags
```

如果标签已经推送到远程，要删除远程标签就麻烦一点，先从本地删除：

```
$ git tag -d v0.9
Deleted tag 'v0.9' (was f52c633)
```

然后，从远程删除。删除命令也是push，但是格式如下：

```
$ git push origin :refs/tags/v0.9
```

要看看是否真的从远程库删除了标签，可以登陆GitHub查看。

### 20、github

```
$ git remote -v
```

删除已有的GitHub远程库：

```
$ git remote rm origin
```

### 21、配置别名

```bash
$ git config --global alias.co checkout
$ git config --global alias.ci commit
$ git config --global alias.br branch
$ git config --global alias.st status
$ git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
$ git config --global alias.last 'log -1'
$ git config --global alias.unstage 'reset HEAD'
```

### 22、删除过大的文件记录

* Step 1 查看哪些历史提交过文件占用空间较大

使用以下命令可以查看占用空间最多的五个文件：

```
git rev-list --objects --all | grep "$(git verify-pack -v .git/objects/pack/*.idx | sort -k 3 -n | tail -5 | awk '{print$1}')"

```
rev-list命令用来列出Git仓库中的提交，我们用它来列出所有提交中涉及的文件名及其ID。 该命令可以指定只显示某个引用（或分支）的上下游的提交。

--objects：列出该提交涉及的所有文件ID。

--all：所有分支的提交，相当于指定了位于/refs下的所有引用。

verify-pack命令用于显示已打包的内容。

* step 2. 重写commit，删除大文件

使用以下命令，删除历史提交过的大文件：

```
git filter-branch --force --index-filter 'git rm -rf --cached --ignore-unmatch big-file.jar' --prune-empty --tag-name-filter cat -- --all
```

上面脚本中的big-file.jar请换成你第一步查出的大文件名，或者这里直接写一个目录。

filter-branch命令可以用来重写Git仓库中的提交

--index-filter参数用来指定一条Bash命令，然后Git会检出（checkout）所有的提交， 执行该命令，然后重新提交。

–all参数表示我们需要重写所有分支（或引用）。

在重写提交的过程中，会有以下日志输出:

```
Rewrite 6cdbb293d453ced07e6a07e0aa6e580e6a5538f4 (266/266)
# Ref 'refs/heads/master' was rewritten
```

如果显示 xxxxx unchanged, 说明repo里没有找到该文件, 请检查路径和文件名是否正确，重复上面的脚本，把所有你想删除的文件都删掉。

* step 3. 推送修改后的repo

以强制覆盖的方式推送你的repo, 命令如下:

```
git push origin master --force
```

* step 4. 清理和回收空间

虽然上面我们已经删除了文件, 但是我们的repo里面仍然保留了这些objects, 等待垃圾回收(GC), 所以我们要用命令彻底清除它, 并收回空间，命令如下:

```
rm -rf .git/refs/original/

git reflog expire --expire=now --all

git gc --prune=now
```

至此，我们已经彻底的删除了我们不想要的文件。


### 23. github 修改remote url

```
git remote set-url origin git@github.com:msgi/xxx.git
```

### 24、保持自己fork之后的仓库与上游仓库同步（github为例）。

点击 fork 到自己帐号下，比如[HanLP](https://github.com/hankcs/HanLP)这个仓库:

![Selection_033](/img/posts/tools/Selection_033.png)

然后就可以在自己的帐号下 clone 相应的仓库

`git clone https://github.com/algteam/HanLP.git`

使用 `git remote -v` 查看当前的远程仓库地址，输出如下：

```
origin	https://github.com/algteam/HanLP.git (fetch)
origin	https://github.com/algteam/HanLP.git (push)
```

可以看到从自己帐号 clone 下来的仓库，远程仓库地址是与自己的远程仓库绑定的

接下来运行 `git remote add upstream https://github.com/hankcs/HanLP.git`

这条命令就算添加一个别名为 upstream（上游）的地址，指向之前 fork 的原仓库地址。`git remote -v` 输出如下：

```
origin	https://github.com/algteam/HanLP.git (fetch)
origin	https://github.com/algteam/HanLP.git (push)
upstream	https://github.com/hankcs/HanLP.git (fetch)
upstream	https://github.com/hankcs/HanLP.git (push)
```

之后运行下面几条命令，就可以保持本地仓库和上游仓库同步了

```
git fetch upstream
git checkout master
git merge upstream/master
```

接着就是熟悉的推送本地仓库到远程仓库

```
git push origin master
```
