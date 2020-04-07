---
title: 装机必备软件一键安装
date: 2018-12-31 21:53:09
categories:
  - Maintenance
tags:
  - software
  - Windows
  - one-key
  - install
  - Chocolatey
---

## 缘起

接着[《编程入门之开发工具一键安装》][1]的思路，**电脑维修一键装机**也可用 [Chocolatey][2] 实现！

回想我 2008~10 年在[川大飞扬][3]做骨干技术员时还没这么好的东西呢，**PowerShell** 也还没普及，要用 CMD、WSH/JS 绞尽脑汁地封装各种**维修工具**…… 今天算是给学弟学妹补上这个遗憾~

## 一键脚本

Talk is cheap, show me the code!

```powershell
function getChoco() {

    Set-ExecutionPolicy Bypass -Scope Process -Force;

    Invoke-Expression (
        (New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1')
    );
}


choco -v

if (! $?) {  getChoco  }


choco install -y \
    boxstarter.winconfig ChocolateyGUI \
    everything notepad3 7zip 360ts \
    firefox tim wps-office-free

Import-Module Boxstarter.Chocolatey

Set-WindowsExplorerOptions -EnableShowHiddenFilesFoldersDrives -EnableShowFileExtensions

Install-WindowsUpdate -AcceptEula
```

懂点计算机的都能大概看明白上面的**脚本程序**在干啥，不懂的只要有*高考英语水平*，用“看日文中的汉字”的笨办法也能了解一二，我就不解释了~

## 软件管家？

上述脚本只装了普通中国大陆人最通用的几个软件，但针对大学不同专业的学生、社会上不同职业的员工，他们常用的专业**软件集合**又各不相同，即便计算机、软件专业，不同技术架构的程序员也需要[不同的开发环境][4]……

大神说：“要有个[软件管家][5]！”

> https://boxstarter.org

Chocolatey 官方团队早给各位“伸手党”准备好了，但他们设计的巧妙之处在于 ——

> 软件集 即是 软件包

可能借鉴了 **UNIX 一切皆文件**的思想，这样只需给[软件集发布者][6]封装一些方便的工具、服务，而用户还是在 [Chocolatey 官方软件仓库][7]中搜索、安装，让大家都简单~

## Awesome

按着官方文档很快就把上述**软件集脚本**发布成一个包 `China-mainland-suite`：

- [软件包主页](https://chocolatey.org/packages/China-mainland-suite/)

- [一键安装](https://boxstarter.org/package/China-mainland-suite/)

依 **GitHub 社群的 awesome（真香）传统**，我来填补一下 Chocolatey 的 awesome 空白 ——

> https://github.com/TechQuery/Chocolatey-awesome

## 参考资料

1.  https://www.pstips.net/powershell-online-tutorials

2.  https://blog.csdn.net/kk185800961/article/details/49026637

[1]: /development/coder-start-kit/
[2]: https://chocolatey.org/
[3]: https://www.fyscu.com/
[4]: /development/coder-start-kit/#%E6%96%B0%E7%94%B5%E8%84%91%E7%9A%84%E5%88%9B%E4%B8%96%E7%BA%AA
[5]: http://soft.360.cn/
[6]: https://boxstarter.org/Learn/SimplePackage
[7]: https://chocolatey.org/packages
