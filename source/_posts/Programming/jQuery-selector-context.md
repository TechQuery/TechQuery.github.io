---
title: 【奇技淫巧】jQuery 的选择器上下文
date: 2014-09-12 03:53:00
author: 南漂一卒
categories:
  - Programming
tags:
  - jQuery
---


用过 **jQuery** 的都知道 —— jQuery() 很强大，啥都能往里塞！

 1. 塞 **HTML 代码** —— 创建 HTML 元素
 2. 塞 **CSS 选择符** —— 检索 HTML 元素
 3. 塞 **XPath 表达式** —— 检索 XML 元素
 4. 塞 **DOM 对象** —— 包装 HTML 元素，使用 jQuery 的强大 API（对象方法）
 5. 塞 **JS 函数** —— 相当于当在 DOM Ready 之后执行主业务逻辑代码

但还有些细节需要注意，比如 **jQuery 选择器的上下文**问题。

----------

jQuery 老手想必都知道，**jQuery() 的第二个参数**可以设置 选择器执行筛选的上下文，也就是**指定元素搜索只能在哪个（些）元素的子元素树形结构范围内**。比如 ——

 1. `$('script', document.body)` 相当于 `$('body').find('script')`
 2. `$('table',
    document.getElementsByTagName('iframe')[0])` 相当于 `$('iframe').eq(0).contents().find('table')`

改变了 jQuery 选择器的上下文之后，这个 jQuery 对象接下来的操作都以这个上下文为参考系，非常方便~

但这里有个例外 —— **若一个 jQuery 对象由 DOM 对象包装而来，那它的上下文会被强制指向 DOM 元素本身，jQuery() 的第二个参数不能改变上下文……**

大家可以用以下代码验证 ——

```javascript
console.log( $('body') );
//  显示的 jQuery 对象的 context 属性指向 document

console.log( $(document.body, document) );
//  显示的 jQuery 对象的 context 属性指向 document.body
```
这有何影响？一个典型的实例就是 —— **HTML 子框架页（比如 `<iframe />`）直接使用父框架加载的 jQuery 及其插件**。

----------

因为不同框架页和其父页面一样，都是一个完整的网页，所以它们的 CSS、JS 之间默认是相互独立的（除了 BOM、DOM 树之间有相互引用）。程序猿一般都分别加载它们所引用的脚本，无论它们是否有相同的脚本。虽然浏览器都有缓存，不会重复从服务器下载相同的文件，但对本机性能而言，重复加载还是有影响。

对 CSS 而言，它的属性最终要应用到具体的 DOM 元素上，不会单独占用内存，重复加载对本机无负面影响；但若**重复加载 JS，同样的对象（各种 JS 库所创建的自身实例）会在每个父/子页面的 Window 对象下创建，造成了内存的浪费**……

综上，因为 **JS 等动态语言的赋值 默认是“引用”（传指针/地址）而非复制**，我们就可以借助框架之间的 BOM、DOM 之间的引用，使父页面加载的 JS 库工作在不同的子页面中。例如 ——

```javascript
self.jQuery = parent.jQuery;
```
----------

当我们使用类似 `$(self.document).find('selector').doSomething()` 的代码时，程序运行正常，所以大家自然会想用更简洁的写法 `$('selector').doSomething()`，但它不会在子页面里查找任何 DOM 元素，只会返回父页面的执行结果…… 因为**父页面的 jQuery 上下文默认是父页面中的 document 对象**~

所以，我们必须**在子页面对父页面的 jQuery 做一层封装** ——

```javascript
self.$ = function () {
    return  self.jQuery(
        arguments[0],
        (arguments.length > 1) ? arguments[1] : self.document
    );
};
```
----------

但其实这样的封装有时无法改变 jQuery 对象 API 的上下文，比如在一个 `<iframe />` 中 ——

```javascript
$('<span class="Float_Button"></span>').appendTo('body');
```
你会发现，新建的元素被追加到了父页面的最后……

不怕，解决方法也得益于 **jQuery 对象 API（公用方法）的优越性 —— CSS 选择符、DOM 对象、jQuery 对象 都能往里塞**，我们用自己封装的 jQuery 来强制指定目的地的上下文 ——

```javascript
$('<span class="Float_Button"></span>').appendTo( $('body') );
```
----------

【参考文档】http://ucren.com/blog/archives/137/comment-page-1
