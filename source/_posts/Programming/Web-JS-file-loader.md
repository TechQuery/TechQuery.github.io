---
title: 【原创】JavaScript 文件响应式异步加载器（单页稳定版）
date: 2014-10-09 06:49:00
author: 南漂一卒
categories:
  - Programming
tags:
  - Web
  - JavaScript
  - module
  - loader
---

我在做[《i 飞扬》电子杂志 HTML5 在线版][1]的过程中，为了在不改变 **Web 前端程序猿的编程思维习惯**的前提下，保证整个 **WebApp 的好用、可靠**，自己开发了一个**【JavaScript 文件响应式异步加载器】**—— [EasyImport.js][2]。

开始只是一段放在 HTML `<head />` 中的小脚本，没觉得是个多么复杂的东西。但随着应用的深入，要想做到上述自己定下的目标，**代码不断地迭代**，期间也因为严重的 Bug 而怀疑它的价值，所以有了后来的一次较大的局部重构。

但辛劳总会有收获 —— 个人更深刻地理解了 **JavaScript**、**DOM** 的一些细节，它本身也到了足够成熟的地步，作为几个**线上实用项目的基础库**，运行在很多人的浏览器中~

## v0.6（单页稳定版）

这是 **EasyImport.js 首个功能完备的稳定版**，能给 **单页应用** 提供有力的**异步加载、响应式适配**支持，但不支持 **`<iframe />` 子页面的 JS、CSS 继承**（也就是说，它独立运行于每一个加载它的父/子页面中，不考虑关联页面的相互联系）。它将提供**长期 Bug 修复的官方维护**，后续版本与本版的公共代码也将同步更新，但不会再改变代码的整体结构。而子页面继承将在下个稳定版给予集成~

```javascript
//
//                >>>  EasyImport.js  <<<
//
//
//      [Version]    v0.6  (2014-11-17)  Stable
//
//      [Usage]      Only for loading JavaScript files in Single-Page Web,
//                   no Inherit support for Frames.
//
//
//              (C)2013-2014    SCU FYclub-RDD
//

(function(BOM, DOM) {
  var UA = navigator.userAgent,
    RootPath,
    JS_List,
    CallBack,
    EIJS = {
      AsyncJS: [],
      DOM_Ready: []
    };
  // ----------- Inner Basic Member ----------- //
  var is_IE = UA.match(/MSIE (\d)|Trident[^\)]+rv:(\d+)/i),
    is_Webkit = UA.match(/Webkit/i);
  var IE_Ver = is_IE ? Number(is_IE[1] || is_IE[2]) : NaN;
  var old_IE = IE_Ver && IE_Ver < 9,
    is_Pad = UA.match(/Tablet|Pad|Book|Android 3/i),
    is_Phone = UA.match(/Phone|Touch|Android 2|Symbian/i);
  var is_Mobile =
    (is_Pad || is_Phone || UA.match(/Mobile/i)) && !UA.match(/ PC /);

  function DOM_Loaded_Event(DOM_E, CB_Func) {
    if (!old_IE) DOM_E.addEventListener('load', CB_Func, false);
    else
      DOM_E.attachEvent('onreadystatechange', function() {
        if (DOM_E.readyState.match(/loaded|complete/)) CB_Func.call(DOM_E);
      });
  }
  function DOM_Ready_Event(_CB) {
    if (!old_IE) DOM.addEventListener('DOMContentLoaded', _CB, false);
    else if (self !== top) DOM_Loaded_Event(DOM, _CB);
    else
      (function() {
        try {
          DOM.documentElement.doScroll('left');
          _CB.call(DOM);
        } catch (Err) {
          setTimeout(arguments.callee, 1);
        }
      })();
  }
  function $_TN(HTML_Elements, TagName) {
    return HTML_Elements.getElementsByTagName(TagName);
  }
  function NewTag(TagName, AttrList) {
    var NE = DOM.createElement(TagName);
    if (AttrList)
      for (var AK in AttrList) {
        if (IE_Ver && AK == 'class') NE.className = AttrList[AK];
        NE.setAttribute(AK, AttrList[AK]);
      }
    return NE;
  }

  var DOM_Head = $_TN(DOM, 'head')[0];
  var DOM_Title = $_TN(DOM_Head, 'title')[0];
  // ----------- Standard Mode Meta Patches ----------- //
  DOM_Head.appendChild(
    NewTag('meta', {
      'http-equiv': 'Window-Target',
      content: '_top'
    })
  );
  if (is_Mobile) {
    if (!old_IE) {
      var is_WeChat = UA.match(/MicroMessenger/i),
        is_UCWeb = UA.match(/UCBrowser|UCWeb/i);
      DOM_Head.insertBefore(
        NewTag('meta', {
          name: 'viewport',
          content: [
            [
              'width',
              is_Phone && (is_WeChat || is_UCWeb) ? 320 : 'device-width'
            ].join('='),
            'initial-scale=1.0',
            'minimum-scale=1.0',
            'target-densitydpi=medium-dpi'
          ].join(',')
        }),
        DOM_Title
      );
    } else
      DOM_Head.insertBefore(
        NewTag('meta', {
          name: 'MobileOptimized',
          content: 320
        }),
        DOM_Title
      );
    DOM_Head.insertBefore(
      NewTag('meta', {
        name: 'HandheldFriendly',
        content: 'true'
      }),
      DOM_Title
    );
  }
  if (IE_Ver)
    DOM_Head.appendChild(
      NewTag('meta', {
        'http-equiv': 'X-UA-Compatible',
        content: 'IE=Edge, Chrome=1'
      })
    );

  // ----------- Inner Logic Module ----------- //
  function xImport(JS_URL, CB_Func) {
    var SE = NewTag('script', {
      type: 'text/javascript',
      charset: 'UTF-8',
      src: JS_URL
    });
    if (CB_Func) DOM_Loaded_Event(SE, CB_Func);
    return $_TN(DOM, 'head')[0].appendChild(SE);
  }
  function LI_Add(LQ, RP, FN) {
    if (!FN.match(/^http(s)?:\/\//)) FN = RP + FN;
    LQ.push({ URL: FN });
  }
  function SL_Set(RP, List0) {
    for (var i = 0; i < List0.length; i++)
      if (!(List0[i] instanceof Array)) List0[i] = [List0[i]];
    var List1 = [];
    for (var i = 0, k = 0; i < List0.length; i++) {
      List1[k] = [];
      for (var j = 0, _URL; j < List0[i].length; j++) {
        if (typeof List0[i][j] == 'string') LI_Add(List1[k], RP, List0[i][j]);
        else {
          var _Rule = {
              old_PC: old_IE,
              Mobile: is_Mobile,
              Phone: is_Phone,
              Pad: is_Pad
            },
            no_Break = true;
          for (RI in _Rule)
            if (_Rule[RI]) {
              if (List0[i][j][RI] === false) no_Break = false;
              else if (List0[i][j][RI]) LI_Add(List1[k], RP, List0[i][j][RI]);
              break;
            }
        }
        if (no_Break && !List1[k][j] && List0[i][j].new_PC)
          LI_Add(List1[k], RP, List0[i][j].new_PC);
      }
      if (List1[k].length) k++;
    }
    return List1;
  }
  function CB_Set(List, Index) {
    var Item = List[Index + 1];
    if (Item[0].CallBack) {
      var AJSQ = EIJS.AsyncJS;
      var AJS_NO = AJSQ.push(0) - 1;
      var AJS_CB = function() {
        if (++AJSQ[AJS_NO] == Item.length) Item[0].CallBack();
      };
    }
    return function TF_Import() {
      for (var n = 0; n < Item.length; n++) xImport(Item[n].URL, AJS_CB);
    };
  }
  function CB_Chain(JS_RC, Final_CB) {
    var DRQ = EIJS.DOM_Ready;
    var DRQ_NO = DRQ.push(0) - 1;
    for (var k = JS_RC.length - 2, l = 0; k > -2; k--, l++) {
      if (!l) {
        if (Final_CB) {
          JS_RC[k + 1][0]['CallBack'] = function(iEvent) {
            if (++DRQ[DRQ_NO] == 2) Final_CB.apply(self);
          };
          DOM_Ready_Event(JS_RC[k + 1][0].CallBack);
        } else if (k > -1) {
          var Last_Script = JS_RC[k + 1][0].URL;
          JS_RC[k][0]['CallBack'] = function(iEvent) {
            if (++DRQ[DRQ_NO] == 2) xImport(Last_Script);
          };
          DOM_Ready_Event(JS_RC[k][0].CallBack);
          continue;
        }
      }
      if (k > -1) JS_RC[k][0]['CallBack'] = CB_Set(JS_RC, k);
    }
  }

  // ----------- Open API ----------- //
  BOM.ImportJS = function() {
    var Func_Args = Array.prototype.slice.call(arguments, 0);

    if (typeof Func_Args[0] == 'string') RootPath = Func_Args.shift();
    else
      RootPath = $_TN(DOM, 'script')[0].src.replace(
        /[^\/]*EasyImport[^\/]*\.js[^\/]*$/i,
        ''
      );
    if (Func_Args[0] instanceof Array) JS_List = Func_Args.shift();
    else throw "Format of Importing List isn't currect !";
    if (typeof Func_Args[0] == 'function') CallBack = Func_Args.shift();
    else CallBack = null;

    var JS_Item = SL_Set(RootPath, JS_List);
    if (!JS_Item[0].length) return;
    CB_Chain(JS_Item, CallBack);
    xImport(JS_Item[0][0].URL, JS_Item[0][0].CallBack);
  };

  BOM.ImportJS.UA = {
    IE: !!old_IE,
    Modern: !old_IE,
    Mobile: !!is_Mobile,
    Pad: !!is_Pad,
    Phone: !!is_Phone
  };
})(window, document);
```

## 【平台支持】

- 主流内核浏览器：IE 7~11、Firefox 3+、Chrome、Safari、Opera
- 国产马甲浏览器：傲游 2+、枫树、搜狗高速 3+、360 安全/极速、猎豹、QQ 等（还可能有个别奇葩 Bug，恕无力一一测试……）
- Android、iOS、Windows Phone、Symbian 等智能设备平台上的自带浏览器及其内核控件

## 【主要特性】

1.  替代 `<script />` 加载 HTML 页面中**所有外置、内置 JavaScript 脚本**，但不改变原有“**顺序即依赖**”的前端编程习惯，且书写形式比原来更简洁
2.  所有外置脚本都可写在 `<head />` 中，但却能与整个页面的其它部分**并行加载**，**代码结构清晰**而又**高性能**
3.  最后一段脚本（无论内置回调、外置脚本）自动在 **DOM Ready** 后加载/执行，既靠谱又方便（`$(document).ready(function () {});` 之类的大包装可以完全不需要了）
4.  无依赖的部分外置脚本只需用 `[]`（数组字面量）括起来就可以**异步加载**
5.  外置脚本可以根据浏览器平台类型**选择性加载**，模块化管理 JavaScript
6.  **移动设备浏览器布局模式自适配**：添加其适用的各种 `<meta />` 标签，让网页源码以标准而通用的简洁思维编写，自然地实施**响应式设计**
7.  浏览器级的 `<iframe />` 套用防御

## 【典型案例】

http://mag.fyscu.com/iWB/iBookView.php?name=iFY&index=19

以下**示例代码**摘抄自上述网页的 HTML 源码 ——

```html
<head>
  ......
  <script type="text/javascript" src="./Libs/EasyImport.js"></script>
  <script>
    ImportJS(
      [
        {
          old_PC: 'jQuery.js',
          new_PC: 'jQuery2.js'
        },
        [
          {
            old_PC: 'Turn.HTML4.js',
            new_PC: 'Turn.js'
          },
          'Smooth_Scroll.js',
          'jPlayer.js',
          'jQuery.Hammer.js',
          {
            old_PC: false,
            new_PC: 'jQuery.PageZoomer.js',
            Mobile: false
          },
          {
            new_PC: 'Hover_Scroll.js',
            Mobile: false
          }
        ],
        'FY_iWeBook.js'
      ],
      function() {
        $('#iWB').iWeBook('#jPlayer_Box');
      }
    );

    var duoshuoQuery = { short_name: 'fyscu' };
    ImportJS(['http://static.duoshuo.com/embed.js']);
  </script>
  ......
</head>
```

## 【参考文章】

1.  http://sunnylost.com/article/js.load.scripts.sequence.html
2.  http://bbs.fyscu.com/forum.php?mod=viewthread&tid=4808
3.  http://www.cnblogs.com/pigtail/archive/2013/03/15/2961631.html
4.  http://www.uiej.com/960.html

[1]: http://mag.fyscu.com
[2]: http://bbs.fyscu.com/forum.php?mod=viewthread&tid=4808
