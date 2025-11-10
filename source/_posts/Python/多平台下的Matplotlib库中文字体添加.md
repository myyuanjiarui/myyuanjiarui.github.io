---
title: 多平台下的Matplotlib库中文字体添加
date: 2024-09-27 11:14 
description: 本文解决了Matplotlib库在不同操作系统下无法显示中文的问题，分别给出了Windows、Mac和Linux平台配置中文字体的方法，其中Linux需要手动下载安装字体文件。
categories:
- Python
tags:
---
<head>
  <meta name="referrer" content="no-referrer" />
</head>


## 问题描述

Matplotlib库默认是不显示中文的，中文会乱码。因此如果想要显示中文，我们需要对代码中Matplotlib使用的字体进行配置。而Windows，Mac，Linux下支持的字体不相同，因此在此记录，方便查阅。

## 问题解决

- Windows

  ```python
  import matplotlib.pyplot as plt
  from matplotlib import rcParams
  
  rcParams['font.sans-serif'] = ['SimHei'] # 指定字体为黑体
  plt.rcParams['axes.unicode_minus']=False # 用来正常显示负号
  
  plt.plot([1, 2, 3, 4])
  plt.ylabel('一些数组')
  plt.show()
  ```

- Mac

  ```py
  import matplotlib.pyplot as plt
  from matplotlib import rcParams
  
  rcParams['font.sans-serif'] = ['Arial Unicode MS']
  plt.rcParams['axes.unicode_minus']=False # 用来正常显示负号
  
  plt.plot([1, 2, 3, 4])
  plt.ylabel('一些数组')
  plt.show()
  ```

- Linux

  参考：[Ubuntu下让matplotlib显示中文字体_ubuntu matplot 使用汉字-CSDN博客](https://blog.csdn.net/takedachia/article/details/131017286)

  Linux平台下的解决麻烦一些，默认没有字体可以支持中文matplotlib绘图，所以我们要下载字体并安装：

  1. 下载字体：[Download SimHei Font - Thousands of fonts to download for free (fontzone.net)](https://fontzone.net/font-download/simhei)，进入网站下载后是一个simhei.tff文件。

  2. 安装字体到matplotlib：

     1. 查找matplotlib库的字体文件夹

        ```py
        import matplotlib
        matplotlib.matplotlib_fname()
        ```

        会输出一个目录，例如：`/home/ck/anaconda3/envs/pytorch2/lib/python3.10/site-packages/matplotlib/mpl-data/matplotlibrc`

     2. 打开该文件所在目录`/home/ck/anaconda3/envs/pytorch2/lib/python3.10/site-packages/matplotlib/mpl-data`下的`fonts`文件夹，把之前选中的 simhei.ttf 复制到 fonts文件夹下。

     3. 删除matplotlib的缓存文件

        ```shell
        cd ~/.cache/matplotlib
        rm -rf *.*
        ```

     4. 显示字体

        如果是在Jupyter notebook中要重启一下内核。

        ```pyt
        import matplotlib.pyplot as plt
        from matplotlib import rcParams
        
        rcParams['font.sans-serif'] = ['SimHei'] # 指定字体为黑体
        plt.rcParams['axes.unicode_minus']=False # 用来正常显示负号
        
        plt.plot([1, 2, 3, 4])
        plt.ylabel('一些数组')
        plt.show()
        ```

        

  

