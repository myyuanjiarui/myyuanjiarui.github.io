---
title: docker操作——录播姬使用手册
date: 2024-06-11 21:27:07
tags:
---
# docker操作——录播姬使用手册

- docker更新

  - 先拉取最新的镜像

    ```shell
    sudo docker pull bililive/recorder:latest
    ```

  - 查看镜像

    ```shell
    sudo docker images
    ```

  - 删除镜像

    ```shell
    sudo docker rmi [IMAGE_ID or IMAGE_NAME]
    ```

  - 停止并删除旧容器

    ```shell
    sudo docker stop <旧容器ID>
    sudo docker rm <旧容器ID>
    ```

  - 查看所有容器（包括正在运行的和已经停止的）

    ```shell
    sudo docker ps -a
    ```

  - 删除所有已经停止的容器

    ```shell
    sudo docker container rmrmprune -f
    ```

  - 重新新建容器运行

    添加了参数`--restart=unless-stopped`: 容器会在退出时自动重新启动，除非显式停止容器。

    添加了参数`--network=host`:使用宿主机网络，获得更好的网络性能。

    旧：

    ```shell
    sudo docker run -d -v ~/rec:/rec -p 2356:2356 --restart=unless-stopped --network=host -e BREC_HTTP_BASIC_USER="yjr" -e BREC_HTTP_BASIC_PASS="915526974512xy" bililive/recorder:latest
    ```

    新：

    ```sh
    sudo docker run -d \
      --name bililive_recorder \
      -p 2356:2356 \
      -v ~/rec:/rec \
      -e BREC_HTTP_BASIC_USER="yjr" \
      -e BREC_HTTP_BASIC_PASS="915526974512xy" \
      bililive/recorder:latest
    ```

    

  - docker查看log

    ```shell
    sudo docker logs 容器ID
    ```

- 连接合并视频

  sudo修改concat.txt的内容为

  ```sh
  file '录制-23264749-20240101-152459-398-新年第一天当然要学习.flv'
  file '录制-23264749-20240101-152923-020-新年第一天当然要学习.flv'
  ```
  
  执行ffmpeg命令
  
  ```sh
  sudo ffmpeg -f concat -safe 0 -i concat.txt -c copy 录制-23264749-20240517-114639-820-清华搭子有仪式感地给毕设收尾_合并.flv
  ```


- 分割视频

  从视频的开始处分割到指定的时间点：

  ```sh
  sudo ffmpeg -i input.mp4 -t 01:18:55 -c copy output1.mp4
  ```

  这从指定的时间点分割到视频的结束：

  ```sh
  sudo ffmpeg -i input.mp4 -ss 01:18:55 -c copy output2.mp4
  ```

- 查看空间大小

  - 查看系统空间

    ```sh
    df -h
    ```

  - 查看当前目录下文件和文件夹大小，并按照名称排序

    ```sh
    du -h | sort -k2
    ```
  
  
  - 查看当前目录下文件和文件夹大小，并按照占用大小逆序排序
  
    ```sh
    du -h | sort -rh
    ```
  
    
  


- 批量删除文件夹下的所有视频文件

  - 保险起见，先查看并输出已有的视频文件

    ```sh
    find . -type f -name "*.flv" -print
    ```

  - 确认文件都可删除后再执行删除

    ```sh
    sudo find . -type f -name "*.flv" -exec rm  {} +
    ```

    

