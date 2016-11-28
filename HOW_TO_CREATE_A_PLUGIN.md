Note: this guide already focuses on version 2 (currently in beta) of the DraftJS Plugins architecture.

> 注:本指南已经关注版本2(目前在beta版本)DraftJS插件架构。

-----

# Basics 
> # 基础知识

A Plugin is simply a plain JavaScript object containing a couple deeper nested objects or functions e.g.

> 一个插件是一个简单的纯JavaScript对象包含几个深层嵌套对象或函数。

```js
const customPlugin = {
  blockStyleFn: (contentBlock) => {
    if (contentBlock.getType() === 'blockquote') {
      return 'superFancyBlockquote';
    }
  },
  customStyleMap: {
    'STRIKETHROUGH': {
      textDecoration: 'line-through',
    },
  },
};
```

As most plugins take some kind of configuration we recommend you to always export a function that creates this object. This is aligned with all the core plugins and leads to a consistent developer experience.

> 大多数插件采取某种配置，我们建议您总是 `export` 一个函数创建该对象。这是符合所有的核心插件和导致一致的开发体验。

```js
export default createCustomPlugin = (config) => {
  const blockStyleFn = (contentBlock) => {
    if (contentBlock.getType() === 'blockquote') {
      return 'superFancyBlockquote';
    }
  };
  
  const customStyleMap = {
    'STRIKETHROUGH': {
      textDecoration: 'line-through',
    },
  };

  return {
    blockStyleFn: blockStyleFn,
    customStyleMap: customStyleMap,
  };
};
```

# Supported Objects & Webhooks

> # 支持对象和钩子

A plugin accepts all standard props that a Draft.js Editor.

> 一个 Draft.js 编辑器 插件接受所有标准的 props.

- [blockRendererFn](https://facebook.github.io/draft-js/docs/api-reference-editor.html#blockrendererfn)
- [blockStyleFn](https://facebook.github.io/draft-js/docs/api-reference-editor.html#blockstylefn)
- [customStyleMap](https://facebook.github.io/draft-js/docs/api-reference-editor.html#customstylemap)
- [handleReturn](https://facebook.github.io/draft-js/docs/api-reference-editor.html#handlereturn)
- [handleKeyCommand](https://facebook.github.io/draft-js/docs/api-reference-editor.html#handlekeycommand)
- [handleBeforeInput](https://facebook.github.io/draft-js/docs/api-reference-editor.html#handlebeforeinput)
- [handlePastedText](https://facebook.github.io/draft-js/docs/api-reference-editor.html#handlepastedtext)
- [handlePastedFiles](https://facebook.github.io/draft-js/docs/api-reference-editor.html#handlepastedfiles)
- [handleDroppedFiles](https://facebook.github.io/draft-js/docs/api-reference-editor.html#handledroppedfiles)
- [handleDrop](https://facebook.github.io/draft-js/docs/api-reference-editor.html#handledrop)
- [onEscape](https://facebook.github.io/draft-js/docs/api-reference-editor.html#onescape)
- [onTab](https://facebook.github.io/draft-js/docs/api-reference-editor.html#ontab)
- [onUpArrow](https://facebook.github.io/draft-js/docs/api-reference-editor.html#onuparrow)
- [onDownArrow](https://facebook.github.io/draft-js/docs/api-reference-editor.html#ondownarrow)

There is one difference compared to the original properties.
All functions receive an additional argument. This argument is an object containing:

> 有一个区别相比原来的属性。所有函数获得一个额外的参数。这个论点是一个对象包含:

```js
// PluginFunctions
{
  getPlugins, // a function returning a list of all the plugins
  getProps, // a function returning a list of all the props pass into the Editor
  setEditorState, // a function to update the EditorState
  getEditorState, // a function to get the current EditorState
  getReadOnly, // a function returning of the Editor is set to readOnly
  setReadOnly, // a function which allows to set the Editor to readOnly
}
```

In addition the a plugin accepts 

> 另外一个插件接受

- `initialize: (PluginFunctions) => void`
- `onChange: (EditorState) => EditorState`
- `willUnMount: (PluginFunctions) => void`
- `decorators: Array<Decorator> => void`
- `getAccessibilityProps: () => { ariaHasPopup: string, ariaExpanded: string }`

-----

### `initialize`

Allows to initialize a plugin once the editor becomes mounted.

> 允许编辑器安装时初始化插件一次。

### `onChange`

Allows a plugin to modify the state at the latest moment before the onChange callback of the Draft.js Editor is fired.

> 允许插件修改状态在Draft.js最新的一刻onChange回调之前触发编辑器

### `willUnMount`

Usually used to clean up.

> 通常用于清理。

### `decorators`

Draft.js allows to initialize an EditorState with a decorator. Many plugins also rely on decorators and therefor we decided
to incorporate them in the plugin architecture. Every plugin can have multiple decorators and all of them are combined by the
plugin editor.

A decorator can contain a `strategy` and a `component` e.g.

> Draft.js允许初始化一个EditorState装饰。许多插件也依靠修饰符, 因此我们决定
将decorators 和 therefor 纳入插件架构。每个插件都可以有多个修饰符和他们相结合的
插件编辑器。

> decorator可以包含一个“策略”和“组件”。

```js
{
  decorators: [
    {
      strategy: hashtagStrategy,
      component: HashtagSpan,
    },
  ],
}
```

You can read more about it in the original Draft.js documentation about [decorators](https://facebook.github.io/draft-js/docs/advanced-topics-decorators.html#compositedecorator).

> 你可以阅读更多关于 Draft.js 的初始文档[decorators](https://facebook.github.io/draft-js/docs/advanced-topics-decorators.html#compositedecorator)。

### `getAccessibilityProps`

In some rare cases like @mentions or Emoji autocompletion a plugin should be able to set certain ARIA attributes.
This currently only allows to set `aria-has-popup` & `aria-expanded`. Let us know in case you have further suggestions.

> 在一些罕见的情况下像 @mentions 或 Emoji 自动完成插件应该可以设置某些 ARIA 属性。
这个目前只允许设置 “aria-has-popup” & “aria-expanded” 。让我们知道如果你有进一步的建议。

# Further Information

Keep in mind that the order of plugins can matter. The first decorator the that matches prevents the following ones to become active
for a certain text block. Same goes for `handleReturn` if returned true. Therefor try to keep your plugins minimal in the sense that
they only match & manipulate the necessary block or text.

> 请记住,插件的顺序问题。第一个装饰相匹配防止以下的活跃起来
对于一个特定的文本块。同样适用于“handleReturn”如果返回真。因此尽量保持你的插件最小的匹配和操作必要的块或文本。

