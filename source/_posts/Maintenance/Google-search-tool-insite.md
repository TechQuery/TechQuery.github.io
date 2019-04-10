---
title: 【原创】Google 站内搜索小工具
date: 2014-11-26 08:32:00
author: 南漂一卒
categories:
  - Maintenance
tags:
  - Google
  - browser
  - bookmark
  - toolkit
---

对于**经常在互联网中查资料去开拓自己的未知领域**的人们来说，**搜索引擎**绝对是必备工具 —— 它不但能在全球浩瀚的信息海洋中对我们需要的信息“大海捞针”，也能有针对性地对某个网站的内容进行搜寻，让我们能不受**内容源网站自身内容组织结构**的负面影响，迅速找到需要的内容。

虽然 **站内搜索** 早已是搜索引擎的标配，但大多做得不好，经常一篇文章已经在某网站发布了比较久了，但就是搜不出来……（特别是 中途调整过**网址结构**的网站，比如 路径格式变更、换子域名、换主域名 等）

目前为止，**Google** 依然是 难得可用且很好用的**站内搜索引擎**，这一点我在以前原创的[【Google 站内搜索助手】v0.9][1]的发布帖中早有论述~

因为我对上述工具的[上游代码][2]做了升级，所以也重构了这个工具。而且，我把自己写的所有实用的 **Bookmarklet** 整合成了一个[**开源项目 iBookmarkLet**][3] 发布，所以博文里就只贴出**安装版代码** ——

<!-- prettier-ignore-start -->
```javascript
javascript: (function(BOM,DOM){function iRegExp(a,b){var c=/ /;return c.compile(a,b),c}var DN,HN,trim=function(){var a=iRegExp("(^s*)|(s*$)","g");return function(b){return b.replace(a,"")}}(),SS=function(WF){var TN,TA,N,n,IS="";for(TN in{input:0,textarea:0})try{TA=DOM.getElementsByTagName(TN);for(N in TA)with(TA[N])IS=trim(value.slice(selectionStart,selectionEnd))}catch(EO){}if(""==IS&&(IS=trim(DOM.selection?DOM.selection.createRange().text:DOM.getSelection().toString())),""==IS)for(n=0;n<WF.length;n++){try{IS=arguments.callee(WF[n])}catch(EO){}if(""!=IS)break}return IS}(BOM.frames);""!=SS?(HN=BOM.location.hostname,DN=HN.match(/(edu|net|org|com|gov)\.\w+$/)?HN.match(/(\w+\.){2}\w+$/):HN.match(/\w+\.\w+$/),BOM.open("https://wen.lu/search?newwindow=1&q="+encodeURIComponent([SS," site:",DN[0]].join("")),"_blank")):BOM.confirm("您未选中任何网页中的文字……\n\n\n『确定』进入问题反馈；『取消』即退出本工具。")?(BOM.prompt("输入框中的是『运行环境』信息，请直接复制它们，按『确认』即可访问 原作者主页～",navigator.userAgent),BOM.open("http://www.fyscu.com/","_blank")):BOM.alert("【Google 站内搜索助手 v1.0】\n\n(C)2013-2014  四川大学·飞扬俱乐部·研发部")})(top,top.document);
```
<!-- prettier-ignore-end -->

[1]: http://bbs.fyscu.com/forum.php?mod=redirect&goto=findpost&ptid=2711&pid=58461
[2]: /programming/google-search-tool/
[3]: http://gitee.com/Tech_Query/iBookmarkLet/
