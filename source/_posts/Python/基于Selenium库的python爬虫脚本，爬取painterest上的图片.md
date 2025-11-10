---
title: 基于Selenium库的python爬虫脚本，爬取painterest上的图片
date: 2023-06-04 18:48 
description: 本文编写了一个基于Selenium的Python爬虫脚本，用于自动化批量下载Pinterest网站上的图片，通过模拟浏览器滚动页面和解析img标签实现图片链接提取和下载。
categories:
- Python
tags:
---
<head>
  <meta name="referrer" content="no-referrer" />
</head>

- 问题背景

  Pinterest是一个社交媒体平台，它提供了一个虚拟的个人兴趣画板，让用户可以收集和分享他们喜欢的图片、视频和链接。上面有许多优秀的图片供浏览和下载，但是一个个点图片下载非常麻烦。于是想要用Python语言写一个自动化脚本，可以根据给定的pinterest的url对页面内的所有图片进行爬取。

- 问题分析

  Pinterest网站上爬取图片要结合该网站的特点（工程要结合实际~），首先该网站图片链接放在html代码里的img标签下的src参数里，其次用户向下滑动页面可以查看更多的图片。

  结合这两个特点，我们可以使用Selenium库来完成脚本。

  > Selenium是一个用于自动化网页浏览器操作的Python库。它提供了一组API，可以通过编写Python代码来控制和操作Web浏览器。Selenium最常用的用途之一是进行Web应用程序的自动化测试，但它也可以用于执行各种Web操作，例如网页截图、表单填写、数据提取等。
  >
  > Selenium支持多种浏览器，包括Chrome、Firefox、Safari和Edge等。它使用浏览器的原生支持或者驱动程序来与浏览器进行交互，并模拟用户在浏览器中的行为，例如点击链接、填写表单、提交数据等。
  >
  > 使用Selenium，您可以编写Python脚本来自动化执行各种Web任务，例如自动登录网站、抓取网页内容、执行交互式操作等。它提供了丰富的功能和灵活的选项，使得Web自动化变得相对简单和可靠。
  >
  > ——ChatGPT

  脚本的执行流程大致如下：

  1. 获取图片链接
  2. 根据链接下载图片到本地
  3. 向下滑动页面
  4. 如果没有到达页面底部则返回第一步

- 代码解释

  下面开始逐步用代码操作

  - 首先导入Selenium库，requests库等相关库

    ```python
    from selenium import webdriver
    from selenium.webdriver.common.by import By
    import urllib.request
    import requests
    from urllib.parse import urljoin, urlparse
    import os
    import time
    ```

  - 给出网站URL和图片保存的本地目录

    ```python
    # 网站URL
    url = "https://www.pinterest.com/search/pins/?q=%E7%8C%AB&rs=typed" #以猫的图片为例
    # 图片保存目录
    directory = ""
    if not os.path.exists(directory):
        os.makedirs(directory)
    ```

  - 使用Selenium库创建浏览器实例，以Chorme浏览器为例

    ```python
    driver = webdriver.Chrome()
    driver.get(url)
    ```

  - 获取页面名字为img的所有标签，并提取属性src内容，与url组成图片的绝对链接

    ```python
    images = driver.find_elements(By.TAG_NAME, "img")
    src = image.get_attribute("src")
    absolute_src = urllib.parse.urljoin(url, src)
    ```

  - 将缩略图的链接换成原图的链接，同时还可以排除无关图像（pinterest链接中'236x'代表缩略图，改成'originals'就是原图了），然后下载图像

    ```python
    if '236x' in absolute_src:
                    absolute_src = absolute_src.replace('236x', 'originals')
                    download_image(absolute_src, directory)
    ```

  - 使用JS判断是否到达底部，当浏览器当前可视区域的高度+页面在垂直方向上的滚动距离>=整个页面的高度时，就代表到达了页面底部

    ```python
    if driver.execute_script("return window.innerHeight + window.pageYOffset >= document.body.scrollHeight;"):
                break
    ```

  - 向下滚动页面。window.scrollBy(0, 1000)代表在水平方向不移动，垂直方向向下移动1000像素，**注意**这个1000像素只是参考值，需要自行调试获得最佳的下滑距离，不会出现重复图片也不会漏掉图片。

    ```python
    driver.execute_script("window.scrollBy(0, 1000);") # 每次垂直滑动1000像素
    time.sleep(2) # 等待图片加载
    ```

  - 下载图像

    ```python
    def download_image(url, directory):
        response = requests.get(url, stream=True)
        if response.status_code == 200:
            # 获取文件名
            filename = os.path.join(directory, os.path.basename(urlparse(url).path))
            # 保存图片
            with open(filename, 'wb') as file:
                for chunk in response.iter_content(1024):
                    file.write(chunk)
            print("Downloaded:", url)
    ```

- 完整代码展示

  ```python
  from selenium import webdriver
  from selenium.webdriver.common.by import By
  import urllib.request
  import requests
  from urllib.parse import urljoin, urlparse
  import os
  import time
  
  def download_image(url, directory):
      response = requests.get(url, stream=True)
      if response.status_code == 200:
          # 获取文件名
          filename = os.path.join(directory, os.path.basename(urlparse(url).path))
          # 保存图片
          with open(filename, 'wb') as file:
              for chunk in response.iter_content(1024):
                  file.write(chunk)
          print("Downloaded:", url)
  
  
  def main():
      # 网站URL
      url = "https://www.pinterest.com/search/pins/?q=%E7%8C%AB&rs=typed"
      # 图片保存目录
      directory = "D:\EdgeDownloads\Cat"
      if not os.path.exists(directory):
          os.makedirs(directory)
      # 创建浏览器实例
      driver = webdriver.Chrome()
      driver.get(url)
      while True:
          images = driver.find_elements(By.TAG_NAME, "img")
          for image in images:
              src = image.get_attribute("src")
              absolute_src = urllib.parse.urljoin(url, src)
              if '236x' in absolute_src:
                  absolute_src = absolute_src.replace('236x', 'originals') # pinterest链接中'236x'代表缩略图，改成'originals'才是原图
                  download_image(absolute_src, directory)
          # 判断是否到达页面底部
          if driver.execute_script("return window.innerHeight + window.pageYOffset >= document.body.scrollHeight;"):
              break
          driver.execute_script("window.scrollBy(0, 1000);") # 每次垂直滑动1000像素
          time.sleep(2) # 等待图片加载
      driver.quit()
  
  if __name__ == '__main__':
      main()
  ```

  