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