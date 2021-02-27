---
title: 让 Web 打包器再次简单！
date: 2021-02-28 00:11:35
categories:
  - Development
tags:
  - Web
  - bundle
  - easy
slidehtml: true
---

> 华为云开发者社区<br>
> 2021 年 3 月 27 日演讲

---

## Web 最初很简单

- HTTP
- HTML
- CSS
- JavaScript

---

<iframe
    style="width: 100%; height: 90vh"
    scrolling="no" frameborder="no" loading="lazy"
    allowtransparency="true" allowfullscreen="true"
    title="Web standard - Resource import"
    src="https://codepen.io/tech_query/embed/MWbVwWv?height=265&theme-id=dark&default-tab=html,result"
></iframe>

---

## Web 后来很强大

- BOM、DOM
- LESS、SASS、Stylus
- ECMAScript 6+、TypeScript
- WebAssembly
- Canvas、WebGL
- WebSocket
- IndexedDB
- Web Worker、PWA
- WebUSB、WebAuthn
- ……

---

### Web 打包器变得很复杂

---

#### JavaScript 库打包器

|                <small>名称及官网</small>                |             <small>简介</small>              |             <small>备注</small>             |
| :-----------------------------------------------------: | :------------------------------------------: | :-----------------------------------------: |
|  [AMD-bundle.js](https://tech-query.me/AMD_bundle.js/)  | <small>AMD、CJS 和 ES 通用模块打包器</small> | <small>本人开发 jQuery 仿制版时自研</small> |
|           [rollup.js](https://rollupjs.org/)            |         <small>ES 模块打包器</small>         | <small>全球首个知名的 ES 模块打包器</small> |
| [MicroBundle](https://github.com/developit/microbundle) |   <small>基于 rollup 的 JS 打包器</small>    |   <small>Preact 等知名库的打包器</small>    |
|          [ESBuild](https://esbuild.github.io/)          | <small>用 Go 语言开发的 ES/TS 打包器</small> |         <small>全球性能最强</small>         |

---

#### Web 应用打包器

|       <small>名称及官网</small>        |                    <small>简介</small>                     |              <small>备注</small>              |
| :------------------------------------: | :--------------------------------------------------------: | :-------------------------------------------: |
| [DevCLI](https://web-cell.dev/DevCLI/) | <small>HTML 5、CSS 3、ECMAScript 6+ 原生资源打包器</small> |  <small>本人开发 WebCell 1.0 时自研</small>   |
|   [webpack](https://webpack.js.org/)   |            <small>插件化 Web 资源打包器</small>            |   <small>全球最著名、最复杂的打包器</small>   |
|    [Parcel](https://parceljs.org/)     |          <small>零配置 Web 原生资源打包器</small>          |   <small>全球次著名、最易用的打包器</small>   |
| [Snowpack](https://www.snowpack.dev/)  |       <small>基于 ESBuild 的 Web 资源打包器</small>        | <small>全球首个知名 ES 模块应用打包器</small> |
|       [Vite](http://vitejs.dev/)       |  <small>基于 rollup 和 ESBuild 的 Web 资源打包器</small>   |   <small>尤雨溪开发 Vue 3.0 时自研</small>    |

---

#### 痛点：配置复杂！！！

1. 标准滞后
2. 实现繁多
3. 配置复杂
4. 迁移困难

---

代码还没写，环境配一天……

---

## 我理想中的打包器

| 本人自研（已废弃） | 社区同款 |                社区现状                |
| :----------------: | :------: | :------------------------------------: |
|   AMD-bundle.js    | ESBuild  | CSS modules 官方尚未支持，但有社区插件 |
|       DevCLI       |  Parcel  |    v1 bug 无人修复，v2 变复杂且难产    |

---

### 设计原则

- **资源文件引入机制**只依赖 **HTML、CSS、JavaScript 原生语法**
- **资源文件扩展名**支持：
  - MarkDown、LESS、SASS、TypeScript 等**高级编译语言**
  - JSON、YAML、CSV 等**文本数据格式**
  - 图片、音视频等**二进制文件的 URI**

---

- 根据依赖树的需要，自动安装**资源编译器**
- 自动读取项目现有的各种**编译器配置文件**
- **打包配置**按社区习惯设置**最优默认值**
- 配置文件支持 [JSON schema](https://json-schema.org/)、**ES 模块**和 **TypeScript 类型定义**

---

### 架构概略

1. 编译器转换各类代码文件
2. 编译器生成**抽象语法树**，并查找**资源引用语句**
3. 构造依赖树
4. 基于观察者模型**增量更新依赖**

---

## 我有一个梦想

---

Web standard first!

---

Make the Web bundler easy again!
