---
layout:     post
title:      devexpress如何使用PropertyGridControl编辑字典(Dictionary)
subtitle:   add/remove dictionary item in devexpress PropertyGridControl
date:       2021-9-24
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

[`PropertyGridControl`](https://docs.devexpress.com/WindowsForms/119885/controls-and-libraries/property-grid)默认使用[`PropertyGrid`](https://docs.microsoft.com/en-us/dotnet/api/system.windows.forms.propertygrid)的集合编辑器，对于一个Dictionary对象是不能添加和删除的

如下图所示：

![image-20210924163519947](/img/dev_PropertyGridControl_defaultDictionaryEditor.png)

要想实现添加/删除功能，需要继承[`UITypeEditor`](https://docs.microsoft.com/en-us/dotnet/api/system.drawing.design.uitypeeditor?view=netframework-4.7.2&f1url=%3FappId%3DDev16IDEF1%26l%3DEN-US%26k%3Dk(System.Drawing.Design.UITypeEditor);k(TargetFrameworkMoniker-.NETFramework,Version%253Dv4.7.2);k(DevLang-csharp)%26rd%3Dtrue)实现一个自定义编辑器

### 解决方案

Google之后发现已经有人实现了这一功能：[GenericDictUiTypeEditor](https://github.com/TechSmith/GenericDictUiTypeEditor)

由于[`PropertyGridControl`](https://docs.devexpress.com/WindowsForms/119885/controls-and-libraries/property-grid)继承自[`PropertyGrid`](https://docs.microsoft.com/en-us/dotnet/api/system.windows.forms.propertygrid)，因此[`PropertyGridControl`](https://docs.devexpress.com/WindowsForms/119885/controls-and-libraries/property-grid)的大部分需求可参照[`PropertyGrid`](https://docs.microsoft.com/en-us/dotnet/api/system.windows.forms.propertygrid)实现

参考[Github](https://github.com/TechSmith/GenericDictUiTypeEditor)为项目添加[Nuget](https://www.nuget.org/packages/GenDictEdit/)引用然后使用`EditorAttribute(typeof(GenericDictionaryEditor<string,string>), typeof(System.Drawing.Design.UITypeEditor))`即有增删功能

![image-20210924165415825](/img/dev_PropertyGridControl_GenericDictUiTypeEditor.png)



### 参考资料

* [Using a Dictionary in a propertygrid](https://stackoverflow.com/a/13107534/7960551)
* [https://github.com/TechSmith/GenericDictUiTypeEditor](https://github.com/TechSmith/GenericDictUiTypeEditor)

