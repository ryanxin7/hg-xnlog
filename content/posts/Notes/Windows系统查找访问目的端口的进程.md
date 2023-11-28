---
author: Ryan
title: Windows 系统查找访问目的端口号的进程
date: 2023-11-24
lastmod: 2023-11-24
tags:
  - 终端安全
categories:
  - 网络安全
expirationReminder:
  enable: false
---

## Windows 系统查找访问目的端口号的进程



要查找当前电脑正在访问目标地址的端口的进程，在 Windows 上，可以使用 `netstat` 命令结合筛选条件进行操作。

但是，Windows 默认的 `netstat` 命令可能不提供访问目标地址的选项。

一个替代方法是使用 PowerShell 来执行这个任务。

<br>



## 1.查找目的地址对应端口的PID

1. **打开 PowerShell**：

   - 在开始菜单中搜索并打开 PowerShell。

2. **执行命令**： 在 PowerShell 中输入以下命令：

   ```powershell
   Get-NetTCPConnection | Where-Object {$_.RemotePort -eq 1777} | Select-Object OwningProcess
   ```

这将列出所有连接到端口 1777 的进程的 PID（OwningProcess）。

<img src="https://cdn1.ryanxin.live/image-20231124103515027.png" alt="image-20231124103515027" style="zoom:80%;" />



<br>



## 2.查找进程

- 使用任务管理器或 `tasklist` 命令来查找该 PID 对应的进程，步骤与上述相同。

例如：查看 tasklist 列表并过滤 141380的进程，查看是那个应用程序。

<img src="https://cdn1.ryanxin.live/image-20231124103748382.png" alt="image-20231124103748382" style="zoom:80%;" />

