---
title: Web 与 Chrome 最新进展 @Google I/O 2019
date: 2019-05-15 10:24:04
categories:
  - Development
tags:
  - Web
  - Chrome
  - Google
  - lecture

slidehtml: true
---

> Google I/O 2019 extend 成都站

---

## DOM 与 BOM

---

### 延迟加载

```html
<img src="path/to/image" loading="lazy" />

<iframe src="path/to/page" loading="lazy"></iframe>
```

---

`loading` 属性值

- `lazy` 延迟加载
- `eager` 立即加载
- `auto` 浏览器自主判断加载策略（默认值）

---

- `loading` 属性是早期提案 `lazyload` 属性的代替
- 将在 Chrome 76 发布
- [补丁库](https://github.com/noahbuscher/img-lazy-loading-poly)

---

### 门户框架

#### 主页面

```html
<portal src="path/to/page"></portal>
```

```javascript
const portal = document.querySelector('portal');

portal
  .activate({ data: { key: 'value' } })
  .then(() => portal.postMessage({ hello: 'portal' }));
```

---

#### 子页面

```javascript
window.addEventListener('portalactivate', event => {
  console.log(event.data); // {key: 'value'}

  const portal = event.adoptPredecessor(); // 主页面

  document.body.append(portal);
});

window.addEventListener('message', event =>
  console.log(
    event.data, // {hello: 'portal'}
    window.portalHost // 主页面 window 对象
  )
);
```

---

![Portal demo](https://web.dev/hands-on-portals/glitch.gif)

---

### 分享网页

```javascript
document.querySelector('button.share').addEventListener('click', async () => {
  const data = { title: 'Hello', text: 'World', url: 'https://web.dev/' };

  if (!navigator.canShare(data)) return;

  await navigator.share(data);

  console.log('Done!');
});
```

- 网页由 HTTPS 打开
- Android Chrome 61+
- 分享调用在**用户操作回调栈**中发起

---

在 PWA 的 `manifest.json` 中多加一段

即可成为**分享目标**（Chrome 71+）

```json
{
  "share_target": {
    "action": "/path/to/share-target/",
    "method": "GET",
    "enctype": "application/x-www-form-urlencoded",
    "params": {
      "title": "your_title_key",
      "text": "your_text_key",
      "url": "you_url_key"
    }
  }
}
```

---

[![Share to PWA](https://developers.google.com/web/updates/images/2018/12/wst-send.png)](https://web-share.glitch.me/)

---

### 形状检测

#### 文字

```javascript
const textDetector = new TextDetector();

  const texts = await textDetector.detect(image);

texts.forEach(text => console.log(text));
```

---

#### 条码

```javascript
const barcodeDetector = new BarcodeDetector({
  formats: [
    'aztec',
    'code_128',
    'code_39',
    'code_93',
    'codabar',
    'data_matrix',
    'ean_13',
    'ean_8',
    'itf',
    'pdf417',
    'qr_code',
    'upc_a',
    'upc_e'
  ]
});

const barcodes = await barcodeDetector.detect(image);

barcodes.forEach(barcode => console.log(barcode));
```

---

#### 人脸

```javascript
const faceDetector = new FaceDetector({
  maxDetectedFaces: 5,
  fastMode: false
});

const faces = await faceDetector.detect(image);

faces.forEach(face => console.log(face));
```

---

- Chrome 74+（开启*实验性 Web 平台特性*标记）
- 支持 Web worker
- [各大平台支持](https://github.com/WICG/shape-detection-api#overview)
- [Web 感知工具包](https://perceptiontoolkit.dev/getting-started/)

---

### 更多草案 API

- [文件系统](https://github.com/WICG/native-file-system)
- [短信验证](https://github.com/sso-google/sms-otp-retrieval)

---

## HTTP

---

### 同站 Cookie

```
Set-Cookie: a=1; SameSite=Strict
Set-Cookie: b=2; SameSite=Lax
Set-Cookie: c=3; SameSite=None
```

[RFC 6265 bis](https://tools.ietf.org/html/draft-ietf-httpbis-cookie-same-site-00)

---

[![](https://web.dev/samesite-cookies-explained/samesite-none-lax-strict.png)](https://web.dev/samesite-cookies-explained/)

---

### Web 包协议

![](https://lh6.googleusercontent.com/n6lWJFxp7RqGt2HalG6-pHCDlGSUso1EHRGzpWP8CZRQ9jmfn18XwbxtUcaFvA5AwcyygTzp_sUu-UTgukCd8C-BTjpyytfyIpiAWuVD7sSuIlbQjeVqNH4ZbqWaqObfpr6cHw_S)

---

<div style="background: white">
![](https://developers.google.com/web/updates/images/2018/11/signed-exchanges.svg)
</div>

---

## JavaScript

---

### WeakRef

---

## Google

---

### Search How to

---

### Material Design 夜间模式

[![](https://storage.googleapis.com/spec-host-backup/mio-design%2Fassets%2F1Eb0bcqf3yyrabY8JLOrfC9zq_nN8wCu9%2Fdarktheme-overview.png)](https://material.io/design/color/dark-theme.html)
