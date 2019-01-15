---
title: 【浏览器小工具】网页浮层广告/锁屏一键清除（完善重发）
date: 2014-09-25 20:40:00
author: 南漂一卒
categories:
  - Programming
tags:
  - Web
  - advertisement
  - browser
  - bookmark
  - toolkit
---


　　我之前写的【[网页浮动广告一键清除工具][1]】虽然不断完善至 v0.3，但对各种实现方式的**浮层广告** 适配得不全面，自己在使用过程中发现有些网站的浮层广告依然无法彻底清除（*某个很亮堂的党报的官网* 新闻阅读页的右下角广告 竟然比某些盗版资源站的还顽固）……
　　进一步调试、分析后发现，这些“地痞无赖”的广告有个特点 —— 在和 `<body />` 一样大的多个 `<div />` 层层包裹下（有的多达 13 层），浮层广告用任意的 CSS 设为 `display: block` 的 HTML 标签包裹（如 `<i />`），这个标签再用 `position: absolute` 定位。（此法也就是“IE 6 不支持 `position: fixed`”的兼容方法）
　　因为我和女友平时都经常使用它（我主要是大量地查技术资料、下经典电影，她主要是看娱乐八卦，反正都是广告满天飞的网站……）于是，继续更新之 ——


## 最新稳定版 全部特性

 1. 可清除 **网页中任一位置的浮动广告**
 2. 可清除 **遮蔽整个网页的登录框**
 3. 可清除 **笼罩整个网页的透明弹窗广告超链接**
 4. 可清除 **网页正文图片边缘的浮动广告**（一般在第二次点击本工具时成功，但此时会误杀一些网页本身的功能浮层，可能会影响某些功能的使用，**建议在纯阅读时二次点击本工具**）


## 开发用源代码

```javascript
//
//    1Key Web Float Layer Cleaner   v0.5
//
//                2014-10-10
//
//        (C)2014  test_32@fyscu.com
//

(function (WO, DO) {
    function $_TN(REO, TagName) {
        return  [].slice.apply( REO.getElementsByTagName(TagName) );
    }
    function get_Style(_TE, _SN) {
        var _Style = _TE.currentStyle ?
            _TE.currentStyle.getAttribute(_SN) :
            DO.defaultView.getComputedStyle(_TE, null).getPropertyValue(_SN);
        var _Number = Number(
            (_Style == '0px') ? 0 : _Style
        );
        return  isNaN(_Number) ? _Style : _Number;
    }
    function CSS_Match(EO, RG) {
        for (var i = 0; i < RG.length; i++)
            if (get_Style(EO, RG[i][0]) == RG[i][1])
                return 1;
    }
    function ParentMatch(EO, Parent, Level) {
        for (var i = 0; i < Level; i++)
            if (EO.parentNode === Parent)
                return 1;
            else if (EO.parentNode === DO)
                break;
            else  EO = EO.parentNode;
    }
    function isFixedLayer(_TE) {
        var inVisible = CSS_Match(_TE, [
            ['display',     'none'],
            ['visibility',  'hidden'],
            ['width',       0]
        ]);
        var Fixed = (! inVisible) && CSS_Match(_TE, [
            ['position',  'absolute'],
            ['position',  'fixed']
        ]);
        if (Fixed && (
            WO.FLC ? true : (get_Style(_TE, 'z-index') > 100)
        ))
            return ParentMatch(
                //  清除效果 基本只取决于“DOM 遍历深度”
                _TE,  DO.body,  (WO.FLC ? 13 : 3)
            );
    }
    var FL_Box = $_TN( $_TN(DO, 'body')[0], '*');
    for (var iKey = FL_Count = 0, _FL; iKey < FL_Box.length; iKey++) {
        _FL = FL_Box[iKey];
        if ( isFixedLayer(_FL) ) {
            _FL.parentNode.removeChild(_FL);
            console.log(_FL);
            FL_Count++;
        }
    }
    if (! self.FLC)  self.FLC = true;
    alert([
        FL_Count,  ' of the fucking Float ADs/Layers have been cleaned !~',
        '\n',  'You can try Clicking my Button again to be cleaner.',
        '\n\n',  '(C)2014   SCU FYclub-RDD'
    ].join(''));
})(self, self.document);
```

## 安装用代码

　　用 **UglifyJS**（[在线版][2]）生成，压缩率 高于 50% ——

```javascript
javascript: (function(a,b){function c(a,b){return[].slice.apply(a.getElementsByTagName(b))}function d(a,c){var d=a.currentStyle?a.currentStyle.getAttribute(c):b.defaultView.getComputedStyle(a,null).getPropertyValue(c),e=Number("0px"==d?0:d);return isNaN(e)?d:e}function e(a,b){for(var c=0;c<b.length;c++)if(d(a,b[c][0])==b[c][1])return 1}function f(a,c,d){for(var e=0;d>e;e++){if(a.parentNode===c)return 1;if(a.parentNode===b)break;a=a.parentNode}}function g(c){var g=e(c,[["display","none"],["visibility","hidden"],["width",0]]),h=!g&&e(c,[["position","absolute"],["position","fixed"]]);return h&&(a.FLC?!0:d(c,"z-index")>100)?f(c,b.body,a.FLC?13:3):void 0}var j,i,h=c(c(b,"body")[0],"*");for(i=FL_Count=0;i<h.length;i++)j=h[i],g(j)&&(j.parentNode.removeChild(j),console.log(j),FL_Count++);self.FLC||(self.FLC=!0),alert([FL_Count," of the fucking Float ADs/Layers have been cleaned !~","\n","You can try Clicking my Button again to be cleaner.","\n\n","(C)2014   SCU FYclub-RDD"].join(""))})(self,self.document);
```

  [1]: http://bbs.fyscu.com/forum.php?mod=viewthread&tid=4929
  [2]: http://www.css-js.cn/
