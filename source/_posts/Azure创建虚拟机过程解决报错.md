---
title: Azure创建虚拟机过程解决报错
date: 2023-05-13 16:33 
tags:
---


## 背景

当我使用Azure学生教育福利开通虚拟机时（具体步骤建议参考：[Azure(az100)和az200建免费虚拟机避抗教程-尘遇 (chenyu.me)](https://chenyu.me/1715.html)），在最后的下载个人私钥环节，遇到了以下报错：无法下载SSH私钥。未注册订阅，无法使用命名空间“Microsoft.Compute"。有关如何注册订阅，请参阅https://aka.ms/rps-not-found。错误详细信息:未注册订阅，无法使用命名空间“Microsoft.Compute"。有关如何注册订阅，请参阅https://aka.ms/rps-not-found。错误详细信息: [objectObject]。

这个问题在百度和谷歌上均搜索不到解决方案，故作文以填补空缺。

## 解决

这个错误信息表明我们的 Azure 订阅中未注册 Microsoft.Compute 命名空间，因此无法在该命名空间中创建资源，包括虚拟机和  SSH 私钥。为了解决这个问题，我们需要在 Azure 订阅中注册 Microsoft.Compute 命名空间。我们可以按照以下步骤来注册：

1. 登录Azure门户（https://portal.azure.com），点击订阅

   ![image-20230513160115141](https://gitee.com/Marches7/piture-bed/raw/master/img/image-20230513160115141.png)

2. 选择需要注册 Microsoft.Compute 命名空间的订阅

   ![image-20230513160303522](https://gitee.com/Marches7/piture-bed/raw/master/img/image-20230513160303522.png)

3. 点击资源提供程序，找到 Microsoft.Compute 并单击它，然后点击注册。

   ![image-20230513160924090](https://gitee.com/Marches7/piture-bed/raw/master/img/image-20230513160924090.png)

4. 注册完成后，就可以在该订阅中创建虚拟机和下载 SSH 私钥了，如果还是无法下载 SSH 私钥，可以尝试在创建虚拟机时使用密码身份验证替代 SSH 私钥身份验证。

5. 使用SSH连接虚拟机

   当创建完虚拟机后，获取虚拟机公网地址，用户名即之前填写的虚拟机名称，用户密钥在下面的页面上传

   ![image-20230513173152454](https://gitee.com/Marches7/piture-bed/raw/master/img/image-20230513173152454.png)