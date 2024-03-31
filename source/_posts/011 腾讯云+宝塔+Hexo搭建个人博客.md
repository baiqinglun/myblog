---
title: 腾讯云+宝塔+Hexo搭建个人博客
excerpt: 网上搜到的搭建教程好多都没用，至此完整记录一下搭建流程
sticky: 
published: true
date: "2024/03/19 22:44:25"
index_img: "https://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/image-20240318110440540.png"
category: 
- [技术]
tags:
- 技术分享
---

# 1、服务器购买

我这里买的是腾讯云的轻量服务器，选择centos版本。[网站](https://cloud.tencent.com/act/pro/2024spring?from=21932)

![image-20240318092649737](https://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/image-20240318092649737.png)

购买完成可在服务器列表中查看

![image-20240318092813799](https://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/image-20240318092813799.png)

点击服务器卡片上【更多】-【查看详情】，进入服务器。可先重置密码

![image-20240318092956444](https://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/image-20240318092956444.png)

![image-20240318093018683](https://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/image-20240318093018683.png)

重置密码后点击【一键登录】

![image-20240318093052025](https://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/image-20240318093052025.png)

# 2、宝塔安装

在终端输入宝塔的安装命令，[宝塔安装](https://www.bt.cn/new/download.html)

```bash
yum install -y wget && wget -O install.sh https://download.bt.cn/install/install_6.0.sh && sh install.sh ed8484bec
```

![image-20240318093320260](https://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/image-20240318093320260.png)

如果出现以下提示说明需要管理员权限。终端输入`su`进入管理员权限，然后执行安装。

![image-20240318093436581](https://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/image-20240318093436581.png)

安装好之后终端会显示宝塔访问端口号、账号和密码。输入【主机域名:端口号】进入宝塔面板，使用账号和密码登录。

# 3、网站搭建

登录成功后，安装php推荐套餐

添加站点

![image-20240318110109451](https://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/image-20240318110109451.png)

填写域名和网站的目录

![image-20240318110156648](https://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/image-20240318110156648.png)

在文件下面会看到刚才的文件夹【`qlbai.fun`】，把里面的文件全部删除。（如果没有自己创建一下）

![image-20240318110239010](https://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/image-20240318110239010.png)

把该文件夹下的全部文件删除，在本地博客目录下运行，生成public目录

```bash
hexo -g
```

生成public目录后压缩一下，然后上传至宝塔解压缩

![image-20240318110424392](https://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/image-20240318110424392.png)

然后启动项目

![image-20240318110440540](https://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/image-20240318110440540.png)

>  使用域名访问，需要先购买个域名，通过DNS解析到该主机地址上，还需要备案才能访问，过程就不演示了。

