---
layout:     post
title:      fastreport 查询系统变量总页数 Report.GetVariableValue("TotalPages")总是返回1
subtitle:   Report.Engine.CurPage的值正常而TotalPages总是1
date:       2020-3-16
author:     王帅
catalog: true
tags:
    - fastreport
    - winfrom
typora-root-url: ..

---



## 解决办法

* 返回0

参考官方文档[**Reference to system variables**](https://www.fast-report.com/documentation/UserManFrNET-en/index.html?usesystemvariablesinexpressions.htm)和[The TOTALPAGES variable always returns 0.](https://www.fast-report.com/en/faq/3/24/)，如果`Report.GetVariableValue("TotalPages")`总是返回0，那你应该在`Report`→`Options`中设置**Double Pass**

![image-20200316175041619](/img/fastreport_doublepass.png)

* 返回1

方法如果总是返回1，那你应该在取值上下文中取消**Reset Page Number**选项

![image-20200316175516777](/img/fastreport_GroupSetResetPageNumber.png)

