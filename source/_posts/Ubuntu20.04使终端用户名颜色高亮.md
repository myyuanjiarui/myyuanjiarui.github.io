---
title: Ubuntu20.04使终端用户名颜色高亮
date: 2023-05-28 18:08 
tags:
---
<head>
  <meta name="referrer" content="no-referrer" />
</head>

## 问题背景

你还在为你的Linux大量输出内容但是不能清晰的看到每次输入的命令而烦恼吗？找找下面图中的两条命令在哪😄

![image-20240717165800894](https://gitee.com/Marches7/piture-bed/raw/master/img/image-20240717165800894.png)

## 解决方法

首先介绍一下.bashrc文件：

在Linux中，.bashrc文件是用于配置Bash（Bourne Again SHell）的用户自定义设置和环境变量的脚本文件。当一个用户登录到系统时，Bash会执行该用户的.bashrc文件来加载用户特定的配置。

.bashrc文件通常位于用户的主目录下（~/或/home/user/），是一个隐藏文件（以.开头），可以使用文本编辑器进行编辑。

然后编辑vim .bashrc：

![image-20240717165846720](https://gitee.com/Marches7/piture-bed/raw/master/img/image-20240717165846720.png)

在第43~45行可以看到官方的一段碎碎念：

uncomment for a colored prompt，if the terminal has the capability; turnedoff by default to not distract the user: the focus in a terminal windowshould be on the output of commands， not on the prompt。

翻译过来就是：

不推荐彩色命令提示符，我们应该把更多的精力放在输出而不是提示符上。

但是咱们偏不，谁让彩色提示符辨识度高又好看呢，并且那么大段的输出有时候也不想看...。解决办法也很简单，只要把第46行的force_color_prompt=yes解注释即可。

然后wq退出，并且source ./bashrc就可以看到效果：

 ![image-20240717165917986](https://gitee.com/Marches7/piture-bed/raw/master/img/image-20240717165917986.png)