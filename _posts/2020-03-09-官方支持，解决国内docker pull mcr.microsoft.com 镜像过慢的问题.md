---
layout:     post
title:      官方支持,解决国内docker pull mcr.microsoft.com 镜像过慢的问题
subtitle:   使用Azure提供的 mcr.azk8s.cn 国内官方镜像仓库pull dotnet 镜像
date:       2020-3-9
author:     王帅
catalog: true
tags:
    - docker
    - mirror
    - docker-images
typora-root-url: ..

---

## 解决办法

~~根据此issue:[dockerhub.azk8s.cn how to get the multiple path image](https://github.com/Azure/container-service-for-azure-china/issues/52)，可使用`mcr.azk8s.cn`仓库替换`mcr.microsoft.com`，即，如果你需要`mcr.microsoft.com/dotnet/core/aspnet:2.2`，那你将`mcr.microsoft.com`替换为`mcr.azk8s.cn`;变成`mcr.azk8s.cn/dotnet/core/aspnet:2.2`即可~~

此仓库已被限制为只能在Azure中使用，如果想要有更快的`docker pull` 速度，可以参考另一篇文章[无需VPS,使用重新打tag的方式 拉取、转存mcr.microsoft.com的镜像](/2020/01/17/无需VPS-使用重新打tag的方式-拉取-转存mcr.microsoft.com的镜像/)