---
title: 2021 年的 Web 标准进展
date: 2021-12-11 14:00:00
categories:
  - Development
tags:
  - Web
  - standard
  - component
  - form
  - API
slidehtml: true
---

> Google DevFest 2021 成都站

---

## Web 组件

> [前情提要](/programming/web-components-practise/slide.html)

---

### 服务端渲染

---

#### Template to Shadow DOM

```html
<host-element>
  <template shadowroot="open">
    <slot></slot>
  </template>
  <h2>Light content</h2>
</host-element>
```

---

```html
<host-element>
  #shadow-root (open)
  <slot>
    ↳
    <h2>Light content</h2>
  </slot>
</host-element>
```

---

#### Shadow DOM to Template

```javascript
const html = element.getInnerHTML({
  includeShadowRoots: true,
  closedRoots: [shadowRoot1, shadowRoot2]
});

console.log(html);
```

---

#### 组件初始化

```html
<menu-toggle>
  <template shadowroot="open">
    <button>
      <slot></slot>
    </button>
  </template>
  Open Menu
</menu-toggle>
```

---

```javascript
class MenuToggle extends HTMLElement {
  constructor() {
    var button = super().shadowRoot?.firstElementChild;

    if (!button) {
      const shadow = this.attachShadow({ mode: 'open' });
      shadow.innerHTML = `<button><slot></slot></button>`;
      button = shadow.firstChild;
    }
    button.addEventListener('click', toggle);
  }
}
customElements.define('menu-toggle', MenuToggle);
```

---

#### 私有 Shadow DOM

```javascript
class MenuToggle extends HTMLElement {
  constructor() {
    var { shadowRoot } = super().attachInternals();

    if (!shadowRoot) {
      shadowRoot = this.attachShadow({ mode: 'open' });
      shadowRoot.innerHTML = `<button><slot></slot></button>`;
    }
    shadowRoot.firstChild.addEventListener('click', toggle);
  }
}
customElements.define('menu-toggle', MenuToggle);
```

---

#### 仅限于解析器 —— 为了安全

```javascript
const div = document.createElement('div');
const template = document.createElement('template');

template.setAttribute('shadowroot', 'open'); // 啥也没做
div.appendChild(template);

console.log(div.shadowRoot); // null
```

---

```javascript
const html = `
    <div>
      <template shadowroot="open"></template>
    </div>
  `;
const div = document.createElement('div');
div.innerHTML = html;

console.log(div.shadowRoot); // null
```

```javascript
const document = new DOMParser().parseFromString(html, 'text/html', {
  includeShadowRoots: true
});
const { shadowRoot } = document.querySelector('div');

console.log(shadowRoot); // # DocumentFragment
```

---

#### CSS 渲染

```html
<nineties-button>
  <template shadowroot="open">
    <style>
      button {
        color: seagreen;
      }
    </style>
    <link rel="stylesheet" href="/comicsans.css" />
    <button>
      <slot></slot>
    </button>
  </template>
  I'm Blue
</nineties-button>
```

---

#### Polyfill 补丁

```javascript
for (const template of document.querySelectorAll('template[shadowroot]')) {
  const mode = template.getAttribute('shadowroot');
  const shadowRoot = template.parentNode.attachShadow({ mode });

  shadowRoot.append(template.content);
  template.remove();
}
```

---

#### 数据绑定（难产）

```html
<template>
  <section>
    <h1>{{name}}</h1>
    Email: <a href="mailto:{{email}}">{{email}}</a>
  </section>
</template>
```

```javascript
const template = document.currentScript.previousElementSibling;

const tree = template.createInstance({
  name: 'Ryosuke Niwa',
  email: 'rniwa@webkit.org'
});
document.body.prepend(tree);

tree.update({ name: 'Ryosuke Niwa', email: 'rniwa@apple.com' });
```

---

## 表单

---

### Event Submitter

https://github.com/idea2app/event-submitter-polyfill

---

#### HTML

```html
<form>
  <input name="data" />

  <button type="submit" data-name="first">Fisrt</button>
  <button type="submit" data-name="second">Second</button>
</form>
```

```javascript
document.querySelector('form')?.addEventListener('submit', event => {
  event.preventDefault();

  const { name } = event.submitter.dataset,
    { data } = event.target.elements;

  fetch(`/api/${name}`, { data: data.value });
});
```

---

#### React

```typescript
import React, { FormEvent } from 'react';
import { render } from 'react-dom';

function handleSubmit(event: FormEvent<HTMLFormElement>) {
  event.preventDefault();

  const { name } = event.nativeEvent.submitter.dataset,
    { data } = event.currentTarget.elements;

  fetch(`/api/${name}`, { data: data.value });
}

render(
  <form onSubmit={handleSubmit}>
    <input name="data" />

    <button type="submit" data-name="first">
      Fisrt
    </button>
    <button type="submit" data-name="second">
      Second
    </button>
  </form>,
  document.body
);
```

---

### FormData 事件

```javascript
const form = document.querySelector('form');

form?.addEventListener('formdata', ({ formData }) => {
  formData.append('my-input', myInputValue);
});
```

> 1. [标准提案作者博文译文](/programming/more-capable-form-controls/)
> 2. [polyfill 开发进展](https://github.com/webcomponents/polyfills/issues/172)

---

### Web 表单组件

---

#### Element Internals

```typescript
class MyCounter extends HTMLElement {
  static formAssociated = true;

  private value_ = 0;
  private internals_ = this.attachInternals();

  get value() {
    return this.value_;
  }
  set value(v) {
    this.value_ = v;
  }

  get form() {
    return this.internals_.form;
  }
  get name() {
    return this.getAttribute('name');
  }
  get type() {
    return this.localName;
  }
  get validity() {
    return this.internals_.validity;
  }
  get validationMessage() {
    return this.internals_.validationMessage;
  }
  get willValidate() {
    return this.internals_.willValidate;
  }

  checkValidity() {
    return this.internals_.checkValidity();
  }
  reportValidity() {
    return this.internals_.reportValidity();
  }
}
customElements.define('my-counter', MyCounter);
```

---

#### HTML

```html
<template>
  <div contenteditable="true"></div>
</template>
```

```javascript
const { content } = document.currentScript.previousElementSibling;

class MyInput extends HTMLInputElement {
  constructor() {
    super().attachShadow({ mode: 'open' }).append(content.cloneNode(true));
  }
}
customElements.define('my-input', MyInput, { extends: 'input' });
```

```html
<input is="my-input" />
```

---

#### WebCell

```typescript
import { component, mixinForm } from 'web-cell';

@component({
  tagName: 'my-counter',
  renderTarget: 'shadowRoot'
})
export class MyCounter extends mixinForm() {
  render() {
    return <div contenteditable="true" />;
  }
}
```
