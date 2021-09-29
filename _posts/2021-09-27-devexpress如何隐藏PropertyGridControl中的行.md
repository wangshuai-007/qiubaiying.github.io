---
layout:     post
title:      devexpress如何隐藏PropertyGridControl中的行
subtitle:   hide row in devexpress PropertyGridControl(VerticalGrid)
date:       2021-9-27
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

[`PropertyGridControl`](https://docs.devexpress.com/WindowsForms/119885/controls-and-libraries/property-grid)实际是[Vertical Grid](https://docs.devexpress.com/WindowsForms/2449/controls-and-libraries/vertical-grid)，它的每一行对应的是[PropertyGridControl.SelectedObject](https://docs.devexpress.com/WindowsForms/DevExpress.XtraVerticalGrid.PropertyGridControl.SelectedObject)中的属性，如果我想根据值隐藏行应该怎么做呢？

### 解决方案

假如我现在只需要显示`Course.Id=2`的`DicClassRoomId_Name`栏位，查看[Tree Traversal](https://docs.devexpress.com/WindowsForms/479/controls-and-libraries/vertical-grid/data-layout-records-rows-and-cells/rows/tree-traversal)可知,可以通过[VGridRowsIterator.DoOperation](https://docs.devexpress.com/WindowsForms/DevExpress.XtraVerticalGrid.Rows.VGridRowsIterator.DoOperation(DevExpress.XtraVerticalGrid.Rows.RowOperation)) 方法遍历整个树达到自定义行内容的功能。

为此需要实现一个[RowOperation](https://docs.devexpress.com/WindowsForms/DevExpress.XtraVerticalGrid.Rows.RowOperation) (完整代码在文末)

```c#
public class SetDicRowVisibleRowOperation : DevExpress.XtraVerticalGrid.Rows.RowOperation
{
    public override void Execute(DevExpress.XtraVerticalGrid.Rows.BaseRow row)
    {
        Console.WriteLine(row.Properties.Caption);   
        if (row.Tag is Course course)
        {
            var fieldName = nameof(Student.ListCourses) + "." + course.IndexName + "." +
                               nameof(Course.DicClassRoomId_Name);
            if (row.Properties.FieldName == fieldName)
                row.Visible = course.Id == 2;
        }
    }
}
```

在遍历之前，还需为Row保存相关信息

```c#
private void Form1_Load(object sender, EventArgs e)
{
    var student = GetTestStudent();
    propertyGridControl1.SelectedObject = student;
    propertyGridControl1.OptionsBehavior.PropertySort = DevExpress.XtraVerticalGrid.PropertySort.NoSort;
    propertyGridControl1.ExpandAllRows();

    //set row Tag info
    student.ListCourses.ForEach(x =>
                {
                    //IndexName set by CourseCollectionPropertyDescriptor constructor
                    var fieldName = nameof(Student.ListCourses) + "." + x.IndexName + "." +
                    nameof(Course.DicClassRoomId_Name);
                    var row = propertyGridControl1.GetRowByFieldName(fieldName);
                    row.Tag = x;
                });

    propertyGridControl1.RowsIterator.DoOperation(new SetDicRowVisibleRowOperation());
}
```

由于上一篇文章[devexpress如何在PropertyGridControl中直接编辑集合成员](/2021/09/26/devexpress如何在PropertyGridControl中直接编辑集合成员/)的影响，`FieldName`发生了变化，此处根据`PropertyDescriptor`的逻辑生成了`FieldName`。

![hide dic row](/img/dev_PropertyGridControl_hideDicRow.png)



### 参考资料

* [PropertyGridControl custom RowOperation doesn't retrieve all properties](https://supportcenter.devexpress.com/ticket/details/t432094/propertygridcontrol-custom-rowoperation-doesn-t-retrieve-all-properties)
* [Tree Traversal](https://docs.devexpress.com/WindowsForms/479/controls-and-libraries/vertical-grid/data-layout-records-rows-and-cells/rows/tree-traversal)
* 示例项目源码：[DevPropertyGridControlExpandCollection_HideDicRow](https://github.com/wangshuai-007/BlogsSample/tree/HideDicRow/DevPropertyGridControlExpandCollection)

