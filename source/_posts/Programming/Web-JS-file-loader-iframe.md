---
title: 【原创】Web 前端脚本文件响应式异步加载器（框架页兼容）
date: 2014-10-28 05:06:00
author: 南漂一卒
categories:
  - Programming
tags:
  - Web
  - JavaScript
  - module
  - loader
---

我之前开源了[【JavaScript 文件响应式异步加载器】v0.6][1]，用于在 Web 前端统一管理单页开发中的 JS 模块加载，方便、优雅、可靠。

但若要用框架页的模式来开发较为复杂的 Web 应用，统一管理父子页面的 JS、CSS 又是一件麻烦的事……

框架页是一种 HTML 模块化的基础技术，我们通常希望整个网页在视觉上浑然一体，而非被框架分割成一块又一块，所以不同的框架页通常使用同一套基础的 JS、CSS 脚本库。一般的做法是在每个框架页的 `<head />` 中都硬编码上完全相同的基础库引用，虽然浏览器缓存会消除文件的重复下载，但 JS 代码会在每个框架页的上下文中再运行一遍，多消耗 N 倍的 CPU 执行时间、多占用 N 份相同大小的内存……

好在框架页之间有 Window 对象之间的引用，好在 JavaScript 的函数可以用 call/apply 方法轻松改变执行上下文

## 框架页稳定版 v0.8

```javascript
//
//                >>>  EasyImport.js  <<<
//
//
//      [Version]    v0.8  (2014-10-31)  Stable
//
//      [Usage]      Only for loading JavaScript files in All Kinds of Web,
//                   with JS/CSS-Inherit support for Frames.
//
//
//              (C)2013-2014    SCU FYclub-RDD
//

(function (BOM, DOM) {
  // ----------- Private Attribute ----------- //
  var UA = navigator.userAgent,
    RootPath,
    JS_List = [],
    CallBack,
    NameSpace = [],
    EIJS = {
      AsyncJS: [],
      DOM_Ready: []
    };
  // ----------- Inner Basic Member ----------- //
  var IE_Ver =
    UA.match(/MSIE (\d+)\.\d/i) || UA.match(/Trident\/.+; rv:(\d+)\.\d/i);
  IE_Ver = IE_Ver ? Number(IE_Ver[1]) : NaN;
  var old_IE = IE_Ver && IE_Ver < 9,
    is_Pad = UA.match(/Tablet|Pad|Book|Android 3/i),
    is_Phone = UA.match(/Phone|Touch|Symbian/i);
  var is_Mobile = is_Pad || is_Phone || UA.match(/Mobile/i);

  function DOM_Loaded_Event(DOM_E, CB_Func) {
    if (!old_IE) DOM_E.addEventListener('load', CB_Func, false);
    else
      DOM_E.attachEvent('onreadystatechange', function () {
        if (DOM_E.readyState.match(/loaded|complete/)) CB_Func.call(DOM_E);
      });
  }
  function DOM_Ready_Event(_CB) {
    if (!old_IE) DOM.addEventListener('DOMContentLoaded', _CB, false);
    else if (self !== top) DOM_Loaded_Event(DOM, _CB);
    else
      (function () {
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
    if (AttrList) for (var AK in AttrList) NE[AK] = AttrList[AK];
    return NE;
  }
  function PagePath_IE(_BOM) {
    var _PP = _BOM.document.URL;
    _PP = _PP.split('/');
    if (_PP.length > 3) _PP.pop();
    _PP.push('');
    return _PP.join('/');
  }

  var DOM_Head = $_TN(DOM, 'head')[0];
  var DOM_Title = $_TN(DOM_Head, 'title')[0];
  try {
    var _DOM = parent.document;
  } catch (Err) {
    var Inherit_Root = true;
  }
  // ----------- Frame Inherit (JS) ----------- //
  if (Inherit_Root || self === top) {
    //  Default NameSpaces (Offical Support Libraries)
    if (!IE_Ver)
      NameSpace = [
        { jQuery: true },
        {
          $: function () {
            return this.jQuery(
              arguments[0],
              arguments.length > 1 ? arguments[1] : this.document
            );
          }
        }
      ];
  } else {
    var _NS;
    if (!IE_Ver) {
      _NS = parent.ImportJS.NS();
      for (var i = 0; i < _NS.length; i++)
        for (__NS in _NS[i]) {
          var _NS_Shell = _NS[i][__NS];
          if (_NS_Shell === true) {
            var _NS_ = parent[__NS];
            self[__NS] = _NS_;
          } else if (typeof _NS_Shell == 'function')
            self[__NS] = (function (_NS_S) {
              return function () {
                return _NS_S.apply(self, arguments);
              };
            })(_NS_Shell);
        }
    } else {
      _NS = [];
      var _SE = $_TN($_TN(_DOM, 'head')[0], 'script');
      for (var i = 2, JS_URL; i < _SE.length; i++) {
        JS_URL = _SE[i].src;
        if (JS_URL == '' || JS_URL.match(/EasyImport.*\.js/i)) continue;
        if (IE_Ver < 8) JS_URL = PagePath_IE(parent) + JS_URL;
        _NS.push(JS_URL);
      }
      BOM.onbeforeunload = function () {
        var SE = $_TN(DOM, 'script');
        for (var i = 0; i < SE.length; i++) SE[i].parentNode.removeChild(SE[i]);
        SE = null;
        DOM.write('');
        DOM.clear();
        BOM.onbeforeunload = null;
        BOM.close();
        top.CollectGarbage();
      };
    }
    NameSpace = _NS;
  }
  // ----------- Frame Inherit (CSS) ----------- //
  if (!Inherit_Root && self !== top) {
    var _LE = _DOM.styleSheets;
    for (var i = 0, CSS_URL; i < _LE.length; i++) {
      if (_LE[i].rel.indexOf('nofollow') == -1) continue;
      CSS_URL = _LE[i].href;
      if (IE_Ver < 8) CSS_URL = PagePath_IE(parent) + CSS_URL;
      DOM_Head.appendChild(
        _LE[i][old_IE ? 'owningElement' : 'ownerNode'].cloneNode(true)
      ).href = CSS_URL;
    }
  }
  // ----------- Standard Mode Meta Patches ----------- //
  if (is_Mobile) {
    if (!old_IE) {
      var is_WeChat = UA.match(/MicroMessenger/i),
        is_UCWeb = UA.match(/UCBrowser|UCWeb/i);
      DOM_Head.insertBefore(
        NewTag('meta', {
          name: 'viewport',
          content: [
            ['width', is_WeChat || is_UCWeb ? 320 : 'device-width'].join('='),
            'initial-scale=1.0',
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
    DOM_Head.insertBefore(
      NewTag('meta', {
        'http-equiv': 'X-UA-Compatible',
        content: 'IE=Edge, Chrome=1'
      }),
      DOM_Title
    );
  // ----------- Inner Logic Module ----------- //
  function xImport(JS_URL, CB_Func) {
    var SE = DOM.createElement('script');
    with (SE) {
      type = 'text/javascript';
      charset = 'UTF-8';
      src = JS_URL;
    }
    if (CB_Func) DOM_Loaded_Event(SE, CB_Func);
    return DOM_Head.appendChild(SE);
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
      var AJS_CB = function () {
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
          JS_RC[k + 1][0]['CallBack'] = function (iEvent) {
            if (++DRQ[DRQ_NO] == 2) Final_CB.apply(self);
          };
          DOM_Ready_Event(JS_RC[k + 1][0].CallBack);
        } else if (k > -1) {
          var Last_Script = JS_RC[k + 1][0].URL;
          JS_RC[k][0]['CallBack'] = function (iEvent) {
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
  BOM.ImportJS = function () {
    var Func_Args = Array.prototype.slice.call(arguments, 0);

    if (typeof Func_Args[0] == 'string') RootPath = Func_Args.shift();
    else RootPath = './';
    if (Func_Args[0] instanceof Array) JS_List = Func_Args.shift();
    else throw "Format of Importing List isn't currect !";
    if (typeof Func_Args[0] == 'function') CallBack = Func_Args.shift();
    else CallBack = null;
    if (Func_Args[0] instanceof Array && !IE_Ver)
      NameSpace = NameSpace.concat(Func_Args[0]);
    if (IE_Ver) {
      JS_List = NameSpace.concat(JS_List);
      NameSpace = [];
    }

    var JS_Item = SL_Set(RootPath, JS_List);
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
  BOM.ImportJS.NS = function () {
    return NameSpace;
  };
})(self, self.document);
```

[1]: /programming/Web-JS-file-loader/
