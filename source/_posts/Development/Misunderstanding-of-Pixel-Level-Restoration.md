---
title: 「像素级还原」的误解
date: 2020-05-22 15:13:46
categories:
  - Development
tags:
  - GUI
  - design
  - Web
  - front-end
  - CSS
  - layout
slidehtml: true
---

> freeCodeCamp 中文社区
>
> 首次直播分享会

---

## 互联网公司“金句”

---

> 这个很简单，技术我不懂，明天我就要！
>
> —— 产品经理

---

> 前端应该“像素级还原”设计图！
>
> —— UI 设计师

---

## Web 前端开发的真相

---

### 几乎不用“像素”这个单位

---

| 设计图<br>数据 |     | 浏览器<br>默认字号 |     | 代码中的数值<br>（两位精度） |
| :------------: | :-: | :----------------: | :-: | :--------------------------: |
|     `X`px      |  ÷  |       `16`px       |  =  |            `Y`rem            |

---

**边框粗细**、**圆角半径**、**阴影范围**、动画中 JS 计算出的**位移量**等除外

---

### 与设计完全不同的布局方式

---

#### 设计师的天马行空

---

PS、AI 等作图工具用**绝对定位**排布**图层**

类似用 CSS 给 `<div />` 设置 `position: absolute`

---

#### 工程师的条条框框

---

把设计图**从全局到局部**

切成一个个**嵌套的行列框**

---

![](https://cloud.netlifyusercontent.com/assets/344dbf88-fdf9-42bb-adb4-46f01eedd629/91bde744-f286-4f89-b3f7-63d26551a958/mrh-css-grid-fig-03-large-opt.png)

---

### 图是死的，页是活的

---

#### 设计效果图

一般最多两套 —— **桌面端**、**手机端**

桌面端分辨率往往是设计师 iMac 的 1920+

---

#### 屏幕适配

- 设备：手机、平板、笔记本、大屏台式机

- 操作：电脑窗口缩放、移动设备横竖屏

---

行列框会在多个范围间**自适应缩放**

<iframe
    style="width: 100%; height:90vh"
    lazyLoad="1"
    loading="lazy"
    src="https://getbootstrap.com/docs/4.5/layout/grid/"
></iframe>

---

### 思维方式的本质差异

---

#### 美工好比建筑设计师

要的就是**惊艳**

---

#### 码农好比土木工程师

要的就是**高效**、**可靠**

---

复用市场**现有规格的构件**

在**施工安全**的前提下

尽力贴近**设计效果图**

---

##### 从零开始开发项目定制组件

**开发工程量**非常大

---

##### 魔改现成组件

- 自适应、响应式样式
- 用户交互逻辑

小心翼翼不破坏它们，但往往事与愿违……

---

### 行百里者半九十

---

开发工时 ≤ “抠细节”工时

---

开发往往集中进行，并能复用现有**组件库**

---

抠 UI 细节往往……

---

- 再深/浅点……
- 再大/小点……
- 再往这/那来点……

---

请先定义“一点”……

---

## 设计规范，了解一下？

---

### Twitter BootStrap 组件库

---

#### 字号等级

<iframe
    style="width: 100%; height:90vh"
    lazyLoad="1"
    loading="lazy"
    src="https://getbootstrap.com/docs/4.5/content/typography/#headings"
></iframe>

---

#### 间距等级

<iframe
    style="width: 100%; height:90vh"
    lazyLoad="1"
    loading="lazy"
    src="https://getbootstrap.com/docs/4.5/utilities/spacing/#notation"
></iframe>

---

#### 配色主题

<iframe
    style="width: 100%; height:90vh"
    lazyLoad="1"
    loading="lazy"
    src="https://getbootstrap.com/docs/4.5/utilities/colors/#color"
></iframe>

---

#### 最大内容宽度

<iframe
    style="width: 100%; height:90vh"
    lazyLoad="1"
    loading="lazy"
    src="https://getbootstrap.com/docs/4.5/layout/overview/#containers"
></iframe>

---

#### 一致的样式、交互

<iframe
    style="width: 100%; height:90vh"
    lazyLoad="1"
    loading="lazy"
    src="https://bootstrap.web-cell.dev/"
></iframe>

---

### Google Material Design

<iframe
    style="width: 100%; height:90vh"
    lazyLoad="1"
    loading="lazy"
    src="https://material.io/"
></iframe>

---

### Microsoft Fluent Design

<iframe
    style="width: 100%; height:90vh"
    lazyLoad="1"
    loading="lazy"
    src="https://www.microsoft.com/design/fluent/#/"
></iframe>

---

### 蚂蚁金服 Ant Design

<iframe
    style="width: 100%; height:90vh"
    lazyLoad="1"
    loading="lazy"
    src="https://ant.design/"
></iframe>

---

### 饿了么 Element UI

<iframe
    style="width: 100%; height:90vh"
    lazyLoad="1"
    loading="lazy"
    src="https://element.eleme.cn/"
></iframe>

---

## 学习收获

---

> 你按设计图标注来就行~
>
> —— UI 设计师

---

行个 🔨！

---

## 讲师带货

<iframe
    style="width: 100%; height:90vh"
    lazyLoad="1"
    loading="lazy"
    src="https://web-cell.dev/"
></iframe>

---

<iframe
    style="width: 100%; height:90vh"
    lazyLoad="1"
    loading="lazy"
    src="https://fcc-cd.dev/"
></iframe>
