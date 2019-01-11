---
title: 再战“所见即所得”富文本编辑器（附 原创工具）
date: 2016-03-16 17:52
author: 南漂一卒
categories:
  - Technology
  - Programming
tags:
  - rich-text
  - editor
  - injection
  - hacker
---


## 未尽的事业

2014年底，我为了搞定 **电邮签名档** 做了次[专门的研发](http://my.oschina.net/TechQuery/blog/350954)，非常有收获！但它依然有个[坑](http://my.oschina.net/TechQuery/blog/350954#OSC_h1_2) ——

> **外置 CSS 样式** 要人工一一填到 HTML 标签的 style 属性中，图片也要自己转换成 **Base64 编码**……

2016年初，我又遇到需要手写 **富文本电邮**的一件事，手工转换一个大网页 —— 累个半死…… 同时，女友也在为“**微信公众平台 图文消息编辑器** 不能在文章正文加超链接”发愁，想用我当年开发的 [**HTML 代码注入工具**](http://gitee.com/Tech_Query/iBookmarkLet#-富文本编辑框-自定义-html-代码片段-插入工具-v0-4) 除了学会写 静态网页代码，还是要我手工转换为 **单一网页格式**（类似 Internet Explorer `.mht`）……


## 编程马拉松

基于不久前对 [**CSS 对象**的深入研究](http://gitee.com/Tech_Query/iQuery/commit/5f2e05676e16b33a81f0639c760738ec9763d487)，我用之前两个夜晚的饭后休息时间完成了一个 [**网页代码 行内化工具**](http://gitee.com/Tech_Query/iBookmarkLet#-网页代码-行内化-v0-2)的开发、测试 ——

```javascript
javascript:  (function (BOM, DOM) {

    /* ---------- 生效样式 ---------- */

    function Tag_Computed_CSS(iDOM) {
        return JSON.parse(JSON.stringify(
            iDOM.ownerDocument.defaultView.getComputedStyle(iDOM)
        ));
    }

    /* ---------- 原始样式 ---------- */

    var Tag_Style = { },  iSandBox = DOM.createElement('iframe');

    iSandBox.style.display = 'none';
    DOM.body.appendChild( iSandBox );

    function Tag_Default_CSS(iTagName) {
        var _DOM_ = iSandBox.contentWindow.document;

        if (! Tag_Style[iTagName]) {
            var iDefault = _DOM_.createElement( iTagName );
            _DOM_.body.appendChild( iDefault );
            Tag_Style[iTagName] = Tag_Computed_CSS(iDefault);
            _DOM_.body.removeChild( iDefault );
        }
        return Tag_Style[iTagName];
    }

    /* ---------- 变更样式 ---------- */

    function Tag_Customed_CSS() {
        var iCustomed = { },
            iComputed = Tag_Computed_CSS( arguments[0] ),
            iDefault = Tag_Default_CSS( arguments[0].tagName.toLowerCase() );

        for (var iAttr in iComputed)
            if (
                isNaN(Number( iAttr ))  &&
                (! iAttr.match(/^(moz|webkit|ms)/))  &&
                (iComputed[iAttr] != iDefault[iAttr])  &&
                (! iAttr.match(/width|height/i))
            )
                iCustomed[iAttr] = iComputed[iAttr];

        return iCustomed;
    }

    /* ---------- 内联化核心 ---------- */

    BOM.CSS_Inline = function () {
        var iStyle = Tag_Customed_CSS( arguments[0] );

        for (var iAttr in iStyle)
            arguments[0].style[iAttr] = iStyle[iAttr];
    };

    BOM.Image_Inline = function (iDOM, iWaterMark) {
        var _Image_ = new Image(),  iCanvas = DOM.createElement('canvas');

        _Image_.crossOrigin = '';
        var iContext = iCanvas.getContext('2d');

        _Image_.onload = function () {
            iCanvas.width = _Image_.width;
            iCanvas.height = _Image_.height;
            iContext.drawImage(_Image_, 0, 0);

            if (iWaterMark) {
                iContext.font = '20px sans-serif';
                iContext.fillStyle = 'white';
                iContext.fillText(iWaterMark,  10,  _Image_.height - 15);
            }
            iDOM.src = iCanvas.toDataURL('image/png');
            _Image_ = null;
        };
        _Image_.src = iDOM.src;
    };

    BOM.Web_Inline = function () {
        var iTag = arguments[0].querySelectorAll('*');

        for (var i = 0;  i < iTag.length;  i++)
            switch ( iTag[i].tagName.toLowerCase() ) {
                case 'meta':      ;
                case 'style':     ;
                case 'script':    ;
                case 'iframe':
                    iTag[i].parentNode.removeChild( iTag[i] );    break;
                case 'img':
                    this.Image_Inline(iTag[i], arguments[1]);     break;
                default:
                    this.CSS_Inline( iTag[i] );
            }
        return arguments[0].innerHTML.trim();
    };

    /* ---------- 使用流程 ---------- */

    BOM.Web_Inline(DOM.body, BOM.prompt("图片水印文字："));

    BOM.setTimeout(function () {
        DOM.body.textContent = DOM.body.innerHTML;
        BOM.alert("请全选、复制当前显示的所以代码~");
    }, 1000);

})(self, self.document);
```
