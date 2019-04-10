---
title: 田窗里面开道场
date: 2019-01-21 20:31:18
author: 南漂一卒
categories:
  - Development
tags:
  - Docker
  - Windows
  - one-key
  - Chocolatey
---

每次大家一谈起 Docker，Windows 总能成为“黑点”，也不知道那些“果珍”哪来的自信……

别跟我说“不好装”！老子抄起 PowerShell 就是一把梭！**Docker 核心、命令行、Compose、kubectl、图形界面客户端**一条命令搞定！

```powershell
choco install docker-desktop docker-kitematic
```

上述不适用于 Win 10 家庭版、Win 7/8 又怎么了？官方也支持老系统啊~

```powershell
choco install docker-toolbox -ia \
    /COMPONENTS="kitematic,virtualbox,dockercompose" \
    /TASKS="desktopicon,modifypath,upgradevm"
```
