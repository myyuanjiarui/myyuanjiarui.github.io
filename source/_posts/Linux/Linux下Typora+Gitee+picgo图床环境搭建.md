---
title: Linux下Typora+Gitee+picgo图床环境搭建
date: 2023-01-15 00:02
categories:
- Linux
tags:
---
<head>
  <meta name="referrer" content="no-referrer" />
</head>

### 前言

在Linux下写Markdown笔记的时候，像上传到github上，结果发现笔记里的图片都成了这个样子：

![Screenshot from 2023-01-15 03-07-24](https://gitee.com/Marches7/piture-bed/raw/master/img/Screenshot from 2023-01-15 03-07-24.png)

图片呢？？？

后来查阅资料理解了，Markdown笔记和Word不同，图片在Markdown中是外链，而Word是直接把图片内置进了文档，所以如果想让被分享的人看到图片，最粗笨最原始的办法是将笔记分享给他的同时，还把图片分享他，并且存放图片的路径要和你的图片路径一模一样。对图片支持的不友好，可以看成Markdown为了轻量级便携输入的一种牺牲。

如何直接分享Markdown文件给别人，别人还能看到图片呢？（这里的“直接“ 也就意味着还可以转成pdf再分享这样不直接的方法...）

解决方法也可以想到，既然图片在markdown中以本地链接的形式存在，那么如果图片是存储在互联网上的，任何能访问互联网的人不就能看到图片了吗！

于是引出了图床这一技术，严格来说，他只是存储图片的一个互联网服务，目前有收费的阿里云OSS，腾讯云COS，免费的Github，Gitee等等。这里选择Gitee作为图床，一是因为对他比较熟悉（觉得不会随便跑路...），二是免费，三是在国内所以国内用户访问很快。

好了我们已经选好了图床，想象一下如果每张图片我们都需要从本地上传到图床，再插入Markdown中，是多么的麻烦。为解决这一难题，有人已经写好了非常成熟的脚本，并且被内置到了著名的Markdown编辑器Typora中，这就是**PicGo**，PicGo本身并不存储图片，他只是自动化上传Markdown中的图片到图床并创建好链接放到Markdown中的脚本，我们只需要作简单的配置就可以了。

### 环境搭建

> 系统：Ubuntu18.04
>
> Typora：v1.4.7
>
> PicGo：v2.3.1

然后开始环境搭建吧。

打开Typora的图片设置

![image-20230115034602675](https://gitee.com/Marches7/piture-bed/raw/master/img/image-20230115034602675.png)

第一栏选择Upload Image（上传图片），图片上传设置选择PicGo-Core，然后点击下方的下载或更新来下载PicGo-Core。然后点击Test Uploader （这步只是为了获得我们PicGo-Core的安装目录）。

![image-20230115034856544](https://gitee.com/Marches7/piture-bed/raw/master/img/image-20230115034856544.png)

进入该目录下，执行以下命令安装插件

```shell
./picgo install gitee-uploader
./picgo install super-prefix
```

如果挂了梯子，这里注意关闭，否则可能会因为不安全连接而中断下载。

点击打开配置文件

```yaml
{
  "picBed": {
    "current": "gitee",
    "uploader": "gitee",
    "gitee": {
      "repo": "username/reponame",
      "branch": "master",
      "token": "xxxxxxx",
      "path": "自定义存储路径（可不填）",
      "customPath": "",
      "customUrl": ""
    }
  },
  "picgoPlugins": {
    "picgo-plugin-gitee-uploader": true,
    "picgo-plugin-super-prefix": true
  },
  "picgo-plugin-gitee-uploader": {
    "lastSync": "2023-01-15 03:48:57"
  }
}
```

需要修改的是repo，token和和path。token是Gitee的私人令牌，百度获得方法。

保存。点击测试上传

![image-20230115035544322](https://gitee.com/Marches7/piture-bed/raw/master/img/image-20230115035544322.png)

大功告成！

### 总结

因为PicGo对Linux的支持问题，前期我还是走了很多弯路，折腾了两天才弄好，没必要使用PicGo GUI，他只是提供Config文件内容，而我这里已经提供了。我觉得这应该是最简单的方法，故分享，再也不用手动上传图片了。
