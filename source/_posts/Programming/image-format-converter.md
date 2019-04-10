---
title: 22 行 JS 写个图片格式转换器
date: 2018-03-07 00:17:00
author: 南漂一卒
categories:
  - Programming
tags:
  - image
  - format
  - converter
---

【原文链接】https://my.oschina.net/TechQuery/blog/1630721

虽然国内大厂（豆瓣、微信公众平台 等）已支持 Google 推出的 **WebP 图片格式**来进一步优化性能，但其它多数软件平台还是只支持 BMP、GIF、JPEG、PNG 等经典格式，有时临时找个支持 WebP 的**图片格式转换器**也挺麻烦的，不如抄起键盘就是一把梭~

---

## 通用源码

```javascript
(function() {
  var canvas = document.createElement('canvas');

  var context2D = canvas.getContext('2d');

  self.convertImage = function(image, format) {
    context2D.clearRect(0, 0, canvas.width, canvas.height);

    canvas.width = image.naturalWidth;

    canvas.height = image.naturalHeight;

    image.crossOrigin = 'Anonymous';

    context2D.drawImage(image, 0, 0);

    return canvas.toDataURL('image/' + (format || 'png'), 1);
  };
})();
```

## 基本用法

直接用 [Google Chrome™](https://www.google.cn/chrome/) 调试器的 Source Snippets（源码片段）功能运行 ——

![代码用法](https://static.oschina.net/uploads/img/201803/07210728_7XZk.png)

若有[跨域相关报错](https://segmentfault.com/q/1010000002459456)，可自行访问该图片的服务器首页，用自创的 `<img src="path/to/image" />` 加载待转换图片 ——

![跨域技巧](https://static.oschina.net/uploads/img/201803/07004925_icAE.png)

```javascript
my_img.src = convertImage(my_img, 'jpeg');
```

上述代码运行完毕后，网页右键菜单中的**图片另存为**功能保存的就是 JPEG 格式的图片。
