---
layout:     post
title:      单元测试CommandLineUtils中的命令
subtitle:   UnitTest the command of CommandLineUtils
date:       2020-4-6
author:     王帅
catalog: true
tags:
    - CommandLineUtils
    - UnitTests
    - dotnet tool
typora-root-url: ..
---

### 引言

使用[CommandLineUtils](https://github.com/natemcmaster/CommandLineUtils)可以创建便捷的dotnet tool工具,官方的[docs / samples /](https://github.com/natemcmaster/CommandLineUtils/blob/master/docs/samples)中包含有很多示例;但是对于如何运行`Command`却没有明示,如果你要单元测试自己创建的命令,要到[test](https://github.com/natemcmaster/CommandLineUtils/tree/master/test/CommandLineUtils.Tests)目录下参考官方的单元测试代码,以下列出自己实践的方法.

## 解决办法

* 对于不使用依赖注入的`Command`,直接使用`CommandLineApplication.Execute<Show>("-a");`即可,其中`Show`为`[Command("show", Description = "")]`
* 对于使用依赖注入的`Command`可以参考这个示例

```c#
var commandLineApplication = new CommandLineApplication<Show>();
var services = new ServiceCollection()
    .AddLogging()
    .AddSingleton(Program.Configuration)
    .AddSingleton(Program.Configuration.Get<APPConfig>())//添加依赖注入
    .BuildServiceProvider();

commandLineApplication.Conventions
    .UseDefaultConventions()
    .UseConstructorInjection(services);

commandLineApplication.Parse();

var result = commandLineApplication.Execute(@"--path=/CUSTDB/1901-0048.rar", "--taskid=1907-7777");//执行命令

result.Should().Be(0);//using FluentAssertions
```

