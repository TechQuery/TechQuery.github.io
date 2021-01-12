---
title: 【译】更强大的表单控件
date: 2020-12-19 13:30:48
categories:
  - Programming
tags:
  - Web
  - component
  - form
  - standard
  - API
img: https://webdev.imgix.net/more-capable-form-controls/hero.jpg?auto=format&fit=max&w=1959
---

> 原文：https://web.dev/more-capable-form-controls/

有了一个新事件，再加上一些自定义元素 API，表单的使用变得更加容易。

---

很多开发者构建自定义表单元素，提供浏览器未内置的控件，或自定义超越内置表单控件的外观与体验。

然而，复制 HTML 内置表单控件的特性很难。想一下当你把一个 `<input>` 元素加进一个表单时，它自动获得的一些特性：

- 输入框自动加进表单的控件列表
- 输入框中的值自动与表单一起提交
- 输入框参与[表单校验][1]，你可以用 `:valid` 和 `:invalid` 伪类为输入框设置样式
- 表单重置、重加载时，或浏览器尝试自动填充表单项时，输入框都会被通知

自定义表单组件通常没有这些特性。开发者能用 JavaScript 解决一些限制，比如向一个表单添加一个隐藏的 `<input>` 以参与表单提交。但其它特性无法单单用 JavaScript 复制。

两个 Web 新特性让构建自定义表单控件更简单，解决了现有自定义控件的限制：

- `formdata` 事件让一个任意的 JavaScript 对象参与到表单提交中，所以你可以无需一个隐藏的 `<input>` 就能添加表单数据
- **Form-associated Custom Elements API** 让自定义元素表现得更像内置表单控件

这两个特性可用于创建效果更好的新型控件。

> 构建自定义表单组件是个高级话题。本文假定您对表单和表单控件有一定的了解。当构建一个自定义表单控件时，有很多因素要考虑，特别是确定你的控件是对所有用户无障碍的。学习更多表单知识，前往 [MDN 表单指南][2].

## 基于事件的 API

`formdata` 事件是一个让任何 JavaScript 代码参与表单提交的底层 API。该机制是这样的：

1.  你添加一个 `formdata` 事件监听器到你想交互的表单
2.  当一个用户点击提交按钮，该表单触发一个 `formdata` 事件，它包含一个持有所有待提交数据的 [`FormData`][3] 对象
3.  每个 `formdata` 监听器都有机会在表单提交前添加或修改数据

这里是一个在 `formdata` 事件监听器中发送一个单值的例子：

```javascript
const form = document.querySelector('form');
// FormData 事件在 <form> 提交时、传输前触发
// 该事件有个 formData 属性
form.addEventListener('formdata', ({ formData }) => {
  // https://developer.mozilla.org/zh-CN/docs/Web/API/FormData
  formData.append('my-input', myInputValue);
});
```

用我们在 Glitch 的示例来尝试。务必在 Chrome 77 或更高版本上运行，以查看该 API 的运行情况。

<iframe
  src="https://glitch.com/embed/#!/embed/formdata-event?path=README.md&previewSize=0"
  title="formdata-event on Glitch"
  allow="geolocation; microphone; camera; midi; vr; encrypted-media"
  style="height: 90vh; width: 100%; border: 0;">
</iframe>

## 表单关联的自定义元素

你可以把基于事件的 API 用于任何类型的组件，但它只允许你与提交过程交互。

标准化的表单控件除了提交外，还参与了表单生命周期的许多部分。表单关联的自定义元素旨在填补自定义控件和内置控件的鸿沟，并匹配了很多标准表单元素的特性：

- 当你把一个表单关联的自定义元素放进一个 `<form>`，它就像一个浏览器提供的控件，自动与该表单关联。
- 该元素可被一个 `<label>` 元素标记
- 该元素可设置一个与表单一起自动提交的值
- 该元素可设置一个标记，指示它是否取得有效输入。如果其中一个表单元素有无效输入，该表单则不能被提交。
- 该元素可提供一些用于表单生命周期多个部分的回调 —— 比如当该表单被禁用或重置到它的默认状态
- 该元素支持标准的 CSS 表单控件伪类，如 `:disabled` 和 `:invalid`

这么多功能！本文不会涉及所有内容，但将阐述把自定义元素与表单集成所需的基础知识。

本节假设你对自定义元素有基本的了解。有关自定义元素的介绍，参见 Web Fundamentals 上的[《Custom Elements v1：可复用的 Web 组件》][4]。

### 定义一个表单关联的自定义元素

把一个自定义元素转变成一个表单关联的自定义元素，需要几个额外步骤:

- 添加一个静态 `formAssociated` 属性到你的自定义元素类，这告诉浏览器把该元素看作一个表单控件
- 在该元素上调用 `attachInternals()` 方法，以访问表单控件的其它方法和属性，如 `setFormValue()` 和 `setValidity()`
- 添加表单控件支持的通用属性和方法，如 `name`、`value` 和 `validity`

来看看如何把这些项目融入一个基础的自定义元素定义：

```javascript
// 表单关联的自定义元素必须是独立的自定义元素 ——
// 意味着它们必须继承自 HTMLElement，而非它的一个子类。
class MyCounter extends HTMLElement {
  // 把该元素标记为一个表单关联的自定义元素
  static formAssociated = true;

  constructor() {
    super();
    // 获得访问内部表单控件 API 的能力
    this.internals_ = this.attachInternals();
    // 该控件的内部值
    this.value_ = 0;
  }
  // 表单控件通常暴露一个“value”属性
  get value() {
    return this.value_;
  }
  set value(v) {
    this.value_ = v;
  }
  // 以下属性和方法并非必须，但浏览器级表单控件提供了它们。
  // 提供它们有助于确保与浏览器提供的控件保持一致。
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
  // ...
}

customElements.define('my-counter', MyCounter);
```

一旦注册，你就可以在你使用浏览器表单控件的任何地方使用这个控件：

```html
<form>
  <label>兔子的数量: <my-counter></my-counter></label>
  <button type="submit">提交</button>
</form>
```

### 设置一个值

`attachInternals()` 方法返回一个可访问表单控件 API 的 `ElementInternals` 对象。这些 API 中最基础的是 `setFormValue()` 方法，用来设置该控件的当前值。`setFormValue()` 方法可接受三种类型之一的值：

- 一个字符串值
- 一个 [`File`][5] 对象
- 一个 [`FormData`][3] 对象，你可以用一个 `FormData` 对象来传多值（例如一个信用卡输入控件可能会传一个卡号、过期日期和验证码）

设置一个简单值：

```javascript
this.internals_.setFormValue(this.value_);
```

设置多值，你可以执行以下操作：

```javascript
// 用该控件名作为提交数据的基础名
const n = this.getAttribute('name');
const entries = new FormData();

entries.append(n + '-first-name', this.firstName_);
entries.append(n + '-last-name', this.lastName_);
this.internals_.setFormValue(entries);
```

> `setFormValue()` 方法接受一个可选的第二参数 `state`，用来存储该控件的内部状态。
> 详情参见本文「恢复表单状态」一节

### 输入校验

你的控件也可以通过调用内部对象上的 `setValidity()` 方法参与表单校验。

```javascript
// 假设在内部值更新时调用此方法
onUpdateValue() {
    if (
        !this.matches(':disabled') &&
        this.hasAttribute('required') &&
        this.value_ < 0
    ) {
        this.internals_.setValidity(
            { customError: true }, 'Value cannot be negative.'
        );
    } else {
        this.internals_.setValidity({});
    }
    this.internals.setFormValue(this.value_);
}
```

你可以用 `:valid` 和 `:invalid` 伪类给一个表单关联的自定义元素设置样式，就像一个内置表单控件一样。

> 尽管你可以设置一个校验消息，但 Chrome 目前不能显示表单关联自定义元素的校验消息。

### 生命周期回调

Form-associated Custom Elements API 包含一组额外的生命周期回调，以绑定到表单生命周期。

这些回调是可选的：仅在你的元素需要在生命周期的某一刻做某事时才实现一个回调。

#### `void formAssociatedCallback(form)`

当浏览器将一个表单元素关联到该元素时调用，或从一个表单元素解除关联该元素时。

#### `void formDisabledCallback(disabled)`

该元素的 `disabled` 状态改变后，无论是因为该元素的 `disabled` 属性被添加或移除；还是因为该元素的一个祖先 `<fieldset>` 的 `disabled` 状态被改变。`disabled` 参数代表该元素的新[禁用状态][6]。例如，当该元素被禁用时，它可能要禁用它影子 DOM 中的元素。

#### `void formResetCallback()`

该表单被重置后调用。该元素应该将其自身重置回某种默认状态。对于 `<input />` 元素，这通常涉及设置 `value` 属性以匹配标签中设置的 `value` 属性（或在复选框的案例中，设置 `checked` 对象属性以匹配 `checked` 标签属性）。

#### `void formStateRestoreCallback(state, mode)`

在两种情形之一被调用：

- 当浏览器恢复该元素状态时（例如，一次导航后，或当浏览器重启时），在此情形下 `mode` 参数是 `"restore"`

- 当浏览器的输入助手特性（诸如表单自动填充）设置一个值时，在此情形下 `mode` 参数是 `"autocomplete"`

第一个参数的类型则取决于 `setFormValue()` 方法被怎样调用。详情参见本文「恢复表单状态」一节。

### 恢复表单状态

在某些情形下 —— 如当导航返回一个页面时，或重启浏览器时，浏览器可能尝试恢复该表单到用户保留的状态。

对于一个表单关联的自定义元素，被恢复的状态来自你传到 `setFormValue()` 方法的值。就像早前示例中展示的那样，你可以用一个单值参数或两个参数调用该方法：

```javascript
this.internals_.setFormValue(value, state);
```

此处的 `value` 代表该控件的可提交参数。此处的可选 `state` 参数是一个该控件状态的 _内部_ 表示，可包含不发送到服务器的数据。该 `state` 参数具有与 `value` 参数相同的类型 —— 它可以是一个字符串、`File` 或 `FormData` 对象。

`state` 参数在你无法只基于值去恢复一个控件状态时很有用。例如，假设你创建了一个多模的颜色选择器：一个调色板或 RGB 色轮。可提交的 _值_ 会是规范形式的选中颜色，如 `"#7fff00"`。但恢复该控件到一个特定状态，你也需要知道它之前处在哪种模式，所以 _状态_ 可能形如 `"palette/#7fff00"`。

```javascript
this.internals_.setFormValue(this.value_, this.mode_ + '/' + this.value_);
```

你的代码可能需要基于存储的状态值来恢复自身状态。

```javascript
formStateRestoreCallback(state, mode) {
    if (mode == 'restore') {
        // 预期一个形如 'controlMode/value' 的状态参数
        [controlMode, value]= state.split('/');
        this.mode_ = controlMode;
        this.value_ = value;
    }
    // Chrome 目前不处理表单关联自定义元素的自动填充。
    // 在自动填充案例中，你可能需要处理一个原始值。
}
```

在一个更简单的控件案例中（例如一个数字输入框），值可能足够该控件恢复之前的状态。当调用 `setFormValue()` 时，若你忽略 `state` ，值会被直接传给 `formStateRestoreCallback()`。

```javascript
formStateRestoreCallback(state, mode) {
    // Simple case, restore the saved value
    this.value_ = state;
}
```

### 一个实际的例子

以下示例整合了表单关联自定义元素的很多特性。务必在 Chrome 77 或更高版本运行它，以查看 API 的运行情况。

<iframe
    src="https://glitch.com/embed/#!/embed/form-associated-ce?path=README.md&previewSize=0"
    title="form-associated-ce on Glitch"
    allow="geolocation; microphone; camera; midi; vr; encrypted-media"
    style="height: 90vh; width: 100%; border: 0;">
</iframe>

## 特性检测

你可以用特性检测去确定 `formdata` 事件和表单关联的自定义元素是可用的。目前每个特性都没有补丁发布。针对这两种情况，你可以回退到添加一个隐藏的表单元素来把该控件值传给表单。很多更高级的表单关联自定义元素特性将可能很难或无法打补丁。

```javascript
if ('FormDataEvent' in window) {
  // 支持 formdata 事件
}
if (
  'ElementInternals' in window &&
  'setFormValue' in window.ElementInternals.prototype
) {
  // 支持表单关联的自定义元素
}
```

> **译注**：原文发表一季度后，社区开发者发布了一个 [ElementInternals polyfill][7]，而 Web Components 标准团队官方 polyfill 也在[计划实现本文所述特性][8]。

## 结论

`formdata` 事件和表单关联自定义元素为创建自定义表单控件提供了新工具。

`formdata` 事件没有给你任何新能力，但它给你一个接口，让你无须一个隐藏的 `<input />` 元素，即可添加表单数据到提交流程。

Form-associated Custom Elements API 通过一组新能力让自定义表单控件像内置表单控件一样工作。

---

## 译者后记

Custom Elements v1 标准刚发布时，自定义表单元素的构建可以通过**扩展原生标签**特性来实现：

```html
<template>
  <div contenteditable="true"></div>
</template>

<script>
  const { content } = document.currentScript.previousElementSibling;

  class MyInput extends HTMLInputElement {
    constructor() {
      super().attachShadow({ mode: 'open' }).append(content.cloneNode(true));
    }
  }
  customElements.define('my-input', MyInput, { extends: 'input' });
</script>

<input is="my-input" />
```

虽然基于 **ES 6 class 继承**语法可以重写表单元素的各种属性、方法，在外部代码操作 DOM 时执行自定义逻辑，但用户直接与浏览器交互不会调用这些重写的接口。因此我们的自定义元素只能单纯地依赖内置表单元素的能力，而无法介入表单的工作流程。

于是，浏览器进一步暴露自己的内部机制，便有了 Element Internals API。它除了包含本文所述 Form-associated Custom Element API 的很多接口，还涉及另一项新标准 —— [AOM][9]（可访问性/无障碍化对象模型），让开发者更轻松地构造更好用的组件。

![](https://web-cell.dev/WebCell-1.fb612fdb.png)

同时，译者自研的 [Web Components 组件框架 WebCell][10] 在原文的帮助下[率先支持新标准][11]，使前文那段最长的示例代码变得非常简单：

```typescript
import { component, mixinForm } from 'web-cell';

@component({
  tagName: 'my-counter'
})
export class MyCounter extends mixinForm() {}
```

最后，欢迎大家持续关注译者对 Web 组件相关标准的后续译文和开源项目更新~

[1]: https://developer.mozilla.org/zh-CN/docs/Learn/HTML/Forms/Data_form_validation
[2]: https://developer.mozilla.org/zh-CN/docs/Learn/HTML/Forms
[3]: https://developer.mozilla.org/zh-CN/docs/Web/API/FormData
[4]: https://developers.google.com/web/fundamentals/web-components/customelements
[5]: https://developer.mozilla.org/zh-CN/docs/Web/API/File
[6]: https://html.spec.whatwg.org/multipage/form-control-infrastructure.html#concept-fe-disabled
[7]: https://github.com/calebdwilliams/element-internals-polyfill
[8]: https://github.com/webcomponents/polyfills#roadmap
[9]: https://w3c.github.io/aria/#ARIAMixin
[10]: https://web-cell.dev/
[11]: https://github.com/calebdwilliams/element-internals-polyfill/network/dependents
