---
layout:     post
title:      无需VPS,使用重新打tag的方式 拉取、转存mcr.microsoft.com的镜像
subtitle:   使用travis-ci的Trigger build功能一键转存
date:       2020-1-17
author:     王帅
catalog: true
tags:
    - docker
    - mirror
    - tag
    - travis-ci
    - docker-images
typora-root-url: ..
---

## 引言

用`docker`的都知道镜像加速，通过配置阿里云、腾讯云的镜像源确实可以大幅提升`docker pull`的效果，但对于某些不常用或新版的镜像却收效甚微。

比如最近想要部署私有ExceptionLess服务，它的dockerfile使用了`mcr.microsoft.com/dotnet/core/sdk:2.2.401`和`mcr.microsoft.com/dotnet/core/aspnet:2.2`，这两个镜像拉取非常慢，只能使用手动`tag`的方式来解决。

## 解决办法

----------

**2020-3-9**更新:

### 使用Azure官方镜像mirror

根据此issue:[dockerhub.azk8s.cn how to get the multiple path image](https://github.com/Azure/container-service-for-azure-china/issues/52)，可使用`mcr.azk8s.cn`仓库替换`mcr.microsoft.com`，即，将`mcr.microsoft.com/dotnet/core/aspnet:2.2`替换为`mcr.azk8s.cn/dotnet/core/aspnet:2.2`

-------

### 手动Tag

如果你有一台国外的VPS，那很简单；只需执行`docker pull`、`docker login`、`docker tag`、`docker push`就OK了。无奈我的VPS到期了，只能另寻它法，google之后从这篇文章找到灵感：[使用重新打 tag 的方式，拉取 k8s.gcr.io 的镜像](https://www.zhoujiangang.com/p/fetch-google-image-use-tag/)。

### 使用travis-ci自动Tag

使用第三方的CI、CD服务`push`镜像到国内私有仓库中解决`docker pull`过慢的问题，本例的CI-CD使用的是[travis-ci](https://travis-ci.com/)，由于自己使用了腾讯云的容器服务，因此私有库使用了腾讯云提供的免费镜像仓库。

* 首先，去腾讯云中创建保存`image`的仓库
![image.png](/img/qcloud_images_list.png)
* 其次，访问[travis-ci](https://travis-ci.com/)，并授权其对自己git库的访问权限(**随便什么仓库都行，因为我们并不需要使用里面的代码**)
* 参照腾讯云提供的镜像使用指引，编写`.travis.yml`脚本文件
![image.png](/img/qcloud_images_guid.png)
我的脚本文件如下：

```bash
language: bash
services:
  - docker
sudo: required
branches:
  only:
    - master
script:
  - docker pull mcr.microsoft.com/dotnet/core/sdk:2.2.401
  - docker pull mcr.microsoft.com/dotnet/core/aspnet:2.2

after_success:
  #登录registry的用户名是您的腾讯云的账号ID，密码是您开通镜像仓库服务时设置的密码
  - sudo docker login --username=[ID] --password=[密码] ccr.ccs.tencentyun.com
  - sudo docker tag mcr.microsoft.com/dotnet/core/sdk:2.2.401 ccr.ccs.tencentyun.com/dotnet-core/sdk:2.2.401
  - sudo docker tag mcr.microsoft.com/dotnet/core/aspnet:2.2 ccr.ccs.tencentyun.com/dotnet-core/aspnet:2.2
  - sudo docker push ccr.ccs.tencentyun.com/dotnet-core/sdk:2.2.401
  - sudo docker push ccr.ccs.tencentyun.com/dotnet-core/aspnet:2.2
```

* 在[travis-ci](https://travis-ci.com/)的首页上，任意选择一个仓库，然后在`More Options`→`Trigger build`的`CUSTOM CONFIG`中粘贴写好的`.travis.yml`脚本并点击`Trigger custom build`
![image.png](/img/travis-ci_Trigger_Custom_Build.png)
* 稍等几分钟即可在`job log`中看到脚本的执行结果
![image.png](/img/travis-ci_wait_job_result.png)