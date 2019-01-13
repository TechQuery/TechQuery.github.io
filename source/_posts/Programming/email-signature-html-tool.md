---
title: 电子邮件签名档 HTML 手写时的折腾（附 原创工具）
date: 2014-12-01 15:47
author: 南漂一卒
categories:
  - Programming
tags:
  - HTML
  - Email
  - toolkit
---


【原文链接】https://my.oschina.net/TechQuery/blog/350954


## 入职必修课

鄙人作为码农进厂开电脑的第一件正事就是 —— 在 Foxmail 客户端中设置自己**公司的域名邮箱**，并加上看起来还挺**专业的签名档**。

接收欢迎新人的邮件，选中老员工的签名档并复制粘贴到写邮件设置中，修改个人信息并保存 —— 这似乎很轻松，但…… 职位一处的文字稍长一些就显示不全…… 旁边那么多空白竟然不能**自适应**？！作为一个**前端程序猿**，简直不能忍 —— 改代码！

虽然 Foxmail 客户端的**富文本编辑器**几乎和 QQ邮箱一样（原本也是同一个团队的作品嘛），但竟然不支持 **HTML 格式化**…… 还好在线代码格式化工具很多。但格式化之后，代码依然很烂 —— 各种样式还用 HTML 4 的标签属性，style 属性也是一坨一坨地堆着，姓名、职位、手机、电邮等竟然还用 `<input type="text" />`……

Foxmail 自己加上去的全局样式不是用 `<style />` 写在最前面吗？这么乱的代码，哥怎么有心情改？！写个简称得了，管你们看不看得懂~


## 收拾“旧河山”

俩月后，公司新产品上线，要在签名档已有的俩二维码中间再加一个 —— 公司网管兄用 DreamWeaver 死活改不好，于是来找我帮忙…… 那就不客气了 —— 直接用自己的 CSS 框架的布局模块来重写，那叫一个畅爽~

写完之后在浏览器里测试完美，便即刻粘贴到 Foxmail 中 —— 样式没了？！

再尝试用 Foxmail **邮件模板** —— 样式又没了？！

只好用浏览器调试器一个一个元素地把**外联样式**填回 style 属性中……

再试 Outlook —— 还是没样式！连 **DataURL 形式的图片**都叉了？！


## 前端猿的 Hack

最后只好登录公司**邮件服务的 Web 版**，在账号选项中设置签名档 —— 连 **HTML 源码编辑模式**都没了……

那就只能把 新签名档源码在浏览器中打开，再全选、复制、粘贴到邮件系统编辑框 —— 测试邮件基本正常，除了**逼死强迫症的水平滚动条**……（马丹，这不在同事面前显得我很没水平？！）

![Web Mail 签名档设置][1]

仔细查看测试邮件的源代码之后发现 —— **所有 style 的长度数值都从百分比变成了像素绝对值**……

我想应该是 浏览器等客户端程序在复制 HTML 等富文本时把相对值转换成了绝对值，可以用下面的原创程序初步验证 ——

```javascript
(function (BOM) {
    var isIE = !(! BOM.document.attachEvent);

    BOM.CB_getData = function (cType, CallBack) {
        var This = this,
            EM = isIE ?
                    ['attachEvent', 'onpaste', 'detachEvent'] :
                    ['addEventListener', 'paste', 'removeEventListener'];

        if (cType && (! isIE))  switch (cType.toLowerCase()) {
            case 'text':    cType = 'text/plain';    break;
            case 'url':     cType = 'text/unicode';  break;
            case 'html':    cType = 'text/html';     break;
        }
        This.document[ EM[cType ? 0 : 2] ](EM[1], function () {
            var PO = isIE ? This : arguments[0];
            var CBR = CallBack.call(
                isIE ? PO.srcElement : PO.target,
                PO.clipboardData.getData(cType)
            );
            if (! CBR)
                if (isIE)  PO.event.returnValue = false;
                else PO.preventDefault();
        }, isIE ? undefined : false);
    };
})(self);

//  粘贴事件发生时，回调函数的首个参数 将赋值为 剪贴板中的内容
CB_getData('HTML', function (Content) {
    console.log(Content);
    return false;
});
//  返回 false 阻止浏览器 事件默认行为，即不在用户粘贴的地方输出 剪贴板内容（同 jQuery）
```
网管小哥只好把浏览器窗口拉窄一些再复制粘贴 —— 至少看起来没啥问题了……（但我知道在一些领导的轻薄笔记本、平板电脑等更窄的屏幕上还会溢出啊）

最后的最后 —— 在 **Web 邮箱的签名档设置框**中右击打开调试器，直接编辑它的 **HTML 文档片段**，把签名档源码粘贴进去再返回来保存 —— 发出来的测试邮件源码完整保留了**长度值的百分比原值**~

![Web Mail 签名档编辑框内容代码注入][2]


## 后话

按理说，**E-mail** 这种兼容 HTML 格式的**经典互联网标准** 应该至少支持 HTML 4 + CSS 2 吧？哥手写的布局系统基于 display: table(-cell) 就不行？

Outlook、Foxmail 等 **Windows 邮件客户端** 渲染 HTML 应该直接调用本机的 **IE 内核**吧？哥用的可是 **IE 11** 啊~

咋就这么多不兼容的坑坑呢……？

----------

其实很多**网站系统的富文本编辑框** 都没有 **HTML 代码编辑模式**，直接复制粘贴**富文本片段**经常会格式出错（比如 **微信公众平台**编辑模式的图文消息发布模块）……

在后台程序过滤之外，**前端工程师** 能一劳永逸地解决此类问题的方法只有 —— 用 **JavaScript** 做 **HTML 代码片段注入**……

所以我写了个 **Bookmarklet**（网页浏览器 **书签栏按钮小工具**）发布在 [**Git@OSC**][3] 上，欢迎大家折腾、**提 issue**~


## 参考文章

 1. http://www.topcss.org/?tag=document-activeelement
 2. http://www.cnblogs.com/hughtxp/p/3939976.html
 3. http://blog.csdn.net/lee_magnum/article/details/17761441
 4. http://www.cnblogs.com/xzhang/p/3968204.html


  [1]: http://static.oschina.net/uploads/space/2014/1201/154333_73Tz_1171658.png
  [2]: http://static.oschina.net/uploads/space/2014/1201/154419_B5gz_1171658.png
  [3]: http://gitee.com/Tech_Query/iBookmarkLet
