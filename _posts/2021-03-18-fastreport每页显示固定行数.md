---
layout:     post
title:      fastreport每页显示固定行数
subtitle:   多余行移动到下一页
date:       2021-3-18
author:     王帅
catalog: true
tags:
    - fastreport
    - winfrom
typora-root-url: ..
---

### 引言
最近遇到客户提的一个需求，他要一页只显示20行明细数据；一番Google之后找到了官方博客介绍相关解决办法:[How to display an estimated number of records on a report page](https://www.fast-report.com/en/blog/148/show/)于是我写了下面的代码

```c#
int counter = 0;
 
 private void Data1_BeforePrint(object sender, EventArgs e)
 {
     if (counter >= 20)
     {
         Engine.StartNewPage();//打印到新的页面
         counter = 0;
     }
     counter++;
 }
```

但是我预览出来，一页却只有18行；而且从设计器打印和直接打印预览的结果还不一样；

一番思考之后，我认为此处是`Report.DoublePass`设置导致的，在[Report settings](https://www.fast-report.com/documentation/UserMan/index.html?report_options.htm)的官方教程中有说明；为解决这个问题，才有了这篇文章

### 解决方案

由于`BeforePrint`事件的问题，我查看官方文档，找到一个`Report.Engine.RowNo`属性，于是我这样写

```c#
if (Report.Engine.RowNo%20==0)
    Engine.StartNewPage();
```

测试发现还是不行，每页数量依旧不对

多次预览调整后找到正确的条件判断：

```c#
if (Report.Engine.RowNo>20&& Report.Engine.RowNo%20==1)
    Engine.StartNewPage();
```

### 参考资料

* [How to display an estimated number of records on a report page](https://www.fast-report.com/en/blog/148/show/)

* [FastReport.Documentation](https://fastreports.github.io/FastReport.Documentation/ClassReference/api/FastReport.Engine.ReportEngine.html)