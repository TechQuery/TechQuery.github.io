---
title: PWA 即刻上手
date: 2019-10-31 22:41:33
updated: 2020-08-10 14:58:09
categories:
  - Development
tags:
  - PWA
  - Web
---

近两年国内前端界很流行*小程序*，但大多数人都不知道小程序很多概念是鹅厂“借鉴”自 Google 主导的 [Progressive Web App（渐进式网页应用）标准][1] ——

1. **“独立”运行**：在“系统正在运行的应用”列表中独立显示
2. **本地缓存**：页面秒开、离线运行
3. **安装到主屏幕**：桌面图标、启动首屏、全屏显示等，看起来和原生 App 别无二致

以上特性已经很香了，更甭说开放的 Web 还有[更丰富的新标准][2]来拓展 Web App 的疆域了~

那这么香的新标准怎么上手呢？会像小程序开发环境那样难用吗？我们老规矩 —— **抄起键盘就是一把梭！**

## 应用元数据

一个**渐进式 Web 应用**首先要声明一下自己的基本信息：

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

一个 PWA 能真正安装到用户系统中，还需要一个 **Service Worker**。但它的工作原理对初学者直接手写有些困难，亲爹 Google 已经做好了开发框架 [Workbox][3]，还配了 [Workbox CLI][4]，几行命令就能生成一个基本的 ServiceWorker，简直不要太贴心：

```shell
npm install workbox-cli --global

cd your_project/

npm run build  # 先执行你项目的构建脚本，生成生产环境目录

workbox wizard
```

跟着向导执行完上述命令，会生成一个名为 `workbox-config.js` 的配置文件，但须稍加修改来适应*天朝国情*：

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

最后在 `index.html` 里注册一下即可：

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

**0 配置、一键化**的 [Parcel][5] 原生支持 **Web Manifest**，不像 webpack 和 Rollup，还需要配置插件，只需将 `package.json` 中的构建脚本修改如下：

```json
{
  "scripts": {
    "start": "workbox generateSW  &&  parcel source/index.html --open",
    "pack": "parcel build source/index.html --public-url .",
    "build": "rm -rf dist/  &&  npm run pack-dist  &&  workbox generateSW"
  }
}
```

每次构建运行 `npm run build` 即可生成可缓存整个 Web App 的 Service Worker 代码。

### webpack

因为我不是 _webpack 配置工程师_，所以我查了好多技术博文才总结出一套最简单的 webpack PWA 配置：

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

对比之后大家会发现，Parcel 这种**面向资源的打包器**自动根据**各种语言的标准资源路径语法**查找依赖项，用应用开发者“想当然”的思路搜集各种资源文件，既不需要遵守尚未成为标准的“约定”，也不需要研究复杂的“配置”，全凭“肌肉记忆”撸代码，轻快舒爽~

## 应用更新

PWA 核心技术 **Service Worker** 的强大之处在于，它**运行在 HTTP 协议与 Web 页面之间**，可**拦截、缓存**它所在 **URL scope 下的所有 HTTP 请求**。但 **Service Worker 只会在它管理的所有页面都关闭后才会更新服务器端的新版本**，若它缓存了 HTML 文件，用户刷新时拿到的依然是缓存的旧版本，HTML 引用的其它所有外置资源的 URL 也不会改变。

所以，我们需要一个**安全又用户友好的更新逻辑**，遂改 JS 入口模块如下：

```javascript
import { serviceWorkerUpdate } from 'web-utility';

const { serviceWorker } = window.navigator;

serviceWorker
  ?.register('sw.js')
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

## 范例项目

1. [WebCell 项目脚手架](https://github.com/EasyWebApp/scaffold)
2. [WebCell 框架官网](https://web-cell.dev/)
3. [成都 Web 开发者大会官网](https://web-conf.dev/)

## 参考文档

- [《用 Workbox CLI 生成一个完整的 Service Worker》](https://developers.google.cn/web/tools/workbox/guides/generate-service-worker/cli)
- [《Service Worker 生命周期》](https://developers.google.cn/web/fundamentals/primers/service-workers/lifecycle)
- [《谨慎处理 Service Worker 的更新》](https://juejin.im/post/6844903792522035208)
- [《Service Worker 工作原理》](https://lavas-project.github.io/pwa-book/chapter04/3-service-worker-dive.html)

[1]: https://developers.google.cn/web/progressive-web-apps/
[2]: /development/web-chrome-update-at-google-io-2019/slide.html
[3]: https://developers.google.cn/web/tools/workbox
[4]: https://developers.google.cn/web/tools/workbox/guides/generate-service-worker/cli
[5]: https://parceljs.org/
[6]: https://developer.mozilla.org/zh-CN/docs/Web/Manifest
