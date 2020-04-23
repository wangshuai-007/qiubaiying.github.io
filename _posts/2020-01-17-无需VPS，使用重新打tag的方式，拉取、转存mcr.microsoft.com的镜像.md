---
layout:     post
title:      无需VPS,使用重新打tag的方式 拉取、转存mcr.microsoft.com的镜像
subtitle:   使用Azure Devops的pipelines功能一键ReTag转存
date:       2020-1-17
author:     王帅
catalog: true
sitemap:
  lastmod:  2020-4-15
tags:
    - docker
    - Azure Devops    
    - mirror
    - tag
    - travis-ci
    - docker-images
typora-root-url: ..
---

## 引言

用`docker`的都知道镜像加速，通过配置阿里云、腾讯云的镜像源确实可以大幅提升`docker pull`的效果，但对于某些不常用或新版的镜像却收效甚微。

比如最近想要部署私有ExceptionLess服务，它的dockerfile使用了`mcr.microsoft.com/dotnet/core/sdk:2.2.401`和`mcr.microsoft.com/dotnet/core/aspnet:2.2`，这两个镜像拉取非常慢，只能使用手动`tag`的方式来解决。

另外，本人在腾讯云中创建了`dotnet-core`命名空间，源自`mcr.microsoft.com`可直接使用`ccr.ccs.tencentyun.com/dotnet-core/runtime`

![image-20200415142923048](/img/tencenyun_aspnet_core_repository.png)

## 解决办法

----------

**2020-04-15更新**

原标题为：*使用travis-ci的Trigger build功能一键转存*

但是本人多次实测，`travis-ci`的`Build`带宽限制严重，如果你要Retag `Runtime`还好，设置`travis_wait 30`基本也能用；但是如果是SDK，基本都会超时

```bash
Still running (50 of 300): sudo docker push ccr.ccs.tencentyun.com/dotnet-core/sdk:5.0
The job exceeded the maximum time limit for jobs, and has been terminated.
```

因此改用`Azure Devops`

------

**2020-3-9**更新:

### ~~使用Azure官方镜像mirror~~

~~根据此issue:[dockerhub.azk8s.cn how to get the multiple path image](https://github.com/Azure/container-service-for-azure-china/issues/52)，可使用`mcr.azk8s.cn`仓库替换`mcr.microsoft.com`，即，将`mcr.microsoft.com/dotnet/core/aspnet:2.2`替换为`mcr.azk8s.cn/dotnet/core/aspnet:2.2`~~

此方案只能在Azure中使用。。。

-------

### 手动Tag

如果你有一台国外的VPS，那很简单；只需执行`docker pull`、`docker login`、`docker tag`、`docker push`就OK了。无奈我的VPS到期了，只能另寻它法，google之后从这篇文章找到灵感：[使用重新打 tag 的方式，拉取 k8s.gcr.io 的镜像](https://www.zhoujiangang.com/p/fetch-google-image-use-tag/)。

### 使用Azure Devops

* 首先，去腾讯云中创建保存`image`的仓库
  ![image.png](/img/qcloud_images_list.png)

* 参考[create-a-service-connection](https://docs.microsoft.com/en-us/azure/devops/pipelines/library/service-endpoints?view=azure-devops&tabs=yaml#create-a-service-connection)，在project→project settings →Service connections →New ervice connection中新建一个名为`tencentyun`的`Docker Registry`，其中`Docker Registry`地址为：https://ccr.ccs.tencentyun.com/v2/，用户名和密码请查看腾讯云的镜像使用指引

![image-20200415111820887](/img/AzureDevops_createTencentyun_DockerRegistry.png)

* 在你项目里面的`Piplines`中新建一个连接到Github仓库的yml，创建模板随便选

![image-20200415135902763](/img/AzureDevops_CreatePipline_yml.png)

* 最后把Review中的yml文件替换成如下：

```yml
trigger:
- master

pool:
  vmImage: 'ubuntu-latest'

variables:
  tag: 'aspnet:5.0'

steps:
- task: Docker@2
  inputs:
    containerRegistry: 'tencentyun'
    command: 'login'
- task: PowerShell@2
  inputs:
    targetType: 'inline'
    script: |
      $env:version="$(tag)"
      
      docker pull mcr.microsoft.com/dotnet/core/$env:version
      
      docker tag mcr.microsoft.com/dotnet/core/$env:version ccr.ccs.tencentyun.com/dotnet-core/$env:version
      docker push ccr.ccs.tencentyun.com/dotnet-core/$env:version
```

* 然后点击`Save and run`(或者也可以在自己的仓库中创建一个上述的`azure-pipelines.yml`文件,在Azure Devops中连接好后提交)就可以了 

### ~~使用travis-ci自动Tag~~

`travis-ci`的`Build`带宽限制严重，已改用Azure Devops，以下步骤仅作参考

-----

使用第三方的CI、CD服务`push`镜像到国内私有仓库中解决`docker pull`过慢的问题，本例的CI-CD使用的是[travis-ci](https://travis-ci.com/)，由于自己使用了腾讯云的容器服务，因此私有库使用了腾讯云提供的免费镜像仓库。

* 首先，去腾讯云中创建保存`image`的仓库
![image.png](/img/qcloud_images_list.png)
* 其次，访问[travis-ci](https://travis-ci.com/)，并授权其对自己git库的访问权限(**随便什么仓库都行，因为我们并不需要使用里面的代码**)
* 参照腾讯云提供的镜像使用指引，编写`.travis.yml`脚本文件
![image.png](/img/qcloud_images_guid.png)
我的脚本文件如下：

```bash
language: bash
env:
  - version=sdk:2.2
services:
  - docker
sudo: required
branches:
  only:
    - master
script:
  - echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin ccr.ccs.tencentyun.com
  - docker pull mcr.microsoft.com/dotnet/core/$version

after_success:
  - sudo docker tag mcr.microsoft.com/dotnet/core/$version ccr.ccs.tencentyun.com/dotnet-core/$version
  - sudo docker push ccr.ccs.tencentyun.com/dotnet-core/$version
```

* 在[travis-ci](https://travis-ci.com/)的首页上，任意选择一个仓库，然后在`More Options`→`Settings`里面添加docker登陆的用户名密码环境变量`DOCKER_USERNAME`和`DOCKER_PASSWORD`

![image-20200410113527361](/img/travis-ci_setEnv.png)

* 在`More Options`→`Trigger build`的`CUSTOM CONFIG`中粘贴写好的`.travis.yml`脚本并点击`Trigger custom build`
![image.png](/img/travis-ci_Trigger_Custom_Build.png)
* ~~稍等几分钟即可在`job log`中看到脚本的执行结果~~

这里并不会立即出结果，图示是因为images已经存在于`ccr.ccs.tencentyun.com`中了，所以给人以速度很快的假象

![image.png](/img/travis-ci_wait_job_result.png)

### 参考链接

* [push-image](https://docs.microsoft.com/en-us/azure/devops/pipelines/ecosystems/containers/push-image?view=azure-devops)

* [share-docker-image](https://docs.travis-ci.com/user/build-stages/share-docker-image/)
* [docker](https://docs.travis-ci.com/user/docker/)