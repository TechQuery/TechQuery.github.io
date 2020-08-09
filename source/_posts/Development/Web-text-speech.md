---
title: Web 文本朗读 60 行代码实现
date: 2020-03-28 15:59:59
categories:
  - Development
tags:
  - Web
  - TTS
---

在[武汉肺炎][1]在中国大陆肆虐的两个月，在后方的大家都在家闷得难受。一开始还在网上时刻关注各种疫情新闻，久而久之头昏眼花，十分疲惫。

当[发哨人艾芬医生][2]的[采访稿][3]被封杀后，**墙国网民**再次群情激奋地力挺，把原文用各种文字、编码、平台记录下来，妥妥的“留取丹青照汗青”！

其中有个网友用一款 Google Chrome **文本朗读** (TTS) 扩展把原文读出来，并把朗读过程录成视频分享出来，让大家闭目养神中就能了解**疫情真相**~

作为一个 Google 重度用户，针对一个事情可以搜出太多网页可读，眼睛会非常累，一直想找个方便的文本朗读工具。但作为一个 Web 全栈工程师，**跨平台兼容性**又是深入骨髓的自觉，于是翻出前一阵扫过一眼的 [Web Speech API][4]，抄起键盘就是一把梭 ——

<iframe
    style="width: 100%; height: 85vh"
    scrolling="no" frameborder="no" allowtransparency="true" allowfullscreen="true"
    title="Web Text Speech"
    src="https://codepen.io/tech_query/embed/WNvLdvB?height=300&theme-id=31315&default-tab=js,result"
>
    See the Pen
    <a href='https://codepen.io/tech_query/pen/WNvLdvB'>Web Text Speech</a>
    by TechQuery (<a href='https://codepen.io/tech_query'>@tech_query</a>) on
    <a href='https://codepen.io'>CodePen</a>.
</iframe>

上述代码所用的[语音合成 API][5] [现代浏览器支持][6]很好，能用它的都支持 ES 6，所以我们只需[压缩代码][7]不需转译，再包上 `javascript: (() => { /* code */ })()` 即可直接存在**浏览器收藏栏**，当个扩展按钮使用了 ——

> javascript:!((e)=>{const voice=speechSynthesis.getVoices().find(({lang:e})=>e===navigator.language);function speak(e){const t=new SpeechSynthesisUtterance(e);t.voice=voice,speechSynthesis.speak(t)}function\*walkRange(e){const t=document.createNodeIterator(e.commonAncestorContainer);for(var n;(n=t.nextNode())&&(e.intersectsNode(n)&&(yield n),n!==e.endContainer););}function getSelectedText(e){const t=self.getSelection().getRangeAt(0);if(t&&t+""&&(!e||e.contains(t.commonAncestorContainer)))return[...walkRange(t)].filter(({nodeType:e,parentNode:t})=>{if(3!==e)return;const{width:n,height:o}=t.getBoundingClientRect();return n&&o}).map(({nodeValue:e},n,{length:o})=>e.slice(0===n?t.startOffset:0,n===o-1?t.endOffset:1/0)).filter(e=>e.trim()).join("").trim()}document.addEventListener("selectionchange",()=>{const e=getSelectedText();e&&!speechSynthesis.speaking?speak(e):speechSynthesis.cancel()}),self.alert("Selected Text will be speak out automatically");})();

<picture>
    <source type="image/webp" srcset="https://caniuse.bitsofco.de/image/speech-synthesis.webp">
    <img
        src="https://caniuse.bitsofco.de/image/speech-synthesis.png"
        alt="Data on support for the speech-synthesis feature across the major browsers from caniuse.com"
    >
</picture>

代码中还有几个重要 API，列出来供大家参考 ——

1. [Selection Range](https://developer.mozilla.org/zh-CN/docs/Web/API/Range)
2. [Selection change 事件](https://developer.mozilla.org/zh-CN/docs/Web/API/Document/selectionchange_event)
3. [NodeIterator](https://developer.mozilla.org/zh-CN/docs/Web/API/NodeIterator)

{% asset_img Web-TTS.png 操作图示 %}

最后，以艾芬医生的两句话**与君共勉** ——

> 每个人还是要**坚持自己独立的思想**，因为**要有人站出来说真话**，必须要有人，**这个世界必须要有不同的声音**。

> 早知道有今天，我管他批评不批评，**『老子』到处说！**

[1]: https://zh.wikipedia.org/wiki/2019%E5%86%A0%E7%8A%B6%E7%97%85%E6%AF%92%E7%97%85
[2]: https://zh.wikipedia.org/wiki/%E8%89%BE%E8%8A%AC
[3]: http://archive.is/OLdHs
[4]: https://developer.mozilla.org/zh-CN/docs/Web/API/Web_Speech_API/Using_the_Web_Speech_API
[5]: https://developer.mozilla.org/zh-CN/docs/Web/API/SpeechSynthesis
[6]: http://caniuse.com/#feat=speech-synthesis
[7]: https://skalman.github.io/UglifyJS-online/
