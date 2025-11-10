---
title: Git学习
date: 2024-02-23 11:14 
description: 本文学习了Git版本控制的基本概念和常用命令，包括工作区与暂存区的关系、分支管理策略、远程仓库的协作流程，以及撤销修改、标签管理等实用操作。
categories:
- Git
tags:
---
<head>
  <meta name="referrer" content="no-referrer" />
</head>

# Git 概念

## 集中式VS分布式

- 集中式的版本库是集中存放在中央服务器的
- 分布式的版本库存在每一个人的电脑中，每个人的电脑都是一个完整的版本库
- 分布式版本控制系统更安全
- 分布式版本控制系统通常也有一台充当“中央服务器”的电脑，但这只是为了方便“交换”大家的修改，没有它并不影响正常工作

## 文件忽略

当我们希望有些文件无需纳入 Git 的管理，也不希望它们总出现在未跟踪文件列表。就可以在仓库根目录下编写 `.gitignore`文件，文件内容支持正则表达式，下面是一个例子：

```bash
.DS_Store
Thumbs.db
db.json
*.log
node_modules/
public/
.deploy*/
_multiconfig.yml
```

> [!TIP]
>
> GitHub 有一个十分详细的针对数十种项目及语言的 `.gitignore` 文件列表， 你可以在 https://github.com/github/gitignore 找到它

## 工作区，版本库和暂存区

![image.png](https://cdn.nlark.com/yuque/0/2024/png/35312186/1706514518606-00b9fca9-4c60-4e47-adc0-a01b510976ef.png#averageHue=%23ececec&clientId=ub7619b40-6c08-4&from=paste&height=255&id=u0b073c97&originHeight=447&originWidth=889&originalType=binary&ratio=1.75&rotation=0&showTitle=false&size=119123&status=done&style=none&taskId=ud973f9ac-ffeb-493d-9fa0-3cad8aacf0f&title=&width=508)

> 其中的stage称为是暂存区

## 子模块

`.gitmodules` 文件是用于管理 Git 子模块（submodules）的配置文件。当你在 Git 仓库中嵌入另一个 Git 仓库（即子模块）时，`.gitmodules` 文件会记录子模块的相关信息，比如子模块的路径和来源。

添加子模块命令：

```bash
git submodule add https://github.com/example/example-submodule.git path/to/submodule
```

然后git会自动生成 `.gitmodules`文件，并将子模块信息写入其中。

`.gitmodules` 文件的内容可能如下所示：

```bash
[submodule "path/to/submodule"]
    path = path/to/submodule
    url = https://github.com/example/example-submodule.git
	#branch = main 还可以自己指定分支
```

# Git流程

## 1. 创建版本库

- 版本库又称为仓库(repository)，仓库里面的所有文件都可以被Git管理起来，每个文件的增删改查都被Git跟踪
- 初始化当前文件夹为仓库：`git init`
- 添加文件到Git仓库，分两步：
  1. `git add <file>`。可以多次使用命令，多次添加文件
  2. `git commmit -m <message>`

## 2. 添加到远程库

在Github/Gitee中添加一个仓库，然后根据Github/Gitee的提示，在本地的仓库下运行

```shell
git remote add origin git@gitee.com:Marches7/仓库名.git # 可以换成https链接
```

上述命令的含义是添加一个名为origin，链接为...的远程仓库，方便我们使用名称来指定远程仓库。

> 当你的仓库是git clone下来的，则会自动添加remote信息，默认名称是origin。

## 3. 多人协作

### 3.1 抓取分支

多人协作时，大家都会往master和dev分支上推送各自的修改。模拟一个新的小伙伴协作开发。

1. 使用git clone将仓库克隆到自己的电脑。默认情况下只能看到本地的master分支。
2. 要在dev分支上开发，就必须创建远程origin的dev分支到本地：`git checkout -b dev origin/dev`
3. 现在就可以在dev分支上开发了，时不时可以把dev分支push到远程

### 3.2 推送分支到远程库

推送本地分支到远程库：`git push <remote> <branch>`

> 远程库的名字就是origin，这是Git默认的叫法，也可以改成别的，但是origin这个名字一看就知道是远程库。
>
> ```bash
> git clone <repo> -o booyah
> ```
>
> 这样远程仓库名称就是booyah

### 3.3 流程

1. 首先，可以试图用 `git push origin <branch-name>`推送自己的修改
2. 如果推送失败，则因为远程分支比你的本地更新，需要先用 `git pull`试图合并
3. 如果合并有冲突，则解决冲突，并在本地提交
4. 没有冲突或者解决掉冲突后，再用 `git push origin <branch-name>`推送就能成功

# Git命令详解

## Git配置

- 因为Git是分布式版本控制系统，所以每台机器都必须指定自己的名字和邮件地址

  ```bash
  git config --global user.name "Your Name"
  git config --global user.email "email@example.com"
  ```
- 查看已有配置和它们所在的文件

  ```bash
  git config --list --show-origin
  ```

  输出可能会有重复的变量名，因为Git会从不同的文件中读取同一个配置（例如：/etc/gitconfig与~/.gitconfig）。这种情况下，Git会使用它找到的每一个变量的最后一个配置。可以通过输入 `git config <key>`来检查某一项配置：

  ```bash
  $ git config user.name
  John Doe
  ```

## 查看提交记录和状态

- 查看所有提交日志：

  ```bash
  git log
  ```

  或者简洁输出（每个提交一行）：

  ```bash
  git log --oneline
  ```
- 查看当前git 分支状态：

  ```bash
  git status # 复杂输出版
  git status -s # 简化输出版
  ```

## 查看历史提交

```bash
git checkout <commit hash>
```

回主分支：

```bash
git checkout main
```

回其他分支：

```bash
git log --all --oneline
git checkout <commit hash>
```

> 因为git checkout相当于穿越回去了，所以单纯使用git log只能看到所在提交之前的所有提交，要加上--all参数才能看到所有提交。

## 撤销操作

### 撤销修改

- 当你改乱了工作区某个文件的内容，想直接丢弃工作区的修改时：

  - `git checkout -- <file>`，会有两种情况：
    1. 一种是readme.txt自修改后还没有被放到暂存区，现在，撤销修改就回到和版本库一模一样的状态。
    2. 一种是readme.txt已经添加到暂存区后，又作了修改，现在，撤销修改就回到添加到暂存区后的状态。
- 当你不但改乱了工作区某个文件的内容，还添加到了暂存区时，想丢弃修改：

  - 第一步：`git reset HEAD <file>`
  - 第二步：`git checkout -- <file>`
- 已经提交了不合适的修改到版本库时，想要撤销本次提交，前提是没有提交到远程库：

  - 修改后重新提交（覆盖上一次提交）：`git commit --amend`

    > 这种方法的价值是可以稍微改进你的最后的提交，而不会让小修补这样的垃圾提交信息弄乱你的仓库历史。
    >
  - 回退到之前某个的版本：`git reset --hard HEAD^`或者 `git reset --hard <完整的comit id>`
- 如果已经提交到了远程库，并且希望远程库也回退

  - 本地回退：`git reset --hard HEAD^`或者 `git reset --hard <完整的comit id>`
  - 远程回退（强制push）：`git push --force`，这里采用默认的分支

git reset: 这是 Git 中的主要命令之一，用于重置当前 HEAD 到指定状态。
--hard: 这是 git reset 命令的一个选项，用于指定重置的模式。在 --hard 模式下，Git 将重置工作目录和索引（暂存区）以匹配你要重置到的提交。这意味着所有自上次提交以来对工作目录和索引的更改都将被丢弃。
HEAD^: 这指定了重置操作的目标提交。HEAD 是一个指向你当前所在分支的最新提交的指针。HEAD^（或者 HEAD~1）表示当前分支的父提交，即当前提交的前一个提交。HEAD^^（或者HEAD~2）表示当前分支的前前个提交。

> [!TIP]
>
> 在Git中任何已提交的东西几乎总是可以恢复的，但是未提交的东西丢失后很可能就无法恢复了。

### 撤销合并

在发生合并冲突或错误时采取中止合并而不是解决冲突时，执行下列命令，将仓库恢复到合并操作之前的状态：

```bash
git merge --abort
```

## 分支管理

因为创建、合并和删除分支非常快，所以Git鼓励你使用分支完成某个任务，合并后再删掉分支，这和直接在master分支上工作效果是一样的，但过程更安全。

- 查看分支：`git branch`
- 创建并切换到分支dev：`git checkout -b dev`
- 切换分支：`git switch <branch name>`
- 合并某分支到**当前分支**：`git merge <branch name>`
- 本地删除分支（删除的时候不能在当前分支）：`git branch -d <branch name>`
- 推送分支到远程库：`git push origin <branch>`

  > 此举会在远程库创建一个新的分支
  >
- 删除远程库分支：`git push origin --delete <branch name>`

> [!TIP]
>
> 分支策略的几个原则：
>
> 只在 `master`分支上保留完全稳定的代码，一些其他develop平行分支，被用来做后续开发或者测试稳定性，待到时机成熟，它们就可以被合并入master分支了。

只在 `master`

![image.png](https://cdn.nlark.com/yuque/0/2024/png/35312186/1706517597971-fed08c2c-03b2-43aa-88e9-d22ab1e743d7.png#averageHue=%23f7f5f4&clientId=ub7619b40-6c08-4&from=paste&height=147&id=ubedd3c26&originHeight=257&originWidth=997&originalType=binary&ratio=1.75&rotation=0&showTitle=false&size=72620&status=done&style=none&taskId=u4852a420-aea6-491c-ba1a-f50dddf2816&title=&width=569.7142857142857)

## 暂存分支内容

当前分支的工作还未做完，需要新建一个Bug分支处理一个紧急Bug， 可以使用 `git stash`可以把当前工作现场“储藏”起来，等以后恢复现场后继续工作。
Bug修复完成后，需要恢复现场，有两种方法：

- 一是用git stash apply恢复，但是恢复后，stash内容并不删除，你需要用git stash drop来删除
- 二是用git stash pop，恢复的同时把stash内容也删了

但是之后会想到一个问题，我们修改的Bug在main分支上，意味着我们工作的分支也有Bug，有必要重复修改本分支的Bug再合并到当前分支上吗？有没有更简单的方法？
为了方便操作，避免重复矛盾，Git专门提供了一个 `git cherry-pick 4c805e2`命令，让我们能复制一个特定的提交到当前分支，4c805e2是Bug提交的commit的号码。

## 标签管理

Git引入Tag的目的是将某个commit绑定上一个容易记住的名字。
操作步骤：

1. 首先切换到需要打标签的分支
2. 敲命令 `git tag <name>`就可以打一个标签，例如 `git tag v1.0`
3. 可以用 `git tag`查看所有的标签
4. 默认标签是打在最新提交的commit上的，如果要打在历史的commit上，就要找到历史的commit id，即 `git log`，然后打上就行，即 `git tag <name> <commit id>`。
5. 删除本地标签：`git tag -d <tagname>`
6. 如果要推送某个标签到远程，使用命令 `git push origin <tagname>`

## 远程仓库

- 查看连接的远程仓库

  ```bash
  $ git remote -v
  origin	https://github.com/schacon/ticgit (fetch)
  origin	https://github.com/schacon/ticgit (push)
  ```
- 查看与远程仓库的交互信息

  ```bash
  $ git remote show origin
  * remote origin
    Fetch URL: https://github.com/schacon/ticgit
    Push  URL: https://github.com/schacon/ticgit
    HEAD branch: master
    Remote branches:
      master                               tracked
      dev-branch                           tracked
    Local branch configured for 'git pull':
      master merges with remote master
    Local ref configured for 'git push':
      master pushes to master (up to date)
  ```
- 远程仓库的重命名和移除

  ```bash
  git remote rename <origin> <target>
  ```

  ```bash
  git remote remove <name>
  ```

## 变基

- 原理

  变基(rebase)的原理就是将提交到某一分支的所有修改都移至另一分支上，就好像"重新播放“一样。变基可以让提交历史变得简洁。
- 变基的使用准则：

  不要在不属于你的提交记录上，或者别人可能会基于某些提交开发的记录上使用变基。

  否则双方的协作拉取会出现相同签名的提交记录，这让事情变得混乱。

## 可视化分支历史

- 在命令行

  ```bash
  git log --oneline --graph
  ```
  ![image-20240815185551739](https://gitee.com/Marches7/piture-bed/raw/master/img/image-20240815185551739.png)
- 在VSCode中

  ![image-20240815185649942](https://gitee.com/Marches7/piture-bed/raw/master/img/image-20240815185649942.png)
