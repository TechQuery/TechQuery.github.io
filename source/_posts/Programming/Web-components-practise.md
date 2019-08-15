---
title: Web 组件标准实践
date: 2019-08-14 22:33:23
categories:
  - Programming
tags:
  - Web
  - component
  - WebCell
  - W3C

slidehtml: true
---

> 水歌 —— WebCell 引擎作者

---

## 水歌其人

- Web/JavaScript 全栈开发者
- WebCell 等多个开源项目的作者
- jQuery、Babel 等多个国际开源项目的贡献者
- freeCodeCamp 成都社区负责人
- 开源社官网开发组长、执委会成员
- 微软 MVP

---

## Web 组件的概念

> 将网页中某块界面的 **HTML 结构**、**CSS 样式**、**JS 逻辑**，封装成一个**可移植的模块**

---

## Web 组件技术简史

---

### 上古时代

- W3C: `<frameset />`、`<iframe />`

- IE: HTML Component (HTC)

---

### 中古时代

```javascript
function MyComponent(options) {
  this.options = options;
}

$.extend(MyComponent.prototype, {
  method: function() {}
});

$.fn.myComponent = function(options) {
  this.each(function() {
    $.data(this, 'instance', new MyComponent(options));
  });
};
```

---

### 近代

```javascript
import React from 'react';

export default class MyComponent extends React.Component {
  render() {
    return <h1>Hello, World!</h1>;
  }
}
```

---

### 历史问题

- **隔离性**好的，**运行时**过重

- **易用性**好的，**工程化**不佳

- **实用性**强的，**工具链**过重

---

## Web 组件标准

https://www.webcomponents.org/

---

### 标准集

- HTML 5.3
- DOM 4.1
- CSS Variables level 1
- ECMAScript 6+

---

### 主要目标

- 运行时 轻量级 隔离环境

- 框架无关的 DOM 接口

---

### 原生写法

<iframe
    height="600" style="width: 100%"
    frameborder="no" scrolling="no" allowtransparency="true" allowfullscreen="true"
    src="https://codepen.io/tech_query/embed/jONqOzj/?height=600&theme-id=31315&default-tab=html,result"
>
</iframe>

---

#### 样式定制

```css
my-component {
  --my-color: blue;
}
```

```css
:host {
}
:host(.class) {
}
:host-context(.class) {
}

::slotted(*) {
  color: var(--my-color);
}
```

---

#### 生命周期

```javascript
class MyComponent extends HTMLElement {
  static get observedAttributes() {
    return ['title'];
  }
  attributeChangedCallback(attrName, oldVal, newVal) {}

  connectedCallback() {}
  disconnectedCallback() {}

  adoptedCallback() {}
}
```

---

#### 扩展原生

```javascript
customElements.define('my-button', class extends HTMLButtonElement {}, {
  extends: 'button'
});
```

```html
<button is="my-button">按钮</button>
```

---

### 规范模式

#### 不要接管一切

```html
<my-button>
  <!-- shadow-root -->
  <button><slot></slot></button>
  <!-- shadow-root -->
  按钮
</my-button>
```

---

#### 渐进增强，优雅降级

```html
<!-- 良 -->
<my-button>
  <!-- shadow-root -->
  <slot></slot>
  <!-- shadow-root -->
  <button>按钮</button>
</my-button>

<!-- 优 -->
<button is="my-button">
  <!-- shadow-root -->
  <slot></slot>
  <!-- shadow-root -->
  按钮
</button>
```

---

### 兼容性

#### 原生

- https://caniuse.com/#feat=custom-elementsv1

- https://caniuse.com/#feat=shadowdomv1

- https://caniuse.com/#feat=css-variables

- https://caniuse.com/#feat=es6-class

#### polyfill

IE 11 +

---

### 工程问题

- 数据绑定

- 资源打包

- 路由定义

- 状态管理

---

## WebCell

> 优雅、轻量的 Web 组件引擎

---

### 极简模板

<iframe
    height="600" style="width: 100%"
    frameborder="no" scrolling="no" allowtransparency="true" allowfullscreen="true"
    src="https://codepen.io/tech_query/embed/WNewpYG/?height=600&theme-id=31315&default-tab=html,result"
>
</iframe>

---

### 一键打包

```javascript
import template from './index.html'; // String

import style from './index.css'; // String

import data from './index.json'; // Object

import icon from './icon.svg'; // Data URI
```

```shell
web-cell pack
```

---

### 声明式组件

```javascript
@component({ template, style, data })
export default class MyComponent extends HTMLElement {
  constructor() {
    super().construct();
  }

  @blobURI
  static get icon() {
    return icon;
  }

  @on('input', ':host input')
  @debounce()
  countLength(event, target) {}

  @at(':host *')
  onAny(event, target) {}
}
```

---

### 扩展生命周期

```javascript
@component()
export default class MyComponent extends HTMLElement {
  @mapProperty
  static get observedAttributes() {
    return ['value', 'name'];
  }

  @mapData
  attributeChangedCallback() {}

  slotChangedCallback(assigned, slot, name) {}

  viewUpdateCallback(newData, oldData, view) {}

  viewChangedCallback(data, view) {}
}
```

---

### 路由即组件

```html
<body>
  <a href="/index">Index</a>
  <a href="/secret">Secret</a>

  <app-router></app-router>
</body>
```

---

```javascript
import App from '../scheme/App';

@component({ store: App })
export default class AppRouter extends HTMLRouter {
  @load('/index')
  indexPage() {
    return '<page-index />';
  }

  @load('/secret/:id')
  secretPage(id) {
    return `<h1>Secret ${id}</h1>`;
  }

  @back('/secret')
  keepSecret() {
    return !!this.store.user;
  }
}
```

---

### 独立数据模型

```javascript
import Model, { mapGetter, is } from 'data-scheme';

import User from './User';

@mapGetter
export default class App extends Model {
  @is(User)
  set user(value) {
    this.set('user', value);
  }
}
```

---

```javascript
@mapGetter
export default class User extends Model {
  @is(/^[\w-]{3,20}$/, '')
  set name(value) {
    this.set('name', value);
  }

  @is(Email, '')
  set email(value) {
    this.set('email', value);
  }

  @is(Phone)
  set phone(value) {
    this.set('phone', value);
  }

  @is([0, 1, 2], 2)
  set gender(value) {
    this.set('gender', value);
  }

  @is(Range(1900))
  set birthYear(value) {
    this.set('birthYear', value);
  }

  @is(URL, 'http://example.com/test.jpg')
  set avatar(value) {
    this.set('avatar', value);
  }

  @is(URL)
  set URL(value) {
    this.set('URL', value);
  }

  @is(String)
  set description(value) {
    this.set('description', value);
  }
}
```

---

### 立即体验

```shell
npm init web-cell ~/Desktop/web-cell-demo
```

---

### 官方组件库

- [Cell Common](https://web-cell.dev/cell-common/)

- [GitHub Web widget](https://tech-query.me/GitHub-Web-Widget/)

- [Material Cell](https://github.com/EasyWebApp/material-cell)

---

### 未来计划

- 从 ECMAScript 6+ 迁移至 TypeScript 3+

- 开发 MobX 适配器

- 支持**嵌套路由**

- 基于 Rollup、Parcel 2 重构工具链

- 在线体验环境

---

### 竞争对手

Google Polymer

Tencent Omi

---

![](/image/WebCell-1.png)

https://web-cell.dev/
