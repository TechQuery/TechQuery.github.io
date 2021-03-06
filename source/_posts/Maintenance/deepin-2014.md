---
title: 【爱之深】Linux Deepin 2014 试用札记
date: 2015-02-28 15:26:00
author: 南漂一卒
categories:
  - Maintenance
tags:
  - Linux
  - Deepin
  - system
  - software
  - bug
---

【原文链接】https://my.oschina.net/TechQuery/blog/380778

## 高度赞扬

作为一个关注 **中国开源社区、深度技术论坛** 近十年的技术员、程序员，私以为 —— **Linux Deepin 2014** 是 第一款真正意义上中国人（主导开发）的 PC 操作系统！也是**继 Ubuntu 之后，全球第二个真正好用的 Linux 桌面发行版**！

主要理由 ——

1.  **稳定的专职开发团队**（感谢 深度技术论坛 创始人 刘闻欢）
2.  **与国际开源社区接轨的运作模式**
3.  **国内外程序员的大力支持**
4.  **自主研发、简洁易用的安装工具、桌面环境**

## 较大的 Bug

### 官方安装 U 盘制作工具 名成实败

- 【环境】自攒台式机 —— Intel 奔腾 64bit 双核、ASUS 主板、SATA - IDE 兼容模式、WinXP Pro 32bit SP3
  简体中文版
- 【操作】制作 Deepin 2014.2 64bit 安装 U 盘
- 【症状】U 盘格式化被格式化为一个 8MB 的隐藏分区、一个占用剩余空间、没有任何文件的 Windows 可用分区，开机引导时（传统 BIOS）只有最低分辨率黑屏闪光标（EFI 引导就直接跳过 U 盘进本机系统了）

### WUBI 安装等进度条时，介绍动画不显示

- 【环境】同上
- 【操作、症状】如题

### QQ 6.7 (in CrossOver) 运行几分钟就崩溃

- 【环境】Deepin 2014.2 64bit （WUBI 安装版）（32bit 安装后无此软件）
- 【症状】QQ 能正常运行客户端账号首次登陆后的所有操作，消息窗口也能正常使用，但几分钟后（即使 登陆后没有任何操作），QQ 自己的崩溃反馈窗口打开，并可以用它来重启，但重新登陆后故障反复
- 【解决】临时用 **DeepWINE-QQ 2012 国际版**

### 系统更新不可用

- 【环境】Deepin 2014.2（WUBI 安装版）、上海交通大学 镜像（自动测速选择的）、中国大陆官方稳定源（手动选择）
- 【操作】用 **深度软件中心** 更新系统软件包
- 【症状】**本机软件源包数据库**更新完之后，32bit 显示系统是最新的，64bit 能显示 187 个更新包，点击【全部升级】后报错“系统软件包依赖损坏”（刷新则反复同样的错误），如下图 ——
  ![Deepin 软件包依赖损坏][1]

### 官方 WUBI 安装工具 重启后不能进 Deepin

- 【环境】ThinkPad T440s、Intel 酷睿 i5、混合硬盘、Win 8.1 Lenovo OEM 中文版、Deepin 2014.2 64bit（本文写作环境）
- 【操作】WUBI 安装提示成功后点“现在重启”后直接 真重启，无人干预时直接进 Windows，且过程无任何变化（即 没有可以选择进 Deepin 的界面），在 msconfig 中也看不到 Deepin 的引导项
- 【解决】进 BIOS 的 Boot 列表看到“Deepin”，于是关掉“**Secure Boot**”并重启，引导前按【F 12】打开“**BIOS/EFI 引导菜单**”，选 Deepin 进入
- 【建议】即使不能操作 EFI 设置，也应该在 WUBI 安装成功后显示一个后续操作简介，用 Google 搜索 linuxdeepin.com 完全没有 **EFI/Win 8 双系统教程**……（Google 全网也没有 **ThinkPad Win 8、Deepin 双系统教程**）

### GRUB 2 引导界面 Deepin 美化版不显示

- 【环境】同上（官方 WUBI 安装）
- 【操作】每次进系统时，Deepin 美化过的 **GRUB 引导界面** 都不能显示，只显示 黑底白字的默认界面

### 快捷键设置【Windows Logo 键 + 方向键】无效

- 【环境】同上（官方 WUBI 安装）
- 【操作】系统设置面板中设置【最大化/恢复窗口】快捷键为【Super + ↑/↓】
- 【症状】虽显示设置成功，但实际各种窗口均只响应方向键的操作
- 【建议】**WinNT 6.x 内核**开始标配的**【Windows Logo 键 + 方向/数字键】系列快捷键** 非常方便，特别是“配合数字键来启动/切换任务栏上的程序”，对**键盘党**来说非常实用！Deepin 的首批铁杆用户必定是一批**开源程序员**，希望能先服务好这样的人群～

## UED 细节不足

### 键盘按键太灵

- 【环境】同上
- 【现象】键盘**组合快捷键** 快速重复响应（如 Google Chrome 中【Ctrl + Tab】切换选项卡，按一下换好几个）
- 【解决】在系统全局设置面板的【键盘和语言】中把“重复延迟”、“重复速率”均调到中间值
- 【建议】各种设备参数默认值 参考 **ThinkPad、Mac Book**

### 默认工作区快捷键与 Google Chrome 冲突

- 【环境】同上
- 【现象】键盘按【Ctrl + 数字键】**切换 Chrome 网页标签** 被系统全局默认快捷键“切换到工作区 n”拦截
- 【解决】在系统全局设置面板的【快捷键 - 工作区】中把“切换到工作区 1~4”的设置清空
- 【建议】**系统全局默认快捷键** 避开 常用软件的默认设置

### 触控板双指滚动方向反了

- 【环境】同上
- 【解决】在系统全局设置面板的【鼠标和触摸板】中开启“自然滚动”（但此时“边缘滚动”的方向也改了）
- 【建议】同上（私以为，在设计的层面 —— **双指滚动** 是从模拟“人手抓握后拖拉”的动作设计出来的，可以叫做“**自然滚动**”；而 **单指边缘滚动** 旨在模拟“鼠标指针拖拽控件滚动条”，应该沿用以前的方向习惯。）

[1]: http://static.oschina.net/uploads/space/2015/0228/144405_Zmtb_1171658.png
