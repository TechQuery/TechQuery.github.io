---
title: 【网页重构】CSS 规则清理（浏览器 BookMarkLet、FireBug 插件）
date: 2014-09-30 18:30:00
author: 南漂一卒
categories:
  - Programming
tags:
  - CSS
  - Web
  - browser
  - bookmark
  - toolkit
---


很多程序猿认为 —— **最蛋疼的工作不在于重写或创新，而是“修改前任程序猿留下的代码”……**

对**前端工程师**来说，若不进行彻底的**网页重构**，而只是修补现有网页的代码（多是之前由 **后端工程师** 乃至 **只会点儿 DreamWeaver 的运维网管** 写的），再加上浏览器兼容性的工作特性（特别是 **IE 8-**），简直是“英年早逝”的节奏……

你能想象一个 HTML 文件中全是大量*乱七八糟、东拼西凑*的 标签结构 和 CSS 规则 吗？不仅如此，`<link rel="stylesheet" />`、`<style />`、`<script />` 也是随时随地出现；很多 HTML 元素都由不止一个 CSS 文件中的多个毫无逻辑联系的规则 叠加影响；文件命名、目录结构也超乱…… **“大海捞针”、“牵一发而动全身”**便是再贴切不过的形容了……

若能用工具清理一遍 CSS，那我们的工作就搞定一半了～


## 原创小工具

这里先分享一下我原创的一个 **BookMarkLet** 形式的 [**网页 CSS 规则使用率检测工具**][1] ——

```javascript
(function (BOM, DOM, $) {

    var CSS_File = DOM.styleSheets,
        CopyRight = '(C)2014-2015  test_32@fyscu.com';

    if (! $) {
        BOM.alert(
            'Please run This Tool in IE 8+ (Standard Mode) or a Modern Web Browser.' + "\n\n" +
            CopyRight
        );
        return false;
    } else if (! console) {
        BOM.alert(
            'Please run This Tool with JavaScript Console opened.' + "\n\n" +
            CopyRight
        );
        return false;
    }

    function URL_Path(URL) {
        if ( URL.match(/^\w+:\/\//) ) {
            URL = URL.split('/').slice(3);
            URL.unshift('.');
            URL = URL.join('/');
        }
        return URL;
    }

    function SelectorCheck(Selector) {
        Selector = Selector.split(',');
        var _NU_ = [ ];

        for (var k = 0;  k < Selector.length;  k++)
            if (! $( Selector[k].trim().split(':')[0] ).length)
                _NU_.push(Selector[k]);

        return _NU_;
    }

    function CSS_Rule_Each(CSS_Rule_Group) {
        try {
            var  CSS_Rule = CSS_Rule_Group.cssRules || CSS_Rule_Group.rules;
        } catch (Err) { }

        var Selector = '',
            NoUsage = [ ],
            Test_Result = {
                media:         CSS_Rule_Group.media.mediaText,
                mediaRules:    [ ],
                fontsRules:    [ ]
            };
        if (CSS_Rule) {
            for (var j = 0, _Rule_;  j < CSS_Rule.length;  j++) {
                _Rule_ = CSS_Rule[j];
                switch (_Rule_.type) {
                    case 1:    {    //  CSSStyleRule
                        _Rule_ = SelectorCheck(_Rule_.selectorText);
                        if (_Rule_.length)  NoUsage.push(_Rule_);
                    }  break;
                    case 4:        //  CSSMediaRule
                        Test_Result.mediaRules.push( arguments.callee(_Rule_)[0] );  break;
                    case 5:        //  CSSFontFaceRule
                        Test_Result.fontsRules.push({
                            fontFamily:    _Rule_.style.fontFamily,
                            src:           _Rule_.style.src
                        });
                }
            }
            Test_Result.WasteRate = ((NoUsage.length / j) * 100).toFixed(2) + '%';
        }
        return [Test_Result, NoUsage];
    }

    for (var i = 0, _File_, _Result_ = [ ];  i < CSS_File.length;  i++) {
        _Result_[i] = CSS_Rule_Each(CSS_File[i]);
        var _File_ = _Result_[i].shift();

        _File_.href = CSS_File[i].href;
        if (_File_.WasteRate && _File_.href)
            _File_.href = URL_Path(_File_.href);
        _File_.element = CSS_File[i].ownerNode;
        console.log(_File_);
    }
    console.log(_Result_);

})(self,  self.document,  self.document.querySelectorAll);
```

### 安装版代码

```javascript
javascript: (function(a,b,c){function f(a){return a.match(/^\w+:\/\//)&&(a=a.split("/").slice(3),a.unshift("."),a=a.join("/")),a}function g(a){var b,d;for(a=a.split(","),b=[],d=0;d<a.length;d++)c(a[d].trim().split(":")[0]).length||b.push(a[d]);return b}function h(a){var b,e,f,i,h;try{b=a.cssRules||a.rules}catch(c){}if(e=[],f={media:a.media.mediaText,mediaRules:[],fontsRules:[]},b){for(h=0;h<b.length;h++)switch(i=b[h],i.type){case 1:i=g(i.selectorText),i.length&&e.push(i);break;case 4:f.mediaRules.push(arguments.callee(i)[0]);break;case 5:f.fontsRules.push({fontFamily:i.style.fontFamily,src:i.style.src})}f.WasteRate=(100*(e.length/h)).toFixed(2)+"%"}return[f,e]}var j,i,k,d=b.styleSheets,e="(C)2014-2015  test_32@fyscu.com";if(!c)return a.alert("Please run This Tool in IE 8+ (Standard Mode) or a Modern Web Browser.\n\n"+e),!1;if(!console)return a.alert("Please run This Tool with JavaScript Console opened.\n\n"+e),!1;for(i=0,k=[];i<d.length;i++)k[i]=h(d[i]),j=k[i].shift(),j.href=d[i].href,j.WasteRate&&j.href&&(j.href=f(j.href)),j.element=d[i].ownerNode,console.log(j);console.log(k)})(self,self.document,self.document.querySelectorAll);
```

### 工具特性

 1. **扫描当前网页中所有内置/外置的 CSS，在控制台显示各 CSS 标签/文件中哪些规则对本页无用、共占比多少**
 2. 外置 CSS 会显示**文件相对路径**
 3. 内置 CSS 会显示其对应的 HTML 元素，便于**在调试器显示的 DOM 树上定位**
 4. **CSS 媒体查询、字体**的规则会在显示的对象中有独立分支
 5. 暂不支持 CSS 选择符中间有 **伪类、伪元素**的情况，只支持它们出现在 选择符链的最后端（这是与下文提及的专业工具之**有效比计算**有误差的主要原因）
 6. **跨域的 CSS 文件**无法扫描（它们一般都是 **CDN 通用库** 或 部署环境中的**静态文件服务器**托管的压缩版，本来就不是用于调试的）


## FireBug 插件 —— CSS Usage

现在比较成熟的 **CSS 有效性检查工具** 当属 `CSS Usage` 了，它不仅能用彩色的富文本显示每个 CSS 标签/文件中**每条规则在当前网页中的使用率**，还能**生成过滤后的 CSS 文本**，可以直接复制粘贴到代码编辑器中保存、使用～

至于它的安装，和 FireBug 一样，在 **FireFox** 的**附加组件**管理界面中搜索、下载、安装、重启，在 FireBug 主界面上端的选项卡最后就能看到它了～


  [1]: http://gitee.com/Tech_Query/iBookmarkLet
