---
layout:     post
title:      devexpress如何在PropertyGridControl中直接编辑集合成员
subtitle:   在 devexpress PropertyGridControl中展开(expand)集合(collection)成员(item)
date:       2021-9-26
author:     王帅
catalog: true
tags:
    - devexpress
    - winfrom
    - propertyGridControl
    - propertyGrid
typora-root-url: ..
---

### 引言

[`PropertyGridControl`](https://docs.devexpress.com/WindowsForms/119885/controls-and-libraries/property-grid)默认使用[`PropertyGrid`](https://docs.microsoft.com/en-us/dotnet/api/system.windows.forms.propertygrid)的集合编辑器，如果你要编辑集合中的项，必须点击按钮打开集合编辑器

如下图所示：

![default collection editor](/img/dev_PropertyGridControl_defaultCollectionExpand.png)

从图中不难发现，List是可以直接编辑`Capacity`属性的，那可以直接把`Course`对象变成List的成员脱离集合编辑器来编辑吗？

### 解决方案

答案当然是可以的，通过观察可以发现，[`PropertyGridControl`](https://docs.devexpress.com/WindowsForms/119885/controls-and-libraries/property-grid)可展开编辑的就是Object，所以只需把集合中的成员映射为List中的Object即可，[`PropertyGrid`](https://docs.microsoft.com/en-us/dotnet/api/system.windows.forms.propertygrid)已经有相应的解决方案了。

使用中间类型`CourseCollection`然后实现[`ICustomTypeDescriptor`](https://docs.microsoft.com/en-us/dotnet/api/system.componentmodel.icustomtypedescriptor?view=netframework-4.7.2&f1url=%3FappId%3DDev16IDEF1%26l%3DEN-US%26k%3Dk(System.ComponentModel.ICustomTypeDescriptor);k(TargetFrameworkMoniker-.NETFramework,Version%253Dv4.7.2);k(DevLang-csharp)%26rd%3Dtrue)接口和[PropertyDescriptor](https://docs.microsoft.com/en-us/dotnet/api/system.componentmodel.propertydescriptor?view=netframework-4.7.2&f1url=%3FappId%3DDev16IDEF1%26l%3DEN-US%26k%3Dk(System.ComponentModel.PropertyDescriptor);k(TargetFrameworkMoniker-.NETFramework,Version%253Dv4.7.2);k(DevLang-csharp)%26rd%3Dtrue)即可完成此需求，完整代码在文末

![expand collection as property](/img/dev_PropertyGridControl_customCollectionExpand.png)

假如你想要自定义类型名称可继承[`ExpandableObjectConverter`](https://docs.microsoft.com/en-us/dotnet/api/system.componentmodel.expandableobjectconverter?view=netframework-4.7.2&f1url=%3FappId%3DDev16IDEF1%26l%3DEN-US%26k%3Dk(System.ComponentModel.ExpandableObjectConverter);k(TargetFrameworkMoniker-.NETFramework,Version%253Dv4.7.2);k(DevLang-csharp)%26rd%3Dtrue)

![custom property name](/img/dev_PropertyGridControl_customExpandableObjectConverter.png)

### 参考资料

* [Customized Display of Collection Data in a PropertyGrid](https://www.codeproject.com/Articles/4448/Customized-Display-of-Collection-Data-in-a-Propert#_articleTop)

* 示例项目源码：[DevPropertyGridControlExpandCollection](https://github.com/wangshuai-007/BlogsSample/tree/master/DevPropertyGridControlExpandCollection)
