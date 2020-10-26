---
layout: post
title: Windows 系统资源使用情况监测脚本
categories: Windows
description: Windows 系统资源使用情况监测脚本
keywords: windows, 资源监测
---

- 半小时监测一次服务器系统资源使用情况，如超过设置阈值，则通过短信api接口发送告警短信

- fbi_warning.vbs
- edit by 佩奇

```vbs
On Error Resume Next
'设定阈值
Dim cpu_lim
cpu_lim=90
Dim mem_lim
mem_lim=90
Dim disk_lim
disk_lim=90

'排除盘符
disk_pc = array("A","C","H")

'接口地址
Dim url
url = "http://xxxx/notice"
Dim params   '传参

'写入文件日志
set fs = createobject("scripting.filesystemobject")
set f = fs.opentextfile("C:\sys_warn.log",8,true)
Dim log

'当前时间
Dim time 
time = now()

strComputer = "."

'获取ip
Dim ip
ip = GetIP()


'获取cpu使用率
Set wmi = GetObject("winmgmts:\\" & strComputer & "\root\cimv2") 
Set items=wmi.ExecQuery("Select * from Win32_Processor",,48)
Dim sum
sum=0
dim num
num = 0
'对CPU使用率求和统计cpu个数
For Each i In items
    sum=sum+i.LoadPercentage
	num = num + 1
Next

If Round(sum/num) > cpu_lim then

params = "{""ip"":""" & ip & """,""content"":""" & "告警：" & time & "目前" & ip & "系统的CPU使用率" & Round(sum/num) & "%,已超过设定阈值, 请排查进程CPU使用情况是否正常。" & """}"
call HttpRequest(url,"POST",params)

log = "告警：" & time & "目前" & ip & "系统的CPU使用率" & Round(sum/num) & "%,已超过设定阈值, 请排查进程CPU使用情况是否正常。"
f.writeline log

else

log = "日志：" & time & "目前" & ip & "系统的CPU使用率" & Round(sum/num) & "%。"
f.writeline log

end If

'获取内存使用率
set objWMI = GetObject("winmgmts:\\" & strComputer & "\root\cimv2")
set colOS = objWMI.InstancesOf("Win32_OperatingSystem")
for each objOS in colOS
dim mem_total,mem_free,mem_bili 
mem_total = round(objOS.TotalVisibleMemorySize / 1024) & "M"
mem_free = round(objOS.FreePhysicalMemory / 1024) & "M"
mem_bili = Round(((objOS.TotalVisibleMemorySize-objOS.FreePhysicalMemory)/objOS.TotalVisibleMemorySize)*100)
Next

If mem_bili > mem_lim then

params = "{""ip"":""" & ip & """,""content"":""" & "告警：" & time & "目前" & ip & "系统内存使用率" & mem_bili & "%,系统总内存" & mem_total & ",空闲内存" & mem_free &",已超过设定阈值, 请排查进程内存使用情况是否正常。" & """}"
call HttpRequest(url,"POST",params)

log = "告警：" & time & "目前" & ip & "系统内存使用率" & mem_bili & "%,系统总内存" & mem_total & ",空闲内存" & mem_free &",已超过设定阈值, 请排查进程内存使用情况是否正常。"
f.writeline log

else

log = "日志：" & time & "目前" & ip & "系统内存使用率" & mem_bili & "%,系统总内存" & mem_total & ",空闲内存" & mem_free &"。"
f.writeline log

end If

'获取硬盘使用率
Set fsoobj = CreateObject("Scripting.FileSystemObject")
 DriversInfo = GetDriversInfo
 DriversInfo = Replace(DriversInfo, "|", vbCrLf)
 sReturn = DriversInfo
Function GetDriversInfo()

   GetDriversInfo = ""
   Set drvObj = fsoobj.Drives
   For Each D In drvObj
       Err.Clear
       If InStr(Join(disk_pc, "|"), D.DriveLetter) = 0 Then
           If D.isReady Then
			   Dim disk_name,disk_total,disk_free,disk_bili
			   disk_name = D.DriveLetter
			   disk_total = cSize(D.TotalSize)
			   disk_free = cSize(D.FreeSpace)
			   disk_bili = Round((100*((D.TotalSize-D.FreeSpace)/D.TotalSize)))
			   			   
			   If disk_bili > disk_lim then
			   
			   params = "{""ip"":""" & ip & """,""content"":""" & "告警：" & time & "目前" & ip & "系统" & disk_name & "盘使用率" & disk_bili & "%,磁盘总空间" & disk_total & ",空闲空间" & disk_free &",已超过设定阈值, 请排查磁盘使用情况是否正常。" & """}"
               call HttpRequest(url,"POST",params)
			   
			   log = "告警：" & time & "目前" & ip & "系统" & disk_name & "盘使用率" & disk_bili & "%,磁盘总空间" & disk_total & ",空闲空间" & disk_free &",已超过设定阈值, 请排查磁盘使用情况是否正常。"
			   f.writeline log
			   
			   else
			   
			   log = "日志：" & time & "目前" & ip & "系统" & disk_name & "盘使用率" & disk_bili & "%,磁盘总空间" & disk_total & ",空闲空间" & disk_free &"。"
			   f.writeline log
			   
			   end IF
             Else
           End If
         Else
       End If
   Next
End Function

'关闭文件
f.close

'处理大小显示
 Function cSize(tSize)

     If tSize >= 1073741824 Then
         cSize = Int((tSize / 1073741824) * 1000) / 1000 & " GB"
       ElseIf tSize >= 1048576 Then
         cSize = Int((tSize / 1048576) * 1000) / 1000 & " MB"
       ElseIf tSize >= 1024 Then
         cSize = Int((tSize / 1024) * 1000) / 1000 & " KB"
       Else
         cSize = tSize & "B"
     End If

End Function

'获取ip
Function GetIP
    GetIP = ""    
    Dim objWMIService, colAdapters, objAdapter
    Set objWMIService = GetObject("winmgmts:\\" & strComputer & "\root\cimv2")
    Set colAdapters = objWMIService.ExecQuery("SELECT * FROM Win32_NetworkAdapterConfiguration WHERE IPEnabled = True")
    If colAdapters.Count = 0 Then
        Exit Function
    End If
    If Ubound(colAdapters.ItemIndex(0).IPAddress) = 0 Then
        'Exit Function
    End If

    GetIP = colAdapters.ItemIndex(0).IPAddress(0)

End Function

'接口调用函数
Function HttpRequest(url,mode,params)
Dim oauth_http
Set oauth_http=CreateObject("Microsoft.XMLHttp")
oauth_http.Open mode,url,False
If UCase(mode) = "POST" Then
oauth_http.setRequestHeader "Content-Type","application/json"
End If
oauth_http.Send(params)
HttpRequest = oauth_http.responseText
Set oauth_http=nothing
End Function
```

