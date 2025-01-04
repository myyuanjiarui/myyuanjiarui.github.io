---
title: Debug箴言
date: 2024-11-20 17:59 
categories:
- 感悟
tags:
---
<head>
  <meta name="referrer" content="no-referrer" />
</head>


1. 一场由`enumerate()`函数引发的数据格式错误

   在Python中最常使用的操作就是for循环了，虽然我们讨厌并避免在代码中进行嵌套for循环。for循环看起来简单，但是就是由于enumerate()函数引发了一场错综复杂的Debug。

   分析下面两个for循环：

   ```py
   # 假设l是tensor([1, 2, 3])
   # case 1:
   for l_i in l:
     print(l_i)
   # case 2:
   for l_i in enumerate(l):
   ```

   实际输出如下：

   ```py
   # case 1:
   tensor(1)
   tensor(2)
   tensor(3)
   # case 2:
   (0, tensor(1))
   (1, tensor(2))
   (2, tensor(3))
   ```

   **所以`enumerate()`返回的是包含索引和对应数据的元组**，如果case 2这样写就与case 1的输出等价：

   ```py
   for index, l_i in enumerate(l): # 对元组进行解包
     print(l_i)
   ```

   思考，如果这样写会有什么错误：

   ```py
   for index, l_i in l: # 没有使用enumerate却进行解包
     print(l_i) # unpack or interation error
   ```

   注意这种写法在遍历深度学习的DataLoader对象却很有可能正常运行，并产生意向不到的错误。因为DataLoader对象是第一维度是大小为batch_size，后续维度是一条数据的大小。上面那种写法如果刚好batch_size=2会把第一维和第二维度解包，**导致第一列的数据丢失了，并且会出现维度的丢失**。例如：

   ```py
   <torch.utils.data.dataloader.DataLoader object>
   dataloader:（batch_size=2）
   tensor([1, 2],
          [5, 6],
          [9, 10])
   ```

   每条数据的大小应该是(2,1)

   如果进行误解包：

   ```py
   tensor(2)
   tensor(6)
   tensor(10)
   ```

   就会丢失第一列的数据，当然如果你的batch_size不等于2，那么就不会正常运行，会出现unpack error。

2. 对shell脚本的错误执行方式导致的指令丢失和重复执行

   如果想批量执行shell命令，我们经常会这样做，将要执行的shell命令全部写入.sh文件当中，再执行这个.sh文件。（例如run.sh）

   如果我们修改了run.sh，在一个新的终端重新执行一批新的命令。此时的情况是：Terminal-1：执行run.sh；Terminal-2：执行run.sh。两个run.sh的内容不同，但文件名一样。

   我们想要的结果是Terminal-1和Terminal-2执行的分别是“新旧”内容的run.sh，互不干扰，但是这取决于我们run.sh文件的内容和系统读取文件内容的方式，下面两种执行方式，都可能会带来意象不到的结果：

   :x:`./run.sh`执行， `sh run.sh`执行。这两种执行方法，当系统是逐行或者逐个chunk读取文件内容时，在运行时修改run.sh都会丢失原本的内容。（可以测试：新加一行或者修改原有行）
   > Python等语言则不会出现这种情况，文件每次被全部读入，并编译成字节码存储到内存就不再改变。

   目前**还没有**很好的将.sh文件一次性全部读到内存中的方法，只能想要修改**先复制一个新的.sh文件**。

   **拓展**：.sh文件中shebang的妙用，可以让你使用./的方法执行任何语言编写的.sh文件。

   Shebang是.sh文件中最顶端的一行，它定义了运行.sh文件的解释器，默认是系统的shell解释器。例如

   ```sh
   #!/usr/bin/bash
   ```

   `#!`是固定语法，后面路径是想要运行.sh文件的解释器路径。

   这样我们想在bash中运行run.sh就可以：

   ```sh
   ./run.sh
   ```

   如果我们的shebang是Python解释器的路径，例如：

   ```sh
   #!/usr/bin/python
   ```

   那么我们`./run.sh`就会以Python解释器运行run.sh的内容，相当于：

   ```sh
   python run.sh
   ```

   所以run.sh里也可以写Python代码，并且shebang 也支持传递参数：

   ```sh
   #!/usr/bin/python [args]
   ```

   相当于：

   ```sh
   python [args] run.sh
   ```

   当然shebang还可以是Ruby，Java等等的解释器路径，由此我们也可以知道，文件的扩展名没有实际意义，所有的文件内容本质上都是字节，具体的执行还是要看解释器对文件的处理方式。

   