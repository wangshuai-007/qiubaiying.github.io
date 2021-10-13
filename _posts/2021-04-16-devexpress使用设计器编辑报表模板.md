---
layout:     post
title:      devexpress使用设计器编辑报表模板
subtitle:   ReportDesigner.exe的自实现
date:       2021-4-16
author:     王帅
catalog: true
tags:
    - devexpress
    - winfrom
    - report
    - XtraReport
typora-root-url: ..
---

### 引言
fastreport由于是单独的报表解决方案，因此原生支持报表模板修改，如果遇到改字体颜色这种小的需求，客户可以自己打开设计器编辑报表搞定；而devexpress的报表如果是在visual studio里面创建的话默认为带设计器的cs文件，如果要让客户可以直接修改报表模板，需要额外做一些操作。

以下示例所用devexpress的版本为18.1

### 解决方案

##### 1.准备repx模板文件

首先需要准备报表模板文件，devexpress的模板文件类型为repx，可以从设计器里面新建或者另存为也可以在visual studio中把cs设计文件导出

![image-20210416094435696](/img/devexpress_saveCSToRepx.png)

repx文件其实是一个xml，它保存了界面布局

##### 2.准备报表编辑器

fastreport在安装程序之后会自动有一个Report Designer.exe的报表编辑器，但是devexpress（版本18.1）却没有这样的设计器，所以还需要新建一个项目来编辑模板（如果你用的旧版如12.1，在安装的DemoCenter目录`"C:\Users\Public\Documents\DXperience 12.1 Demos\XtraReports\Bin\ReportDesigner.exe"`会有一个自带的设计器，也可以用这个exe来编辑报表）

设计器启动代码如下，完整demo见文末

```c#
private string repxPath = "";
private void Form1_Load(object sender, EventArgs e)
{
    this.Hide();

    var rpt = new XtraReport();
    if (File.Exists(repxPath))
        rpt.LoadLayout(repxPath);
    ReportDesignTool designTool = new ReportDesignTool(rpt);

    // Access the standard or ribbon-based Designer form and its MDI Controller.
    // IDesignForm designForm = designTool.DesignForm;
    XRDesignRibbonForm designForm = designTool.DesignRibbonForm as XRDesignRibbonForm;
    if (designForm != null)
    {
        designForm.OpenReport(rpt);
        if (File.Exists(repxPath))
        {
            var fileInfo = new FileInfo(repxPath);
            designForm.RibbonControl.AutoSaveLayoutToXmlPath = repxPath;
            designForm.Text = fileInfo.Name;
        } 

        designForm.ShowDialog();
    }

    this.Close();
}
```

##### 3.界面效果

编译好后打开生成的exe文件，打开之前导出的repx文件，这样就有了报表编辑功能，界面如图所示：

![report](/img/devexpress_DevReportDesigner_v18.1.4.0.png)

此设计器已发布，下载地址：[DevReportDesigner_v18.1.4.0](https://github.com/wangshuai-007/BlogsSample/releases/tag/DevReportDesigner_v18.1.4.0)

对于repx文件，`右键`→`打开方式`→`在这台电脑上查找其它应用`→`选择DevReportDesigner.exe`



如果要直接使用repx文件打印出报表，请参考官方文章：

[Loads the report definition from the specified REPX file.](https://docs.devexpress.com/XtraReports/DevExpress.XtraReports.UI.XtraReport.FromFile(System.String-System.Boolean))

数据绑定方法可查看官方问答：

[Binding Datatable to XRTable in XtraReport](https://supportcenter.devexpress.com/ticket/details/t369657/binding-datatable-to-xrtable-in-xtrareport)

### 参考资料

* [https://docs.devexpress.com/XtraReports/2666/detailed-guide-to-devexpress-reporting/store-and-distribute-reports/store-report-layouts-and-documents/load-report-layouts](https://docs.devexpress.com/XtraReports/2666/detailed-guide-to-devexpress-reporting/store-and-distribute-reports/store-report-layouts-and-documents/load-report-layouts)
* [https://docs.devexpress.com/XtraReports/DevExpress.XtraReports.UI.ReportDesignTool.ShowRibbonDesigner](https://docs.devexpress.com/XtraReports/DevExpress.XtraReports.UI.ReportDesignTool.ShowRibbonDesigner)
* 设计器demo地址：[https://github.com/wangshuai-007/BlogsSample/tree/master/DevReportDesigner](https://github.com/wangshuai-007/BlogsSample/tree/master/DevReportDesigner)
* 设计器下载地址：[DevReportDesigner_v18.1.4.0](https://github.com/wangshuai-007/BlogsSample/releases/tag/DevReportDesigner_v18.1.4.0)

