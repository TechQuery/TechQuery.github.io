---
title: 【JavaScript 进阶】面向对象编程与 jQuery 式 API 的实现
date: 2014-09-26 20:24:00
author: 南漂一卒
categories:
  - Technology
  - Programming
tags:
  - JavaScript
  - OOP
  - jQuery
---


**JavaScript 简洁、灵活，jQuery 优雅、强大** —— 它们共同开拓了新的思维方式，影响了一代程序员和他们的 Web 前端开源项目，这是毋庸置疑的~

要想让它们的简洁、灵活、优雅、强大为我所用，内化为编程上的精神法宝，亲手“临摹”一下既有趣又有力。一番死磕之后，jQuery 中蕴含的 **JavaScript 面向对象的思想方法** 自然就很容易理解了~


## 山寨版 jQuery 半成品

目前仅简单地实现了**最常用的选择器**（标签、ID、类、first/last/only/nth-child 伪类）、**选择器上下文**、**最常用的 jQuery 对象方法**（each、find、on、off 等）、**连缀语法**等 ——

```javascript
// ----------- xDOM  Core Object ----------- //
function DOMxx(Selector, Context) {
    this.xDOM = '0.5';
    this.jquery = '1.9.0';    //  Work with the version of jQuery API

    // ----------- Atom Functions (Private Method) ----------- //
    function Each(Object, CallBack) {
        for (var key in Object)  if (key != 'length') {
            var _RV_ = CallBack.call(
                Object[key],
                isNaN(Number(key)) ? key : Number(key)
            );
            if (_RV_ === false) break;
        }
    }
    function inArray(Value, iArray) {
        for (var i = 0; i < iArray.length; i++)
            if (iArray[i] === Value)  return i;
        return false;
    }
    function Child_PC(EOS, PCS) {
        var PES = [], CES = [];
        Each(EOS, function () {
            if (inArray(this.parentNode, PES) === false)
                PES.push(this.parentNode);
        });
        var PCS_KW = PCS.match(/^(\\w+)-child/i)[1];
        if (PCS_KW == 'nth')
            var PCS_EC = PCS.match(/^nth-child\\((.+)\\)/i)[1];
        Each(PES, function () {
            EOS = this.children;
            if (! EOS) return;
            switch (PCS_KW) {
                case 'first':    {
                    CES.push(EOS[0]);
                    return;
                }
                case 'last':     {
                    CES.push(EOS[EOS.length - 1]);
                    return;
                }
                case 'only':     {
                    if (EOS.length == 1)
                        CES.push(EOS[0]);
                    return;
                }
                case 'nth':      {
                    PCS_EC = PCS_EC.replace(/(\d+)(\w|\()/g, '$1 * $2');
                    for (var n = 1, N = 0; ; n++) {
                        N = eval(PCS_EC);
                        if (N >= EOS.length) return;
                        CES.push(EOS[N]);
                    }
                }
            }
        });
        return CES;
    }
    function $_Atom(REO, SE) {
        SE = SE.split(':');
        if (SE.length > 1)  var PC = SE[1];
        SE = SE[0];
        switch (SE.slice(0, 1)) {
            case '#':    {
                REO = self.document;
                MN = 'getElementById';
                SE = SE.slice(1);
                break;
            }
            case '.':    {
                MN = 'getElementsByClassName';
                SE = SE.slice(1);
                break;
            }
            case '*':    ;
            default:     MN = 'getElementsByTagName';
        }
        REO = REO[MN](SE);
        REO = PC ? Child_PC(REO, PC) : REO;
        return  (! REO.length) ? [REO] : (
            (REO instanceof Array) ? REO : [].slice.apply(REO)
        );
    }
    function Select(Selector, Context) {
        var Selector_Group = Selector.split(' '),  Level = 0,
            Select_Result = [];
        Each(
            $_Atom( Context, Selector_Group[Level] ),
            function (Index) {
                if (! Index) Level++;
                while (Selector_Group[Level] == '')
                    Level++;
                if (! Selector_Group[Level]) {
                    Select_Result.push(this);
                    return;
                }
                if (! this.childNodes.length)  return;
                Each(
                    $_Atom(this, Selector_Group[Level]),
                    arguments.callee
                );

            }
        );
        return Select_Result;
    }
    function Event_Bind(_Element, _Type, _CallBack, _unBind) {
        var _CB_ = function (Event) {
            EO = self.event || Event;
            EO.target = EO.srcElement || EO.target;
            if (! _CallBack.call(_Element, EO))
                if (EO.returnValue) {
                    EO.returnValue = false;
                    EO.cancelBubble = true;
                } else {
                    EO.preventDefault();
                    EO.stopPropagation();
                };
        };
        if (_Element.attachEvent)
            _Element[ (_unBind ? 'de' : 'at') + 'tachEvent'](
                'on' + ((_Type == 'input') ? 'propertychange' : _Type),
                _CB_
            );
        else
            _Element[
                (_unBind ? 'remove' : 'add') + 'EventListener'
            ](_Type, _CB_, false);
    }

    // ----------- Powerful API (Public Method / Data) ----------- //
    this.selector = Selector;
    this.context = (Context || self.document);
    this.DOM = Select(Selector, this.context);
    if (this.DOM.length == 1)
        this.context = this.DOM[0];

    this.get = function (Index) {
        return this.DOM[Index];
    };
    this.each = function (CallBack) {
        Each(this.DOM, CallBack);
        return this;
    };
    this.find = function (Selector) {
        return  new this.constructor(
            [this.selector, Selector].join(' '),  this.context
        );
    }
    this.on = function (Event_Type, CallBack) {
        return this.each(function () {
            Event_Bind(this, Event_Type, CallBack);
        });
    };
    this.off = function (Event_Type, CallBack) {
        return this.each(function () {
            Event_Bind(this, Event_Type, CallBack, true);
        });
    }
}
// ----------- xDOM  Shortcut Function ----------- //
function xDOM(CSS_Selector, DOM_Context) {
    return  new DOMxx(CSS_Selector, DOM_Context);
}
```

有木有一种步 **Zepto.js** 后尘的感觉……？


## 【参考文献】

 1. http://www.cnblogs.com/newyorker/archive/2013/02/14/2891298.html
