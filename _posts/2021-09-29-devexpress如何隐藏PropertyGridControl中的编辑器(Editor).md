---
layout:     post
title:      devexpress如何隐藏PropertyGridControl中的编辑器(Editor)
subtitle:   hide row value cell(editor) in devexpress PropertyGridControl(VerticalGrid)
date:       2021-9-29
author:     王帅
catalog: true
tags:
    - devexpress
    - winfrom
    - propertyGridControl
    - propertyGrid
    - VerticalGrid
typora-root-url: ..
---

### 引言

[`PropertyGridControl`](https://docs.devexpress.com/WindowsForms/119885/controls-and-libraries/property-grid)有两种视图模式([ActiveViewType](https://docs.devexpress.com/WindowsForms/DevExpress.XtraVerticalGrid.PropertyGridControl.ActiveViewType))，经典和office

如果使用office视图（经典视图为Grid展示方式，因此无法隐藏，只能自定义[ExpandableObjectConverter](https://docs.microsoft.com/en-us/dotnet/api/system.componentmodel.expandableobjectconverter?view=netframework-4.7.2&f1url=%3FappId%3DDev16IDEF1%26l%3DEN-US%26k%3Dk(System.ComponentModel.ExpandableObjectConverter);k(TargetFrameworkMoniker-.NETFramework,Version%253Dv4.7.2);k(DevLang-csharp)%26rd%3Dtrue)，具体可参考[CourseConverter](https://github.com/wangshuai-007/BlogsSample/blob/daf39bab640c7fdd4b8a5504ed64c4d9b1368f97/DevPropertyGridControlExpandCollection/DevPropertyGridControlExpandCollection/Student.cs#L123)）绑定一个前文：[devexpress如何在PropertyGridControl中直接编辑集合成员](/2021/09/26/devexpress如何在PropertyGridControl中直接编辑集合成员/)中的对象结果就是这样:

![office view](/img/dev_PropertyGridControl_customCollectionExpand_officeView.png)

现在,`ListCourses`这个对象已经被展开了,所以它的集合编辑器没那么必要,如果我想要把这个`Editor`去掉,有什么办法吗?

### 解决方案

官方并没有针对这个问题提供简单的解决方案,但是它有[CustomDrawRowValueCell](https://docs.devexpress.com/WindowsForms/DevExpress.XtraVerticalGrid.VGridControlBase.CustomDrawRowValueCell)事件

这个事件可以自定义Editor

所以,问题就比较简单了;只需要不绘制Cell即可达到隐藏控件的目的

```c#
private void PropertyGridControl1_CustomDrawRowValueCell(object sender, DevExpress.XtraVerticalGrid.Events.CustomDrawRowValueCellEventArgs e)
{
    if(e.Row.Properties.FieldName== nameof(Student.ListCourses))
    {
        e.Properties.AllowEdit = false;//解决点击编辑框位置时Editor出现的问题
        e.Handled = true;//不执行绘制操作即可隐藏编辑器
    }
}
```

效果如图:

![hide collection editor](/img/dev_PropertyGridControl_hideCollection_officeView.png)

### 参考资料

* [CustomDrawRowValueCell](https://docs.devexpress.com/WindowsForms/DevExpress.XtraVerticalGrid.VGridControlBase.CustomDrawRowValueCell)Event

