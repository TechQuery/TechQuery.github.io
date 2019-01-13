---
title: Intel XDK 跨平台 App 开发初体验
date: 2016-06-21 16:32
author: 南漂一卒
categories:
  - Development
tags:
  - Intel
  - SDK
  - IDE
  - PhoneGap
  - Cordova
  - Mobile
  - Web
  - Software
---


　　用 **HTML/CSS/JavaScript** 进行**移动端 App 跨平台开发**的开源旗手非 [**Adobe PhoneGap**](http://phonegap.com/) 莫属，它的开源核心 **[Apache Cordova](http://cordova.apache.org/)** 就像 Apple Safari、Google Chrome 的 Webkit 内核一样，驱动着国内外不少同类解决方案（如 [**Intel XDK**](https://software.intel.com/zh-cn/intel-xdk)、[WeX5](http://www.wex5.com)）。

　　但 PhoneGap 在开发时也有一些问题 ——

 1. Adobe 官方支持 PhoneGap 的 IDE 是 Dreamweaver（开源人肯定优先选开源产品）
 2. [Android 开发者真机预览 App](https://play.google.com/store/apps/details?id=com.adobe.phonegap.app) 没有官方下载链接（天朝将会上线的“谷歌市场”估计也不会同步 Google Play 所有的 App）
 3. Adobe 官方提供的 [PhoneGap 构建服务](https://build.phonegap.com/) _私有 App 免费服务配额_ 很有限
 4. Android 开发时用 **[CrossWalk](https://crosswalk-project.org/)**（Intel 开源的 Chromium 核心）替换 **WebView** 要自己折腾
 5. 群众反映的[某些问题](https://github.com/phonegap/phonegap-app-developer/issues/287#issuecomment-210536817) 似乎解决缓慢

　　上述这些问题正好被“牛逼”已久的 **Intel XDK** 解决了~

<iframe src="http://player.youku.com/embed/XNzU1Njk1MDU2" frameborder="0" allowfullscreen
    style="width: 100%; height: 80vh"></iframe>

[（在新网页中观看视频）](http://v.youku.com/v_show/id_XNzU1Njk1MDU2.html)

　　Intel XDK 整个上手过程还是比较顺利 ——

 1. 官网下载_安装包_（[中文版](https://software.intel.com/zh-cn/intel-xdk-early-access)还不是稳定版）
 2. 安装、启动后注册 **Intel 开发者账号**
 3. 从 _Template 或 Samples and Demos _中选一项创建应用（建议勾选“use **App Designer**”，有些模板有[“所见即所得”的**拖拽 UI 控件模式**](https://software.intel.com/en-us/xdk/docs/app-designer-overview)）
    ![App 设计器](http://static.oschina.net/uploads/img/201606/22103244_0LsM.png)
 4. 写好自己的程序后即可到 _Build 选项卡_中选择 **App 打包目标平台
    ![构建前提示添加数字证书](http://static.oschina.net/uploads/img/201606/22095531_mjsf.png)**
 5. 点击 IDE 界面上的提示链接，会跳转到 App Build Settings 页面，再其中完善一下 App 相关信息（若需要 **CrossWalk** 来优化性能，请选择 **Embedded 运行时**，因为 _Shared_ 只会从 _Google Play_ 自动安装共享库，天朝用户只能用 20+ MB 的**静态编译版 APK** 了……）
    ![Build 设置 CrossWalk 优化](http://static.oschina.net/uploads/img/201606/22103245_mgR9.png)
    ![CrossWalk 运行时类型](http://static.oschina.net/uploads/img/201606/22103245_rwa7.png)
 6. 在上述界面中还需要新建一个** Developer Certificate**（相关信息的填写可参考 _Android 数字证书_ 的生成方法）
    ![添加开发者证书](http://static.oschina.net/uploads/img/201606/22095532_hmWi.png)
    ![新建 Android 数字证书 KeyStore](http://static.oschina.net/uploads/img/201606/22095532_wQHX.png)
 7. 再回到 _Build 选项卡_时可能会提示你 Unlock Certificate，输入之前设置的**证书密钥**即可
    ![Build 选项卡](http://static.oschina.net/uploads/img/201606/22095540_E52L.png)
 8. 终于，我们可以点击期盼已久 _Start Builds 按钮_了（_等进度条_是天朝擅长的……）
    ![等待构建](http://static.oschina.net/uploads/img/201606/22095550_kFO2.png)
 9. 构建成功后，你注册开发者账号的邮箱会收到一封内含下载链接的电邮（直接在 IDE 界面上点_下载按钮_是单线程下载……）
    ![App 构建下载链接](http://static.oschina.net/uploads/space/2016/0621/182435_X0QM_1171658.png)

　　Android App 安装、运行亲测结果 ——

 1. **ARM 架构版**：在 **[MIUI](http://www.miui.com/)** v7 上需开启“安装未知来源的应用”，运行正常！~
 2. **x86 架构版**：[BlueStacks](http://www.bluestacks.com/) 虚拟机安装成功，运行黑屏……

【参考文档】

 1. [Apache Cordova 官方中文文档](http://cordova.apache.org/docs/zh-cn/5.4.0/)
 2. [Intel XDK 构建选项](https://software.intel.com/en-us/xdk/docs/using-build-tab-and-build-settings)
 3. [Intel XDK 开发者账号证书管理](https://software.intel.com/en-us/xdk/docs/intel-xdk-certificate-management)
 4. [Android 数字证书概述](http://docs.apicloud.com/Dev-Guide/Android-License-Application-Guidance)
 5. [Java 证书工具讲解](http://www.micmiu.com/lang/java/keytool-start-guide/)
 6. [Android 应用签名机制](http://www.ourunix.org/post/146.html)
 7. [Intel CrossWalk 运行时的选择](https://software.intel.com/en-us/xdk/docs/choosing-crosswalk-build-options-shared-or-embedded)
 8. [通过 Cordova 插件添加 iOS WKWebView 支持](https://software.intel.com/en-us/xdk/docs/building-cordova-ios-app-with-wkwebview)
 9. [Intel XDK AJAX 域名白名单](https://software.intel.com/en-us/articles/cordova-whitelisting-with-intel-xdk-for-ajax-and-launching-external-apps)
