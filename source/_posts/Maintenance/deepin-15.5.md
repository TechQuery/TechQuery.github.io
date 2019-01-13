---
title: Deepin 15.5 使用体验
date: 2018-04-02 22:28
author: 南漂一卒
categories:
  - Technology
  - Maintenance
tags:
  - Linux
  - Deepin
  - system
  - software
  - bug
---


## 硬盘真机安装

 - http://reboot.pro/topic/19464-booting-linux-distros-iso-with-grub4dos/
 - https://sourceforge.net/p/clonezilla/discussion/Clonezilla_live/thread/35ef89a5/


## 虚拟硬盘安装

因一时没配置好 Deepin Live ISO 的 GRUB4DOS 引导脚本，就暂时用了 WUBI（Windows 体验式安装）。但意外的是，公司 Intel Core i7-7700K 4.2GHz、32GB 内存的台式机，安装后半段进度条几度便秘……


## 坑爹的 N 卡开源驱动

安装后首次进桌面，感觉鼠标指针、dock、侧边栏都有卡顿，心中不妙…… 再点 launcher，逐行扫描过程清晰可见，快赶上当年装 WINE-QQ 调不起显卡、全靠 CPU 绘图一样…… 再加上“检查系统更新”菊花转半天，“失望”情绪油然而生……

放弃之前，还是点开“驱动管理器”碰碰运气，还好检测到了 N 卡私有驱动，等了一阵子装好了，似乎仍不见起色…… 重启？就好了……（为啥不弹框引导用户一下呢？）


## 漫长的首次装机

虽然“同步阻塞”是大多数**操作系统内置包管理器**的基本设计原则，但 Linux 大多数软件包最方便的安装方式就是包管理器，不像 Windows、Mac OS X 很多常用软件都可以从官网下载编译好的二进制包，所以系统升级、软件安装都要占用包管理器，系统安装到配齐环境往往要等很久……


## 版本之痛

Web 前端、JavaScript 全栈开发现在都依赖 NodeJS，它不像 Python 被大多数 Linux 软件包依赖，基本只用在 Web 应用开发，而且又因社区活跃而迭代很快，在不少 Linux 软件源中的版本都比较旧，只能手动安装新版。

可谁承想，[官方安装脚本](https://nodejs.org/en/download/package-manager/#debian-and-ubuntu-based-linux-distributions)却把 Deepin 的系统信息识别为 `unstable` 版本…… 只能又求助于 NVM —— https://github.com/creationix/nvm#install-script ，终于装上了……
