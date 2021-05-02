## Git学习笔记

### 1 Git是什么

#### 1.1 直接记录快照，而非差异比较

其他的版本控制系统（SVN等）都是基于差异（delta-based）的，将他们存储的信息看作是一组基本文件和每个文件随时间逐步积累的差异。

![存储每个文件于初始版本的差异](.\res\deltas.png "存储每个文件于初始版本的差异")

Git 不按照以上方式对待或保存数据。反之，Git 更像是把数据看作是对小型文件系统的一系列快照。 在 Git 中，每当你提交更新或保存项目状态时，它基本上就会对当时的全部文件创建一个快照并保存这个快照的索引。 为了效率，如果文件没有修改，Git 不再重新存储该文件，而是只保留一个链接指向之前存储的文件。 Git 对待数据更像是一个 **快照流**。

![存储项目随时间改变的快照](.\res\snapshots.png "存储项目随时间改变的快照")

如上图，虚线框中的是未改变的文件，Git并未重新存储这些文件，而是建立一个指向之前文件的链接。

#### 1.2 近乎所有操作都在本地执行

Git是一个分布式系统，在本地有一个完整的仓库副本，这使得在Git中大部分操作都只需要访问本地的文件和资源。

#### 1.2.1 Git保证完整性

Git中所有的数据在存储前都计算校验和，然后以检验和来引用。这是一个由40个十六进制字符组成的字符串，是基于文件内容或目录结构计算出来的。所以如果在传送过程中丢失信息或损坏文件，Git都能发现，这保证了文件的完整性。

#### 1.2.2 Git一般只添加数据

Git操作几乎只往Git数据库添加数据，Git几乎不会执行任何可能导致文件不可恢复的操作，一提交快照到Git中，就难以再丢失数据。因此使用Git特别安全。

#### 1.2.3 Git的三种状态

+ 已修改（modified）：表示修改了文件，但还没保存到数据库中
+ 已暂存（staged）：表示对一个已修改文件的当前版本做了标记，使之包含在下次提交的快照中
+ 已提交（committed）：表示数据已经安全地保存在本地数据库中

这会让我们的Git项目拥有三个阶段：工作区、暂存区、以及Git目录。

![工作目录、暂存区域以及Git仓库](.\res\areas.png "工作目录、暂存区域以及Git仓库")

工作区是对项目的某个版本独立提取出来的内容。 这些从 Git 仓库的压缩数据库中提取出来的文件，放在磁盘上供你使用或修改（即在磁盘上看到的文件）。

暂存区是一个文件，保存了下次将要提交的文件列表信息，一般在 Git 仓库目录中。 按照 Git 的术语叫做“索引”，不过一般说法还是叫“暂存区”。

Git 仓库目录是 Git 用来保存项目的元数据和对象数据库的地方。 这是 Git 中最重要的部分，从其它计算机克隆仓库时，复制的就是这里的数据（压缩了的数据）。

基本的 Git 工作流程如下：

1. 在工作区中修改文件。
2. 将你想要下次提交的更改选择性地暂存，这样只会将更改的部分添加到暂存区。
3. 提交更新，找到暂存区的文件，将快照永久性存储到 Git 目录。

### 2 Git基础

#### 2.1 获取Git仓库

两种方法创建Git仓库：

+ 在想要创建Git仓库的目录下执行：

  ```shell
  $ git init
  ```

  该命令将创建一个名为 .git 的子目录，这个目录含有初始化的Git仓库中所有的必须文件。如果在一个已存在文件的文件夹中进行版本控制，要通过 git add 命令来指定所需的文件进行追踪，然后执行 git commit：

  ```shell
  $ git add *.c
  $ git add LICENSE
  $ git commit -m 'initial project version'
  ```

  

+ 从其他服务器克隆一个已存在的Git仓库

  Git克隆的是该服务器上几乎所有的数据，而不仅仅复制完成你的工作所需要的文件。当执行git clone命令时，默认配置下远程Git仓库的每一个文件的每一个版本都将拉取下来。

  ```shell
  $ git clone https://github.com/libgit2/libgit2
  ```

  这会在当前目录下创建一个名为libgit2目录，并在这个目录下初始化一个 .git 文件夹，从远程仓库拉取所有数据放入 .git 文件夹，然后从中读取最新版本的文件拷贝。可以通过额外的参数自定义本地仓库的名字：

  ```shell
  $ git clone https://github.com/libgit2/libgit2 mylibgit
  ```

  Git支持多种数据传输协议上面的例子使用 https:// 协议，也可以使用 git:// 协议或使用SSH传输协议，比如 user@server:path/to/repo.git 。

#### 2.2 记录每次更新到仓库

##### 2.2.1 检查当前文件状态

可以使用 git status 查看哪些文件处于什么状态。如果在克隆仓库后立即使用此命令：

```shell
$ git status
On branch master    # 当前所在分支为master分支
Your branch is up-to-date with 'origin/master'.   # 这个分支同远程服务器上的分支没有偏离
nothing to commit, working directory clean    # 当前工作目录很干净，所有已跟踪文件在上次                                               # 提交后都未被更改过，且当前目录下没有出现                                               # 任何未跟踪状态的新文件
```

在该项目下新建一个README文件，再使用该命令：

```shell
$ git status
On branch master
Your branch is up-to-date with 'origin/master'.
Untracked files:    # 未跟踪的文件列表
  (use "git add <file>..." to include in what will be committed)
  
    README
    
nothing added to commit but untracked files present (use "git add" to track)
```

##### 2.2.2 跟踪新文件

使用 git add 开跟踪一个文件：

```shell
$ git add README
```

此时再使用 git status：

```shell
$ git status
On branch master
Your branch is up-to-date with 'origin/master'.
Changes to committed:    # 改变且已暂存的文件列表
  (use "git restore --staged <file>..." to unstage)
  
    new files:  README
```

##### 2.2.3 暂存已修改的文件

如果修改了一个名为CONTRIBUTING.md的已被跟踪的文件，然后运行 git status 命令：

```shell
$ git status
On branch master
Your branch is up-to-date with 'origin/master'.
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    new file:   README

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

    modified:   CONTRIBUTING.md
```

总之，不管是新建了文件或修改了已存在的文件，都需要用 git add 命令将新建的或修改过的文件添加到暂存区。将 git add 命令理解为“精确地将内容添加到下一次提交中”。

##### 2.2.4 忽略文件

可以创建一个名为 .gitignore 文件，列出要忽略的文件的模式：

```shell
$ cat .gitignore
*.[oa]    # 忽略所有以.a或.a结尾的文件
*~        # 忽略以~结尾的文件
build/    # 忽略build文件夹
```

文件 .gitignore 的格式规范如下

+ 所有空行或者以 # 开头的行都会被忽略
+ 可以使用标准的 glob 模式匹配，它会递归地应用在整个工作区中
+ 匹配模式可以以 (/) 开头防止递归
+ 匹配模式可以以 (/) 结尾指定目录
+ 要忽略指定模式以外地文件或目录，可以在模式前加上谈好 (!) 取反

##### 2.2.5 查看已暂存地和未暂存地修改

不加参数直接输入 git diff 查看尚未暂存地文件更新了哪些部分。

可以用 git diff --staged 命令查看已暂存的将要添加到下次提交里的内容，这条命令将比对已暂存文件与最后一次提交的文件差异。

##### 2.2.6 提交更新

现在暂存区已经准备就绪，可以提交了。在此之前，请务必确认还有什么已修改或新建的文件还没有 git add 过，否则提交的时候不会记录那些尚未暂存的变化。这些已修改但为暂存的文件只会保留在本地磁盘。所以，每次准备提交前，先用 git status 查看一下，你所需要的文件是不是都已暂存起来，然后再运行 git commit：

```shell
$ git commit
```

这样会启动你选择的文本编辑器（vim等）来输入提交说明。

另外，也可以在commit命令后添加 -m 选项，将提交信息与命令放在同一行。如下所示：

```shell
$ git commit -m "Story 182: Fix benchmarks for speed"
[master 463dc4f] Story 182: Fix benchmarks for speed    # master分支，463dc4f是检验和
 2 files changed, 2 insertions(+)    # 多少文件修改过，虽少行添加和删改过
 create mode 100644 README
```

可以加上 -a 选项跳过 git add 命令，直接将所有已被跟踪的文件暂存起来一并提交，不过不建议使用。

##### 2.2.7 移除文件

手动从工作目录中删除文件，再使用 git rm 命令记录此次一处文件的操作：

```shell
$ git rm PROJECTS.md
rm 'PROJECTS.md'
$ git status
On branch master
Your branch is up-to-date with 'origin/master'.
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    deleted:    PROJECTS.md
```

下一次提交时，该文件就不再纳入版本管理了。加上 -f 选项强制删除已放到暂存区的文件。

使用 --cached 选项：

```shell
$ git rm --cached README
```

可以将文件从Git仓库中删除，但任然保留在当前工作目录中。适用于当忘记添加 .gitignore 文件，不小心把一个很大的日志文件或一堆 .a 这样的编译生成文件添加到暂存区时。

##### 2.2.8 移动文件

使用 git mv 命令移动文件或改名：

```shell
$ git mv README.md README
$ git status
On branch master
Your branch is up-to-date with 'origin/master'.
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    renamed:    README.md -> README
```

其实，运行 git mv 就相当于运行了下面三条命令：

```shell
$ mv README.md README
$ git rm README.md
$ git add README
```

如此分开操作，Git也会意识到这是一次重命名，所以不管那何种方式结果都是一样的。

#### 2.3 查看提交历史

使用 git log 命令查看提交历史：

```shell
$ git log
commit ca82a6dff817ec66f44342007202690a93763949    # SHA-1校验和
Author: Scott Chacon <schacon@gee-mail.com>        # 作者的名字和电子邮件地址
Date:   Mon Mar 17 21:52:11 2008 -0700             # 提交时间

    changed the version number                     # 提交说明

commit 085bb3bcb608e1e8451d4b2432f8ecbe6306e7e7
Author: Scott Chacon <schacon@gee-mail.com>
Date:   Sat Mar 15 16:40:33 2008 -0700

    removed unnecessary test

commit a11bef06a3f659402fe7563abf99ad00de2209e6
Author: Scott Chacon <schacon@gee-mail.com>
Date:   Sat Mar 15 10:31:28 2008 -0700

    first commit
```

使用 -p 或 --path 选项显示每次提交引入的差异，类似于 git diff 命令的输出。

使用 --stat 选项查看每次提交的简略统计信息，列出所有被修改过的文件、有多少文件被修改了以及被修改过的文件哪些行被移除或是添加了。

使用 --since 和 --until 按照时间作限制筛选输出，如：

```shell
$ git log --since=2.weeks
```

列出最近两周的所有提交。可以用 “2021-04-30” 这样的格式指定某一天。

使用 --author 选项显示指定作者的提交，用 --grep 选项搜索提交说明中的关键字。

-S 选项，它接受一个字符串参数，并且只会显示那些添加或删除了该字符串的提交。 假设你想找出添加或删除了对某一个特定函数的引用的提交，可以调用：

```shell
$ git log -S function_name
```

#### 2.4 撤销操作

如果提交完了之后发现漏了几个文件没有添加，或者提交信息写错了，可以运行 git commit --amend 命令来重新提交：

```shell
$ git commit --amend
```

有两种情况：

+ 自上次提交后没有修改，那么快照不变，只会修改提交信息
+ 做了修改，并使用 git add 添加到了暂存区，第二次提交将替代第一次提交的结果

##### 2.4.1 取消暂存的文件

可以使用 git reset 命令来取消暂存文件：

```shell
$ git reset HEAD CONTRIBUTING.md
Unstaged changes after reset:
M	CONTRIBUTING.md
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    renamed:    README.md -> README

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

    modified:   CONTRIBUTING.md
```

##### 2.4.2 撤销对文件的修改

使用 git checkout 命令撤销对文件的修改，将其还原成上次提交时的样子：

```shell
$ git checkout -- CONTRIBUTING.md    # -- 接文件名，即检出这个文件
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    renamed:    README.md -> README
```

这样对这个文件在本地的任何修改都会消失——Git会用最近提交的版本覆盖掉它。

#### 2.5 远程仓库的使用

##### 2.5.1 查看远程仓库

使用以下命令查看远程仓库：

```shell
$ git remote
origin      # 远程仓库的默认名字
```

指定 -v 选项，会显示远程仓库的URL：

```shell
$ git remote -v
origin https://github.com/schacon/ticgit (fetch)    # 读
origin https://github.com/schacon/ticgit (push)     # 写
```

##### 2.5.2 添加远程仓库

使用 git clone 命令可以克隆远程仓库，也可以使用 git remote add 命令为本地仓库添加一个新的远程仓库，同时指定一个方便的简写：

```shell
$ git remote add pb https://github.com/paulboone/ticgit  # pb为远程仓库的简写，后接URL
$ git remote -v
origin https://github.com/schacon/ticgit (fetch)
origin https://github.com/schacon/ticgit (push)
pb	https://github.com/paulboone/ticgit (fetch)
pb	https://github.com/paulboone/ticgit (push)
```

##### 2.5.3 从远程仓库抓取与拉取

执行以下命令从远程仓库获得数据：

```shell
$ git fetch <remote>
```

这个命令只会将数据下载到本地仓库，不会自动合并或修改当前的工作。

也可以执行

```shell
$ git pull
```

命令来自动抓取后合并该远程分支到当前分支。

##### 2.5.4 推送到远程仓库

执行

```shell
$ git push <remote> <branch>
```

命令分享自己的项目，以下命令将master分支推送到origin服务器：

```shell
$ git push origin master    # origin,master是默认远程仓库名，主分支名
```

如果和其他人同一时间克隆，但其他人先推送，自己的推送将会被拒绝，这时要先从远程仓库拉取数据并合并到自己的工作中，再推送。

##### 2.5.5 查看远程仓库

使用以下命令查看远程仓库的跟多信息：

```shell
$ git remote show origin       # show后接仓库名
* remote origin
  Fetch URL: https://github.com/schacon/ticgit
  Push  URL: https://github.com/schacon/ticgit
  HEAD branch: master          # 当前工作分支
  Remote branches:             # 分支列表
    master                               tracked
    dev-branch                           tracked
  Local branch configured for 'git pull':    # 直接使用git pull拉取所有远程引用
    master merges with remote master
  Local ref configured for 'git push':
    master pushes to master (up to date)
```

##### 2.5.6 远程仓库重命名与移除

执行 git remote rename 来修改一个远程仓库的简写名：

```shell
$ git remote rename pb paul
$ git remote
origin
paul
```

执行 git remote remove 移除一个远程仓库，或 git remote rm：

```shell
$ git remote remove paul
$ git remote
origin
```

#### 2.6 打标签

可以给仓库历史中某个提交打上标签，以示重要。

##### 2.6.1 列出标签

执行 git tag 列出标签：

```shell
$ git tag
v1.0      # 按字母顺序列出标签
v2.0
```

也可以按照特定模式查找标签，加上 -l 或 --list 选项：

```shell
$ git tag -l "v1.8.5*"
v1.8.5
v1.8.5-rc0
v1.8.5-rc1
v1.8.5.1
v1.8.5.2
```

##### 2.6.2 创建标签

Git支持两种标签:

+ 轻量标签（lightweight）：很像一个不会改变的分支，它只是对某个特定提交的引用
+ 附注标签（annotated）：附注标签是存储在Git数据库中的一个完整对象，它们可以被校验，其中包含打标签者的名字、电子邮箱地址、日期时间，还有一个标签信息，可以使用GPG（GNU Privacy Guard）签名并验证

##### 2.6.3 附注标签

使用以下命令创建附注标签：

```shell
$ git tag -a v1.4 -m "version 1.4"
```

使用 -a 选项创建附注标签，-m 后写上标签信息，没有 -m 选项的话会启动编辑器来要求输入。

##### 2.6.4 轻量标签

轻量标签本质上是将提交校验和存储到一个文件中，没有保存其他任何信息。创建轻量标签不需要 -a -m 等选项，只需要提供标签名字：

```shell
$ git tag v1.4-1w
```

##### 2.6.5 后期打标签

为过去的提交打上标签：

```shell
git tag -a v1.2 9fceb02
```

只需要在命令末尾加上校验和或部分校验和。

##### 2.6.6 共享标签

git push 默认不会传送标签到远程服务器，必须显式地推送标签：

```shell
$ git push origin <tagname>
```

使用以下命令将所有不在远程服务器上地标签全部推送：

```shell
$ git push origin --tags
```

##### 2.6.7 删除标签

使用以下命令删除标签：

```shell
$ git tag -d <tagname>
```

再使用以下命令更新远程仓库：

```shell
$ git push origin --delete <tagname>
```

##### 2.6.8 检出标签

可以使用以下命令查看某个标签所指向的文件版本：

```shell
$ git checkout <tagname>
```

这会使仓库处于“分离头指针（detached HEAD）”状态，在这个状态下，如果你做了某些更改然后提交它们，标签不会发生变化， 但你的新提交将不属于任何分支，并且将无法访问，除非通过确切的提交哈希才能访问。因此，如需更改，通常要创建一个新的分支：

```shell
$ git checkout -b version2 v2.0.0
Switched to a new branch 'version2'
```

### 3 Git分支

#### 3.1 分支简介

提交对象会包含一个指向暂存内容快照的指针。 但不仅仅是这样，该提交对象还包含了作者的姓名和邮箱、提交时输入的信息以及指向它的父对象的指针。 首次提交产生的提交对象没有父对象，普通提交操作产生的提交对象有一个父对象， 而由多个分支合并产生的提交对象有多个父对象。

Git 的分支，其实本质上仅仅是指向提交对象的可变指针。 Git 的默认分支名字是 `master`。

![分支及其提交历史](./res/branch-and-history.png "分支及其提交历史")

##### 3.1.1 分支创建

创建一个testing分支，使用`git branch` 命令：

```shell
$ git branch testing
```

这会在当前所在的提交对象上创建一个指针。

![两个指向相同提交历史的分支](./res/two-branches.png "两个指向相同提交历史的分支")

Git怎么知道当前在哪一个分支上？它有一个名为`HEAD`的特使指针，`HEAD`是一个指向当前所在的本地分支的指针，将其想象成当前分支的别名。在本例中，当前分支任然是`master`分支，因为`git branch`分支仅仅创建一个分支，并不会主动切换到新分支中去。

![HEAD指向当前所在分支](.\res\head-to-master.png "HEAD指向当前所在分支")

##### 3.1.2 分支切换

要切换到一个已存在的分支，使用 `git checkout`命令：

```shell
$ git checkout testing
```

这样`HEAD`就指向`testing`分支了。

![HEAD指向当前所在分支](./res/head-to-testing.png "HEAD指向当前所在分支")

这是再提交一次，`testing`分支向前移动了，但`master`分支却没有：

![HEAD分支随着提交操作自动向前移动](.\res\advance-testing.png "HEAD分支随着提交操作自动向前移动")

切回`master`分支：

![检出时HEAD随之移动](.\res\checkout-master.png "检出时HEAD随之移动")

这条命令做了两件事：一是使HEAD指回`master`分支，二是将工作目录恢复成`master`分支所指向的快照内容。

做些修改并提交，现在这个项目的提交历史已经产生了分叉：

![项目分叉历史](./res/advance-master.png "项目分叉历史")

可使用 `git log`查看分叉历史。

可以使用`git checkout -b <newbranchname>`创建一个新分支的同时切换过去。

#### 3.2 分支的新建与合并

让我们来看一个简单的分支新建与分支合并的例子，实际工作中你可能会用到类似的工作流。 你将经历如下步骤：

1. 开发某个网站。
2. 为实现某个新的用户需求，创建一个分支。
3. 在这个分支上开展工作。

正在此时，你突然接到一个电话说有个很严重的问题需要紧急修补。 你将按照如下方式来处理：

4. 切换到你的线上分支（production branch）。
5. 为这个紧急任务新建一个分支，并在其中修复它。
6. 在测试通过之后，切换回线上分支，然后合并这个修补分支，最后将改动推送到线上分支。
7. 切换回你最初工作的分支上，继续工作。

##### 新建分支

解决问题追踪系统中的`#53`问题，新建一个分支并切换到这个分支：

```shell
$ git checkout -b iss53
Switched to a new branch "iss53"
```

在`#53`问题上工作并做了一些提交，在此过程中，`iss53`分支在不断向前推进，`HEAD`指向`iss53`分支。

接下来，要修复一个紧急问题。新建立一个`hotfix`分支，在该分支上工作直到问题解决：

```shell
$ git checkout -b hotfix
Switched to a new branch 'hotfix'
```

然后将`hotfix`分支合并回`master`分支来部署到线上：

```shell
$ git checkout master
$ git merge hotfix
Updating f42c576..3a0874c
Fast-forward
 index.html | 2 ++
 1 file changed, 2 insertions(+)
```

fast-forward：

由于想要合并的分支 `hotfix` 所指向的提交 `C4` 是你所在的提交 `C2` 的直接后继， 因此 Git 会直接将指针向前移动。换句话说，当你试图合并两个分支时， 如果顺着一个分支走下去能够到达另一个分支，那么 Git 在合并两者的时候， 只会简单的将指针向前推进（指针右移），因为这种情况下的合并操作没有需要解决的分歧——这就叫做 “快进（fast-forward）”。

删除`hotfix`分支：

```shell
$ git branch -d hotfix
Deleted branch hotfix (3a0874c).
```

切回到`iss53`分支继续之前的工作：

```shell
$ git checkout iss53
Switched to branch "iss53"
```

##### 无冲突的分支合并

已修正`#53`问题，将所做的工作合并回`master`分支：

```shell
$ git checkout master
Switched to branch 'master'
$ git merge iss53
Merge made by the 'recursive' strategy.
index.html |    1 +
1 file changed, 1 insertion(+)
```

和之前将分支指针向前推进不同，Git将此次三方合并地结果做了一个新的快照并自动创建一个新的提交指向它，这个称作一次合并提交，它地特别之处在于它有不止一个父提交。

##### 遇到冲突时的分支合并

如果在两个不同的分支都对同一个文件同一部分进行了修改，Git没法干净地合并：

```shell
$ git merge iss53
Auto-merging index.html
CONFLICT (content): Merge conflict in index.html
Automatic merge failed; fix conflicts and then commit the result.
```

可以使用`git status`命令查看那些因包含合并冲突而处于未合并（unmerged）状态地文件。

Git会在出现冲突地文件中加入标准的冲突解决标记：

```shell
<<<<<<< HEAD:index.html       # 上半部分是HEAD亦既是当前所在分支（主分支）的内容
<div id="footer">contact : email.support@github.com</div>
=======                       # 分割线
<div id="footer">
 please contact us at support@github.com
</div>
>>>>>>> iss53:index.html      # iss53分支的内容
```

解决完冲突后，暂存这些文件并提交。

#### 3.3 分支管理

如果不加任何参数运行`git branch`命令，会得到当前所有分支的一个列表：

```shell
$ git branch
  iss53
* master
  testing
```

注意 `master` 分支前的 `*` 字符：它代表现在检出的那一个分支（也就是说，当前 `HEAD` 指针所指向的分支）。 这意味着如果在这时候提交，`master` 分支将会随着新的工作向前移动。 如果需要查看每一个分支的最后一次提交，可以运行 `git branch -v` 命令。

`--merged` 与 `--no-merged` 这两个选项可以过滤这个列表中已经合并或尚未合并到当前分支的分支。 如果要查看哪些分支已经合并到当前分支，可以运行 `git branch --merged`：

```shell
$ git branch --merged
  iss53
* master
```

因为之前已经合并了 `iss53` 分支，所以现在看到它在列表中。 在这个列表中分支名字前没有 `*` 号的分支通常可以使用 `git branch -d` 删除掉；你已经将它们的工作整合到了另一个分支，所以并不会失去任何东西。

查看所有包含未合并工作的分支，可以运行 `git branch --no-merged`：

```shell
$ git branch --no-merged
  testing
```

这里显示了其他分支。 因为它包含了还未合并的工作，尝试使用 `git branch -d` 命令删除它时会失败：

```console
$ git branch -d testing
error: The branch 'testing' is not fully merged.
If you are sure you want to delete it, run 'git branch -D testing'.
```

如果真的想要删除分支并丢掉那些工作，如同帮助信息里所指出的，可以使用 `-D` 选项强制删除它。

#### 3.4 远程分支

##### 3.4.1 抓取

假设当前远程服务器与本地的提交记录如下：

![本地与远程的工作可以分叉](./res/remote-branches-2.png "本地与远程的工作可以分叉")

如果要与给定的远程仓库同步数据，运行 `git fetch <remote>` 命令，如：

```shell
$ git fetch origin
```

这个命令查找 “origin” 是哪一个服务器（在本例中，它是 `git.ourcompany.com`）， 从中抓取本地没有的数据，并且更新本地数据库，移动 `origin/master` 指针到更新之后的位置。

![git fetch更新跟踪的远程分支](./res/remote-branches-3.png "git fetch更新跟踪的远程分支")

##### 3.4.2 推送

想要分享一个远程服务器没有，只在本地存在的分支，必须显示地推送想要分享地分支，使用`git push <remote> <branch>`，命令：

```shell
$ git push origin serverfix
Counting objects: 24, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (15/15), done.
Writing objects: 100% (24/24), 1.91 KiB | 0 bytes/s, done.
Total 24 (delta 2), reused 0 (delta 0)
To https://github.com/schacon/simplegit
 * [new branch]      serverfix -> serverfix
```

这意味着“推送本地的 `serverfix` 分支，将其作为远程仓库上的 `serverfix` 分支”。

特别注意，当抓取到新的远程跟踪分支时，本地不会自动生成一份可编辑的副本。可以运行`git merge origin/serverfix`将这些工作合并到当前所在的分支。也可以运行`git checkout -b <branch> <remote>/<branch>`来建立自己的`serverfix`分支：

```shell
$ git checkout -b serverfix origin/serverfix
Branch serverfix set up to track remote branch serverfix from origin.
Switched to a new branch 'serverfix'
```

##### 3.4.3 跟踪分支

从一个远程跟踪分支检出一个本地分支会自动创建所谓的“跟踪分支”（它跟踪的分支叫做“上游分支”）。 跟踪分支是与远程分支有直接关系的本地分支。 如果在一个跟踪分支上输入 `git pull`，Git 能自动地识别去哪个服务器上抓取、合并到哪个分支。

当克隆一个仓库时，它通常会自动地创建一个跟踪 `origin/master` 的 `master` 分支。

`git checkout -b <branch> <remote>/<branch>`命令有一种快捷方式：

```shell
$ git checkout --track origin/serverfix
Branch serverfix set up to track remote branch serverfix from origin.
Switched to a new branch 'serverfix'
```

该捷径本身还有一个捷径，如果要检出的分支在本地不存在，但有一个同名的远程分支，那么Git会自动创建一个跟踪分支：

```shell
$ git checkout serverfix
Branch serverfix set up to track remote branch serverfix from origin.
Switched to a new branch 'serverfix'
```

设置已有的本地分支跟踪一个刚刚拉取下来的远程分支，或者想要修改正在跟踪的上游分支，使用`-u` 或 `--set-upstream-to` 选项运行 `git branch` 来显式地设置：

```shell
$ git branch -u origin/serverfix
Branch serverfix set up to track remote branch serverfix from origin.
```

上游分支可以使用`@{upstream}`或`@{u}`简写。

##### 3.4.4 拉取

当 `git fetch` 命令从服务器上抓取本地没有的数据时，它并不会修改工作目录中的内容。 它只会获取数据然后让你自己合并。 然而，有一个命令叫作 `git pull` 在大多数情况下它的含义是一个 `git fetch` 紧接着一个 `git merge` 命令。`git pull` 会查找当前分支所跟踪的服务器与分支， 从服务器上抓取数据然后尝试合并入那个远程分支。

##### 3.4.5 删除远程分支

运行带有 `--delete` 选项的 `git push` 命令来删除一个远程分支：

```shell
$ git push origin --delete serverfix
To https://github.com/schacon/simplegit
 - [deleted]         serverfix
```

这个命令做的只是从服务器上移除这个指针，Git服务器通常会保留数据一段时间直到垃圾回收运行，如果不小心删除了，通常很容易恢复。

#### 3.5 变基

##### 3.5.1 变基的基本操作

整合来自不同分支的修改有两种方法：

+ `merge`
+ `rebase`

![分叉的提交历史](./res/basic-rebase-1.png "分叉的提交历史")

`merge`命令会把两个分支的最新快照（c3和c4）以及二者最近的共同祖先进行三方合并，合并的结果是生成一个新的快照（并提交）。

![通过合并操作来整合分支](./res/basic-rebase-2.png "通过合并操作来整合分支")

还有一种方法：提取在`C4`中引入的补丁和修改，然后再`C3`的基础上应用一次。在Git上，这种操作叫做**变基（rebase）**。可以使用`rebase`命令将提交到某一分支上的所有修改都移至另一分支上：

```shell
$ git checkout experiment
$ git rebase master
First, rewinding head to replay your work on top of it...
Applying: added staged command
```

注意是将当前分支上的修改移到`master`分支，`rebase`后接的是目标分支。

它的原理首先找到这两个分支的最近共同祖先`C2`，然后对比当前分支相对于该祖先的历次提交，提取相应的修改，并存为临时文件，然后将当前分支指向目标基底`C3`，最后以将之前另存为临时文件的修改依序应用。

![将C4中的修改变基到C3上](./res/basic-rebase-3.png "将C4中的修改变基到C3上")

再回到`master`分支，进行一次快进合并：

```shell
$ git checkout master
$ git merge experiment
```

![master分支的快进合并](./res/basic-rebase-4.png "master分支的快进合并")

这两种整合方法的最终结果没有任何区别，但是变基是的提交历史更加整洁，成一条直线。

注意，无论是通过变基，还是通过三方合并，整合的最终结果所指向的快照始终是一样的，只不过提交历史不同罢了。变基是将一系列提交按照原有次序应用到另一分支上，而合并是把最终结果合在一起。

可以使用`git rebase <basebranch> <topicbranch>`命令在主分支进行变基，而省去先切换到`server` 分支。

##### 3.5.2 变基的风险

变基的操作实质上是丢弃一些现有的提交，然后相应的新建一些内容一样但实际上不同的提交。变基要遵守一条准则：

**如果提交存在于你的仓库之外，而别人可能基于这些提交进行开发，那么不要执行变基。**

使用`git pull --rebase`命令或`git pull -r`命令相当于先`git fetch`，再`git rebase <remote>/<branch>`，通过rebase而不是merge整合分支。

##### 3.5.3 变基 vs. 合并

有一种观点认为，仓库的提交历史即是 **记录实际发生过什么**。 它是针对历史的文档，本身就有价值，不能乱改。 从这个角度看来，改变提交历史是一种亵渎，你使用 *谎言* 掩盖了实际发生过的事情。 如果由合并产生的提交历史是一团糟怎么办？ 既然事实就是如此，那么这些痕迹就应该被保留下来，让后人能够查阅。

另一种观点则正好相反，他们认为提交历史是 **项目过程中发生的事**。 没人会出版一本书的第一版草稿，软件维护手册也是需要反复修订才能方便使用。 持这一观点的人会使用 `rebase` 及 `filter-branch` 等工具来编写故事，怎么方便后来的读者就怎么写。

总的原则是，只对尚未推送或分享给别人的本地修改执行变基操作清理历史， 从不对已推送至别处的提交执行变基操作，这样，你才能享受到两种方式带来的便利。

### 4 Git工具

#### 4.1 选择修订版本

可以通过任意一个提交的 40 个字符的完整 SHA-1 散列值来指定单个修订版本， 不过还有很多更人性化的方式来做同样的事情。

Git 十分智能，只需要提供 SHA-1 的前几个字符就可以获得对应的那次提交， 当然你提供的 SHA-1 字符数量不得少于 4 个，并且没有歧义——也就是说， 当前对象数据库中没有其它对象以这段 SHA-1 开头。

#### 4.2 贮藏与清理

有时，当在项目的一部分上已经工作了一段时间后，所有东西都进入了混乱状态，而这时想要切换到另一个分支做一点别的事情，可以使用`git stash`命令。

##### 4.2.1 贮藏工作

当前工作区状态：

```shell
$ git status
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

	modified:   index.html

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	modified:   lib/simplegit.rb
```

现在想要切换分支，但不想提交之前的工作，所以贮藏修改，运行`git stash`或`git stash push`：

```shell
$ git stash
Saved working directory and index state \
  "WIP on master: 049d078 added the index file"
HEAD is now at 049d078 added the index file
(To restore them type "git stash apply")
```

可以看到工作目录是干净的：

```shell
$ git status
# On branch master
nothing to commit, working directory clean
```

此时可以切换分支并在其他地方工作，原来的修改存储在栈上。要查看贮藏的东西，可以使用`git stash list`：

```shell
$ git stash list
stash@{0}: WIP on master: 049d078 added the index file
stash@{1}: WIP on master: c264051 Revert "added file_size"
stash@{2}: WIP on master: 21d80a5 added number to log
```

可以使用`git stash apply`命令将刚刚贮藏的工作重新应用。如果想要应用一个更旧的贮藏，可是通过名字指定：`git stash apply stash@{2}`。

```shell
$ git stash apply
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	modified:   index.html
	modified:   lib/simplegit.rb

no changes added to commit (use "git add" and/or "git commit -a")
```

可以在一个分支上保存一个贮藏，切换到另一个分支，然后藏尸重新应用这些修改。当应用贮藏时工作目录中也可以有修改与未提交的文件——如果有任何东西不能干净地应用，Git 会产生合并冲突。

使用`--index`选项来运行`git stash apply`命令，来重新应用暂存的修改。

应用选项只会尝试应用贮藏的工作，在堆栈上还有它。可以使用`git stash drop`加上将要移除的贮藏的名字来移除它：

```shell
$ git stash list
stash@{0}: WIP on master: 049d078 added the index file
stash@{1}: WIP on master: c264051 Revert "added file_size"
stash@{2}: WIP on master: 21d80a5 added number to log
$ git stash drop stash@{0}
Dropped stash@{0} (364e91f3f268f0900bc3ee613f9f733e82aaed43)
```

也可以运行`git stash pop`来应用贮藏然后立即从堆栈上移除。
