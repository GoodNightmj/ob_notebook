## 起步
### Git的优势
1. 分布式存储系统
	在Git中并不只提取最新版本的文件快照， 而是把代码仓库完整地镜像下来，包括完整的历史记录。 这么一来，任何一处协同工作用的服务器发生故障，事后都可以用任何一个镜像出来的本地仓库恢复。 因为每一次的克隆操作，实际上都是一次对代码仓库的完整备份。
2. 直接记录快照，而非差异比较
	Git 不按照保存不同版本文件差异方式对待或保存数据。反之，Git 更像是把数据看作是对小型文件系统的一系列快照。 在 Git 中，每当你提交更新或保存项目状态时，它基本上就会对当时的全部文件创建一个快照并保存这个快照的索引。 为了效率，如果文件没有修改，Git 不再重新存储该文件，而是只保留一个链接指向之前存储的文件。 Git 对待数据更像是一个 **快照流**。
3. 近乎所有操作都是本地执行
	 举个例子，要浏览项目的历史，Git 不需外连到服务器去获取历史，然后再显示出来——它只需直接从本地数据库中读取。 你能立即看到项目历史。如果你想查看当前版本与一个月前的版本之间引入的修改， Git 会查找到一个月前的文件做一次本地的差异计算，而不是由远程服务器处理或从远程服务器拉回旧版本文件再来本地处理。
4.  Git 保证完整性
	 Git 用以计算校验和的机制叫做 SHA-1 散列（hash，哈希）。 这是一个由 40 个十六进制字符（0-9 和 a-f）组成的字符串，基于 Git 中文件的内容或目录结构计算出来。 SHA-1 哈希看起来是这样：
	`24b9da6552252987aa493b52f8696cd6d3b00373`
	Git 中使用这种哈希值的情况很多，你将经常看到这种哈希值。 实际上，Git 数据库中保存的信息都是以文件内容的哈希值来索引，而不是文件名。
5. Git 一般只添加数据
	你执行的 Git 操作，几乎只往 Git 数据库中 **添加** 数据。 你很难使用 Git 从数据库中删除数据，也就是说 Git 几乎不会执行任何可能导致文件不可恢复的操作。
### git的三种状态
Git 有三种状态，你的文件可能处于其中之一： **已提交（committed）**、**已修改（modified）** 和 **已暂存（staged）**。
- 已修改表示修改了文件，但还没保存到数据库中。
- 已暂存表示对一个已修改文件的当前版本做了标记，使之包含在下次提交的快照中。
- 已提交表示数据已经安全地保存在本地数据库中。
这会让我们的 Git **项目**拥有三个阶段：工作区、暂存区以及 Git 目录。
![image.png](https://raw.githubusercontent.com/GoodNightmj/PicGo/master/202411271924769.png)
基本的 Git 工作流程如下：

1. 在工作区中修改文件。
    
2. 将你想要下次提交的更改选择性地暂存，这样只会将更改的部分添加到暂存区。
    
3. 提交更新，找到暂存区的文件，将快照永久性存储到 Git 目录。


###  初次运行 Git 前的配置
git拥有git config文件用于保留git控制信息，有三个位置
1. `/etc/gitconfig` 文件: 包含系统上每一个用户及他们仓库的通用配置。 如果在执行 `git config` 时带上 `--system` 选项，那么它就会读写该文件中的配置变量。 （由于它是系统配置文件，因此你需要管理员或超级用户权限来修改它。）
2. `~/.gitconfig` 或 `~/.config/git/config` 文件：只针对当前用户。 你可以传递 `--global` 选项让 Git 读写此文件，这会对你系统上 **所有** 的仓库生效。
3. 当前使用仓库的 Git 目录中的 `config` 文件（即 `.git/config`）：针对该仓库。 你可以传递 `--local` 选项让 Git 强制读写此文件，虽然默认情况下用的就是它。 （当然，你需要进入某个 Git 仓库中才能让该选项生效。）
*优先级依次更高*
可通过`
`git config --list --show-origin`来查看配置和所在文件
![image.png](https://raw.githubusercontent.com/GoodNightmj/PicGo/master/20241203015438.png)



## Git基础

### 获取 Git 仓库

通常有两种获取 Git 项目仓库的方式：

1. 将尚未进行版本控制的本地目录转换为 Git 仓库；git init
    
2. 从其它服务器 **克隆** 一个已存在的 Git 仓库。git clone


如果你想在克隆远程仓库的时候，自定义本地仓库的名字，你可以通过额外的参数指定新的目录名：

```
$ git clone https://github.com/libgit2/libgit2 mylibgit
```

### 记录每次更新到仓库

* 工作目录下的每一个文件都不外乎这两种状态：**已跟踪** 或 **未跟踪**
![image.png](https://raw.githubusercontent.com/GoodNightmj/PicGo/master/202411271905228.png)

已跟踪的文件是指那些被纳入了版本控制的文件，在上一次快照中有它们的记录，在工作一段时间后， 它们的状态可能是未修改，已修改或已放入暂存区。简而言之，已跟踪的文件就是 Git 已经知道的文件。

工作目录中除已跟踪文件外的其它所有文件都属于未跟踪文件，它们既不存在于上次快照的记录中，也没有被放入暂存区。
#### git add
git add这是个多功能命令：
	可以用它开始跟踪新文件
	或者把已跟踪的文件放到暂存区
	还能用于合并时把有冲突的文件标记为已解决状态
 **将这个命令理解为“精确地将内容添加到下一次提交中”而不是“将一个文件添加到项目中”要更加合适。**

#### git status
查看哪些文件处于什么状态-未跟踪、已修改、未修改
 Git 有一个选项可以帮你缩短状态命令的输出，这样可以以简洁的方式查看更改。 如果你使用 `git status -s` 命令或 `git status --short` 命令，你将得到一种格式更为紧凑的输出。

```console
$ git status -s
 M README
MM Rakefile
A  lib/git.rb
M  lib/simplegit.rb
?? LICENSE.txt
```

新添加的未跟踪文件前面有 `??` 标记，新添加到暂存区中的文件前面有 `A` 标记，修改过的文件前面有 `M` 标记。 输出中有两栏，左栏指明了暂存区的状态，右栏指明了工作区的状态。例如，上面的状态报告显示： `README` 文件在工作区已修改但尚未暂存，而 `lib/simplegit.rb` 文件已修改且已暂存。 `Rakefile` 文件已修改，暂存后又作了修改，因此该文件的修改中既有已暂存的部分，又有未暂存的部分。

#### 头指针
即HEAD指针，指向当前所在分支（即所在分支的最新一次提交），如果checkout 到过去的某次提交后，即，当前状态不处于任何一个分支上时，我们称之为**分离头指针** 状态，此时，如果修改代码并提交并不会在任何一个分支上，代码的修改会丢失，因此，这种状态仅仅用于查看即可

#### .gitignore
文件 `.gitignore` 的格式规范如下：

- 所有空行或者以 `#` 开头的行都会被 Git 忽略。
    
- 可以使用标准的 glob 模式匹配，它会递归地应用在整个工作区中。
    
- 匹配模式可以以（`/`）开头防止递归。
    
- 匹配模式可以以（`/`）结尾指定目录。
    
- 要忽略指定模式以外的文件或目录，可以在模式前加上叹号（`!`）取反。
一个 `.gitignore` 文件的例子：

```
# 忽略所有的 .a 文件
*.a

# 但跟踪所有的 lib.a，即便你在前面忽略了 .a 文件
!lib.a

# 只忽略当前目录下的 TODO 文件，而不忽略 subdir/TODO
/TODO

# 忽略任何目录下名为 build 的文件夹
build/

# 忽略 doc/notes.txt，但不忽略 doc/server/arch.txt
doc/*.txt

# 忽略 doc/ 目录及其所有子目录下的 .pdf 文件
doc/**/*.pdf
```


#### git diff

git diff命令比较的是工作目录中当前文件和暂存区域快照之间的差异。 也就是修改之后还没有暂存起来的变化内容。
`git diff --staged`比对已暂存文件与最后一次提交的文件差异


提交时记录的是放在**暂存区域**的快照。 任何还未暂存文件的仍然保持已修改状态，可以在下次提交时纳入版本管理。 每一次运行提交操作，都是对你项目作一次快照，以后可以回到这个状态，或者进行比较。

#### git commit
只要在提交的时候，给 `git commit` 加上 `-a` 选项，Git 就会自动把所有已经跟踪过的文件暂存起来一并提交，从而跳过 `git add` 步骤：
#### git rm
1.删除文件并放弃跟踪：
```console
rm PROJECTS.md
git rm PROJECTS.md
```
使用`git rm`命令从 Git 中移除某个文件，如果要删除之前修改过或已经放到暂存区的文件，则必须使用强制删除选项 `-f`（译注：即 force 的首字母）。 这是一种安全特性，用于防止误删尚未添加到快照的数据，这样的数据不能被 Git 恢复。
2.文件保留在磁盘，但是并不想让 Git 继续跟踪：

```console
$ git rm --cached README
```


#### git mv
如果在 Git 中重命名了某个文件，仓库中存储的元数据并不会体现出这是一次改名操作。

要在 Git 中对文件改名，可以这么做：

```console
$ git mv file_from file_to
```

相当于运行了下面三条命令：

```console
$ mv README.md README
$ git rm README.md
$ git add README
```

如此分开操作，Git 也会意识到这是一次重命名，所以不管何种方式结果都一样。 两者唯一的区别在于，`git mv` 是一条命令而非三条命令，直接使用 `git mv` 方便得多。 不过**在使用其他工具重命名文件时，记得在提交前 `git rm` 删除旧文件名，再 `git add` 添加新文件名。**






### 查看提交历史（git log）
 `git log` 的常用选项

|选项                | 说明                                                                    |
| ----------------- | --------------------------------------------------------------------- |
| `-p`              | 按补丁格式显示每个提交引入的差异。                                                     |
| `--stat`          | 显示每次提交的文件修改统计信息。                                                      |
| `--shortstat`     | 只显示 --stat 中最后的行数修改添加移除统计。                                            |
| `--name-only`     | 仅在提交信息后显示已修改的文件清单。                                                    |
| `--name-status`   | 显示新增、修改、删除的文件清单。                                                      |
| `--abbrev-commit` | 仅显示 SHA-1 校验和所有 40 个字符中的前几个字符。                                        |
| `--relative-date` | 使用较短的相对时间而不是完整格式显示日期（比如“2 weeks ago”）。                                |
| `--graph`         | 在日志旁以 ASCII 图形显示分支与合并历史。                                              |
| `--pretty`        | 使用其他格式显示历史提交信息。可用的选项包括 oneline、short、full、fuller 和 format（用来定义自己的格式）。 |
| `--oneline`       | `--pretty=oneline --abbrev-commit` 合用的简写。                             |


### 撤销操作（后悔药）
![Pasted image 20250403154418](https://raw.githubusercontent.com/GoodNightmj/PicGo/master/Pasted%20image%2020250403154418.png)
#### git reset&& git revert
git reset 即为撤回提交
通过还原成前x次的commit状态，即实现了撤回x+1次之后的commit状态
命令为
`git reset --mix [commit id]`
--mix的意思为保留当前工作区的状态
通过reset命令使本地分支和远程分支不同步（本地分支少了x+1次之后的commit)
通过使用git push -f 命令强制向远程推送

**注意**： 不能在协作分支操作，会影响他人

git revert
当本地的提交已经push到远端，那么就不能使用git reset命令了（使用必然造成git push -f)
此时可以使用revert命令，该命令会再次生成一个commit，会实现revert 指定commit的修改全部复原
**commit --amend 只适用于最新的提交，历史提交不能使用amend**
用于提交完了才发现漏掉了几个文件没有添加，或者提交信息写错了。这种情况下，只会存在一次提交，提交信息为xxx


```console
$ git commit -m 'initial commit' 发现有漏加的文件或者提交信息写错
$ git add forgotten_file
$ git commit --amend -m 'xxx'
```
#### 取消暂存的文件
你已经修改了两个文件并且想要将它们作为两次独立的修改提交， 但是却意外地输入 `git add *` 暂存了它们两个。如何只取消暂存两个中的一个呢？
```
$ git add *
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    renamed:    README.md -> README
    modified:   CONTRIBUTING.md
```
通过提示的命令输入即可
```
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
#### 取消工作区文件的修改（恢复为上次提交状态）
通过`git reset HEAD xx`可以使暂存区的文件进入工作区，那如果我们现在不需要他在工作区的改变，回到刚克隆或上次提交后的情况呢，最后的提示也给了我们解决方案
`$ git checkout -- CONTRIBUTING.md`
#### 保存工作区文件的修改
![image.png](https://raw.githubusercontent.com/GoodNightmj/PicGo/master/202504031500206.png)

### 远程仓库
#### 查看远程仓库
查看远程仓库：显示需要读写远程仓库使用的 Git 保存的简写与其对应的 URL。
```console
$ git remote -v
origin	https://github.com/schacon/ticgit (fetch)
origin	https://github.com/schacon/ticgit (push)
```
#### 添加远程仓库：
1. git clone 默认添加
2. git remote add`git remote add <shortname> <url>`
#### 抓取&&拉取
`git fetch <remote>`
这个命令会访问远程仓库，从中拉取所有你还没有的数据。 执行完成后，你将会拥有那个远程仓库中所有分支的引用，可以随时合并或查看。
使用clone 命令后远程仓库会默认名为origin
**注意**：fetch 不会自动合并你的工作。当设置了当前分支跟踪远程分支后，可以使用git pull 进行抓取和合并
#### 推送
`git push <remote> <branch>`
条件：1.当你有所克隆服务器的写入权限
2.你必须先抓取他们的工作并将其合并进你的工作后才能推送。
#### 查看远程仓库的信息：
`git remote show <remote>`
```console
$ git remote show origin
* remote origin
  URL: https://github.com/my-org/complex-project
  Fetch URL: https://github.com/my-org/complex-project
  Push  URL: https://github.com/my-org/complex-project
  HEAD branch: master
  Remote branches:
    master                           tracked
    dev-branch                       tracked
    markdown-strip                   tracked
    issue-43                         new (next fetch will store in remotes/origin)
    issue-45                         new (next fetch will store in remotes/origin)
    refs/remotes/origin/issue-11     stale (use 'git remote prune' to remove)
  Local branches configured for 'git pull':
    dev-branch merges with remote dev-branch
    master     merges with remote master
  Local refs configured for 'git push':
    dev-branch                     pushes to dev-branch                     (up to date)
    markdown-strip                 pushes to markdown-strip                 (up to date)
    master                         pushes to master                         (up to date)
```

这个命令列出了当你在特定的分支上执行 `git push` 会自动地推送到哪一个远程分支。 它也同样地列出了哪些远程分支不在你的本地，哪些远程分支已经从服务器上移除了， 还有当你执行 `git pull` 时哪些本地分支可以与它跟踪的远程分支自动合并。
#### 重命名和删除
`git remote rm <remote>`
一旦你使用这种方式删除了一个远程仓库，那么所有和这个远程仓库相关的远程跟踪分支以及配置信息也会一起被删除。
`git remote rename <new> <old>`

### [打标签](https://git-scm.com/book/zh/v2/Git-%E5%9F%BA%E7%A1%80-%E6%89%93%E6%A0%87%E7%AD%BE)

### git alias
```console
$ git config --global alias.<to> <from>
```

```console
$ git config --global alias.co checkout
$ git config --global alias.br branch
$ git config --global alias.ci commit
$ git config --global alias.st status
```

这意味着，当要输入 `git commit` 时，只需要输入 `git ci`。 随着你继续不断地使用 Git，可能也会经常使用其他命令，所以创建别名时不要犹豫。

## Git 分支

### 分支简介
例子：
假设现在有一个工作目录，里面包含了三个将要被暂存和提交的文件。 暂存操作会为每一个文件计算校验和（SHA-1 哈希算法），然后会把当前版本的文件快照保存到 Git 仓库中 （Git 使用 _blob_ 对象来保存它们），最终将校验和加入到暂存区域等待提交：
```console
$ git add README test.rb LICENSE
$ git commit -m 'The initial commit of my project'
```
当使用 `git commit` 进行提交操作时，Git 会先计算每一个子目录（本例中只有项目根目录）的校验和， 然后在 Git 仓库中这些校验和保存为树对象。随后，Git 便会创建一个提交对象， 它除了包含上面提到的那些信息外，还包含指向这个树对象（项目根目录）的指针。 如此一来，Git 就可以在需要的时候重现此次保存的快照。
Git 仓库中有五个对象：三个 _blob_ 对象（保存着文件快照）、一个 **树** 对象 （记录着目录结构和 blob 对象索引）以及一个 **提交** 对象（包含着指向前述树对象的指针和所有提交信息）。
![image.png](https://raw.githubusercontent.com/GoodNightmj/PicGo/master/202411281329190.png)
***
做些修改后再次提交，那么这次产生的提交对象会包含一个指向上次提交对象（父对象）的指针。
![image.png](https://raw.githubusercontent.com/GoodNightmj/PicGo/master/202411281415074.png)

#### 分支创建
它只是为你创建了一个可以移动的新的指针。 比如，创建一个 testing 分支， 你需要使用 `git branch` 命令：

`$ git branch testing`这会在当前所在的提交对象上创建一个指针。
![image.png](https://raw.githubusercontent.com/GoodNightmj/PicGo/master/202412041938918.png)
通过HEAD这个特殊指针使Git知道当前是在哪个分支
![image.png](https://raw.githubusercontent.com/GoodNightmj/PicGo/master/202412041939352.png)
#### 分支切换
`$ git checkout testing`来切换一个已经存在的分支
因此Git 的分支实质上仅是包含所指对象校验和（长度为 40 的 SHA-1 值字符串）的文件，所以它的创建和销毁都异常高效。

###  分支的新建与合并
![Pasted image 20250403160404](https://raw.githubusercontent.com/GoodNightmj/PicGo/master/Pasted%20image%2020250403160404.png) 

**当你新建和合并分支的时候，所有这一切都只发生在你本地的 Git 版本库中 —— 没有与服务器发生交互。**

当切换分支的时候，Git 会重置你的工作目录，使其看起来像回到了你在那个分支上最后一次提交的样子。 Git 会自动添加、删除、修改文件以确保此时你的工作目录和这个分支最后一次提交时的样子一模一样。因为本质上分支不过是指向某个commit的指针。
#### 三种合并情况
1. 合并master分支和hotfix分支![image.png](https://raw.githubusercontent.com/GoodNightmj/PicGo/master/202411281420411.png)
```console
$ git checkout master
$ git merge hotfix
Updating f42c576..3a0874c
Fast-forward
 index.html | 2 ++
 1 file changed, 2 insertions(+)
```
由于c4是c2的直接后继，那么 Git 在合并两者的时候， 只会简单的将指针向前推进（指针右移），因为这种情况下的合并操作没有需要解决的分歧——这就叫做 “快进（fast-forward）”。
***
2.合并master分支和iss53分支，对于这种开发历史从一个更早的地方开始分叉开来（diverged）。即`master` 分支所在提交并不是 `iss53` 分支所在提交的直接祖先，Git 不得不做一些额外的工作。 出现这种情况的时候，Git 会使用两个分支的末端所指的快照（`C4` 和 `C5`）以及这两个分支的公共祖先（`C2`），做一个简单的三方合并。
![image.png](https://raw.githubusercontent.com/GoodNightmj/PicGo/master/202411281422880.png)
![image.png](https://raw.githubusercontent.com/GoodNightmj/PicGo/master/202411281425831.png)
和之前将分支指针向前推进所不同的是，Git 将此次三方合并的结果做了一个新的快照并且自动创建一个新的提交指向它。 这个被称作一次合并提交，它的特别之处在于他有不止一个父提交。

***
3.带有冲突的提交，即在两个不同的分支中，对同一个文件的同一个部分进行了不同的修改
这种情况下，git会做出合并但不会进行提交，打开冲突文件会类似
```html
<<<<<<< HEAD:index.html
<div id="footer">contact : email.support@github.com</div>
=======
<div id="footer">
 please contact us at support@github.com
</div>
>>>>>>> iss53:index.html
```
解决冲突，你必须选择使用由 `=======` 分割的两部分中的一个，或者你也可以自行合并这些内容

#### git  merge --squash main
squash 即压缩，将待合并的分支上的提交压缩成一个合并进新分支
![image.png](https://raw.githubusercontent.com/GoodNightmj/PicGo/master/202504101440098.png)


![Pasted image 20250403155500](https://raw.githubusercontent.com/GoodNightmj/PicGo/master/Pasted%20image%2020250403155500.png)
#### git reabse (变基)：一定要push -f
#todo 
疑问是： 假如将一个分支rebase 到另一个分支上后，是否另一个分支就消失了
即git branch 只会出现一个分支
Answer：两个分支，变为线性
***
`git rebase [branch name]`
有两种理解，假设当前分支为A ，branch name 为B
1.从最终效果来看，可以理解为将B分支合并进了A分支
2.A 变基到了B分支，A的地基不要了
优点就是不会生成merge 的提交记录，使历史树更加干净
*** 
git rebase main
![image.png](https://raw.githubusercontent.com/GoodNightmj/PicGo/master/202504101441246.png)

![Pasted image 20250403160015](https://raw.githubusercontent.com/GoodNightmj/PicGo/master/Pasted%20image%2020250403160015.png)
### 分支管理
`git branch`查看当前都有哪些分支
`--merged` 与 `--no-merged` 这两个有用的选项可以过滤这个列表中已经合并或尚未合并到当前分支的分支。 如果要查看哪些分支已经合并到当前分支，可以运行 `git branch --merged`：

###  远程分支
如果你在本地的 `master` 分支做了一些工作，在同一段时间内有其他人推送提交到 `git.ourcompany.com` 并且更新了它的 `master` 分支，这就是说你们的提交历史已走向不同的方向。 但只要你保持不与 `origin` 服务器连接（并拉取数据），你的 `origin/master` 指针就不会移动。
![image.png](https://raw.githubusercontent.com/GoodNightmj/PicGo/master/202411281557861.png)
如果要与给定的远程仓库同步数据，运行 `git fetch <remote>` 命令（在本例中为 `git fetch origin`）。  从中抓取本地没有的数据，并且更新本地数据库，移动 `origin/master` 指针到更新之后的位置。
![image.png](https://raw.githubusercontent.com/GoodNightmj/PicGo/master/202411281557887.png)
假定你有另一个内部 Git 服务器,你可以运行
`git remote add <shortname> <url>`添加一个新的远程仓库引用到当前的项目，

![image.png](https://raw.githubusercontent.com/GoodNightmj/PicGo/master/20241128215218.png)
####  推送
如果希望和别人一起在名为 `serverfix` 的分支上工作，你可以像推送第一个分支那样推送它。 运行 
`git push <remote> <branch>`或`git push <remote> <local branch > <remote branch> ` 相同 
#### 跟踪
其他协作者从服务器上抓取数据时，即
`git fetch origin`他们会在本地生成一个远程分支 `origin/serverfix`，指向服务器的 `serverfix` 分支的引用
可以运行 `git merge origin/serverfix` 将这些工作合并到当前所在的分支。 如果想要在自己的 `serverfix` 分支上工作，可以将其建立在远程跟踪分支之上：
```console
git checkout -b serverfix origin/serverfix
这会给你一个用于工作的本地分支，并且起点位于 `origin/serverfix`。
这个命令=
git checkout --track origin/serverfix
=
git checkout serverfix（如果你尝试检出的分支 (a) 不存在且 (b) 刚好只有一个名字与之匹配的远程分支，那么 Git 就会为你创建一个跟踪分支：）
git pull 相当于fetch + merge
```
如果想要查看设置的所有跟踪分支，可以使用 `git branch` 的 `-vv` 选项。 这会将所有的本地分支列出来并且包含更多的信息，如每一个分支正在跟踪哪个远程分支与本地分支是否是领先、落后或是都有。

```console
$ git branch -vv
  iss53     7e424c3 [origin/iss53: ahead 2] forgot the brackets
  master    1ae2a45 [origin/master] deploying index fix
* serverfix f8674d9 [teamone/server-fix-good: ahead 3, behind 1] this should do it
  testing   5ea463a trying something new
```

这里可以看到 `iss53` 分支正在跟踪 `origin/iss53` 并且 “ahead” 是 2，意味着本地有两个提交还没有推送到服务器上。 也能看到 `master` 分支正在跟踪 `origin/master` 分支并且是最新的。 接下来可以看到 `serverfix` 分支正在跟踪 `teamone` 服务器上的 `server-fix-good` 分支并且领先 3 落后 1， 意味着服务器上有一次提交还没有合并入同时本地有三次提交还没有推送。 最后看到 `testing` 分支并没有跟踪任何远程分支。
#### 删除远程分支
假设你已经通过远程分支做完所有的工作了——也就是说你和你的协作者已经完成了一个特性， 并且将其合并到了远程仓库的 `master` 分支
```console
git push origin --delete serverfix
```
来删除服务器上删除 `serverfix` 分支

### 变基（rebase）
合并对比变基
![image.png](https://raw.githubusercontent.com/GoodNightmj/PicGo/master/202412022058167.png)
*** 
![image.png](https://raw.githubusercontent.com/GoodNightmj/PicGo/master/202412022059678.png)
它的原理是首先找到这两个分支（即当前分支 `experiment`、变基操作的目标基底分支 `master`） 的最近共同祖先 `C2`，然后对比当前分支相对于该祖先的历次提交，提取相应的修改并存为临时文件， 然后将当前分支指向目标基底 `C3`, 最后以此将之前另存为临时文件的修改依序应用。
c4'和c5内容完全一样，只不过跟合并相比，少了一个提交记录
#### 变基的风险
在和同事共同处理一个项目时，如果你进行rebase后进行push ，对方的本地提交历史中会出现远程git库中没有的历史，就会非常麻烦
相反的，你在push操作后，远程git仓库中又会出现对方rebase操作后不愿意看到的一些文件



总的原则是，只对尚未推送或分享给别人的本地修改执行变基操作清理历史， 从不对已推送至别处的提交执行变基操作，这样，你才能享受到两种方式带来的便利。



## 四、服务器上的git

###  协议
#### HTTP 协议
优点
不同的访问方式只需要一个 URL 以及服务器只在需要授权时提示输入授权信息，这两个简便性让终端用户使用 Git 变得非常简单。 相比 SSH 协议，可以使用用户名／密码授权是一个很大的优势，这样用户就不必须在使用 Git 之前先在本地生成 SSH 密钥对再把公钥上传到服务器。 对非资深的使用者，或者系统上缺少 SSH 相关程序的使用者，HTTP 协议的可用性是主要的优势。 与 SSH 协议类似，HTTP 协议也非常快和高效。
另一个好处是 HTTPS 协议被广泛使用，一般的企业防火墙都会允许这些端口的数据通过。

 缺点

在一些服务器上，架设 HTTPS 协议的服务端会比 SSH 协议的棘手一些。 除了这一点，用其他协议提供 Git 服务与智能 HTTP 协议相比就几乎没有优势了。


#### SSH 协议
##### 优势
用 SSH 协议的优势有很多。 首先，SSH 架设相对简单 —— SSH 守护进程很常见，多数管理员都有使用经验，并且多数操作系统都包含了它及相关的管理工具。 其次，通过 SSH 访问是安全的 —— 所有传输数据都要经过授权和加密。 最后，与 HTTPS 协议、Git 协议及本地协议一样，SSH 协议很高效，在传输前也会尽量压缩数据。
##### 缺点
SSH 协议的缺点在于它不支持匿名访问 Git 仓库。 如果你使用 SSH，那么即便只是读取数据，使用者也 **必须** 通过 SSH 访问你的主机， 这使得 SSH 协议不利于开源的项目，毕竟人们可能只想把你的仓库克隆下来查看。 如果你只在公司网络使用，SSH 协议可能是你唯一要用到的协议。 如果你要同时提供匿名只读访问和 SSH 协议，那么你除了为自己推送架设 SSH 服务以外， 还得架设一个可以让其他人访问的服务。

#### Git 协议
##### 优点
目前，Git 协议是 Git 使用的网络传输协议里最快的。 如果你的项目有很大的访问量，或者你的项目很庞大并且不需要为写进行用户授权，架设 Git 守护进程来提供服务是不错的选择。 它使用与 SSH 相同的数据传输机制，但是省去了加密和授权的开销。
##### 缺点
Git 协议缺点是缺乏授权机制。 把 Git 协议作为访问项目版本库的唯一手段是不可取的。 一般的做法里，会同时提供 SSH 或者 HTTPS 协议的访问服务，只让少数几个开发者有推送（写）权限，其他人通过 `git://` 访问只有读权限。 Git 协议也许也是最难架设的。 它要求有自己的守护进程，这就要配置 `xinetd`、`systemd` 或者其他的程序，这些工作并不简单。 它还要求防火墙开放 9418 端口，但是企业防火墙一般不会开放这个非标准端口。 而大型的企业防火墙通常会封锁这个端口。

### 在服务器上搭建 Git
#### 1.把现有仓库导出为裸仓库
```console
$ git clone --bare my_project my_project.git
Cloning into bare repository 'my_project.git'...
done.
```
命令 == `console$ cp -Rf my_project/.git my_project.git`

#### 2.把裸仓库放到服务器上
```console
$ scp -r my_project.git user@git.example.com:/srv/git
```
其他用户就可以进行clone了


#### 3.用户上传ssh公钥到服务器
##### 用户
有的话
默认情况下，用户的 SSH 密钥存储在其 `~/.ssh` 目录下。 进入该目录并列出其中内容，你便可以快速确认自己是否已拥有密钥：

```console
$ cd ~/.ssh
$ ls
authorized_keys2  id_dsa       known_hosts
config            id_dsa.pub
```
没有的话，在Linux/macOs上使用
`$ ssh-keygen -o`进行生成
##### 服务器
我们将使用 `authorized_keys` 方法来对用户进行认证。
将这些公钥加入系统用户 `git` 的 `.ssh` 目录下 `authorized_keys` 文件的末尾：
例如：
```console
$ cat /tmp/id_rsa.john.pub >> ~/.ssh/authorized_keys
$ cat /tmp/id_rsa.josie.pub >> ~/.ssh/authorized_keys
$ cat /tmp/id_rsa.jessica.pub >> ~/.ssh/authorized_keys
```




## 分布式Git


### 分布式工作流程

#### 集中式工作流
![image.png](https://raw.githubusercontent.com/GoodNightmj/PicGo/master/202412052052700.png)
如果两个开发者从中心仓库克隆代码下来，同时作了一些修改，那么只有第一个开发者可以顺利地把数据推送回共享服务器。 第二个开发者在推送修改之前，必须先将第一个人的工作合并进来，这样才不会覆盖第一个人的修改。

#### 集成管理者工作流
1. 项目维护者推送到主仓库。
    
2. 贡献者克隆此仓库，做出修改。
    
3. 贡献者将数据推送到自己的公开仓库。
    
4. 贡献者给维护者发送邮件，请求拉取自己的更新。
    
5. 维护者在自己本地的仓库中，将贡献者的仓库加为远程仓库并合并修改。
    
6. 维护者将合并后的修改推送到主仓库。




## 相关资源
[1.常用命令](https://juejin.cn/post/6978812726411788295#heading-2)


## b站视频备忘录
