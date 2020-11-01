---
title: 为何新手可用 TypeScript？
date: 2020-08-11 00:06:54
updated: 2020-11-2 00:46:07
categories:
  - Development
tags:
  - TypeScript
  - ECMAScript
---

[TypeScript 官网][1]开宗明义 ——

> TypeScript 是适用于任何规模应用的 JavaScript

其中奥妙，且看我一个边干边学的 JS 码农娓娓道来~

## 安装简单

因为 TypeScript 官方支持 [ECMAScript 最新标准][2]和[进入 Stage 3 的提案][3]，绝大多数项目的代码可以直接编译，polyfill 和运行时工具函数全内置，不像 Babel 那样需要自己找**运行时补丁库**或**提案编译插件**。

### TypeScript

```shell
npm install typescript -D
```

### Babel

```shell
npm install @babel/core @babel/preset-env @babel/cli -D
npm install @babel/polyfill
```

```javascript
import '@babel/polyfill';
```

## 配置方便

### TypeScript

TypeScript 默认不需要配置，在写 `.ts(x)` 文件的过程中，TS 编译器会在缺少配置的地方提示你需要添加什么配置，此时在项目根目录新建一个 `tsconfig.json` 文件，支持 **TS 语言服务协议**的代码编辑器还能提示配置文件的选项，全程按指导操作即可，官方文档都很少需要看~

### Babel

Babel 虽然编译标准语法只需一个 preset 包，但更多的**常用提案语法**、**框架语法糖**就需要一大堆插件了，而且所有配置选项都没有代码提示，必须各种查文档，有时文档也不甚详细，容易踩坑……

```json
{
  "presets": ["@babel/preset-env"]
}
```

## 类型可选

TypeScript 的核心卖点是**类型系统**，这让很多从 JavaScript、PHP、Python 等**脚本语言**入门编程的程序员非常畏惧，认为是非常大的**学习成本**。

但 TS 的类型系统与 C/C++、Java、C# 等传统**强类型语言**从设计理念上非常不同 ——

> TS 代码不需要*处处声明类型*，而只需程序员在编译器推导无效的少许地方**标注类型**

而 TS **类型推导**的基础数据则是 JS、DOM、Node.js 等标准/技术中的 **API 类型定义**，编译器循着**语法结构**的上下文，自然能推导出大多数变/常量的类型，以便帮程序员**检查潜在错误**、**显示代码提示**。

## 开发轻松

### Node.js 运行时

虽然 Node.js 创始人又另起炉灶搞了第一个**全功能 TypeScript 运行时环境** [Deno][4]，但并不意味着 Node.js 不能方便地运行 TS 代码，因为我们有 [`ts-node`][5] ——

```shell
// 全局安装运行时
npm install ts-node -g
// 执行一个 TS 脚本
ts-node index.ts
```

### Parcel 零配置打包

如果你用 [Parcel][6] 这种**零配置打包器**开发 Web 前端项目，你甚至完全不用安装包括 TypeScript、LESS、SCSS 在内的任何常见**预编译语言**的编译器。在项目首次启动时，Parcel 会自动帮你安装好项目依赖的所有代码文件类型对应的编译器！你只需在启动项目时装好 Parcel，然后就按 **HTML、CSS、JS 原生语法**写逻辑、引用外置资源即可 ——

```shell
npm install parcel-bundler -D
```

## 总结：TS，真香！

我的 JavaScript 生涯从 ES 3 入门，后来用 ES 5 结合 JSDoc 开发小框架，再到 ES 6+ 配合 ESDoc 开发 [WebCell v1][7]，最终用 TS + TSDoc 重写出 [WebCell v2][8]。在充分利用 [JavaScript 这门“无所不能”的语言][9]强大灵活性的同时，**更简洁的 ECMAScript 新语法**和**更规范的 TypeScript 类型系统**，为开源库和日常项目开发带来突飞猛进的效率提升 —— TS，真香！

## 彩蛋

再分享一个 TypeScript **JSX 类型推导**的新玩法：

> [CommanderJSX][10] —— 用 JSX 声明**命令行参数**，并支持**回调传参类型推导**

[1]: https://www.typescriptlang.org/zh/
[2]: https://tc39.es/ecma262/
[3]: https://jscig.github.io/#proposals?stage=3
[4]: https://www.denojs.cn/
[5]: https://github.com/TypeStrong/ts-node
[6]: https://zh.parceljs.org/
[7]: https://github.com/EasyWebApp/WebCell/tree/master
[8]: https://web-cell.dev/
[9]: /programming/javascript-in-one-word/
[10]: https://github.com/TechQuery/CommanderJSX
