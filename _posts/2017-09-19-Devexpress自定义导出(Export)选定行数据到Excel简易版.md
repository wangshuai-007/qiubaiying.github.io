---
layout:     post
title:      Devexpress自定义导出(Export)选定行数据到Excel简易版
subtitle:   使用自带GridView.ExportToXls方法快速导出
date:       2017-09-19
author:     王帅
catalog: true
tags:
    - Devexpress
    - Excel
    - winform
typora-root-url: ..
---
## 引言

我们知道，一般在winform中导出GridView的内容为Excel时会引用第三方库；如Aspose、NPOI等，它们需要你逐行设置导出的内容，虽然功能强大，但有时略显麻烦，可喜的是Devexpress中已经为我们实现了导出的方法[GridControl.ExportToXls](https://documentation.devexpress.com/WindowsForms/DevExpress.XtraGrid.GridControl.ExportToXls.overloads)，它可以将整个GridView全部导出，但通常我们仅需要导出选定行，此时需要做一些设置。

## 步骤

- 首先设置GridView允许选择多行以及选中行导出模式

```c#
//选择多行
GridView.OptionsSelection.MultiSelect = true;
//仅打印选中行
GridView.OptionsPrint.PrintSelectedRowsOnly = true;
```

- 然后使用Devexpress自带的导出功能自动打印

```c#
var fileName = string.Empty;

var dialog = new SaveFileDialog();
dialog.Title = "保存文件";
dialog.RestoreDirectory = true;
dialog.AddExtension = true;

////判断默认文件名称是否存在?若存在,则赋值
//if (string.IsNullOrEmpty(defaultFileName) == false)
//{
// dialog.FileName = defaultFileName;
//}

dialog.Filter = @"Excel 97-2003 文件|*.xls";

if (dialog.ShowDialog() == DialogResult.OK)
{
    fileName = dialog.FileName;
}
if (string.IsNullOrEmpty(fileName)) return;
try
{
    GridView.ExportToXls(fileName);
    MessageBox.Show("导出Excel成功");
}
catch (Exception ex)
{
    throw ex;
}
```

## 效果图

devexpress导出选中行效果图
![效果图](/img/导出选中行.png)

