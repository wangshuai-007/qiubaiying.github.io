---
layout:     post
title:      Devexpress打印预览(PrintPreview)时修改打印数据
subtitle:   所见即所得的打印数据快速修改方法
date:       2017-10-7
author:     王帅
catalog: true
tags:
    - Devexpress
    - Print
    - winform
typora-root-url: ..
---
## 引言

打印功能对于一个系统来说通常是强需求，每个系统都会根据自己所提供的服务生成不同的打印模板，但光有模板往往不够用，因为有时会需要在打印预览(PrintPreview)时根据打印内容临时修改打印的数据，而且这种修改并不需要应用于打印数据源的修改，只需要打印出变更后的内容就行了；所以我们只需修改内存中的数据，然后重新打印就可以了。

在最新的Devexpress 17.1中[XRLable](https://documentation.devexpress.com/XtraReports/17403/Concepts/Report-Controls/Label)，[XRTableCell](https://documentation.devexpress.com/XtraReports/9741/Concepts/Report-Controls/Table)和[Character Comb](https://documentation.devexpress.com/XtraReports/117157/Concepts/Report-Controls/Character-Comb)有[EditOptions](https://documentation.devexpress.com/XtraReports/DevExpress.XtraReports.UI.XRLabel.EditOptions.property)属性，可以实现直接在打印预览界面(PrintPreview)上修改打印的数据，具体请参考[官方示例](https://documentation.devexpress.com/XtraReports/117343/Concepts/Creating-Reports/Navigation-and-Interaction/Content-Editing-in-Print-Preview)；但很可惜我相信大部分的系统都还和我用的一样，没有使用17.1，所以只能乖乖修改数据然后重新打印。

## 效果图
![打印预览界面双击修改打印数据](/img/reportPreview.gif)

## 步骤

- #### 首先我们设计好打印内容的模板

打印内容模板:

![打印内容模板](/img/report1.png)

​	打印预览效果图:

![打印预览效果图](/img/printpreviewbase0.png)

打印预览(PrintPreview)中的每一个Page表示一个model，我们需要一种方式来修改对应model的数据；通过查看[官方示例](https://documentation.devexpress.com/CoreLibraries/DevExpress.XtraPrinting.Brick.class)我们知道，打印预览界面(PrintPreview)的元素实际上都是Brick类型的子类，你点击每一个Page都触发的是[BrickClick](https://documentation.devexpress.com/WindowsForms/DevExpress.XtraPrinting.Control.PrintControl.BrickClick.event)事件：

![](/img/brickclick.png)

通过调试可以发现打印预览界面(PrintPreview)元素构成的层级关系：

![层级关系](/img/bricktable.png)

所以我们可以把每一个model的主键保存在每一个Page中，我选用了BrickTable（你也可以选择其它元素，只是在我的例子中刚好每一个Page对应于一个Table）

- 所以第二步是保存model的主键到BrickTable的AnchorName中（如果你在打印预览界面(PrintPreview)使用了目录功能，可能就需要用其它方式保存数据了）

```c#
/// <summary>
/// 保存的打印页主键列表
/// </summary>
private List<int> ListStudentId { get;set; }
/// <summary>
/// 生成Table明细数据之后的事件处理方法
/// </summary>
/// <param name="sender"></param>
/// <param name="e"></param>
private void Detail1_AfterPrint(object sender, EventArgs e)
{
    var dtRowView = DetailReport.GetCurrentRow() as DataRowView;
    if (dtRowView != null)
    {
        var studentId =Convert.ToInt32(dtRowView["StudentId"]);
     
        ListStudentId.Add(studentId);//将Key记录下来
    }
}
```
* 第三步，在生成打印明细数据时将主键赋值到BrickTable的AnchorName上

```c#
/// <summary>
/// 将生成打印明细的数据打印到界面上的事件处理方法
/// </summary>
/// <param name="sender"></param>
/// <param name="e"></param>
private void tbStudentDetail_PrintOnPage(object sender, PrintOnPageEventArgs e)
{
    if (ListStudentId.Any())
    {
        //将主键保存到生成的打印对象的ID中
        var iterator = new DevExpress.XtraPrinting.Native.NestedBrickIterator(Pages[e.PageIndex].InnerBricks);
        while (iterator.MoveNext())
        {
            var visualBrick = iterator.CurrentBrick as VisualBrick;
            if (visualBrick != null)
                //PutStateToBrick(visualBrick, PrintingSystem);
            {
                if (visualBrick.BrickType == "Table")
                //if (string.IsNullOrEmpty(visualBrick.Text) && string.IsNullOrEmpty(visualBrick.ID))
                {
                    if (ListStudentId.Any())
                    {
                        visualBrick.ID = "StudentId";
                        //visualBrick.Text = listKey[visualBrick.ID];
                        visualBrick.AnchorName = ListStudentId[0].ToString();
                        ListStudentId.RemoveAt(0);
                    }
                    else
                    {
                        break;
                    }
                }
            }
            //Do some actions based on your requirements
        }
        //}
        //ListStudentId.RemoveAt(0);
    }
}
```

- #### 剩下的就是在打印预览界面(PrintPreview)双击Page弹出修改属性窗体后重新打印数据

```c#
/// <summary>
/// 双击打印预览页面的事件处理方法
/// </summary>
/// <param name="sender"></param>
/// <param name="e"></param>
private void documentViewer1_BrickDoubleClick(object sender, DevExpress.XtraPrinting.Control.BrickEventArgs e)
{
    var studentId = string.Empty;
    var iterator = new DevExpress.XtraPrinting.Native.NestedBrickIterator(e.Page.InnerBricks);
    while (iterator.MoveNext())
    {
        VisualBrick visualBrick = null;
        try
        {
            visualBrick = iterator.CurrentBrick as VisualBrick;
        }
        catch (ArgumentOutOfRangeException)
        {
            //对弹出的窗口点击确认后会报此异常,因为超出了report的范围
            //对这种异常不做处理
        }
        if (visualBrick != null)
            //PutStateToBrick(visualBrick, PrintingSystem);
            //读取打印模板界面保存的report主键数据
        {
            if (visualBrick.ID == "StudentId")
            {
                studentId = visualBrick.AnchorName;
            }
        }

        //Do some actions based on your requirements
    }

    if(string.IsNullOrEmpty(studentId))return;
    else
    {
        //MessageBox.Show($"点击的学生主键为{studentId}");
        var newForm = new EditProperties(ListReport.First().PModel.ListStudents
            .SingleOrDefault(c => c.StudentId == Convert.ToInt32(studentId)));
        var dlg= newForm.ShowDialog();
        if (dlg == DialogResult.OK)
        {
            //重新打印
            var newReport=ListReport.First();
            newReport.SetDataSource(); 
            LoadData();
        }
    }
}
```

其中，修改属性的窗体使用了类似设计器属性修改界面的[PropertyGridControl](https://documentation.devexpress.com/WindowsForms/DevExpress.XtraVerticalGrid.PropertyGridControl.class)控件，当然你也可以自定义界面来实现类似功能，只要可以修改属性的值就行了

文章描述内容的示例项目已放在了[Github](https://github.com/wangshuai-007/BlogsSample)上。