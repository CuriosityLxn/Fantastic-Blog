`JSX` 会被 `Babel` 编译为 `React.createElement`，所以在 `JSX` 文件中需要显式引入 `React`：

```js
import React from 'react';
```

在 React 17 中已经不需要这样做了，因为 React 团队和 Babel 合作，重构了 JSX。**新的 JSX 转换不会将 JSX 转换为 React.createElement**，会自动从 React 的 package 中引入新的入口函数并调用。

新 JSX 被编译后的结果：

```js
// 编译器自动引入
import {jsx as _jsx} from 'react/jsx-runtime';
```

> ⚠️ 若要使用 Hooks 或其他导出，仍需显式引入 React ！！

## JSX 和 Fiber 的关系

- JSX：React.createElement() 的语法糖，创建 DOM 元素及 React 节点。
- Fiber：在 commit 阶段，ReactDom.render() 实现 Fiber 节点及 Fiber 树的创建及更新（调用 diff 算法）。

Fiber Node 不仅包含 JSX 创建出的 DOM 元素，还包含组件 **schedule、reconcile、render** 所需的信息。以下信息是 Fiber Node 有，JSX 元素没有的：

- 组件更新优先级
- 组件的 state
- 组件是否会 rerender 的标记
