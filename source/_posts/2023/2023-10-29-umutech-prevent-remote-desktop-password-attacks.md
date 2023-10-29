---
layout: post
title: 防爆破远程桌面密码
date: 2023-10-29 02:16:00
description: ChatGPT 说：网络太危险了！
categories: UMUTech
tags:
- ops
- powershell
- security
- windows
---
## 问题

面向公网开了个远程桌面端口，无法用多少号端口，都会被爆破！

## 分析

改端口是没用的，因为只有 65535 个，只要机器 IP 被发现，扫描端口很快就能完成。

封 IP 是有用的，虽然爆破者（攻击者）有很多 IP，但一定是有限的，有多少封多少！

## 解决

1. 从系统日志里分析爆破者 IP

打算使用 PowerShell 7 来编写脚本，首先学习 `Get-WinEvent` 命令：

```powershell
Get-Help Get-WinEvent -Online
```

日志的过滤条件可以用“事件查看器”来协助生成：

![WinEvent UI](/images/2023/20231029-winevent-ui.png)
![WinEvent XML](/images/2023/20231029-winevent-xml.png)

当然，以上全部 XML 是 -FilterXml 的参数，比较长，可以用 -FilterXPath 来简化，只需要中间一部分。

条件还可以再加上 LogonType，以缩小范围，其中 3 表示“网络登录”：

```powershell
Get-WinEvent -LogName 'Security' -FilterXPath '*[System[EventID=4625] and EventData[Data[@Name="LogonType"]=3]]' -MaxEvents 1 | Format-List Message
```

以下是[完整代码](https://github.com/UMU618/windows-scripts/blob/master/pwsh/list-logon-failed-ips.ps1)，它会打印出 IP，和这个 IP 的登陆失败次数：

```powershell
$ips = @{}
Get-WinEvent -LogName Security -FilterXPath '*[System[band(Keywords,4503599627370496)] and EventData[Data[@Name="LogonType"]=3]]' | %{
    $xml = [xml]$_.toXml()
    foreach ($data in $xml.Event.EventData.Data) {
        if ($data.Name -eq 'IpAddress') {
            $ip = $data.'#text'
            if ($ips.ContainsKey($ip)) {
                ++$ips[$ip]
            } else {
                $ips.Add($ip, 1)
                'Fisrt find ' + $ip + ' on ' + $_.TimeCreated
            }
        }
    }
}
Write-Output 'All IPs:', $ips
```

2. 把爆破者 IP 加入防火墙，阻止它们

目前收集到这些：

```
112.184.96.197
118.45.92.239
122.38.193.166
138.199.21.245
14.116.196.99
14.49.207.162
141.98.11.119
141.98.11.58
147.78.47.57
148.113.4.245
176.111.174.173
176.111.174.174
179.60.147.13
182.180.92.224
185.161.248.145
188.250.64.50
222.191.242.228
45.143.201.62
58.221.4.54
58.33.52.84
62.122.184.88
67.159.237.58
77.90.185.132
78.128.114.18
89.248.163.94
89.248.163.95
91.191.209.202
91.240.118.187
91.240.118.29
```

打开 `wf.msc`，新建一个阻止型的防火墙策略，然后加入到“作用域”的“远程 IP 地址”里。

![Firewall](/images/2023/20231029-firewall.png)

加防火墙也可以用 PowerShell 搞定，这次先偷个懒，下次再说吧！
