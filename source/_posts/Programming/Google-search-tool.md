---
title: 【浏览器小工具】墙内 Google 划词搜索
date: 2014-11-25 04:01:00
author: 南漂一卒
categories:
  - Programming
tags:
  - Google
  - browser
  - bookmark
  - toolkit
---


天朝**习大大**继位之后，政坛、社会暗流更加汹涌，各路高官皆有落马者…… **好在江山依然稳固，彭阿姨依然美丽**，但我**大谷歌**却牺牲更大，义务封口近一岁，重放新声遥无期……

好在我朝素不缺**侠肝义胆之士**，树立数个**海外镜像**来招魂，其魂最真者 —— [某 **V2EX** 大神的一声**天问** —— 路在何方！~][1]

鄙人不才，仅能**低调地传承侠客的义举** —— 把大神的招魂镜像封装在[我之前做的一个 Bookmarklet][2] 中，并再次优化，方便大家使用 ——

## 安装用代码

```javascript
javascript: (function(BOM,DOM,SL){function iRegExp(a,b){var c=/ /;return c.compile(a,b),c}var trim=function(){var a=iRegExp("(^s*)|(s*$)","g");return function(b){return b.replace(a,"")}}(),SS=function(WF){var TN,TA,N,n,IS="";for(TN in{input:0,textarea:0})try{TA=DOM.getElementsByTagName(TN);for(N in TA)with(TA[N])IS=trim(value.slice(selectionStart,selectionEnd))}catch(EO){}if(""==IS&&(IS=trim(DOM.selection?DOM.selection.createRange().text:DOM.getSelection().toString())),""==IS)for(n=0;n<WF.length;n++){try{IS=arguments.callee(WF[n])}catch(EO){}if(""!=IS)break}return IS}(BOM.frames);""!=SS?BOM.open(["https://wen.lu/search?newwindow=1&lr=lang_",SL,"&q=",encodeURIComponent(SS)].join(""),"_blank"):BOM.confirm("您未选中任何网页中的文字……\n\n\n『确定』进入问题反馈；『取消』即退出本工具。")?(BOM.prompt("输入框中的是『运行环境』信息，请直接复制它们，按『确认』即可访问 原作者主页～",navigator.userAgent),BOM.open("http://www.fyscu.com/","_blank")):BOM.alert("【Google 中文搜索助手 v0.9】\n\n(C)2013-2014  四川大学·飞扬俱乐部·研发部")})(top,top.document,"zh-CN");
```

## 开发用源码

```javascript
(function (BOM, DOM, SL) {

    function iRegExp(Pattern, Mode) {
        var RegEx = / /;
        RegEx.compile(Pattern, Mode);
        return RegEx;
    }
    var trim = (function () {
        var RegEx = iRegExp('(^\s*)|(\s*$)', 'g');
        return  function (Str) {
            return Str.replace(RegEx, '');
        };
    })();

    var SS = (function (WF) {
        var IS = '';
        for (var TN in {input: 0, textarea: 0}) try {
            var TA = DOM.getElementsByTagName(TN);
            for (var N in TA) with (TA[N])
                IS = trim( value.slice(selectionStart, selectionEnd) );
        } catch (EO) {};
        if (IS == '')  IS = trim(
            DOM.selection ?
                DOM.selection.createRange().text :
                DOM.getSelection().toString()
        );
        if (IS == '')  for (var n = 0; n < WF.length; n++) {
            //  以下异常处理 很神奇地 绕过了 IE、Firefox 对 iframe 的跨域安全限制～
            //  但可能在它们中运行时会有一瞬间的卡顿……
            try { IS = arguments.callee( WF[n] ); } catch (EO) {};
            if (IS != '') break;
        }
        return IS;
    })(BOM.frames);

    if (SS != '')
        BOM.open([
            'https://wen.lu/search?newwindow=1&lr=lang_', SL,
            '&q=', encodeURIComponent(SS)
        ].join(''), '_blank');
    else if ( BOM.confirm(
        "您未选中任何网页中的文字……\n\n\n『确定』进入问题反馈；『取消』即退出本工具。"
    ) ) {
        BOM.prompt(
            "输入框中的是『运行环境』信息，请直接复制它们，按『确认』即可访问 原作者主页～",
            navigator.userAgent
        );
        BOM.open('http://www.fyscu.com/', '_blank');
    } else  BOM.alert(
        "【Google 中文搜索助手 v0.9】\n\n(C)2013-2014  四川大学·飞扬俱乐部·研发部"
    );
})(top, top.document, 'zh-CN');
```

## 为何还坚持 Google ？

 1. **棱镜门丑闻**虽然揭露了**美帝在国际互联网隐藏多年的大木马**，Google 等业界典范也未能幸免…… 但一个**世界最大的移民国家对自由执着的追求**，让那些阴暗面总是不能得逞太久；而**天朝的秘密**大多都躺在几十年后还不一定能解密的档案中……
 2. **百度**这样一个**唯利是图、没啥技术**的搜索引擎，**搜索结果中假冒伪劣、坑蒙拐骗丛生，人为和谐比比皆是** —— 叫“中国黄页”还差不多~
 3. **搜狐搜狗**虽是**中国大陆搜索引擎中最用心、最努力的**，2010年和**腾讯搜搜**一样，接收了一些**谷歌中国危机**中跳槽的人才，2014年又全面合并人才储备不少的搜搜，加上**搜狗输入法**多年积累的**语料数据、技术创新**，已经很棒了~ 但个人近两年对它的持续使用发现 —— **垃圾站屏蔽、小众站抓取** 还有较大进步空间……
 4. **奇虎360搜索** 有点像当年 115搜索 等**结果二次筛选**的网站，虽然它一直力行用户体验第一的产品原则，但作为 3B大战的直接产物，加上360一直屡教不改的“**流氓推广**”方式，让我没法信任它提供的内容……


  [1]: http://www.v2ex.com/t/126018
  [2]: http://bbs.fyscu.com/forum.php?mod=viewthread&tid=3677
