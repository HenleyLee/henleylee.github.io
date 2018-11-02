---
title: Git 介绍与常用命令
date: 2018-08-08 18:45:35
top: true
categories: 工具
tags:
  - Git
  - 工具
---

![Git](https://user-gold-cdn.xitu.io/2018/11/2/166d3058fc3e4c8c?w=455&h=190&f=jpeg&s=10850)

## 一、Git是什么？ ##
 - Git是一个开源的分布式版本控制系统，用于敏捷高效地处理任何或小或大的项目。
 - Git 是 Linus Torvalds 为了帮助管理 Linux 内核开发而开发的一个开放源码的版本控制软件。
 - Git 与常用的版本控制工具 CVS, Subversion 等不同，它采用了分布式版本库的方式，不必服务器端软件支持。

## 二、Git与SVN区别 ##
GIT不仅仅是个版本控制系统，它也是个内容管理系统(CMS),工作管理系统等。

如果你是一个具有使用SVN背景的人，你需要做一定的思想转换，来适应GIT提供的一些概念和特征。
Git 与 SVN 区别点：
1. GIT是分布式的，SVN不是：这是GIT和其它非分布式的版本控制系统，例如SVN，CVS等，最核心的区别。
2. GIT把内容按元数据方式存储，而SVN是按文件：所有的资源控制系统都是把文件的元信息隐藏在一个类似.svn,.cvs等的文件夹里。
3. GIT分支和SVN的分支不同：分支在SVN中一点不特别，就是版本库中的另外的一个目录。
4. GIT没有一个全局的版本号，而SVN有：目前为止这是跟SVN相比GIT缺少的最大的一个特征。
5. GIT的内容完整性要优于SVN：GIT的内容存储使用的是SHA-1哈希算法。这能确保代码内容的完整性，确保在遇到磁盘故障和网络问题时降低对版本库的破坏。

## 三、Git快速入门 ##
Git官网：[git-scm.com](https://git-scm.com/)

Git完整命令手册：[git-scm.com/docs](https://git-scm.com/docs)

Git命令手册(pdf版)：[github-git-cheat-sheet.pdf](http://www.runoob.com/manual/github-git-cheat-sheet.pdf)

Git简明指南：[git-guide](http://www.runoob.com/manual/git-guide/)

## 四、Git安装 ##
在使用Git前我们需要先安装 Git。Git 目前支持 Linux/Unix、Solaris、Mac和 Windows 平台上运行。
Git 各平台安装包下载地址为：[git-scm.com/downloads](https://git-scm.com/downloads)

## 五、Git配置 ##
Git 提供了一个叫做 git config 的工具，专门用来配置或读取相应的工作环境变量。

### 5.1 配置文件的存储位置 ###
这些环境变量，决定了 Git 在各个环节的具体工作方式和行为。这些变量可以存放在以下三个不同的地方：
 * /etc/gitconfig 文件：系统中对所有用户都普遍适用的配置。若使用 git config 时用 --system 选项，读写的就是这个文件。
 * ~/.gitconfig 文件：用户目录下的配置文件只适用于该用户。若使用 git config 时用 --global 选项，读写的就是这个文件。
 * 当前项目的 Git 目录中的配置文件（也就是工作目录中的 .git/config 文件）：这里的配置仅仅针对当前项目有效。每一个级别的配置都会覆盖上层的相同配置，所以 .git/config 里的配置会覆盖 /etc/gitconfig 中的同名变量。

在 Windows 系统上，Git 会找寻用户主目录下的 .gitconfig 文件。主目录即 $HOME 变量指定的目录，一般都是 C:\Users\$USER。此外，Git 还会尝试找寻 /etc/gitconfig 文件，只不过看当初 Git 装在什么目录，就以此作为根目录来定位。 

### 5.2 查看git的版本信息 ###
查看git的版本信息：
```shell
git --version
```

### 5.3 配置用户信息 ###
当Git安装完成后首先要做的事情是配置个人的用户名称和电子邮件地址。这是非常重要的，因为每次Git提交都会使用该信息。它被永远的嵌入到了你的提交中：
```shell
git config --global user.name 'user_name'
git config --global user.email 'user_email'
```
具体可参考 [Git Configuration](https://git-scm.com/book/en/v2/Customizing-Git-Git-Configuration)

### 5.4 配置文本编辑器 ###
设置Git默认使用的文本编辑器, 一般可能会是 Vi 或者 Vim。如果你有其他偏好，比如 Emacs 的话，可以重新设置：
```shell
git config --global core.editor emacs
```

### 5.5 配置差异分析工具 ###
还有一个比较常用的是，在解决合并冲突时使用哪种差异分析工具。比如要改用 vimdiff 的话：
```shell
git config --global merge.tool vimdiff
```

### 5.6 查看配置信息 ###
要检查已有的配置信息，可以使用 git config --list 命令：
```shell
git config --list
```
 有时候会看到重复的变量名，这是因为Git从不同的的配置文件中(例如：/etc/gitconfig以及~/.gitconfig)读取相同的变量名。在这种情况下，对每个唯一的变量名，Git使用最后的那个值。
 
 也可以直接查阅某个环境变量的设定，使用如下命令 git config {key}： 
```shell
git config user.name
```

也可以直接查看某个配置文件的配置信息：
```shell
git config --global -l
or
git config --system -l
```

### 5.7 git 配置文件 ###
1. 系统级文件 $(prefix)/etc/gitconfig，本文即 /usr/etc/gitconfig 文件。
git config --system 用来指定读写系统级文件。初始不存在，若不存在则无影响。
2. 用户级文件 ~/.gitconfig
git config --global 指定只操作用户级文件。初始不存在，若不存在则无影响。
3. Repository 级文件 .git/config
git config --local 对写操作，则只写入 Repository 级文件（默认行为）；对读操作，则只从 Repository 级文件读。
4. git config --file config-file 则指定 config-file。

### 5.8 清除认证信息 ###
```shell
git config --global --unset credential.helper
or
git config --system --unset credential.helper
```
上面的命令可以解决 `remote: HTTP Basic: Access denied` 错误。

### 5.9 保存认证信息 ###
```shell
git config --local credential.helper store
or
git config --system credential.helper store
```
上面的命令可以解决每次提交都要输入用户名和密码的问题。具体可参考 [Git Credential Storage](https://git-scm.com/book/en/v2/Git-Tools-Credential-Storage)

## 六、Git常用命令 ##
### 6.1 创建代码仓库 ###
#### 6.1.1 git init ####
`git init` 的命令格式为：
```shell
git init [-q | --quiet] [--bare] [--template=<template_directory>]
      [--separate-git-dir <git dir>]
      [--shared[=<permissions>]] [directory]
```

第一步：创建一个代码仓库非常简单，首先，选择一个合适的地方，创建一个空目录：
```shell
mkdir repository
cd repository
pwd
```
`mkdir`命令用于创建目录，`cd`命令用于进入目录，`pwd`命令用于显示当前目录。
第二步，通过`git init`命令把这个目录变成Git可以管理的仓库或者通过`git init <directory>`命令直接指定一个目录作为Gi仓库：
```shell
git init
git init <directory>
```
Git瞬间就把仓库建好了，而且告诉你是一个空的仓库(empty Git repository)，细心的读者可以发现当前目录下多了一个`.git`的目录，这个目录是Git来跟踪管理版本库的，没事千万不要手动修改这个目录里面的文件，不然改乱了，就把Git仓库给破坏了。如果你没有看到`.git`目录，那是因为这个目录默认是隐藏的，用ls -ah命令就可以看见。

#### 6.1.2 git clone ####
`git clone` 的命令格式为：
```shell
git clone [--template=<template_directory>]
      [-l] [-s] [--no-hardlinks] [-q] [-n] [--bare] [--mirror]
      [-o <name>] [-b <name>] [-u <upload-pack>] [--reference <repository>]
      [--dissociate] [--separate-git-dir <git dir>]
      [--depth <depth>] [--[no-]single-branch]
      [--recurse-submodules] [--[no-]shallow-submodules]
      [--jobs <n>] [--] <repository> [<directory>]
```

克隆到当前目录，可以使用以下命令格式：
```shell
git clone <repository>
```
如果我们需要克隆到指定的目录，可以使用以下命令格式：
```shell
git clone <repository> <directory>
```
参数说明：
 - `repository`:Git 仓库。
 - `directory`:本地目录。

### 6.2 添加暂存区 ###
`git add` 的命令格式为：
```shell
git add [--verbose | -v] [--dry-run | -n] [--force | -f] [--interactive | -i] [--patch | -p]
      [--edit | -e] [--[no-]all | --[no-]ignore-removal | [--update | -u]]
      [--intent-to-add | -N] [--refresh] [--ignore-errors] [--ignore-missing]
      [--chmod=(+|-)x] [--] [<pathspec>…​]
```

`git add`命令可以在提交之前多次执行。它只在运行`git add`命令时添加指定文件的内容; 如果希望随后的更改包含在下一个提交中，那么必须再次运行`git add`将新的内容添加到索引。默认情况下，`git add`命令不会添加忽略的文件。

#### 6.2.1 基本用法 ####
```shell
git add [path]
```
通常是通过`git add [path]`的形式把`[path]`添加到索引库中，`[path]`可以是文件也可以是目录。
git不仅能判断出`[path]`中，修改(不包括已删除)的文件，还能判断出新添的文件，并把它们的信息添加到索引库中。

#### 6.2.2 常用命令 ####
```shell
git add .               # 将所有修改添加到暂存区
git add *Presenter      # 将以Presenter结尾的文件的所有修改添加到暂存区
git add Base*           # 将所有以Base开头的文件的修改添加到暂存区(例如:BaseActivity.java,BaseFragment.java)
git add Model?          # 将以Model开头且后面只有一位的文件的修改添加到暂存区(例如:Model1.java,ModelA.java)
git add model/*.java    # 将model目录及其子目录下所有*.java文件的修改添加到暂存区
```

### 6.3 代码提交 ###
`git commit` 的命令格式为：
```shell
git commit [-a | --interactive | --patch] [-s] [-v] [-u<mode>] [--amend]
       [--dry-run] [(-c | -C | --fixup | --squash) <commit>]
       [-F <file> | -m <msg>] [--reset-author] [--allow-empty]
       [--allow-empty-message] [--no-verify] [-e] [--author=<author>]
       [--date=<date>] [--cleanup=<mode>] [--[no-]status]
       [-i | -o] [-S[<keyid>]] [--] [<file>…​]
```

`git commit`命令将索引的当前内容与描述更改的用户和日志消息一起存储在新的提交中。

#### 6.3.1 提交暂存区到本地仓库区 ####
```shell
git commit -m [message]
```

#### 6.3.2 将未添加到暂存区的文件，同时提交到本地仓库区 ####
```shell
git commit –am <message>
git commit –a –m <message>
```

#### 6.3.3 提交暂存区的指定文件到仓库区 ####
```shell
git commit <file1> <file2> ... -m <message>
```

#### 6.3.4 提交时显示所有diff信息 ####
```shell
git commit -v
```

#### 6.3.5 修改最近一次提交 ####
```shell
git commit --amend
```

#### 6.3.6 修改最近一次提交，并改写上一次commit的提交信息 ####
```shell
git commit --amend -m <message>
```

### 6.4 分支管理 ###
`git branch` 的命令格式为：
```shell
git branch [--color[=<when>] | --no-color] [-r | -a]
    [--list] [-v [--abbrev=<length> | --no-abbrev]]
    [--column[=<options>] | --no-column] [--sort=<key>]
    [(--merged | --no-merged) [<commit>]]
    [--contains [<commit]] [--no-contains [<commit>]]
    [--points-at <object>] [--format=<format>] [<pattern>…​]
git branch [--set-upstream | --track | --no-track] [-l] [-f] <branchname> [<start-point>]
git branch (--set-upstream-to=<upstream> | -u <upstream>) [<branchname>]
git branch --unset-upstream [<branchname>]
git branch (-m | -M) [<oldbranch>] <newbranch>
git branch (-d | -D) [-r] <branchname>…
git branch --edit-description [<branchname>]
```

`git branch`命令用于列出，创建或删除分支。

#### 6.4.1 列出所有本地分支 ####
```shell
git branch
```

#### 6.4.2 列出所有远程分支 ####
```shell
git branch -r
```

#### 6.4.3 列出所有本地分支和远程分支 ####
```shell
git branch -a
```

#### 6.4.4 新建一个分支，但依然停留在当前分支 ####
```shell
git branch <branch-name>
```

#### 6.4.5 新建一个分支，并切换到该分支 ####
```shell
git checkout -b <branch-name>
```
`git checkout` 命令加上 `-b` 参数表示创建并切换，相当于以下两条命令：
```shell
git branch <branch-name>
git checkout <branch-name>
```

#### 6.4.6 新建一个分支，指向指定commit ####
```shell
git branch <branch> <commit>
```

#### 6.4.7 新建一个分支，与指定的远程分支建立追踪关系 ####
```shell
git branch --track <branch> <remote-branch>
```

#### 6.4.8 切换到指定分支，并更新工作区 ####
```shell
git checkout <branch-name>
```

#### 6.4.9 切换到上一个分支 ####
```shell
git checkout -
```

#### 6.4.10 建立追踪关系，在现有分支与指定的远程分支之间 ####
```shell
git branch --set-upstream <branch> <remote-branch>
```

#### 6.4.11 合并指定分支到当前分支 ####
```shell
git merge <branch>
```

#### 6.4.12 选择一个commit，合并进当前分支 ####
```shell
git cherry-pick <commit>
```

#### 6.4.13 删除本地分支 ####
```shell
git branch -d <branch-name>
```

#### 6.4.14 删除远程分支 ####
```shell
git push origin :<branch-name>
或
git push origin --delete <branch-name>
```

### 6.5 标签管理 ###
`git tag` 的命令格式为：
```shell
git tag [-a | -s | -u <keyid>] [-f] [-m <msg> | -F <file>]
    <tagname> [<commit> | <object>]
git tag -d <tagname>…
git tag [-n[<num>]] -l [--contains <commit>] [--no-contains <commit>]
    [--points-at <object>] [--column[=<options>] | --no-column]
    [--create-reflog] [--sort=<key>] [--format=<format>]
    [--[no-]merged [<commit>]] [<pattern>…]
git tag -v [--format=<format>] <tagname>…

```

`git tag`命令用于创建，列出，删除或验证使用GPG签名的标签对象。

#### 6.5.1 列出所有标签 ####
```shell
git tag
```

#### 6.5.2 新建一个标签在当前commit ####
```shell
git tag <tag-name>
```

#### 6.5.3 新建一个标签在指定commit ####
```shell
git tag <tag-name> <commit>
```

#### 6.5.4 新建一个带标签信息的标签在当前commit ####
```shell
git tag -a <tag-name> -m [message]
```

#### 6.5.5 删除本地标签 ####
```shell
git tag -d <tag-name>
```

#### 6.5.6 删除远程标签 ####
方法一：直接删除远程标签：
```shell
git push origin --delete tag <tag-name>
```
方法二：先删除本地标签，再删除远程标签：
```shell
git tag -d <tag-name>
git push origin :refs/tags/<tag-name>
```

#### 6.5.7 查看标签信息 ####
```shell
git show <tag-name>
```

#### 6.5.8 推送某个标签到远程 ####
```shell
git push origin <tag-name>
```

#### 6.5.9 一次性推送所有尚未推送到远程的本地标签 ####
```shell
git push origin --tags
```

### 6.6 查看信息 ###
#### 6.6.1 查看Git版本号 ####
```shell
git --version
```

#### 6.6.2 显示有变更的文件 ####
```shell
git status
```

#### 6.6.3 显示当前分支的提交历史记录 ####
下表介绍了一些 git log 命令支持的一些常用的选项及其释义：
| 选项            |  说明                             |
| --------------- | :-------------------------------- |
| -p 	          | 按补丁格式显示每个更新之间的差异。 |
| --word-diff     | 按 word diff 格式显示差异。 |
| --stat 	      | 显示每次更新的文件修改统计信息。 |
| --shortstat     | 只显示 --stat 中最后的行数修改添加移除统计。 |
| --name-only 	  | 仅在提交信息后显示已修改的文件清单。 |
| --name-status   | 显示新增、修改、删除的文件清单。 |
| --abbrev-commit | 仅显示 SHA-1 的前几个字符，而非所有的 40 个字符。 |
| --relative-date | 使用较短的相对时间显示(比如，“2 weeks ago”)。 |
| --graph 	      | 显示 ASCII 图形表示的分支合并历史。 |
| --pretty 	      | 使用其他格式显示历史提交信息。可用的选项包括 oneline，short，full，fuller 和 format(后跟指定格式)。|
| --oneline 	  | --pretty=oneline --abbrev-commit 的简化用法。 |
默认不用任何参数的话，`git log` 会按提交时间列出所有的更新，最近的更新排在最上面。可以看到，每次更新都有一个 SHA-1 校验和、作者的名字和电子邮件地址、提交时间，最后缩进一个段落显示提交说明。
```shell
git log
```
`git log` 有许多选项可以帮助你搜寻感兴趣的提交，接下来我们介绍些最常用的。

我们常用 `-p` 选项展开显示每次提交的内容差异，用 `-<n>` 则仅显示最近的两次更新：
```shell
git log -p -2
```
该选项除了显示基本信息之外，还在附带了每次 commit 的变化。当进行代码审查，或者快速浏览某个搭档提交的 commit 的变化的时候，这个参数就非常有用了。

`--stat`，仅显示简要的增改行数统计。
```shell
git log --stat
```

`--pretty` 选项，可以指定使用完全不同于默认格式的方式展示提交历史。比如用 `oneline` 将每个提交放在一行显示，这在提交数很大时非常有用。另外还有 `short`，`full` 和 `fuller` 可以用，展示的信息或多或少有些不同，后面也可以指定提交历史的次数(比如：`-<n>` )，具体展示效果请自己动手实践一下。
```shell
git log --pretty=oneline
```

 `format`选项，可以定制要显示的记录格式，这样的输出便于后期编程提取分析。
```shell
git log --pretty=format:"%h - %an, %ar : %s"
```
下表列出了常用的格式占位符写法及其代表的意义：
| 选项   |  说明  |
| ------ | :---- |
| %H     |   提交对象(commit)的完整哈希字串    |
| %h     |   提交对象的简短哈希字串   |
| %T     |   树对象(tree)的完整哈希字串    |
| %t     |   树对象的简短哈希字串    |
| %P     |   父对象(parent)的完整哈希字串    |
| %p     |   父对象的简短哈希字串    |
| %an    |   作者(author)的名字    |
| %ae    |   作者的电子邮件地址    |
| %ad    |   作者修订日期(--date= 制定的格式)    |
| %aD    |   作者修订日期(RFC2822格式)    |
| %ar    |   作者修订日期(相对格式，如：1 day ago)   |
| %at    |   作者修订日期(UNIX timestamp)  |
| %ai    |   作者修订日期(ISO 8601 格式)   |
| %cn    |   提交者(committer)的名字    |
| %ce    |   提交者的电子邮件地址    |
| %cd    |   提交日期 (--date= 制定的格式)    |
| %cD    |   提交日期(RFC2822格式)    |
| %cr    |   提交日期(相对格式，如：1 day ago)    |
| %ct    |   提交日期(UNIX timestamp)    |
| %ci    |   提交日期(ISO 8601 格式)    |
| %s     |   提交说明    |

除了定制输出格式的选项之外，`git log` 还有许多非常实用的限制输出长度的选项，也就是只输出部分提交信息。用 --since 和 --until选项显示按照时间作限制的提交，比如说具体的某一天（“2008-01-15”），或者是多久以前（“2 years 1 day 3 minutes ago”）。用 --author 选项显示指定作者的提交，用 --grep 选项搜索提交说明中的关键字。
下表还列出了其他常用的类似选项：
| 选项   |  说明  |
| ------ | :---- |
| -(n)              | 仅显示最近的 n 条提交 |
| --since, --after  | 仅显示指定时间之后的提交 |
| --until, --before | 仅显示指定时间之前的提交 |
| --author          | 仅显示指定作者相关的提交 |
| --committer       | 仅显示指定提交者相关的提交 |
 	
### 6.7 远程同步 ###
#### 6.7.1 显示所有远程仓库 ####
```shell
git remote -v
```

#### 6.7.2 获取某个远程主机的全部更新 ####
```shell
git fetch <remote>
```

#### 6.7.3 获取某个远程主机的某个分支的更新 ####
```shell
git fetch <remote> <branch>
```
比如，取回origin主机的master分支的更新：
```shell
git fetch origin master
```

#### 6.7.4 显示某个远程仓库的信息 ####
```shell
git remote show <remote>
```

#### 6.7.5 获取某个远程主机的某个分支的更新与当前分支合并 ####
```shell
git pull <remote> <remote-branch>
```
比如，要取回origin主机的dev分支，与当前分支合并：
```shell
git pull origin dev
```
上面命令表示，取回origin/dev分支，再与当前分支合并。实质上，这等同于先做git fetch，再执行git merge。
```shell
git fetch origin
git merge origin/dev
```

#### 6.7.6 获取某个远程主机的某个分支的更新与本地的某个分支合并 ####
```shell
git pull <remote> <remote-branch>:<local-branch>
```
比如，要取回origin主机的dev分支，与本地的master分支合并：
```shell
git pull origin dev:master
```

#### 6.7.7 将本地的当前分支自动与对应的远程主机”追踪分支”进行合并 ####
```shell
git pull <remote>
```

#### 6.7.8 将当前分支推送到远程主机的对应分支 ####
```shell
git push
```
#### 6.7.9 将当前分支到远程主机的对应分支 ####
```shell
git push <remote>
```

#### 6.7.10 将本地指定分支到远程主机的对应分支 ####
```shell
git push <remote> <branch>
```

#### 6.7.11 强行推送当前分支到远程主机的对应分支(忽略冲突) ####
```shell
git push <remote> --force
```

#### 6.7.12 推送所有分支到远程仓库 ####
```shell
git push <remote> --all
```

### 6.8 代码回滚 ###
#### 6.8.1 恢复暂存区的指定文件到工作区 ####
```shell
git checkout <file-name>
```

#### 6.8.2 恢复某个commit的指定文件到暂存区和工作区 ####
```shell
git checkout <commit> <file-name>
```

#### 6.8.3 恢复暂存区的所有文件到工作区 ####
```shell
git checkout .
```

#### 6.8.4 回滚添加操作 ####
```shell
git reset
```

#### 6.8.5 回滚最近一次提交 ####
```shell
git reset --soft HEAD^
```

#### 6.8.6 永久删除最后几个提交 ####
```shell
git reset --hard HEAD~3
```

## 七、 Command line instructions ##
### 7.1 Git global setup ###
```shell
git config --global user.name "user-name"
git config --global user.email "user-email"
```

### 7.2 Create a new repository ###
```shell
git clone <remote-repository>
cd <directory>
touch README.md
git add README.md
git commit -m "add README"
git push -u origin master
```

### 7.3 Existing folder ###
```shell
cd <existing-folder>
git init
git remote add origin <remote-repository>
git add .
git commit -m "Initial commit"
git push -u origin master
```

### 7.4 Existing Git repository ###
```shell
cd <existing-repository>
git remote rename origin old-origin
git remote add origin <remote-repository>
git push -u origin --all
git push -u origin --tags
```

## 八、致谢 ##
1. [Git教程](https://git-scm.com/book/zh/)
2. [易百教程](https://www.yiibai.com/git/)
3. [菜鸟教程](http://www.runoob.com/git/git-tutorial.html)
