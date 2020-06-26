---
layout:     post
title:      使用PowerShell查看使用RDP登录到远程服务器的客户端IP
subtitle:   Get the login ip of RDP client
date:       2020-6-26
author:     王帅
catalog: true
tags:
    - RDP
    - mstsc
    - ip
    - PowerShell
typora-root-url: ..
---

### 引言
TeamViewer是一个非常好的远程支持工具，但是我们公司资金有限，所以只在局域网里面配备了一台正版客户端；当同事需要远程支持的时候再使用局域网RDP（mstsc）连接到那台PC。这个teamviewer有一个限制，就是一台电脑只能一个用户使用，当你其它用户登录的时候再连接就会提示无法连接。基于这个原因，我需要知道是谁登录到这个用户了。

### 解决方案

参考[这篇文章](https://serverfault.com/questions/361565/how-can-i-get-the-ip-address-of-a-remote-desktop-client-and-how-can-i-trigger-a)我编写了一个PowerShell脚本来通知当前登录的IP

```powershell
$output= netstat -n | find --% ":3389" | find --% "ESTABLISHED"

##Invoke-RestMethod -Uri "https://sc.ftqq.com/[SCKey].send?text=mstscLogin&desp=$output"
$Uri = 'https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=[workWXkey]'

$json2= "{        ""msgtype"": ""text"",        ""text"": {           ""content"": ""$output""        }   }"
 $json2
$Result = Invoke-RestMethod -Uri $Uri -Method Post -Body $JSON2 -ContentType "application/json"
$Result
```

通知的方式我选择的是微信，如果你常用企业微信，那你可以参考[这篇文章](https://work.weixin.qq.com/api/doc/90000/90136/91770)配置群机器人得到通知消息，如果不常用企业微信，那可以用[Server酱](http://sc.ftqq.com/3.version)，登录后把脚本中的Key替换成自己的就能收到通知了（微信不能发送空消息，所以你要在远程的电脑上执行脚本才有效果）

### 常见问题

* 脚本的执行方式

由于我登录的用户只有一般的权限，所以我使用任务计划（Task scheduler），创建一个每次连接到用户（虽然我期望的是登录用户执行我的脚本，但其实这里触发器的触发条件应该用连接用户)就执行的计划来执行这个脚本

* 连接到用户后弹出命令行执行窗口

如果你选择连接到用户后执行上述的PowerShell脚本，那会弹出一个一闪而过的命令行黑窗，参考[这个文章](https://social.technet.microsoft.com/Forums/windows/en-US/24d1b052-b56d-4a34-b39b-602ca84cf4bd/task-scheduler-hidden-powershell-with-no-popup?forum=winserverpowershell)，使用以下VBS脚本执行PowerShell代码，这样就不会弹窗了

```powershell
command = "powershell.exe -nologo -command C:\Scripts\YourScript.ps1"
 set shell = CreateObject("WScript.Shell")
 shell.Run command,0
```



