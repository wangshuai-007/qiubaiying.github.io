---
layout:     post
title:      如何简单的启动一组dotnet服务,powershell?
subtitle:   start a group dotnet server with powershell or tye
date:       2022-3-23
author:     王帅
catalog: true
tags:
    - powershell
    - server
    - dotnet
    - tye
typora-root-url: ..
---

### 引言

我们知道使用`dotnet`命令可以方便的启动c#程序，假如我有一组相互关联的服务，一般在测试的时候可以打开多个visual studio启动新实例

不过这只在需要调试它们的时候才有必要，一般也只需要分别在`bin\debug`中启动命令就行

每天写代码的时候都去启动这些服务未免有些麻烦,因此有了这篇文章.



### 解决方案

##### 复杂的解决方案

要用命令行启动这些服务是要解决一些问题的,最关键的是状态检测;因为你直接用`dotnet`命令启动的是一个`dotnet`命令行窗口

一旦有多个服务启动了,且每个服务有很多的日志你就不知道哪些服务没启动成功;下面放出我写的一个示例

```powershell
# 确保管理员权限运行脚本
if (-NOT ([Security.Principal.WindowsPrincipal][Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole] "Administrator"))  
{  
  $arguments = "& '" +$myinvocation.mycommand.definition + "'"
  Start-Process powershell -Verb runAs -ArgumentList $arguments
  Break
}

$version="net5.0"
# 相对路径启动dll
$dlls=@(
    "XXXX.XXXX"
    "YYYY.YYYY"
)

# $dlls | ForEach-Object { &()}
for ( $index = 0; $index -lt $dlls.count; $index++)
{
    # $path=" $PSScriptRoot\{0}\bin\Debug\$version\{0}.dll" -f $dlls[$index]
    $dllName=$dlls[$index];

    $path=" $PSScriptRoot\{0}\bin\Debug\$version\" -f $dllName

    $cmd='cd '+ $path;
    Invoke-Expression $cmd

    $file=$path+$dllName+".dll"

    cmd.exe /c "start  ""$dllName"" powershell.exe -NoExit -Command  ""dotnet .\$dllName.dll"""

    "$dllName execed"
}

# 绝对路径启动dll
$exes=@(
    "D:ZZZZ\bin\Debug\$version",
    "D:XXXX\bin\Debug\$version"
)
for ( $index = 0; $index -lt $exes.count; $index++)
{
    # $path=" $PSScriptRoot\{0}\bin\Debug\$version\{0}.dll" -f $dlls[$index]
    $path=$exes[$index]

    $cmd='cd '+ $exes[$index];
    Invoke-Expression $cmd

    $name=Get-ChildItem -filter *.exe -name
    $name=[System.IO.Path]::GetFileNameWithoutExtension($name)
    $file=$path+$name

    cmd.exe /c "start  ""$file"" powershell.exe -NoExit -Command  ""dotnet .\$name.dll"""


    "$file execed"
}

"all server started."
```

这里还推荐使用[Seq](https://datalust.co/seq),方便在启用这些服务后查看日志。

-------

以上为文章原文：

下面放出简单的解决方案：

##### 简单的解决方案

那就是[Tye](https://github.com/dotnet/tye)

使用Tye,只需要:

* 安装`tye`命令行工具

 ```powershell
 dotnet tool install -g Microsoft.Tye --version "0.11.0-alpha.22111.1"
 ```

* 编写程序启动`yaml`配置文件

```yaml
name: servername
services:
- name: identityService
  project: .\InterFace\xxxx.IdentityService.csproj
  build: false #不编译项目直接启动
  bindings:
  - port: 8080
- name: xxxxService
  project: .\InterFace\xxxxService.csproj
  build: false
  bindings:
  - port: 8088  
```

* 使用`tye run D:\test\tye.yml`或直接使用`tye run`启动工具
* 打开`localhost:8000`查看服务启动情况

![tye dashboard](/img/tye_dashboard.png)

现在,你的服务都由`tye`帮你接管了

### 参考资料
* [tye documentation](https://github.com/dotnet/tye/tree/main/docs)
* [https://thecloudblog.net/post/distributed-tracing-in-asp.net-core-with-jaeger-and-tye-part-2-project-tye/](https://thecloudblog.net/post/distributed-tracing-in-asp.net-core-with-jaeger-and-tye-part-2-project-tye/)
* [https://stackoverflow.com/questions/10262231/obtaining-exitcode-using-start-process-and-waitforexit-instead-of-wait](https://stackoverflow.com/questions/10262231/obtaining-exitcode-using-start-process-and-waitforexit-instead-of-wait)
* [https://stackoverflow.com/questions/3382082/whats-the-nohup-on-windows](https://stackoverflow.com/questions/3382082/whats-the-nohup-on-windows)
* [https://stackoverflow.com/questions/6604089/dynamically-generate-command-line-command-then-invoke-using-powershell](https://stackoverflow.com/questions/6604089/dynamically-generate-command-line-command-then-invoke-using-powershell)
* [https://github.com/dotnet/runtime/issues/2688#issuecomment-339577646](https://github.com/dotnet/runtime/issues/2688#issuecomment-339577646)

