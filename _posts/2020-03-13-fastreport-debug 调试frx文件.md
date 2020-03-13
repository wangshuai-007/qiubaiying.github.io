---
layout:     post
title:      fastreport debug 调试frx文件
subtitle:   使用visual studio 跟踪frx流程查看frx变量
date:       2020-3-13
author:     王帅
catalog: true
tags:
    - fastreport
    - winfrom
    - Debug
typora-root-url: ..

---

## 引言

`Fastreport`自带设计器的代码编辑功能非常糟糕，如果你写了很多逻辑在frx文件里面，你几乎无法debug调试，唯有使用`MessageBox.Show()`方法查看变量的值，而如果你要查看一个循环中的值。。。

## 解决办法

`Fastreport`的frx文件提供有导出为C#代码的功能，使用visual studio执行C#代码即可解决调试难的问题

## 步骤

打开frx文件，选择file→Save As→保存类型→C#file

![saveAsC#](/img/fastreport_saveAsCSharpfile.png)

生成的文件可能有错误，删除无效的引用后一般就可以编译通过了

在控制台中使用以下代码

```c#
//using FastReport;
var reportCtrl = new frx();//frx保存的c#类

//设置报表参数
foreach (var item in paramsDic.Keys)
{
    reportCtrl.SetParameterValue(item, paramsDic[item]);
}

//注册报表数据源
reportCtrl.RegisterData(dataByte);

reportCtrl.Show();

```

在生成的frx里面打上断点,执行上述代码就能中断并查看变量了

![image-20200313175310786](/img/fastreport_debug.png)