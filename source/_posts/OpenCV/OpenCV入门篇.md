---
title: OpenCV入门篇
date: 2024-10-15 10:02
description: 本文学习了OpenCV图像处理的基础操作，包括图片的加载显示保存、摄像头和视频处理、颜色空间转换、阈值分割、图像几何变换和绘图功能的使用方法。
categories:
- OpenCV
tags:
---
<head>
  <meta name="referrer" content="no-referrer" />
</head>

# OpenCV入门篇

## OpenCV Basic Knowledge

- OpenCV中的彩色图是以**B-G-R**通道顺序存储的，灰色图则只有一个通道

- 图像坐标的起始点是在左上角，所以行对应y，列对应x

  ![image-20241010192034957](https://gitee.com/Marches7/piture-bed/raw/master/img/image-20241010192034957.png)

- 常用的图片格式

  - bmp
    - 全称：Bitmap
    - **不压缩**
  - jpg
    - 全称：Joint Photographic Experts Group
    - **有损压缩方式**
  - png
    - 全称：Portable Network Graphics
    - **无损压缩方式**

## 图片基本操作

- 加载图片

  ```py
  cv2.imread(/path/to/image，0)
  ```

  参数2：读入方式，省略则采用默认值

  - `cv2.IMREAD_GRAYSCALE`：灰度图(0)

  - `cv2.IMREAD_COLOR`：彩色图，默认值(1)
  - `cv2.IMREAD_UNCHANGED`：包含透明通道的彩色图(-1)

- 显示图片

  ```py
  img = cv2.imread(/path/to/image)
  cv2.imshow('window_name', img)
  ## Wait for 'q' key to be pressed
  # while True:
  #     if cv2.waitKey(1) & 0xFF == ord('q'):
  #         break
  ```

  参数1是窗口的名字。

- 保存图片

  ```py
  cv2.imwrite('new_file_name', img)
  ```

- 获取和修改像素点值

  ```py
  px = img[100, 90] # 获取第99行(行对应y)第89列(列对应x)的元素三通道像素值（包含三个元素的数组）
  px_b = img[100, 90, 0] # 只获取蓝色通道的值
  img[100, 90] = np.arry([100, 100, 100])# 修改像素点的值
  ```

- 图片属性

  - `img.shape`获取图像的形状，图片是彩色的话，则返回一个包含行数（高度），列数（宽度）和通道数的元组。
  - `img.dtype`获取图像数据类型，例如uint8
  - `img.size`返回图像总像素数，即行数*列数\*通道数

- 获取某个通道的像素值

  可以使用`cv2.split()`：

  ```py
  b, g, r = cv2.split(img)
  ```

  更高效的方式使用numpy中的索引，如提取蓝色通道：

  ```py
  b = img[:, :, 0]
  ```

## 摄像头操作

- 打开摄像头

  ```py
  capture = cv2.VideoCapture(0)
  ```

  参数0指的是摄像头的编号，如果电脑上有两个摄像头的话，访问第二个摄像头就可以传入1。

- 获取摄像头的一帧

  ```py
  ret, frame = capture.read()
  ```

  第一个参数是一个布尔值，表示当前这一帧是否获取正确，第二个参数是摄像头的一帧

- 以原分辨率的一倍来捕获

  ```py
  width, height = capture.get(3), capture.get(4)
  capture.set(cv2.CAP_PROP_FRAME_WIDTH, width*2)
  capture.set(cv2.CAP_PROP_FRAME_HEIGHT, height*2)
  ```

  `capture.get(propID)`可以获取摄像头的一些属性，例如分辨率，亮度，对比度等。

## 视频基本操作

- 播放本地视频

  ```py
  # 读取本地视频
  capture = cv2.VideoCapture('path/to/video')
  
  while(capture.isOpened()):
    # 读取视频的一帧
    ret, frame = capture.read()
    # 展示这一帧
    cv2.imshow('Video', frame)
    # 帧的播放间隔和q键退出操作
    if cv2.waitKey(30) == ord('q'):
      break
  ```

  跟打开摄像头一样，如果把摄像头的编号换成视频的路径就可以本地播放视频了。

- 从摄像头录制视频

  要保存视频，我们需要创建一个`VideoWriter`的对象，需要传入四个参数：

  - 输出的文件名：例如'output.avi'
  - 编码方式FourCC码：`FourCC`是用来指定视频编码方式的四字节码，例如MJPG编码可以这样写：`cv2.VideoWriter_fourcc(*'MJPG')`或`cv2.VideoWriter_fourcc('M','J','P','G')`
  - 帧率FPS：浮点数
  - 要保存的分辨率大小

  ```py
  # 获取摄像头
  capture = cv2.VideoCapture(0)
  # 定义编码方式
  fourcc = cv2.VideoWriter_fourcc(*'MJPG')
  # 创建VideoWriter对象
  outfile = cv2.VideoWriter('output.avi', fourcc, 25., (640, 480))
  # 循环获取视频每一帧并写入到输出视频文件
  while(capture.isOpened()):
    # 读取视频的一帧
    ret, frame = capture.read()
    if ret: # 如果读取成功
      # 帧写入输出视频文件
      outfile.write(frame)
      # 帧的播放间隔和q键退出操作
      if cv2.waitKey(1) == ord('q'):
        break
  ```

## 颜色空间转换

用`cv2.cvtColor()`来进行颜色模型转换：

```py
img_gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
```

参数1是转换的图片，参数2是转换的模式，例如`COLOR_BGR2GRAY`表示BGR→Gray，常用的还有BGR→HSV(`COLOR_BGR2HSV`)等各种颜色转换。

## 阈值分割

阈值分割方法有多种，例如固定阈值，自适应阈值和Otsu阈值法"二值化"图像

- 固定阈值分割

  即先设定一个阈值，然后将该像素点中所有值大于阈值的变为一类值，小于阈值的变为另一类值。

  ```py
  ret, th = cv2.threshold(img, 127, 255, cv2.THRESH_BINARY)
  ```

  函数返回ret是return value的缩写，代表当前阈值，th代表处理后的图像矩阵。

  函数有4个参数：

  - 参数1：要处理的原图，一般是灰度图

  - 参数2：设定的阈值

  - 参数3：对于`THRESH_BINARY`，`THRESH_BINARY_INV`阈值方法所选用的最大阈值，一般为255

  - 参数4：阈值的方式，主要有5种。结合下面的代码和结果理解：

    ```py
    import matplotlib.pyplot as plt
    
    # 应用 5 种不同的阈值方法
    ret, th1 = cv2.threshold(img, 127, 255, cv2.THRESH_BINARY)
    ret, th2 = cv2.threshold(img, 127, 255, cv2.THRESH_BINARY_INV)
    ret, th3 = cv2.threshold(img, 127, 255, cv2.THRESH_TRUNC)
    ret, th4 = cv2.threshold(img, 127, 255, cv2.THRESH_TOZERO)
    ret, th5 = cv2.threshold(img, 127, 255, cv2.THRESH_TOZERO_INV)
    
    titles = ['Original', 'BINARY', 'BINARY_INV', 'TRUNC', 'TOZERO', 'TOZERO_INV']
    images = [img, th1, th2, th3, th4, th5]
    
    # 使用 Matplotlib 显示
    for i in range(6):
        plt.subplot(2, 3, i + 1)
        plt.imshow(images[i], 'gray')
        plt.title(titles[i], fontsize=8)
        plt.xticks([]), plt.yticks([])  # 隐藏坐标轴
    
    plt.show()
    ```

    ![image-20241010202555014](https://gitee.com/Marches7/piture-bed/raw/master/img/image-20241010202555014.png)

    ![image-20241010202720424](https://gitee.com/Marches7/piture-bed/raw/master/img/image-20241010202720424.png)

- 自适应阈值

  固定阈值是在整个图片上应用一个阈值进行分割，它 并不使用于敏感分布不均的图片。而自适应分割会每次去图片的一小部分计算阈值，在小区域内进行分割，不同区域的阈值不尽相同。使用`cv2.adaptiveThreshold()`函数进行自适应阈值分割。下面的例子展示了自适应阈值和固定阈值的结果比较以及不同自适应阈值的结果比较：

  ```py
  # 自适应阈值对比固定阈值
  img = cv2.imread('sudoku.jpg', 0)
  
  # 固定阈值
  ret, th1 = cv2.threshold(img, 127, 255, cv2.THRESH_BINARY)
  # 自适应阈值
  th2 = cv2.adaptiveThreshold(
      img, 255, cv2.ADAPTIVE_THRESH_MEAN_C, cv2.THRESH_BINARY, 11, 4)
  th3 = cv2.adaptiveThreshold(
      img, 255, cv2.ADAPTIVE_THRESH_GAUSSIAN_C, cv2.THRESH_BINARY, 17, 6)
  
  titles = ['Original', 'Global(v = 127)', 'Adaptive Mean', 'Adaptive Gaussian']
  images = [img, th1, th2, th3]
  
  for i in range(4):
      plt.subplot(2, 2, i + 1), plt.imshow(images[i], 'gray')
      plt.title(titles[i], fontsize=8)
      plt.xticks([]), plt.yticks([])
  plt.show()
  ```

  ![image-20241010210057691](https://gitee.com/Marches7/piture-bed/raw/master/img/image-20241010210057691.png)

  `cv2.adaptiveThreshold`的参数解释：

  - 参数1：要处理的原图
  - 参数2：最大阈值，一般为255
  - 参数3：小区域阈值的计算方式
    - `ADAPTIVE_THRESH_MEAN_C`：小区域内取均值
    - `ADAPTIVE_THRESH_GAUSSAN_C`：小区域内加权求和，权重是个高斯核。
  - 参数4：阈值方法，只能使用`THRESH_BINARY`、`THRESH_BINARY_INV`
  - 参数5：小区域的面积，如11就是11*11的小块
  - 参数6：最终阈值等于小区域计算出的阈值再减去此值

- Otsu阈值

  在固定阈值的基础上，Otsu可以自动计算出较优的阈值，而不用手动分割。

  ```py
  ret, thresh = cv2.threshold(img, 0, 255, cv2.THRESH_BINARY+cv2.THRESH_OTSU)
  ```

  参数解释：

  1. `img`：输入的灰度图像
  2. `0`：手动设定的阈值，但由于后面使用了Ostu的方法，这个值会被忽略
  3. `255`：阈值处理后，高于阈值的像素将被设置为255，表示完全白色。
  4. `cv2.THRESH_BINARY+cv2.THRESH_OTSU`：使用的阈值方法

## 图像几何变换

![image-20241010171700106](https://gitee.com/Marches7/piture-bed/raw/master/img/image-20241010171700106.png)

从原理上来看，图像的几何变换分为2D的仿射变换（平移、缩放、旋转和翻转等），3D的透视变换。

### 仿射变换

仿射变换是基于2x3的变换矩阵进行计算

- 缩放图片

  ```py
  # 按照指定的宽度和高度缩放图片
  res = cv2.resize(img, (132, 150))
  # 按照比例缩放，如x,y轴均放大一倍，并指定差值方法为线性插值
  res2 = cv2.resiez(img, None, fx=2, fy=2, interpolation=cv2.INTER_LINEAR)
  ```

  - 翻转图片

    ```py
    dts = cv2.flip(img, 1)
    ```

    参数2：

    - =0：垂直翻转（沿x轴）

    - \>0：水平翻转（沿y轴）

    - <0：垂直+水平翻转

      ![image-20241010212248235](https://gitee.com/Marches7/piture-bed/raw/master/img/image-20241010212248235.png)

  - 平移图片

    要平移图片，可以利用仿射变换函数`cv2.warpAffine()`，首先需要构建一个2x3的平移矩阵：
    $$
    M=\begin{bmatrix}
    1& 0 & t_x \\
    0 & 1& t_y
    \end{bmatrix}
    $$
    t_x和t_y分别是x方向和y方向的平移距离。

    ```py
    import numpy as np
    # 获取图像矩阵的行数和列数
    rows, cols = img.shape[:2]
    # 定义平移矩阵x轴平移100，y轴平移50
    M = np.float32([[1, 0, 100], [0, 1, 50]])
    # 用仿射变换实现平移
    dst = cv2.warpAffine(img, M, (cols, rows))
    ```

    ![image-20241010213134224](https://gitee.com/Marches7/piture-bed/raw/master/img/image-20241010213134224.png)

  - 旋转图片

    旋转也使用放射变换实现，因此也需要定义一个变换矩阵。因为旋转矩阵涉及到正余弦值，书写不方便，所以OpenCV直接提供了一个`cv2.getRotationMatrix2D()`函数来生成这个矩阵。

    ```py
    # 45°旋转图片并缩小一半
    M = cv2.getRotationMatrix2D((cols / 2, rows / 2), 45, 0.5)
    # 使用仿射变换函数旋转图片
    dst = cv2.wrapAffine(img, M, (cols, rows))
    ```

    

    `cv2.getRotationMatrix2D()`函数有三个参数：

    - 参数1：图片的旋转中心
    - 参数2：旋转角度（正：逆时针，负：顺时针）
    - 参数3：缩放比例，0,5表示缩小一半

### 透视变换

透视变换是基于3x3的变换矩阵进行计算。是将二维的图片投影到一个三维平面，再转换到二维坐标下。

## 绘图功能

绘制形状的函数有一些共同的参数：

- Img：要回执形状的图片
- Color：绘制的颜色
- Thickness：线宽，默认为1，对于矩形/圆之类的封闭形状而言，传入-1表示填充形状。

1. 画线

   ```py
   cv2.line(img, (0, 0), (512, 512), (255, 0, 0), 5) # 画一条线宽为 5 的蓝色直线，参数 2：起点，参数 3：终点
   ```

2. 画矩形

   ```py
   # 画一个绿色边框的矩形，参数 2：左上角坐标，参数 3：右下角坐标
   cv2.rectangle(img, (384, 0), (510, 128), (0, 255, 0), 3)
   ```

3. 画圆

   ```py
   # 画一个填充红色的圆，参数 2：圆心坐标，参数 3：半径
   cv2.circle(img, (447, 63), 63, (0, 0, 255), -1)
   ```

4. 画椭圆

5. 画多边形

6. 添加文字

## 代码性能优化

1. 数据元素少时用Python语法，数据元素多用Numpy
2. 尽量避免循环，尤其是嵌套循环
3. 优先使用OpenCV/Numpy中封装好的函数
4. 尽量将数据向量化，变成Numpy的数据格式
5. 尽量避免数组的复制操作