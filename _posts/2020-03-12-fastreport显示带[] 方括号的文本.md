---
layout:     post
title:      fastreport显示带[] 方括号的文本或表达式
subtitle:   修改Brackets属性改变变量标识符
date:       2020-3-12
author:     王帅
catalog: true
tags:
    - fastreport
    - winfrom
    - Brackets
typora-root-url: ..

---

## 引言

Fastreport默认使用`[]`标识变量及表达式，如果你显示的文本中有方括号就会报错

```c#
    FastReport.TextObjectBase.CalcAndFormatExpression(String expression,Int32 expressionIndex)
    FastReport.TextObject.GetData()
    FastReport.BandBase.GetData()
    FastReport.Engine.ReportEngine.PrepareBand(BandBase band, Boolean getData)        	FastReport.Engine.ReportEngine.ShowBandToPreparedPages(BandBase band, Boolean getData)
    FastReport.Engine.ReportEngine.ShowBand(BandBase band, Boolean getData)
    FastReport.Engine.ReportEngine.ShowDataBand(DataBand dataBand, Int32 rowCount)
    FastReport.Engine.ReportEngine.RunDataBand(DataBand dataBand, Int32 rowCount, Boolean keepFirstRow, Boolean keepLastRow)
    FastReport.Engine.ReportEngine.RunDataBand(DataBand dataBand)
    FastReport.Engine.ReportEngine.RunBands(BandCollection bands)       FastReport.Engine.ReportEngine.RenderOuterSubreports(BandBase parentBand)
    FastReport.Engine.ReportEngine.ShowBand(BandBase band, Boolean getData)
    FastReport.Engine.ReportEngine.ShowDataBand(DataBand dataBand, Int32 rowCount)
    FastReport.Engine.ReportEngine.RunDataBand(DataBand dataBand, Int32 rowCount, Boolean keepFirstRow, Boolean keepLastRow)    FastReport.Engine.ReportEngine.ShowGroupTree(GroupTreeItem root)     FastReport.Engine.ReportEngine.ShowGroupTree(GroupTreeItem root)
    FastReport.Engine.ReportEngine.RunGroup(GroupHeaderBand groupBand)
    FastReport.Engine.ReportEngine.RunBands(BandCollection bands)
    FastReport.Engine.ReportEngine.RunReportPage(ReportPage page)
    FastReport.Engine.ReportEngine.RunReportPages()
    FastReport.Engine.ReportEngine.RunReportPages(ReportPage page)
    FastReport.Engine.ReportEngine.Run(Boolean runDialogs, Boolean append, Boolean resetDataState, ReportPage page)
    FastReport.Report.Prepare(Boolean append)
```

此时你首先想到的就是转义，其实`Fastreport`提供了简单的方法

## 解决办法

变量标识符叫`Brackets`，它默认是`[,]`，你可以把它替换成其它的`"<,>",`、` "<!,!>"`；但是任何其它的标识符也难免有作为文本的时候，此时你可以删除`Brackets`属性，那它就会直接把你的值作为变量来使用，而不会解析文本中的`[]`

## 参考资料：

* [escape-square-brackets-in-fast-report-strings](https://stackoverflow.com/questions/12636732/escape-square-brackets-in-fast-report-strings)
* [Displaying the expressions](https://www.fast-report.com/documentation/UserManFrNET-en/index.html?textobjectexpressions.htm)
* [Expressions](https://fastreports.github.io/FastReport.Documentation/Expressions.html)

