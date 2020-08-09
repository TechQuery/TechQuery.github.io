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
  <!-- 老浏览器私有 meta 标签兼容 -->
  <script src="https://cdn.jsdelivr.net/npm/pwacompat@2.0.15/pwacompat.min.js"></script>
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
  "icons": [
    {
      "src": "logo.png",
      "type": "image/png",
      "sizes": "144x144"
    }
  ]
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

```javascript
module.exports = {
  globDirectory: 'dist/',
  globPatterns: ['**/*.{html,css,js,json,ico,gif,jpg,jpeg,png,webp}'],
  swDest: 'dist/sw.js',
  // 禁用 Google Cloud CDN
  importWorkboxFrom: 'disabled',
  // 启用 第三方 CDN
  importScripts: [
    'https://cdn.jsdelivr.net/npm/workbox-sw@4.3.1/build/workbox-sw.min.js'
  ],
  // 首次打开页面后尽快接管网络请求
  clientsClaim: true,
  cleanupOutdatedCaches: true
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
  <!-- 老浏览器私有 meta 标签兼容 -->
  <script src="https://cdn.jsdelivr.net/npm/pwacompat@2.0.15/pwacompat.min.js"></script>
  <script>
    navigator.serviceWorker?.register('sw.js');
  </script>
</head>
```

## 工具链兼容性

### Parcel

虽然官方提供了 [webpack 和 Rollup 的插件][4]，但 **0 配置、一键化**的 [Parcel][5] 却还没有靠谱的相关插件。于是自己先踩了一遍坑，秘籍如下：

1. 在源码目录新建一个上述的 `sw.js`，内容留空，只为 Parcel 这类**面向资源的打包器**不报错

2. 对 `package.json` 中的构建脚本修改如下：

```json
{
  "scripts": {
    "start": "workbox generateSW  &&  parcel source/index.html --open",
    "pack-dist": "parcel build source/index.html --public-url .",
    "pack-sw": "rm -f dist/sw.js.map  &&  workbox generateSW",
    "build": "rm -rf dist/  &&  npm run pack-dist  &&  npm run pack-sw"
  }
}
```

### webpack

尽管 Parcel 没有 Workbox 官方插件，有一点要填的小坑，但还是比 webpack 配置起来简单。下面就来看一下我查了好多技术博文总结出的 webpack PWA 配置：

```shell
npm install webpack-pwa-manifest workbox-webpack-plugin -D
```

安装好上述 webpack 插件才能开始配置，而且需要注意的是，`webpack-pwa-manifest` 这个第三方插件对 [PWA Manifest 字段][6]的支持不完整，如遇报错就删除相应字段。

```javascript
// webpack.config.js
const { resolve } = require('path');

const PWA_Manifest = require('webpack-pwa-manifest'),
  Workbox = require('workbox-webpack-plugin');

const PWA_manifest = require('./src/manifest.json'),
  Workbox_config = require('./workbox.config');

for (const icon of PWA_manifest.icons) icon.src = resolve('src', icon.src);

module.exports = {
  plugins: [
    new PWA_Manifest(PWA_manifest),
    new WorkboxPlugin.GenerateSW(Workbox_config)
  ]
};
```

## 应用更新

PWA 核心技术 **Service Worker** 的强大之处在于，它**运行在 HTTP 协议与 Web 页面之间**，可**拦截、缓存**它所在 **URL scope 下的所有 HTTP 请求**。但 **Service Worker 只会在它管理的所有页面都关闭后才会更新服务器端的新版本**，若它缓存了 HTML 文件，用户刷新时拿到的依然是缓存的旧版本，HTML 引用的其它所有外置资源的 URL 也不会改变。

所以，我们需要一个**安全又用户友好的更新逻辑**，遂改 JS 入口模块如下：

```javascript
import { serviceWorkerUpdate } from 'web-utility';

const { serviceWorker } = window.navigator;

serviceWorker
  ?.register('/sw.js')
  .then(serviceWorkerUpdate)
  .then(worker => {
    if (window.confirm('检测到新版本，是否立即启用？'))
      // 触发 Workbox CLI 生成的 Service Worker 中监听的消息回调
      worker.postMessage({ type: 'SKIP_WAITING' });
  });

serviceWorker?.addEventListener('controllerchange', () =>
  window.location.reload()
);
```

---

好了，马上重新部署你的 Web 应用，在 Chrome、Opera、Edge、Firefox 上试试把它**添加到桌面**吧！

![](https://www.atyantik.com/wp-content/uploads/2017/10/PWA-States.png)

## 参考文档

- [《用 Workbox CLI 生成一个完整的 Service Worker》](https://developers.google.com/web/tools/workbox/guides/generate-service-worker/cli)
- [《Service Worker 生命周期》](https://developers.google.com/web/fundamentals/primers/service-workers/lifecycle)
- [《谨慎处理 Service Worker 的更新》](https://juejin.im/post/6844903792522035208)
- [《Service Worker 工作原理》](https://lavas-project.github.io/pwa-book/chapter04/3-service-worker-dive.html)

[1]: https://developers.google.cn/web/progressive-web-apps/
[2]: https://developers.google.cn/web/tools/workbox
[3]: https://developers.google.cn/web/tools/workbox/guides/generate-service-worker/cli
[4]: https://developers.google.cn/web/tools/workbox/guides/using-bundlers
[5]: https://parceljs.org/
[6]: https://developer.mozilla.org/zh-CN/docs/Web/Manifest
