---
title: 【笔记】移动 Web 前端开发 那些蛋疼事儿
date: 2014-10-31 05:34:00
author: 南漂一卒
categories:
  - Development
tags:
  - Web
  - mobile
  - issue
  - collection
---


**移动 Web 前端开发的设备适配** 现在有种“**兼容老 IE**”的蛋疼感…… 希望刚刚正式颁布的 **HTML 5 标准**可以让这种现在渐渐改变，而不是又一个“Web 噩梦时代”的开始……

以下是一些**优秀参考文章**的搜集，是**本人肉眼过滤**于 sogou.com；而非 URL 的条目则是 **肉测心得** ——


## 响应式设计入门

 1. http://www.cnblogs.com/vajoy/p/3903591.html
 2. http://jinjuan.me/webapp-share-zte/


## User Agent（网页客户端 标识符）

 - http://outofmemory.cn/code-snippet/1901/mobile-liulanqi-Us


## Meta/Viewport 标签（移动设备适配元数据）

 1. http://sowm.cn/wpjam/article/5FBDBB430CC9E424299B61EC03BF5C42.html
 2. http://wiki.eoeandroid.com/Targeting_Screens_from_Web_Apps
 3. http://wanglery.iteye.com/blog/1926747
 4. http://www.cnblogs.com/stephenykk/p/3822441.html


## 屏幕像素比

 - http://www.zhangxinxu.com/wordpress/2012/08/window-devicepixelratio/


## Target DensityDPI（安卓中等屏幕专用）

 - http://blog.csdn.net/fengri5566/article/details/9414599


## WebView 怪癖（***微信内置浏览器*** 等）

 1. **JavaScript 动态设置 Viewport** 时 `device-width` 无效，需手动指定 具体像素值（如 320）
 2. 其 **device-dpi** ≠ medium-dpi（设置无效），最佳效果是手动指定为 `medium-dpi`
 3. **Android** 有些版本的 **WebView 组件**中 `<input type="file" />` 无效，它的调用从底层被拦截，在 JavaScript 层面无异常抛出，程序没被中断


## position: fixed 大坑

 1. https://github.com/maxzhang/maxzhang.github.com/issues/11
 2. https://github.com/maxzhang/maxzhang.github.com/issues/2
 3. https://github.com/maxzhang/maxzhang.github.com/issues/7


## 移动版网页布局技巧

 1. http://www.w3cplus.com/mobile/mobile-terminal-refactoring-mobile-layout.html
 2. http://blog.csdn.net/hursing/article/details/9186199


----------


## 移动端网页调试 —— HTTP

### 公网（weinre + 花生壳）
 1. http://www.cnblogs.com/lhb25/p/debug-mobile-site-and-app-with-weinre.html
 2. http://segmentfault.com/q/1010000000941321
 3. http://service.oray.com/question/2480.html

### 内网（UC 浏览器开发者版 + WiFi）
 1. http://www.uc.cn/business/developer/
 2. http://plus.uc.cn/document/webapp/doc5.html

### 内网（Firefox 42 + WiFi）
 1. https://developer.mozilla.org/zh-TW/docs/Tools/Remote_Debugging/Debugging_Firefox_for_Android_over_Wifi


----------


## 线上网页 JS 日志

 1. http://jslogger.com


----------


## 触控事件的蛋疼

 1. http://select.yeeyan.org/view/213582/202991
 2. http://www.cnblogs.com/pifoo/archive/2011/05/23/webkit-touch-event-1.html
 3. http://www.aliued.cn/2013/04/27/%E7%A7%BB%E5%8A%A8%E5%BC%80%E5%8F%91%E4%B9%8Btouch-event%E7%AF%87.html
 4. http://www.web-tinker.com/article/20364.html


## GPU 硬件加速渲染

 1. http://blog.bingo929.com/transform-translate3d-translatez-transition-gpu-hardware-acceleration.html

（绝对是 **未完待续**……）
