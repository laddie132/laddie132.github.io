---
title: 谷歌云平台免费搭建个人博客
urlname: app_engine_wp
date: 2019-04-04 11:25:58
categories:
- Dev

tags: 
- GCP
- WordPress
---

谷歌云平台（Google Cloud Platform，GCP）的$300体验政策对开发者真的是十分友好，网上有各种各样的无限试用教程，大家可以参考。基于此，本文介绍下如何使用谷歌云平台部署WordPress博客。
<!-- more -->

## 方法一：虚拟机服务
最简单的方法，当然是直接使用谷歌云创建VM实例，启动虚拟机后，就可以和本地一样搭建WordPress应用了。

## 方法二：App Engine
这个服务可以说是谷歌云的特色了，部署起来更为简单直接，自带一个以项目ID开头的域名。创建项目后，并开启结算服务就可以部署了。

### Step1：创建Cloud SQL
直接在Console中创建SQL实例，并选择MySQL类型，注意选择合适的地区，国内推荐asia-east-2（香港）。此外，也可以选择主机类型，这跟付费价格有关。
![SQL](https://ws4.sinaimg.cn/large/006tKfTcly1g1qgbcaok5j3085085q34.jpg)

之后在该SQL实例上创建一个WordPress使用的数据库，如下图所示：
![DB](https://ws3.sinaimg.cn/large/006tKfTcly1g1qgb8nkfmj30be08c3z0.jpg)

这里数据库就创建完成了，可以在Cloud Shell中测试，也可以使用`cloud_sql_proxy`工具映射到本地进行测试。

### Step2：创建WordPress项目
首先需要安装Composer工具，这是一个PHP的包管理器，请自行下载安装。

在本地的某个空文件夹中，执行如下命令，安装谷歌云工具。
```
composer require google/cloud-tools
```

下载完成后，执行如下命令，会创建一个新的WordPress项目，这期间会询问你若干个问题，包括数据库名称、密码等配置。
```
php vendor/bin/wp-gae create
```

配置完成后，你会发现这里已经有了一个新的WordPress目录，该目录与直接从WordPress官网下载解压的文件不太相同，主要包含了部署至App Engine所必须的一些文件，如app.yaml、cron.yaml、php.ini和gae-app.php等。

### Step3：部署至App Engine
部署过程需要安装GCloud SDK工具，请自行下载安装。之后，直接在WordPress主目录，执行如下命令，即可完成部署。
```
gcloud app deploy app.yaml cron.yaml
```
> 注意上述命令需要事先配置好gcloud，包括你的google账号、项目ID等。

## 注意事项
- App Engine中的文件是不允许修改的，因此，如果需要更新项目，需要重新部署一遍，在App Engine启动另一个版本的服务即可
- 因为App Engine不允许修改，所以WordPress无法直接上传文件等，可以借助自带的GCS插件，绑定谷歌云的存储服务，之后的文件都会上传至谷歌云的存储器中
- WordPress安装插件无法借助谷歌云的存储器，需要在本地安装后，重新部署至App Engine

## 参考
- 官方文档[https://cloud.google.com/community/tutorials/run-wordpress-on-appengine-standard]