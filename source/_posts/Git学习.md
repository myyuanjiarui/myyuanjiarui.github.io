---
title: Git学习
date: 2024-02-23 11:14 
tags:
---
<head>
  <meta name="referrer" content="no-referrer" />
</head>

## 集中式VS分布式

- 集中式的版本库是集中存放在中央服务器的
- 分布式的版本库存在每一个人的电脑中，每个人的电脑都是一个完整的版本库
- 分布式版本控制系统更安全
- 分布式版本控制系统通常也有一台充当“中央服务器”的电脑，但这只是为了方便“交换”大家的修改，没有它并不影响正常工作

## Git的安装

- 因为Git是分布式版本控制系统，所以每台机器都必须指定自己的名字和邮件地址

```shell
git config --global user.name "Your Name"
git config --global user.email "email@example.com"
```

## 查看提交记录

`git log`

## 创建版本库

- 版本库又称为仓库(repository)，仓库里面的所有文件都可以被Git管理起来，每个文件的增删改查都被Git跟踪
- 初始化当前文件夹为仓库：`git init`
- 添加文件到Git仓库，分两步：
  1. `git add <file>`。可以多次使用命令，多次添加文件
  2. `git commmit -m <message>`

## 添加到远程库

在Github/Gitee中添加一个仓库，然后根据Github/Gitee的提示，在本地的仓库下运行

```shell
git remote add origin git@gitee.com:Marches7/仓库名.git
```

## 撤销修改

- 工作区，版本库和暂存区

![image.png](https://cdn.nlark.com/yuque/0/2024/png/35312186/1706514518606-00b9fca9-4c60-4e47-adc0-a01b510976ef.png#averageHue=%23ececec&clientId=ub7619b40-6c08-4&from=paste&height=255&id=u0b073c97&originHeight=447&originWidth=889&originalType=binary&ratio=1.75&rotation=0&showTitle=false&size=119123&status=done&style=none&taskId=ud973f9ac-ffeb-493d-9fa0-3cad8aacf0f&title=&width=508)

> 其中的stage称为是暂存区

- 当你改乱了工作区某个文件的内容，想直接丢弃工作区的修改时：
  - `git checkout -- <file>`，会有两种情况：
    1. 一种是readme.txt自修改后还没有被放到暂存区，现在，撤销修改就回到和版本库一模一样的状态。
    2. 一种是readme.txt已经添加到暂存区后，又作了修改，现在，撤销修改就回到添加到暂存区后的状态。
- 当你不但改乱了工作区某个文件的内容，还添加到了暂存区时，想丢弃修改：
  - 第一步：`git reset HEAD <file>`
  - 第二步：`git checkout -- <file>`
- 已经提交了不合适的修改到版本库时，想要撤销本次提交，前提是没有提交到远程库：
  - 回退到之前的版本：`git reset --hard HEAD^`

git reset: 这是 Git 中的主要命令之一，用于重置当前 HEAD 到指定状态。
--hard: 这是 git reset 命令的一个选项，用于指定重置的模式。在 --hard 模式下，Git 将重置工作目录和索引（暂存区）以匹配你要重置到的提交。这意味着所有自上次提交以来对工作目录和索引的更改都将被丢弃。
HEAD^: 这指定了重置操作的目标提交。HEAD 是一个指向你当前所在分支的最新提交的指针。HEAD^（或者 HEAD~1）表示当前分支的父提交，即当前提交的前一个提交。HEAD^^（或者HEAD~2）表示当前分支的前前个提交

## 分支管理

因为创建、合并和删除分支非常快，所以Git鼓励你使用分支完成某个任务，合并后再删掉分支，这和直接在master分支上工作效果是一样的，但过程更安全。

- 查看分支：`git branch`
- 创建分支dev：`git checkout -b dev`
- 切换分支：`git switch <branch name>`
- 合并某分支到当前分支：`git merge <branch name>`
- 删除分支（删除的时候不能在当前分支）：`git branch -d <branch name>`

分支策略的几个原则：

- 首先，master分支应该是非常稳定的，也就是仅用来发布新版本，平时不能在上面干活
- 干活都在dev分支上，也就是说，dev分支是不稳定的，到某个时候，比如1.0版本发布时，再把dev分支合并到master上，在master分支发布1.0版本
- 你和你的小伙伴们每个人都在dev分支上干活，每个人都有自己的分支，时不时地往dev分支上合并就可以了

![image.png](https://cdn.nlark.com/yuque/0/2024/png/35312186/1706517597971-fed08c2c-03b2-43aa-88e9-d22ab1e743d7.png#averageHue=%23f7f5f4&clientId=ub7619b40-6c08-4&from=paste&height=147&id=ubedd3c26&originHeight=257&originWidth=997&originalType=binary&ratio=1.75&rotation=0&showTitle=false&size=72620&status=done&style=none&taskId=u4852a420-aea6-491c-ba1a-f50dddf2816&title=&width=569.7142857142857)

## 暂存分支内容

当前分支的工作还未做完，需要新建一个Bug分支处理一个紧急Bug， 可以使用`git stash`可以把当前工作现场“储藏”起来，等以后恢复现场后继续工作。
Bug修复完成后，需要恢复现场，有两种方法：

- 一是用git stash apply恢复，但是恢复后，stash内容并不删除，你需要用git stash drop来删除
- 二是用git stash pop，恢复的同时把stash内容也删了

但是之后会想到一个问题，我们修改的Bug在main分支上，意味着我们工作的分支也有Bug，有必要重复修改本分支的Bug再合并到当前分支上吗？有没有更简单的方法？
为了方便操作，避免重复矛盾，Git专门提供了一个`git cherry-pick 4c805e2`命令，让我们能复制一个特定的提交到当前分支，4c805e2是Bug提交的commit的号码。

## 多人协作

### 抓取分支

多人协作时，大家都会往master和dev分支上推送各自的修改。模拟一个新的小伙伴协作开发。

1. 使用git clone将仓库克隆到自己的电脑。默认情况下只能看到本地的master分支。
2. 要在dev分支上开发，就必须创建远程origin的dev分支到本地：`git checkout -b dev origin/dev`
3. 现在就可以在dev分支上开发了，时不时可以把dev分支push到远程

### 推送分支到远程库

推送dev分支到远程库：`git push origin dev`

> 远程库的名字就是origin，这是Git默认的叫法，也可以改成别的，但是origin这个名字一看就知道是远程库。

### 流程

1. 首先，可以试图用git push origin <branch-name>推送自己的修改
2. 如果推送失败，则因为远程分支比你的本地更新，需要先用git pull试图合并
3. 如果合并有冲突，则解决冲突，并在本地提交
4. 没有冲突或者解决掉冲突后，再用git push origin <branch-name>推送就能成功

## 标签管理

Git引入Tag的目的是将某个commit绑定上一个容易记住的名字。
操作步骤：

1. 首先切换到需要打标签的分支
2. 敲命令`git tag <name>`就可以打一个标签，例如`git tag v1.0`
3. 可以用`git tag`查看所有的标签
4. 默认标签是打在最新提交的commit上的，如果要打在历史的commit上，就要找到历史的commit id，即`git log`，然后打上就行，即`git tag <name> <commit id>`。
5. 还可以创建带有说明的标签，用-a指定标签名，-m指定说明文字：`git tag -a v0.1 -m "version 0.1 released" 1094adb`
6. 如果要推送某个标签到远程，使用命令`git push origin <tagname>`

