---
title: Web 组件标准实践
date: 2019-08-14 22:33:23
updated: 2024-07-30 03:14:00
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
- freeCodeCamp 成都社区主理人
- 开源社理事、项目委员会主席
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
  method: function () {}
});

$.fn.myComponent = function (options) {
  this.each(function () {
    $.data(this, 'instance', new MyComponent(options));
  });
};
```

---

### 近代

```javascript
import { Component } from 'react';

export class MyComponent extends Component {
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
  <template shadowrootmode="open">
    <button><slot></slot></button>
  </template>
  按钮
</my-button>
```

---

#### 渐进增强，优雅降级

```html
<!-- 良 -->
<my-button>
  <template shadowrootmode="open">
    <slot></slot>
  </template>
  <button>按钮</button>
</my-button>

<!-- 优 -->
<button is="my-button">
  <template shadowrootmode="open">
    <slot></slot>
  </template>
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

### 声明式组件

<iframe
    src="https://codesandbox.io/p/devbox/9gyll?embed=1&file=%2Fsrc%2FClock.tsx"
    style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
    title="WebCell scaffold"
></iframe>

---

### 官方适配 MobX

```javascript
import { observer } from 'web-cell';

import { app } from '../model';

export const PageIndex = observer(() => (
  <div onClick={app.increase}>count: {app.count}</div>
));
```

---

```javascript
import { component, observer } from 'web-cell';

import { app } from '../model';

@component({ tagName: 'page-index' })
@observer
export class PageIndex extends HTMLElement {
  render() {
    return <div onClick={app.increase}>count: {app.count}</div>;
  }
}
```

---

### 极简路由 —— Cell Router

> 路径即状态，容器即组件

---

```javascript
import { DOMRenderer } from 'dom-renderer';
import { FC } from 'web-cell';
import { createRouter, PageProps } from 'cell-router';

const { Route, Link } = createRouter();

const TestPage: FC<PageProps> = ({
    className,
    style,
    path,
    history,
    ...data
}) => (
    <ul {...{ className, style }}>
        <li>Path: {path}</li>
        <li>Data: {JSON.stringify(data)}</li>
    </ul>
);

new DOMRenderer().render(
    <>
        <nav>
            <Link to="test?a=1">Test</Link>
            <Link to="example/2">Example</Link>
        </nav>
        <main className="router">
            <Route
                path=""
                component={props => <div {...props}>Home Page</div>}
            />
            <Route path="test" component={TestPage} />
            <Route path="example/:id" component={TestPage} />
        </main>
    </>
);
```

---

#### 异步加载页面

`tsconfig.json`

```json
{
  "compilerOptions": {
    "module": "ESNext"
  }
}
```

`source/page/index.ts`

```javascript
import { lazy } from 'web-cell';

export default [
  {
    path: '',
    component: lazy(() => import('./Home'))
  },
  {
    path: 'list',
    component: lazy(() => import('./List'))
  }
];
```

---

### 开箱即用

```bash
npm init -y

npm i dom-renderer web-cell mobx cell-router web-utility boot-cell

npm i parcel -D
```

---

### 官方组件库 —— BootCell

<iframe
    src="https://bootstrap.web-cell.dev/"
    style="width: 100%; height: 35rem"
></iframe>

---

### 推荐组件库 —— MDUI

<iframe
    src="https://www.mdui.org/zh-cn/"
    style="width: 100%; height: 35rem"
></iframe>

---

### 竞争对手

- Google Polymer/Lit

- Ionic Stencil

- Tencent Omi

---

![](/image/WebCell-1.png)

https://web-cell.dev/
