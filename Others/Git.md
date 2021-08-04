## 0verview

> 　　2002年Linus选择了一个商业的版本控制系统BitKeeper来管理Linux系统，秉持着开源精神BitKeeper授权Linux社区免费使用这个版本控制系统。而2005年Linux社区的一些人企图破解BitKeep的协议，被发现后Linux社区失去了BitKeeper的免费使用权。正是因此，Linus Torvalds于2005年用__C语言__开发了Git这一款__分布式的__版本控制系统。

### 集中式与分布式

- _集中式版本控制系统_ :版本库存放在中央服务器上,当我们准备工作的时候,首先需要从中央服务器获取最新的版本,工作完成后再进行上传.而且这个方式最大的弊端在于必须要联网才能工作.
- _分布式版本控制系统_ : 每个人的电脑都是一个完整的版本库,所以工作的时候不需要联网.理论上可以没有中央服务器,而为了多人协作方便,我们同常会使用一台中央服务器作为我们每台电脑交换信息的媒介.

### Git的安装

- 安装过程略
- 安装完成后,需要设置用户名和邮箱

```bash
$ git config --global user.name "Your Name"
$ git config --global user.email "email@email.com" 
```

> --global表示全局设定

### 创建版本库repository

- 通过`git init`命令 将某一目录初始化为Git可以管理的仓库.

```bash
$ git init
Initialized empty Git repository in D:/GitTest/.git/
```

- 初始化后会多出来一个名为`.Git`的隐藏目录,其中存放的是Git管理版本库的信息,小心不要删除了.

```bash
$ ls -ah
./  ../  .git/
```

## 常规操作

### add与commit

> 目录中添加了一个readme.txt文件...
> 内容为 I'm studying Git...

- `git add readme.txt`将readme.txt添加到暂存区 
- `git commit -m "a new readme file"`将文件提交到仓库

```bash
$ git add readme.txt
$ git commit -m"a new readme file"
[master (root-commit) 1d9a89f] a new readme file
 1 file changed, 1 insertion(+)
 create mode 100644 readme.txt
```

	- `-m`控制参数后面是本次提交的说明,建议每次提交都填上正确合理的说明,方便以后的查找.
	- 为什么是两步呢?因为我们可以add多个文件到暂存区然后一次性commit.

> 改动了一下readme.txt的内容...
> 增加 Git is easy to use...

- `git status`命令可以查看当前仓库的状态

```bash
$ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   readme.txt

no changes added to commit (use "git add" and/or "git commit -a")
```

- `git diff readme.txt` 可以查看工作区改动了哪些内容

```bash
$ git diff readme.txt
diff --git a/readme.txt b/readme.txt
index ba4782a..f2d0b06 100644
--- a/readme.txt
+++ b/readme.txt
@@ -1 +1,2 @@
-I'm studying Git...
\ No newline at end of file
+I'm studying Git...
+Git is easy to use...
\ No newline at end of file
```

> 再次提交修改过的文件 `$ git commit -m"add git is easy"`

#### 工作区与版本库

![mark](D:/OneDrive/_mine/docsify/_img/8YPxYOM6lAE0.png)

- 工作区add后到版本库中的暂存区,commit操作将暂存区的内容提交到HEAD指向的某分支上.

### 版本回退

- `git log` 显示从最近到最远的提交日志
  - 添加参数`--pretty=oneline`可以过滤Author和Date信息,只显示___commitId___和说明
  - HEAD指针指向当前版本

```bash
$ git log --pretty=oneline
4d806ebe133fdb37fc7f047c24d276fd47f9d604 (HEAD -> master) add git is easy
1d9a89f3eb4cc4aa4b75b3d5e3be5ce212a998eb a new readme file
```

- `git reset --hard HEAD^`回退到上一个版本
  - 加几个`^`就是回退几个版本 

```bash
$ git reset --hard HEAD^
HEAD is now at 1d9a89f a new readme file

$ git log
commit 1d9a89f3eb4cc4aa4b75b3d5e3be5ce212a998eb (HEAD -> master)
Author: khy_99 <khy_99@163.com>
Date:   Wed Nov 27 16:42:49 2019 +0800

    a new readme file
```

- ___问题___ : 再使用log发现找不到修改后的提交信息了?
  - 解决:如果你的黑窗口还没有关闭,找到第二次提交的版本号 `$ git reset --hard 版本ID` 注意ID不需要全部输入.

```bash
$ git reset --hard 4d80
HEAD is now at 4d806eb add git is easy
```

	- 黑窗口关了怎么办? `git reflog` 用来显示你的历史操作,从而找到需要的版本ID

```bash
$ git reflog
4d806eb (HEAD -> master) HEAD@{0}: reset: moving to 4d80
1d9a89f HEAD@{1}: reset: moving to HEAD^
4d806eb (HEAD -> master) HEAD@{2}: reset: moving to HEAD
4d806eb (HEAD -> master) HEAD@{3}: commit: add git is easy
1d9a89f HEAD@{4}: commit (initial): a new readme file
```


### 撤销与删除

- `git checkout -- readme.txt` 撤销__工作区__的修改
  - 若文件已经add到暂存区后又做了修改,则只会回退到add后的状态,就是把工作区版本还原成为暂存区版本.
  - 切换分支的命令是`git checkout`,注意`--`符号.
- `git reset HEAD readme.txt` 撤销__暂存区__的修改
  - 暂存区的修改撤销到了工作区

- `rm readme.txt` 删除工作区文件.
  - 确认删除需要add与commit.
  - `git checkout -- readme.txt`误删了,从版本库里恢复过来. 
- `git rm readme.txt`删除文件同时add到了暂存区.
  - `git commit`确定要删除就commit.
  - 恢复需要两步:
    - `git reset HEAD readme.txt` 从分支恢复到暂存区.
    - `git checkout -- readme.txt`再从暂存区恢复到工作区.

## 远程仓库

> 使用GitHub或Gitee充当远程仓库,托管我们的代码,方便多人协作.

### 创建远程仓库

- 首先需要创建SSH密钥对
  ` ssh-keygen -t rsa -C "email@example.com"` + 仨回车.
  会生成私钥`id_rsa`与公钥`id_rsa.pub`存放在当前用户下的`.ssh`目录中.

- 在GitHub上添加自己的公钥`id_rsq.pub`.
- 在GitHub上创建一个新的仓库.
- 将本地仓库与之关联
  - `git remote add origin git@github.com:HenryKang99/GitLearning.git`
    -上面`git@...`一串是远程仓库ssh地址,当然也可以使用HTTPS的方式,坏处就是速度慢而且每次都要输入密码.

### 推送与克隆

- `git push -u origin master`推送本地内容到远程仓库.
  - `-u`参数将本地的master与远程的master建立的关联,以后的push或pull就可以简化命令.

```bash
$ git push -u origin master
The authenticity of host 'github.com (52.74.223.119)' can't be established.
RSA key fingerprint is SHA256:nThbg6kXUpJWGl7E1IGOCspRomTxdCARLviKw6E5SY8.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes  <-- 第一次推送的时候这里要确认一下
Warning: Permanently added 'github.com,52.74.223.119' (RSA) to the list of known hosts.
Enumerating objects: 6, done.
Counting objects: 100% (6/6), done.
Delta compression using up to 4 threads
Compressing objects: 100% (2/2), done.
Writing objects: 100% (6/6), 474 bytes | 118.00 KiB/s, done.
Total 6 (delta 0), reused 0 (delta 0)
To github.com:HenryKang99/GitLearning.git
 * [new branch]      master -> master
Branch 'master' set up to track remote branch 'master' from 'origin'.
```

- `git clone git@github.com:HenryKang99/GitLearning.git`从远程仓库clone到本地.

## 分支管理

> 前面一直使用的都是名叫`master`的主分支

### 分支的创建,合并与删除

- `git checkout -b dev`创建并切换分支,相当于
  - `git branch dev` 和 `git checkout dev`

- 也可以使用`git switch -c dev`创建并切换分支.
- `git switch master` 切换到已有分支.

- `git branch`列出所有分支,当前分支前面会带有一个`*`

```bash
$ git branch
* dev
  master
```

> 在dev分支上commit一个devtest.txt文件,再切换回master分支,会发现刚才的文件不见了.

- `git merge dev` 将dev分支合并到master上.

```bash
$  git merge dev
Updating 4d806eb..3242396
Fast-forward
 devtest.txt | 1 +
 1 file changed, 1 insertion(+)
 create mode 100644 devtest.txt
 
 $ git log --pretty=oneline
32423966aa6d6dc0135a48a993d865a668766b43 (HEAD -> master, dev) add a new devtest file
4d806ebe133fdb37fc7f047c24d276fd47f9d604 (origin/master) add git is easy
1d9a89f3eb4cc4aa4b75b3d5e3be5ce212a998eb a new readme file
```

- <b>注意</b> : 这里其实合并的时候没有发生冲突,默认使用的是Git中的`快速合并`模式,相当于直接把`HEAD`指针指向了合并分支的最新一次commit;
  - 这样的优点是速度快,缺点是当我们使用`git log`时我们看不到这一次合并情况.
  - 想要看到分支合并情况,我们需要在合并的时候使用参数`git merge --no-ff`,表示禁用`快速合并`模式.

- `git branch -d dev` 删除dev分支

```bash
$ git branch -d dev
Deleted branch dev (was 3242396).

$ git branch
* master
```

### 解决冲突

> 在dev分支中修改devtest.txt文件,增加内容"test conflict...",add&commit;
> 切换到master分支,新增内容Test conflict...",add&commit;

1. 合并分支dev到master 发现冲突

```bash
$ git merge dev
Auto-merging devtest.txt
CONFLICT (content): Merge conflict in devtest.txt
Automatic merge failed; fix conflicts and then commit the result.
```

2. 打开冲突文件,解决冲突

```bash
This is a devtest file...
<<<<<<< HEAD
Test conflict...
=======
test conflict
>>>>>>> test conflict
```

3. 再次add&commit;使用'git log --graph' , `git log --graph --pretty=oneline --abbrev-commit` 查看合并情况,最后删除dev分支`git branch -d dev`

```bash
$ git log --graph --pretty=oneline --abbrev-commit
*   b4fa617 (HEAD -> master) fix conflict
|\
| * 491516a (dev) test conflict
* | 3609db5 Test conflict
|/
* 3242396 add a new devtest file
* 4d806eb (origin/master) add git is easy
* 1d9a89f a new readme file
```

### 使用`git stash` 隐藏工作区工作

> stash 隐藏,贮藏.
> issue 重要议题,争论的问题.
> 背景 : 当我们在dev分支工作到一半的时候需要紧急去修改master分支上的一个issue...

1. 使用`git stash`存储现在在dev上的工作;
   - 可以使用 `git status`查看工作区,发现是干净的.
2. `git switch -c issue01`创建并切换到issue01分支进行工作,完成后add&commit&merge&删除issue01分支.
3. 回到dev分支,恢复之前的工作:
   - `git stash list` 查看之前隐藏的工作区
   - `git stash apply` 和 `git stash drop`来恢复和删除stash
   - `git stash pop`恢复的同时自动删除stash 

```bash
$ git stash pop
On branch dev
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   devtest.txt

no changes added to commit (use "git add" and/or "git commit -a")
Dropped refs/stash@{0} (e936304bc07d6ed71bd6a0f421a432abfffa7fec)
```

> 若dev上也有和master同样的issue需要解决,可以直接把issue01提交到master的commit复制到dev上.

4. `cherry-pick commitID`复制某一个commit到当前分支.

```bash
$ git cherry-pick 7bbada6
[dev 2eb3829] issue fixed...
 Date: Thu Nov 28 21:27:09 2019 +0800
 1 file changed, 2 insertions(+), 1 deletion(-)
```

## 标签管理

### 本地

- `git tag v1.0` 给当前分支上最新的commit打上标签
- `git tag v2.0 commitID` 给某一个commit打上标签
- `git tag -d v1.0` 删除标签
- `git tag` 查看所有标签
- `git show v1.0` 查看标签的详细信息

### 远程

- `git push origin v1.0` 将某个标签推到远程
- `git push origin --tags` 将所有未push的标签推到远程
- 删除远端标签分为两步
  1. `git tag -d v1.0` 先删除本地标签
  2. `git push origin :refs/tags/v1.0`删除远程的标签