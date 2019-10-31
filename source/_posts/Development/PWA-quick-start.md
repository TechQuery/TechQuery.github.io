---
title: PWA 即刻上手
date: 2019-10-31 22:41:33
categories:
  - Development
tags:
  - PWA
  - Web
---

## 应用元数据

一个[渐进式 Web 应用][1]首先要声明一下自己的基本信息：

`index.html`

```html
<head>
  <link rel="manifest" href="index.webmanifest" />
</head>
```

`index.webmanifest`

```json
{
  "name": "应用全名（启动页）",
  "short_name": "应用短名（桌面图标）",
  "start_url": "https://example.dev/",
  "description": "应用简介",
  "scope": "/",
  "display": "standalone",
  "orientation": "any",
  "lang": "zh-CN",
  "dir": "ltr",
  "theme_color": "rgba(0,0,0,0.5)",
  "background_color": "transparent",
  "icons": [{ "src": "logo.png", "type": "image/png", "sizes": "144x144" }]
}
```

## 后台服务

一个 PWA 能真正安装到用户系统中，还需要一个 **Service Worker**。但它的工作原理对初学者直接手写有些困难，亲爹 Google 已经做好了开发框架 [Workbox][2]，还配了 [Workbox CLI][3]，几行命令就能生成一个基本的 ServiceWorker，简直不要太贴心：

```shell
npm install workbox-cli --global

cd your_project/

npm run build  # 先执行你项目的构建脚本，生成生产环境目录

workbox wizard
```

跟着向导执行完上述命令，会生成一个名为 `workbox-config.js` 的配置文件，但须稍加修改来适应“天朝国情”：

```js
module.exports = {
  globDirectory: 'dist/',
  globPatterns: ['**/*.{jpg,png,jpeg,ico,html,css,js}'],
  swDest: 'dist/sw.js',
  // 禁用 Google Cloud CDN
  importWorkboxFrom: 'disabled',
  // 启用 第三方 CDN
  importScripts: ['https://cdn.jsdelivr.net/npm/workbox-sw']
};
```

然后就可以一键生成 ServiceWorker 了：

```json
workbox generateSW
```

最后在 `index.html` 里引用一下即可：

```html
<head>
  <link rel="manifest" href="index.webmanifest" />
  <script>
    if ('serviceWorker' in navigator)
      window.addEventListener('load', function() {
        navigator.serviceWorker.register('sw.js');
      });
  </script>
</head>
```

## 工具链兼容性

虽然官方提供了 [webpack 和 Rollup 的插件][4]，但 **0 配置、一键化**的 [Parcel][5] 却还没有靠谱的相关插件。于是自己先踩了一遍坑，秘籍如下：

1. 在源码目录新建一个上述的 `sw.js`，内容留空，只为 Parcel 这类**面向资源的打包器**不报错

2. 对 `package.json` 中的构建脚本修改如下：

```json
{
  "scripts": {
    "build-sw": "rm -f dist/sw.js.map  &&  workbox generateSW",
    "build": "parcel build source/index.html --public-url .  &&  npm run build-sw"
  }
}
```

---

好了，马上重新部署你的 Web 应用，在手机 Chrome 上试试把它**添加到桌面**吧！

![](https://www.atyantik.com/wp-content/uploads/2017/10/PWA-States.png)

[1]: https://developers.google.cn/web/progressive-web-apps/
[2]: https://developers.google.cn/web/tools/workbox
[3]: https://developers.google.cn/web/tools/workbox/guides/generate-service-worker/cli
[4]: https://developers.google.cn/web/tools/workbox/guides/using-bundlers
[5]: https://parceljs.org/
